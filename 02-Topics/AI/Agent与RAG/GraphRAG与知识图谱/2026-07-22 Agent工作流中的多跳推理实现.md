# Agent工作流中的多跳推理实现

**Created:** 7/22/2026 9:41:50  
**Updated:** 7/22/2026 9:45:28  
**Exported:** 7/22/2026 9:46:26  
**Link:** [https://claude.ai/chat/55b6074b-46ba-464c-a591-51bc81ec8829](https://claude.ai/chat/55b6074b-46ba-464c-a591-51bc81ec8829)  

## User:

> 7/22/2026 9:41:52

Agent/Workflow中的多跳推理，工程中是怎么实现的

## Assistant:

> 7/22/2026 9:42:48

> 综合架构模式与工程实现细节，阐述多跳推理的核心机制。
>
> - 用户用中文问关于Agent/Workflow中的多跳推理如何在工程中实现的问题。这是一个关于Agent架构的技术问题，我可以从现有知识出发给出实际的答案，不需要搜索——核心是讲清楚架构模式和实现思路。
>   
>   多跳推理的本质是一个问题需要多个顺序的检索或推理步骤，后一步依赖前一步的结果。实现方式有几种：静态工作流通过查询分解和DAG来处理可预测的模式；动态Agent循环（如ReAct）让LLM根据观察结果决定下一步工具调用；还有迭代检索方法如Self-Ask和Self-RAG。另外还有先规划后执行的方式，以及基于图的方法如GraphRAG来做知识图谱遍历。
>   
>   工程实现的核心是状态管理（用上下文或黑板存储中间结果）、提示词工程来生成子问题和提取关键实体、设定终止条件（最大跳数、充分性判断、置信度）、以及管理上下文避免爆炸——每步都要总结而不是无限堆积。
>   
>   还要处理死路检测和回溯、检测依赖关系来决定并行还是串行执行、缓存去重检索结果，以及追踪每一跳的可观测性和评估指标。
>   
>   实际应用中的陷阱包括错误复合（0.9的5次方会快速衰减）、上下文污染、幻觉出来的桥接实体、无限循环和延迟问题。选择实现方式时，固定模式用工作流，开放式问题用Agent循环。
>   
>   我决定用中文写，结构是：本质上是状态机加每轮决策、三种主流实现方式对比、工程骨架代码、关键细节处理、常见坑点，保持简洁有料。
> - **Done**

多跳推理的工程本质其实很朴素：**把"一次问答"变成一个带状态的循环，每一轮的检索/工具调用的输入，依赖上一轮的输出**。区别只在于"下一跳查什么"这个决策由谁做、什么时候做。

## 三种主流实现路线

**1. 静态分解（Query Decomposition / DAG）**

先让 LLM 把问题拆成子问题并标注依赖关系，再按拓扑序执行，无依赖的并行跑。

```
"马斯克创立的第二家公司的现任CEO是谁"
→ Q1: 马斯克创立的第二家公司是什么？        (无依赖)
→ Q2: {Q1.answer} 的现任CEO是谁？           (依赖 Q1)
```

代表是 LLMCompiler、DSPy 的 multi-hop 模块。优点是可并行、延迟可控、可缓存；缺点是拆解在"还不知道答案"时就要做完，遇到需要看到中间结果才知道下一步怎么走的问题就废了。

**2. 动态循环（ReAct / Self-Ask 系）**

不预先规划，每轮让模型看着已有观察决定下一个动作，直到它输出 finish。

```
Thought → Action(tool, args) → Observation → Thought → ... → Answer
```

这是 LangGraph、Agno、绝大多数 agent 框架的默认形态。灵活，但每跳一次串行 LLM 调用，延迟高、成本高、容易跑偏。

**3. 检索驱动的迭代（IRCoT / FLARE / Self-RAG）**

介于两者之间：不显式规划，而是**用生成过程本身触发下一跳**。比如 IRCoT 是"生成一步 CoT → 拿这句 CoT 当 query 再检索 → 继续生成";FLARE 是检测到下一句的 token 置信度低时才触发检索。这类在 RAG 场景下性价比最高，因为它避免了为每一跳都付一次"规划"的 LLM 开销。

实践中常混用:外层用 plan-and-execute 定大方向,单步内部用 ReAct 兜底,再加一个 replan 分支处理执行失败。

## 工程骨架

核心是一个显式的 state 对象在节点间流转,而不是把所有东西塞进 message 数组滚雪球:

```python
@dataclass
class HopState:
    original_query: str
    hops: list[Hop]              # 每跳的 query / evidence / 抽出的实体
    facts: dict[str, str]        # 结构化的中间结论（bridge entity 等）
    visited: set[str]            # 已检索过的 query 指纹，去重防环
    budget: int = 5

def multi_hop(state: HopState):
    while state.budget > 0:
        # 1. 充分性判断：现有 facts 够不够回答原问题
        verdict = sufficiency_check(state.original_query, state.facts)
        if verdict.enough:
            return synthesize(state)

        # 2. 生成下一跳 query —— 关键在于把 facts 显式注入
        next_q = gen_next_query(state.original_query, state.facts, verdict.missing)
        if fingerprint(next_q) in state.visited:
            next_q = rewrite(next_q)          # 换个说法，防死循环

        # 3. 执行 + 压缩
        docs = retrieve(next_q)
        extracted = extract_facts(next_q, docs)   # 只留结构化事实，不留原文
        state.facts.update(extracted)
        state.visited.add(fingerprint(next_q))
        state.budget -= 1

    return synthesize(state, degraded=True)      # 预算耗尽走降级
```

几个容易被忽略但决定成败的点:

**桥接实体要显式抽出来。** 多跳失败最常见的原因不是检索不到,而是第一跳的答案被埋在一大段文本里,第二跳的 query 生成时模型没抓准。所以在每跳结束后加一个结构化抽取(`{"第二家公司": "SpaceX"}`),下一跳的 prompt 里直接填槽,而不是让模型自己从上下文里再读一遍。

**上下文必须每跳压缩。** 如果把每跳的原始检索结果全部累积进 context,5 跳之后既贵又会被噪声淹没(lost in the middle)。正确做法是原文只在当跳可见,跳出后只保留抽取出的事实和引用 ID,最终合成答案时再按需回捞原文。

**终止条件要三重保险。** 单靠模型说 "我可以回答了" 不可靠。实际要同时有:硬性 max_hops(通常 3–5)、token/时间预算、以及一个独立的充分性判断(可以用小模型或规则)。另外必须有 query 指纹去重,否则模型反复用同义句检索同一个东西是很常见的失效模式。

**错误会复利衰减。** 每跳 90% 准确率,5 跳只剩 59%。所以工程上要么减少跳数(把常见的两跳模式预编译成一次结构化查询,或者直接建知识图谱让一次图遍历顶掉多跳),要么加验证——每跳产出的事实做一次交叉检索确认,冲突时回溯换路径。GraphRAG 这类方案本质就是把"多跳推理"的成本从推理期挪到了索引期。

**可观测性从第一天就要做。** 每跳记录 query、检索结果 ID、抽取的事实、决策理由。评估时要分开看:每跳的检索召回率、桥接实体的抽取准确率、最终答案正确率——只看最后一个指标你根本不知道是哪跳断的。HotpotQA、2WikiMultiHopQA、MuSiQue 是这类任务的标准评测集,MuSiQue 最难,因为它专门构造了无法靠单跳捷径蒙对的样本。

## 怎么选

问题模式固定(比如客服里就那么几类查询链)就用静态分解甚至硬编码 workflow,别上 agent——延迟和成本能差一个数量级。只有在跳数和路径真的不可预测时才用动态循环,并且一定要配预算上限。

## User:

> 7/22/2026 9:44:52

所以是先定大方向，内部动态决定？一个普通的检索，怎么复杂到了这个地步，和Workflow/Agent一样复杂了

## Assistant:

> 7/22/2026 9:45:28

> 权衡了检索复杂度与实际需求的矛盾。
>
> - The user's pushing back on the complexity here, and they're right to question it. Multi-hop retrieval is structurally an agent—there's no real distinction between them—but most retrieval problems don't actually need this level of sophistication. The added complexity only makes sense when the question genuinely demands multiple reasoning steps.
>   
>   The critical difference is that single-hop retrieval is stateless and parallelizable, but the moment one query depends on the previous result, you've introduced a loop with data-dependent termination. That's no longer a search problem—it's a control flow problem with all the attendant costs: error handling, state management, observability, and budget constraints. Most of the industry over-applies this pattern; the vast majority of RAG questions are single-hop, and multi-hop is actually the exception.
>   
>   The honest framing is that the complexity isn't really in retrieval itself—it's in the reasoning about what to retrieve next. You could also sidestep the whole loop by shifting cost to index time with approaches like GraphRAG, or by expanding recall on a single hop with longer context and larger top-k results, which often outperforms a multi-hop chain anyway.
> - **Done**

是的,但先说清楚:**"内外都上"通常是过度设计**。默认应该二选一,只有当外层步骤本身有明确阶段划分、且某个阶段的失败模式确实需要模型自己纠错时,才值得套两层。多数系统直接用一种就够了。

至于第二个问题——你的怀疑是对的,而且比你想的更彻底:**它已经不是检索了**。

单跳检索是个纯函数:query 进,docs 出。无状态、可缓存、可并行、失败重试代价为零。整个 RAG 之所以工程上简单,全靠这个性质。

而第二跳的 query 依赖第一跳的结果的那一刻,你引入了两样东西:**跨调用的状态**,和**一个退出条件由数据决定的循环**。这就不是查询了,这是一段程序。于是所有程序执行的问题一次性全回来了——会不会不终止、状态怎么传、错误怎么传播、跑到哪一步了怎么看。这不是谁把它设计复杂了,是循环的固有成本。

所以"多跳检索"和"agent"没有实质区别,它就是一个动作空间只有 retrieve 一个工具的 agent。行业里把它们叫成两个词,纯粹是历史原因:一个从 RAG 那边长出来,一个从 tool use 那边长出来,中间撞上了。真正的分界线不在"检索 vs 工具",而在**这一步的输入是否依赖上一步的输出**。不依赖就是查询,依赖就是程序。

真正值得记住的是复杂度可以绕开:

**大部分问题根本不是多跳的。** 生产 RAG 里真需要跳的query通常是个位数百分比,剩下的是被切块切碎了导致召回不全,看起来像多跳。先查清楚这一点,很多时候答案是改分块策略而不是上循环。

**把跳数挪到索引期。** "马斯克的第二家公司的CEO"要两跳,是因为这两个事实存在不同文档里。如果建索引时就把实体关系抽出来连好(GraphRAG、知识图谱、或者干脆预计算常见的 join),查询期一次图遍历就完事。多跳的成本从每次查询付,变成建库时付一次。

**用宽召回换深度。** 一跳 top-50 + 长上下文,经常能打平三跳循环,而且延迟低一个数量级、没有终止问题、没有误差累积。上下文窗口便宜下来之后,这个方案的性价比一直在涨。

**最后才是循环。** 只有当关系是开放的、无法预先建模,而且答案确实散落在检索不到的地方时,才付这个复杂度。

一个实用的判断标准:如果你能写出这个问题的 SQL(哪怕数据不在数据库里),那它就不需要 agent,它需要的是一个更好的索引。

Powered by Claude Exporter (https://www.ai-chat-exporter.net)
