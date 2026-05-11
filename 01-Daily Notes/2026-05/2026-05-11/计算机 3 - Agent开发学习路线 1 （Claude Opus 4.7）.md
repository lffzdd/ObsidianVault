# Agent 开发学习路线（2026年版）

我先把整个学习路径分成五个阶段，每个阶段说明"学什么"和"怎么学"。**最重要的一条原则**：Anthropic 在《Building Effective Agents》中反复强调——从直接调用 LLM API 开始，许多模式只需几行代码就能实现；如果使用框架，要确保理解底层代码。不要一上来就学框架，那是大部分人卡住的原因。

## 阶段一：打好 LLM 基础（1–2 周）

这一步的目标是吃透"一次 LLM 调用"的全部细节。Agent 本质上就是循环调用 LLM + 工具 + 记忆，底层调不明白，上层再花哨也是空中楼阁。

需要掌握的：Transformer 和注意力机制的直觉理解（不需要会推公式，但要知道上下文窗口、token、温度这些为什么是这样）；prompt engineering（system/user/assistant 角色、few-shot、CoT、结构化输出/JSON mode）；function calling / tool use 的请求和响应格式；流式输出；token 计费和成本控制。

怎么学：直接读 Anthropic 和 OpenAI 的官方文档（docs.claude.com、platform.openai.com），跟着写代码。用纯 Python + requests 或者官方 SDK 自己手撸一个能调用工具的小 demo（比如让模型查天气、查股价），**不要用任何框架**。这一步建议产出 200–500 行自己写的代码。

## 阶段二：理解 Agent 核心概念和模式（2–3 周）

**必读材料**：Anthropic 的《Building Effective Agents》是这个领域最经典的入门文章，配套有 anthropic-cookbook 仓库的可运行代码。读完你会理解几个关键区分：

工作流（Workflow）vs Agent：工作流是 LLM 和工具通过预定义代码路径编排的系统；Agent 是 LLM 动态决定自己流程和工具使用的系统。大部分商业场景其实只需要工作流。

需要吃透的几种基础模式 Prompt Chaining（顺序处理，每步基于上一步）、Routing（智能分类与路由）、Parallelization（并发执行多个子任务）、Orchestrator-Workers（动态拆分与委派）、Evaluator-Optimizer（通过反馈循环迭代改进）。

增强型 LLM（Augmented LLM）的三件套：tools（工具调用）、retrieval（检索/RAG）、memory（记忆）。

ReAct 范式：Thought → Action → Observation 的循环，几乎所有 Agent 都基于这个思路。

怎么学：把上面 5 种模式各用纯 API 自己实现一遍。比如 prompt chaining 实现一个"提取合同条款 → 校验合规 → 生成摘要"；evaluator-optimizer 实现一个"写代码 → 跑测试 → 根据失败信息修改"的循环。每种模式 50–150 行代码。

## 阶段三：上框架（3–4 周）

有了手撸经验再上框架，才能看穿框架的抽象。2026 年的主流选择：

LangGraph 是复杂状态化工作流的首选，提供显式控制、分支、重试、HITL（人在回路）和时间旅行调试；学习曲线最陡但生产成熟度最高，已经超过 CrewAI 的 GitHub star 数。如果你只学一个，学这个。

Claude Agent SDK 是 Anthropic 的官方 Agent 框架，和驱动 Claude Code 的是同一套架构，原生支持 hooks、MCP、skills、subagents。如果你主要用 Claude，这个非常值得学。

CrewAI 是从想法到多 Agent 原型最快的路径，适合按角色分解的任务（researcher/writer/reviewer）。门槛最低，适合先建立直觉。

⚠️ 一个重要的更新：AutoGen 实际已经进入维护模式，微软将精力转向了更广义的 Microsoft Agent Framework。新项目不要从 AutoGen 入手了。

怎么学：选 LangGraph 作为主力。读官方教程把每种基础图（线性、分支、循环、子图、HITL）走一遍。然后用 LangGraph 重写阶段二里的那 5 种模式，对比手撸版本的差异——你会突然理解 checkpoint、reducer、state schema 这些概念为什么存在。

## 阶段四：协议与工程化（2–3 周）

2026 年 Agent 生态的两大协议：

**MCP（Model Context Protocol）**：由 Anthropic 贡献给 Linux Foundation 的 Agentic AI Foundation，正在成为 Agent 连接工具和数据源的标准。学会写一个自己的 MCP server，把它接到 Claude Desktop 或 Cursor 里用。

**A2A（Agent2Agent Protocol）**：Google 联合 50+ 技术伙伴发布，标准化 Agent 间的通信和发现。让不同框架的 Agent 能互相调用。

工程化必须掌握的几件事：

- **评估（Evals）**：这是分水岭。Agent 不是写完跑通就完事，必须有测试集和指标。先定义"成功"是什么（任务完成率、步骤数、token 成本、人类满意度），再用 LangSmith / Braintrust / 自己写的脚本做回归测试。
- **可观测性**：每一次 LLM 调用、每一次工具调用、每一次状态变化都要能追溯。LangSmith 或 Langfuse 选一个。
- **成本与延迟控制**：多 Agent 系统比单 Agent 多消耗 10–15 倍的 token，承诺复杂架构前先算账。学会缓存、用小模型做路由、并行化。
- **安全与 HITL**：在不可逆操作前设置人工审核检查点。

## 阶段五：动手做真实项目（持续）

学习的真正分水岭在这里。我推荐按难度递增做这四个项目，每个都要做到能给别人用的程度，不是 demo 级别：

1. **个人知识助手**（RAG + 记忆）：把你的笔记、PDF、网页书签做成可问答的 Agent。考验检索质量和长期记忆。
2. **代码 Agent**（工具使用 + 循环）：能读取仓库、运行测试、修 bug 的小 Agent。考验工具设计和错误恢复。
3. **多 Agent 研究系统**（编排）：给一个问题，研究员 Agent 搜资料、写手 Agent 起草、审稿人 Agent 评审、循环迭代。考验 orchestrator-workers 和 evaluator-optimizer 模式。
4. **生产部署**：把上面任意一个挂到 Web 上，加上认证、限流、监控、计费。这一步会暴露你之前所有偷懒的地方。

## 学习节奏建议

如果是全职学习，整个路径 2–3 个月可以走完一遍；业余每天 1–2 小时大概 4–6 个月。一个常见的失败模式是**模式追逐**：今天看 LangGraph，明天看 CrewAI，后天看 AutoGen，没有一个项目跑完。**选一条主线，做完一个完整项目，再去看其他框架的差异**，这时候你才有判断力。

## 核心资源清单

- **必读文章**：Anthropic 的 Building Effective Agents（anthropic.com/research/building-effective-agents）、anthropic-cookbook 的 patterns/agents 目录
- **官方文档**：docs.claude.com（Anthropic）、LangGraph 官方教程、modelcontextprotocol.io
- **视频**：Barry Zhang 在 AI Engineer Summit 的 "How We Build Effective Agents" 演讲
- **代码仓库**：anthropic-cookbook、langchain-ai/langgraph 的 examples 目录
- **进阶论文方向**：ReAct、Reflexion、Toolformer、Voyager、SWE-agent

最后给一个 Anthropic 的核心心法，照着做能少走很多弯路：在 LLM 领域成功的关键不是构建最复杂的系统，而是为你的需求构建正确的系统。从简单的 prompt 开始，用全面的评估优化它，只在简单方案不够用时才加多步 Agent 系统。

需要的话我可以针对某个阶段（比如阶段二的 5 种模式实现，或者阶段五的某个项目）展开讲具体怎么做。你目前的编程基础和方向（Python 熟练度、是否做过 LLM 应用、想做哪类 Agent）是怎样的？我可以根据这个再细化。