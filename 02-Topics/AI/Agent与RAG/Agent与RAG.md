---
type: 入口笔记
---
# Agent与RAG

## 当前理解

Agent = LLM + 工具 + 循环：从 CoT、ReAct 到 Plan 模式，本质是让模型在“思考—调用工具—观察”的循环里推进任务。工程上真正难的是上下文管理（压缩）、多智能体协同、评测，以及让 skill / 工具对模型“可见且好用”。

## 笔记（按时间）

- [[02-Topics/AI/Agent与RAG/2026-01-27 AI上线生产代码|2026-01-27 AI上线生产代码]] — 讲了：Research / Proposal
- [[02-Topics/AI/Agent与RAG/2026-03-22 AI Agent开发要学什么|2026-03-22 AI Agent开发要学什么]] — 讲了：Gemini / 第一部分：历史脉络与底层逻辑 (The Evol / 第二部分：开发要学什么？（自底向上的技术栈）
- [[02-Topics/AI/Agent与RAG/2026-03-22 CoT|2026-03-22 CoT]] — 讲了：第二步：历史脉络与发展阶段 (The Evolu
- [[02-Topics/AI/Agent与RAG/2026-03-22 Prompt Engineering|2026-03-22 Prompt Engineering]] — 讲了：一、先建立正确理解 / 二、最有用的一个框架 / 三、先看最直观的对比
- [[02-Topics/AI/Agent与RAG/2026-03-25 现代ReAct|2026-03-25 现代ReAct]] — 讲了：现代 ReAct：从 Prompt Hack 到 / 一、先说结论 / 二、经典 ReAct：模型真正看到的文本
- [[02-Topics/AI/Agent与RAG/2026-03-27 AI岗位要求|2026-03-27 AI岗位要求]] — 讲了：AI岗位要求
- [[02-Topics/AI/Agent与RAG/2026-03-27 CoT与ReAct之后的学习路径|2026-03-27 CoT与ReAct之后的学习路径]] — 讲了：学完 ReAct 后的进阶路线 / 一、三条主要路线 / 路线 A：多 Agent 协作（最热门）
- [[02-Topics/AI/Agent与RAG/2026-03-27 多智能体协同|2026-03-27 多智能体协同]] — 讲了：第一部分：历史脉络与底层逻辑 —— 为什么要“拉 / 第二部分：底层通信机制 (Topologies  / 第三部分：自底向上 —— 纯 Python 手写
- [[02-Topics/AI/Agent与RAG/2026-03-27 状态机和图工作流|2026-03-27 状态机和图工作流]] — 讲了：第一步：历史脉络 —— 为什么要从“循环”走向“ / 第二步：图工作流的三大底层概念 (The Cor / 第三步：自底向上 —— 纯 Python 手写一
- [[02-Topics/AI/Agent与RAG/2026-03-27 评测|2026-03-27 评测]] — 讲了：一、 记忆系统与知识外挂 (RAG: Retri / 二、 评测与可观测性 (Evals & Obse / 三、 异步架构与流式输出 (Async & St
- [[02-Topics/AI/Agent与RAG/2026-05-11 Agent开发学习路线 1 （Claude Opus 4.7）|2026-05-11 Agent开发学习路线 1 （Claude Opus 4.7）]] — 讲了：Agent 开发学习路线（2026年版） / 阶段一：打好 LLM 基础（1–2 周） / 阶段二：理解 Agent 核心概念和模式（2–3
- [[02-Topics/AI/Agent与RAG/2026-05-11 Agent开发学习路线 2|2026-05-11 Agent开发学习路线 2]] — 讲了：阶段二展开：吃透 Agent 核心模式 / 0. 地基：增强 LLM（Augmented L / 1. Prompt Chaining（提示链）
- [[02-Topics/AI/Agent与RAG/2026-06-13 Agent开发 - 上下文压缩|2026-06-13 Agent开发 - 上下文压缩]] — 讲了：1. Server-side compactio / 2. Context editing：不是总结， / 3. Thinking block cleari
- [[02-Topics/AI/Agent与RAG/2026-08-01 Agent开发 - Plan模式的实现 Explaining ReAct Planning Logic|2026-08-01 Agent开发 - Plan模式的实现 Explaining ReAct Planning Logic]] — 讲了：Chat Conversation / User Input / Planner Response
- [[02-Topics/AI/Agent与RAG/2026-08-18 Agent - LLM眼里的skill|2026-08-18 Agent - LLM眼里的skill]] — 讲了：第 1 轮：大模型刚调用完  load_skil / 第 2 轮：大模型执行完了  pytest ，准 / 第 3 轮：大模型全部执行完毕，调用了  unl
- [[02-Topics/AI/Agent与RAG/2026-09-19 AI做架构|2026-09-19 AI做架构]] — 讲了：一、 时间节点：AI 是从什么时候开始“真能做架 / 二、 目前真正能用来做架构的模型有哪些？ / 1. Claude Fable 5.1 / Cl
- [[02-Topics/AI/Agent与RAG/2026-10-07 AI Agent实习简历草稿|2026-10-07 AI Agent实习简历草稿]] — 讲了：姓名（待填） / 教育经历 / 专业技能
