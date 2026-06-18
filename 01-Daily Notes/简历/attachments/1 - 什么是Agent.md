# Agent 面试复习笔记

## 一句话定义

**Agent = 具备自主闭环能力的 AI 系统。**

核心不是工具调用，而是：

> 给定目标 → 自主规划 → 调用工具执行 → 感知结果 → 调整策略 → 完成目标

记住关键词：

**自主性（Autonomy） + 闭环（Loop） + 行动能力（Action）**

---

# 面试标准答案（2 分钟版）

Agent 本质上是一个能够自主完成目标的 AI 系统。

与普通大模型最大的区别在于：

1. Agent 有自主规划能力，可以将复杂任务拆解成多个步骤。
2. Agent 能通过工具调用与外部世界交互，不只是生成文本。
3. Agent 具备感知 - 规划 - 行动 - 反馈的闭环机制，会根据执行结果动态调整后续决策。

因此普通 LLM 更像一个问答专家，而 Agent 更像一个能够完成任务的执行者。

---

# 普通 LLM 的三大局限

## 1. 知识冻结（Knowledge Cutoff）

训练数据有截止时间。

例如：

- 今天天气
- 最新股价
- 最新新闻

模型本身不知道。

---

## 2. 无法行动（No Action）

只能告诉你怎么做：

- 发邮件
- 查询数据库
- 调用 API
- 执行代码

但自己做不了。

本质：

> LLM = Text Generator

输出永远只是文字。

---

## 3. 没有持续记忆（No Persistent Memory）

每次调用都是新的开始。

不会记得：

- 用户偏好
- 历史任务
- 长期状态

除非开发者手动传入上下文。

---

# Agent 的核心闭环

Agent 最重要的概念：

```text
感知（Perceive）
        ↓
规划（Plan）
        ↓
行动（Act）
        ↓
反馈（Observe）
        ↓
重新规划
```

即：

```text
Perceive → Plan → Act → Observe
```

这就是面试中经常提到的：

**Agent Loop**

---

# Agent 四大核心能力

---

## 1. Planning（规划）

给 Agent 一个目标：

```text
帮我调研 AI Agent 框架
```

Agent 不会直接回答。

而是：

```text
步骤1：搜索框架
步骤2：收集资料
步骤3：比较优缺点
步骤4：整理报告
```

先拆解任务。

---

## 2. Tool Use（工具调用）

Agent 真正做事的能力来源。

常见工具：

- Search
- Browser
- Database
- Code Interpreter
- API
- Email

例如：

```text
查天气
↓
获取结果
↓
生成邮件
↓
发送邮件
```

---

### 面试高频考点

很多人会答错：

> Agent = 工具调用

这是错误的。

正确说法：

> 工具调用只是 Agent 的能力之一，不等于 Agent。

---

### Tool Calling 本质

模型负责：

```text
决策
```

程序负责：

```text
执行
```

例如：

```json
{
  "tool": "get_weather",
  "city": "北京"
}
```

模型只负责输出：

```text
调用哪个工具
参数是什么
```

真正执行的是外部程序。

---

## 3. Memory（记忆）

Agent 通常有两层记忆。

---

### 短期记忆（Short-Term Memory）

当前任务状态。

例如：

```text
已搜索哪些网站
已获得哪些结果
```

一般放在 Context Window。

---

### 长期记忆（Long-Term Memory）

跨任务保存。

例如：

```text
用户喜欢中文回答
用户是技术面试准备者
```

通常：

```text
Embedding
+
Vector Database
```

实现。

常见：

- Milvus
- Weaviate
- Pinecone
- Chroma

---

## 4. Reflection（反思）

Agent 和 Workflow 最大区别之一。

Agent 会：

```text
检查结果
发现错误
修正策略
再次尝试
```

例如：

```text
关键词A搜索失败
↓
自动换关键词B
↓
重新搜索
```

或者：

```text
API调用失败
↓
分析报错
↓
重新调用
```

这就是：

**Self-Reflection**

---

# Agent 与 LLM 的本质区别

|维度|LLM|Agent|
|---|---|---|
|工作模式|一问一答|持续执行|
|是否规划|❌|✅|
|是否调用工具|一般不具备|✅|
|是否有记忆|很弱|✅|
|是否行动|❌|✅|
|是否自我修正|❌|✅|
|是否闭环|❌|✅|

一句话：

> LLM 负责思考，Agent 负责完成任务。

---

# 为什么 Agent 2024~2025 才爆发

三个条件成熟。

---

## 1. 大模型能力成熟

代表：

- OpenAI GPT-4
- Anthropic Claude 3

具备：

- 推理能力
- 指令遵循能力
- 任务拆解能力

---

## 2. Function Calling 标准化

2023 年后主流模型支持：

```json
{
  "tool": "...",
  "arguments": {...}
}
```

开发成本大幅下降。

---

## 3. Agent 生态成熟

代表框架：

- [LangChain](https://www.langchain.com/?utm_source=chatgpt.com)
- [LlamaIndex](https://www.llamaindex.ai/?utm_source=chatgpt.com)

配套：

- 向量数据库
- 工具生态
- Agent Framework

逐步完善。

---

# MCP 与 A2A

面试很容易问。

---

## MCP（Model Context Protocol）

提出者：

Anthropic

定位：

> Agent 调工具的标准协议

类似：

```text
AI世界的 USB-C
```

解决：

```text
Agent ↔ Tool
```

问题。

---

### MCP 架构

```text
Host
 ↓
Client
 ↓
Server
```

例如：

```text
Claude Desktop
 ↓
MCP Client
 ↓
GitHub MCP Server
```

---

## A2A（Agent2Agent）

提出者：

Google

定位：

> Agent 与 Agent 通信协议

解决：

```text
Agent ↔ Agent
```

问题。

---

### Agent Card

每个 Agent 提供：

```text
我是谁
我能做什么
我需要什么输入
```

方便协作。

---

# MCP 与 A2A 区别（高频题）

|协议|解决问题|
|---|---|
|MCP|Agent 调工具|
|A2A|Agent 之间协作|

记忆口诀：

```text
MCP 管工具
A2A 管协作
```

[[01-Daily Notes/简历/attachments/attachments/53e81dc69945d7dc33a2baf67e837297_MD5.png|Open: Pasted image 20260617173432.png]]
![[01-Daily Notes/简历/attachments/attachments/53e81dc69945d7dc33a2baf67e837297_MD5.png]]

---

# 面试官最喜欢听到的总结

如果最后只能说一句话：

> Agent 不是简单的工具调用，而是具备自主规划、工具使用、记忆管理和反思能力的智能体。它通过感知 - 规划 - 行动 - 反馈的闭环机制，自主完成复杂任务，这是 Agent 与普通大模型最本质的区别。

---

# 三个最容易踩的坑

### 错误①

```text
Agent = 插件
```

❌

---

### 错误②

```text
Agent = Tool Calling
```

❌

---

### 错误③

只说工具调用，不说闭环

❌

---

# 面试速记（背这个）

```text
Agent = 自主闭环系统

核心能力：
1. Planning
2. Tool Use
3. Memory
4. Reflection

核心流程：
Perceive
→ Plan
→ Act
→ Observe
→ Repeat

核心区别：
LLM负责思考
Agent负责完成任务

协议：
MCP = Agent ↔ Tool
A2A = Agent ↔ Agent
```

如果准备 Agent 岗位，这一题基本是整个 Agent 面试体系的总纲，后面的 ReAct、Memory、Multi-Agent、Reflection、MCP、A2A 都是在展开这几个核心概念。