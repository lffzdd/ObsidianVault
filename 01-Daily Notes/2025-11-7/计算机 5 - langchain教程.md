- ~ [Title Unavailable \| Site Unreachable](https://chatgpt.com/g/g-p-68f734897bcc81919f529cb94fdb7f49-chao-ji-ge-ti/shared/c/68f745a5-9bd4-8320-afed-deb64060c590)
>给我一个langchain教程，一步一步来，不用一次性回答完

太好了！我给你做一套「循序渐进」的 LangChain 实战小课。我们一次上一小节，每节都有明确产出与练习。你说“下一步”，我就继续。

# 课程地图（先给你个整体框架，不一次讲完）

1. 准备环境与第一次跑通（今天这节）
    
2. 用 LCEL 串起 Prompt → LLM → 解析器
    
3. 加入工具与检索（RAG）
    
4. 多轮对话与记忆
    
5. LangGraph：把流程画成状态机（稳健可控）
    
6. 部署成 API / CLI / 批处理脚本
    
7. 进阶性能与成本优化（缓存、并发、评测）
    

---

# 第 1 课：准备环境 & 第一个“可复用链”

目标：

- 装好必要包
    
- 用 **LCEL**（LangChain Expression Language）把 Prompt、模型、输出解析拼成一条“链”
    
- 跑出一个最小可用 demo
    

## 1. 安装

建议用 Python 3.10+ 与虚拟环境：

```bash
python -m venv .venv && source .venv/bin/activate  # Windows 用 .venv\Scripts\activate
pip install -U langchain langchain-openai langchain-core
```

> 可选：后面做 RAG 再装 `langchain-community`, `faiss-cpu` 或 `chromadb`。

## 2. 配置密钥

使用 OpenAI 兼容模型最省事（也可换成你自己的兼容端点）。

```bash
export OPENAI_API_KEY="你的key"      # Windows PowerShell:  $env:OPENAI_API_KEY="你的key"
```

## 3. 写下首条链（Prompt → 模型 → 文本解析）

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

# 1) Prompt：把输入变量化，便于复用
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一名简洁有力的写作助手。"),
    ("user", "请把下面内容改写得更清晰但保留原意：{text}")
])

# 2) LLM：选一个聊天模型；temperature 控制创意度
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0.3)

# 3) 解析器：把聊天消息变成纯文本
parser = StrOutputParser()

# 4) 用 LCEL 组合：像 Unix 管道一样
chain = prompt | llm | parser

# 5) 调用
result = chain.invoke({"text": "LangChain能让你把复杂AI流程拆成可复用的部件。"})
print(result)
```

运行后你应看到一段更清晰的改写文本。  
这条“链”是可复用组件：你可以把 `chain` 当函数，多次 `invoke`，也能用于批处理（`batch`）或流式（`stream`）。

## 4. 再来一个带“风格控制”的小封装

把“可复用”做得再像产品一点——我们做个函数，把风格、长度等作为参数暴露：

```python
from typing import Literal

style_map = {
    "严谨": "使用正式、客观、学术化的语气；避免夸张修辞。",
    "科普": "面向非专业读者，举例贴近生活，避免术语或配合解释。",
    "广告": "突出卖点，强调利益点和行动号召，但不夸大其词。"
}

def make_rewriter(style: Literal["严谨","科普","广告"]="科普", max_words: int = 120):
    sys = f"你是一名写作助手。目标风格：{style}。风格说明：{style_map[style]}。请控制在 {max_words} 字以内。"
    prompt = ChatPromptTemplate.from_messages([
        ("system", sys),
        ("user", "改写并提炼重点：{text}")
    ])
    return prompt | llm | parser

rewrite = make_rewriter(style="科普", max_words=80)
print(rewrite.invoke({"text": "量子计算利用量子叠加与纠缠实现并行计算，但目前物理实现仍面临纠错与扩展性挑战。"}))
```

## 5. 小测验（动手 10 分钟）

1. 把上面的改写器改成“摘要器”：输入长文本，输出 3 点要点（项目符号形式）。
    
2. 再写一个“翻译器”：自动识别输入语言，输出为简体中文，并在末尾给出 1 句术语解释。
    
3. 用 `batch` 一次处理 3 段文本：
    

```python
items = [{"text": "第一段..."}, {"text": "第二段..."}, {"text": "第三段..."}]
for out in chain.batch(items):
    print(out, "\n---")
```

## 6. 常见坑与排查

- **模块导入错位**：`ChatPromptTemplate` 来自 `langchain_core.prompts`，`ChatOpenAI` 来自 `langchain_openai`。
    
- **Key 未生效**：确认终端里 `echo $OPENAI_API_KEY`（或 `$env:OPENAI_API_KEY`）能看到值。
    
- **超时/网络**：可在 `ChatOpenAI(timeout=...)` 指定超时时间，或切换网络再试。
    
- **成本控制**：开发期把 `temperature` 降低、用小模型 `gpt-4o-mini` 之类，先跑通逻辑。
    

---

到这里，你已经有了一个真正可复用的小“链”。接下来第 2 课，我们会把“输出解析”做得更可靠（比如强制返回 JSON 结构），并演示把多个步骤拼成更长的 LCEL 管道（例如：清洗 → 生成 → 校对）。

准备好就说：**“下一步”**。