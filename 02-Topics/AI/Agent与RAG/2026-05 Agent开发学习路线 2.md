# 阶段二展开：吃透 Agent 核心模式

这一阶段的目的不是学语法，而是建立**模式直觉**——看到一个业务问题就知道该用哪种结构。我把它拆成"地基（增强 LLM）+ 五种工作流模式 + 真正的 Agent 循环"。每一节给你：概念、最小代码骨架、什么时候用、配套练习。

代码用 Python + Anthropic SDK 演示（OpenAI 类似），重点在结构而不是 API 细节。

## 0. 地基：增强 LLM（Augmented LLM）

所有模式都建立在这个原子单元上：一次 LLM 调用 + 工具 + 检索 + 记忆。

```python
from anthropic import Anthropic
client = Anthropic()

def llm(messages, tools=None, system=None):
    resp = client.messages.create(
        model="claude-sonnet-4-5",
        max_tokens=2000,
        system=system or "",
        messages=messages,
        tools=tools or [],
    )
    return resp
```

**练习 0**：写一个 `llm_with_tools()`，能处理 tool_use 响应——调用本地函数，把结果作为 tool_result 喂回去，循环直到模型返回 `end_turn`。这是最重要的基本功，后面所有模式都假设你会写这个循环。**没写过这个就直接跳框架，你永远会被框架抽象绊倒。**

---

## 1. Prompt Chaining（提示链）

**形状**：A → B → C，每一步的输出是下一步的输入。可以在中间插入程序化检查（gate），不通过就回退或终止。

```python
def write_blog_post(topic):
    # Step 1: 出大纲
    outline = llm([{"role": "user", 
                    "content": f"为'{topic}'写一个5节大纲，每节一句话"}])
    
    # Gate: 检查格式是否正确
    if outline.count("\n") < 4:
        raise ValueError("大纲不够细")
    
    # Step 2: 基于大纲写初稿
    draft = llm([{"role": "user",
                  "content": f"基于以下大纲写文章：\n{outline}"}])
    
    # Step 3: 改写润色
    polished = llm([{"role": "user",
                     "content": f"把这篇文章改成更口语化的风格：\n{draft}"}])
    
    return polished
```

**什么时候用**：任务能干净地拆成固定的步骤，每一步比合在一起做更准确。典型场景：合同分析（提条款 → 校合规 → 出摘要）、翻译（直译 → 润色 → 本地化）、内容生成（大纲 → 写作 → 改写）。

**什么时候不用**：步骤需要根据中间结果动态变化时——那是 Agent 循环的活儿。

**练习 1**：实现一个"会议纪要生成器"。输入是一段会议转录，链路是：提取行动项 → 按负责人分组 → 校验每个行动项有截止日期 → 生成 Markdown 纪要。在每一步加一个程序化 gate（比如行动项数量大于 0、每项必须有动词开头）。

---

## 2. Routing（路由）

**形状**：先用一个 LLM 调用做分类，把输入分发给专门的下游处理器。每个下游处理器可以是独立的 prompt、独立的模型，甚至独立的工作流。

```python
def route_customer_query(query):
    # 路由器：用便宜/快的模型分类
    classification = llm(
        messages=[{"role": "user", "content": query}],
        system="""把用户问题分类成以下之一，只输出类别名：
        - billing（账单/付款）
        - technical（技术问题）
        - refund（退款）
        - general（其他）"""
    )
    category = classification.content[0].text.strip()
    
    # 分发到专门的 handler，每个有自己的 system prompt 和工具集
    handlers = {
        "billing": handle_billing,
        "technical": handle_technical,
        "refund": handle_refund,
        "general": handle_general,
    }
    return handlers[category](query)
```

**关键技巧**：路由器用小模型（Haiku、GPT-4o-mini），下游处理器才用大模型——能省 70% 的成本。

**什么时候用**：输入种类多、每类需要的处理方式差异大；想分开优化每条路径的 prompt；想根据复杂度路由到不同档位的模型（简单问题用 Haiku，复杂问题用 Opus）。

**什么时候不用**：只有 2 类且差别不大——直接一个 prompt 配条件分支就够了，多一次 LLM 调用纯属浪费。

**练习 2**：实现一个"代码助手路由"。输入是一段自然语言请求，分类成 [explain（解释代码）, refactor（重构）, debug（找 bug）, write_test（写测试）] 四类，每类用不同的 system prompt。要求路由器用 Haiku，下游用 Sonnet。统计两种方式的 token 成本差。

---

## 3. Parallelization（并行化）

有两个变体，搞混会踩坑。

**变体 A：Sectioning（分片）**——任务能切成独立子任务并行做，最后汇总。

```python
import asyncio

async def analyze_company(company_name):
    # 三个独立的子任务并行跑
    financial, news, competitors = await asyncio.gather(
        analyze_financials(company_name),
        analyze_recent_news(company_name),
        analyze_competitors(company_name),
    )
    # 最后用一个 LLM 调用汇总
    return llm([{"role": "user", "content": 
                 f"基于以下三份分析写一段投资建议：\n{financial}\n{news}\n{competitors}"}])
```

**变体 B：Voting（投票/多数决）**——同一个任务跑 N 次，取最一致或最好的答案。用于需要高准确度或捕捉边缘情况的场景。

```python
async def safety_check(content):
    # 同一个安全检查跑 5 次（不同 temperature）
    results = await asyncio.gather(*[
        llm_async([{"role": "user", "content": f"这段内容是否有害？yes/no\n{content}"}],
                   temperature=0.7) for _ in range(5)
    ])
    yes_count = sum(1 for r in results if "yes" in r.lower())
    return yes_count >= 2  # 5 次里有 2 次说有害就拦截
```

**什么时候用**：子任务真正独立（A）；或者准确度比延迟更重要（B）；或者想用一个"检查员"和"主任务"并行跑，互不干扰。

**关键陷阱**：很多人以为多 Agent 就是并行，其实大部分多 Agent 系统是 orchestrator-workers（下一节）。Parallelization 的关键是**子任务在调用前就已经确定**。

**练习 3**：实现一篇文档的"三视角评审"。同一份 PRD 文档，让三个 LLM 调用并行从【产品经理视角】【工程师视角】【法务视角】给评价，最后用第四次调用汇总成一份统一意见。对比串行做同样事情的延迟。

---

## 4. Orchestrator-Workers（编排者-工作者）

**形状**：一个 orchestrator LLM **动态地**决定要拆成哪些子任务，分发给 workers，再综合结果。和 Parallelization 的关键区别：**子任务是运行时由 LLM 决定的，不是预先写死的**。

```python
def orchestrator_workers(task):
    # 步骤1：orchestrator 拆任务
    plan = llm(
        messages=[{"role": "user", "content": task}],
        system="""把任务拆成 1-5 个独立的子任务，输出 JSON：
        [{"id": 1, "description": "..."}]"""
    )
    subtasks = parse_json(plan)
    
    # 步骤2：workers 并行执行（每个 worker 是一次独立的 LLM 调用，可能带工具）
    results = []
    for subtask in subtasks:
        result = llm_with_tools(
            messages=[{"role": "user", "content": subtask["description"]}],
            tools=ALL_TOOLS,
        )
        results.append({"subtask": subtask, "result": result})
    
    # 步骤3：orchestrator 综合
    final = llm([{"role": "user", "content": 
                  f"原始任务：{task}\n子任务结果：{results}\n请综合成最终答案"}])
    return final
```

**什么时候用**：任务复杂度不可预测的场景。Anthropic 的 coding agent 就用这种模式处理 GitHub issue——不知道一个 issue 要改几个文件，所以由 orchestrator 动态决定。研究类任务（不知道要查几个来源）、跨多个未知文件的代码修改、复杂的客户问题诊断。

**什么时候不用**：能预先确定子任务时，用 Parallelization 更便宜更快。

**练习 4**：实现一个"研究员 Agent"。给一个开放问题（如"对比 LangGraph 和 CrewAI 的状态管理"），orchestrator 决定要搜哪些关键词、读哪些类型的资料，workers 各自去搜索和总结，最后 orchestrator 综合成报告。注意 token 成本会蹭蹭上涨，记得记录。

---

## 5. Evaluator-Optimizer（评估-优化循环）

**形状**：Generator LLM 生成 → Evaluator LLM 打分/给反馈 → 不达标就拿着反馈重新生成 → 直到通过或达到最大轮次。

```python
def evaluator_optimizer(task, max_rounds=5):
    feedback = ""
    for round_num in range(max_rounds):
        # Generator
        prompt = task if round_num == 0 else f"{task}\n\n上一版的反馈：{feedback}\n请改进"
        attempt = llm([{"role": "user", "content": prompt}])
        
        # Evaluator
        eval_result = llm(
            messages=[{"role": "user", "content": 
                       f"任务：{task}\n候选答案：{attempt}\n"
                       f"按以下标准评分（1-10）并给出改进建议..."}],
            system="你是严格的评审员"
        )
        score, feedback = parse_eval(eval_result)
        
        if score >= 8:
            return attempt
    
    return attempt  # 达到最大轮次也返回最后一版
```

**什么时候用**：有明确的评估标准；初稿质量不够但迭代会显著改善。经典场景：翻译（评审找漏译/不自然之处）、代码生成（跑测试拿失败信息回去修）、文学创作（评审挑结构问题）、数学推理（验证步骤）。

**关键设计**：Evaluator 必须比 Generator 严格。常见做法是用同一个模型但不同 prompt，或用更强的模型当 evaluator。**评估标准必须可执行**——能跑代码就跑代码，能用 regex 检查就用 regex，不要全靠 LLM 主观判断。

**什么时候不用**：任务一遍就能做对（迭代是浪费）；或者评估标准本身比生成还难定义（那 LLM-as-judge 不可靠）。

**练习 5**：实现一个"SQL 生成器 with 自检"。输入自然语言 + 数据库 schema，generator 生成 SQL，evaluator 是个**真实运行**——把 SQL 跑在 SQLite 上，捕获报错。报错就把错误信息扔回 generator 让它修。这是 evaluator-optimizer 最强的形态：评估器是真实环境而不是另一个 LLM。

---

## 6. 真正的 Agent Loop（自主 Agent）

前 5 种是工作流——结构是你写死的。真正的 Agent 是把"下一步做什么"完全交给 LLM 决定，循环直到任务完成。

```python
def agent(task, tools, max_steps=20):
    messages = [{"role": "user", "content": task}]
    
    for step in range(max_steps):
        response = llm(messages, tools=tools)
        messages.append({"role": "assistant", "content": response.content})
        
        # Agent 觉得做完了
        if response.stop_reason == "end_turn":
            return response.content[-1].text
        
        # Agent 要调工具
        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": str(result),
                    })
            messages.append({"role": "user", "content": tool_results})
    
    raise Exception("达到最大步数仍未完成")
```

就这么简单——**Agent 的核心就是这 20 行**。所有"自主性"来自 LLM 在每一轮自己决定调哪个工具、何时停止。

**什么时候用**：任务开放、步骤数不可预测、环境会反馈（错误信息、用户回复、工具输出）。典型场景：编码 Agent、Computer Use、研究 Agent、调试 Agent。

**生产级 Agent 还需要**：步数上限、token 预算上限、对不可逆操作的人工确认（HITL）、可见的思考过程（让用户能看到它在干嘛）、错误恢复（工具失败时怎么办）。

**练习 6（这是阶段二的毕业项目）**：写一个能修 bug 的小 Agent。给它三个工具：`read_file`、`run_tests`、`edit_file`。提供一个含 bug 的 Python 项目，让它读测试失败信息、定位问题、修代码、再跑测试，直到通过。规模：100 行以内，单文件项目。完成这个你就真正"会"Agent 了。

---

## 怎么选模式：决策树

实际开发中我用这套判断顺序：

1. **能不能一次 LLM 调用搞定？**——能就别复杂化。Anthropic 反复说"只在简单方案不够用时才加多步系统"。
2. **任务能不能切成固定步骤？**——能就用 Prompt Chaining。
3. **输入是不是要分类处理？**——是就用 Routing。
4. **子任务能不能预先列出且独立？**——能就用 Parallelization。
5. **要不要根据中间结果决定下一步做什么？**——要就用 Orchestrator-Workers 或 Agent Loop。
6. **有没有可执行的评估标准？**——有就加 Evaluator-Optimizer 在外层包一圈。

实际项目往往是组合：一个 Routing 把请求分类 → 简单类用 Chaining → 复杂类用 Orchestrator → 关键步骤再套 Evaluator-Optimizer。

---

## 阶段二的两周节奏建议

第一周：练习 0–3（增强 LLM、Chaining、Routing、Parallelization）。每个练习 100–200 行代码，跑通就行，不追求完美。每天 2–3 小时大概一周。

第二周：练习 4–6（Orchestrator-Workers、Evaluator-Optimizer、Agent Loop）。难度陡升，特别是练习 6——预期会调 bug 调到怀疑人生，但调完之后你对 Agent 的理解会有质变。

**强烈建议**：每个练习完成后，去看 [anthropic-cookbook](https://github.com/anthropics/anthropic-cookbook) 的 `patterns/agents` 目录里的对应实现，对比你的写法和 Anthropic 工程师的写法。你会捡到很多小技巧（比如怎么处理 tool_use 的边缘情况、怎么写 system prompt、怎么解析结构化输出）。

完成阶段二后你会有一种"祛魅感"——再看任何 Agent 框架的文档，都能一眼看穿"这一坨抽象其实就是某某模式"。这时候再上阶段三的 LangGraph 才是真正高效的。

要不要我针对其中某个练习展开讲（比如练习 6 那个修 bug 的 Agent，可以给一个完整的可运行版本）？或者如果你想直接开始动手，告诉我你现在最想做的是哪个练习，我可以给你更具体的起手式。