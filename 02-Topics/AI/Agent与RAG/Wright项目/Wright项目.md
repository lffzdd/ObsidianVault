---
type: 入口笔记
---
# Wright项目

## 当前理解

Wright 是自己做的 Agent 项目：从 ReActMulti 内核出发，逐步演进到 ToolExecutor / Execution 分层、工具调用权限把关、交互邮箱与双线程设计，并暴露出测试体系上的隐患。这里汇总架构评估、重构交接和简历描述。

## 笔记（按时间）

- [[02-Topics/AI/Agent与RAG/Wright项目/2026-06-10 Agent内核升级路线|2026-06-10 Agent内核升级路线]] — 讲了：Agent内核升级路线
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-06-10 ReActMulti验收与上下文管理|2026-06-10 ReActMulti验收与上下文管理]] — 讲了：我顺手改了这些 / ③：上下文管理（token 预算 → 历史压缩）
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-07-24 论文版本和思考深度选择|2026-07-24 论文版本和思考深度选择]] — 讲了：论文版本和思考深度选择
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-09-19 ToolExecutor与Execution架构评估|2026-09-19 ToolExecutor与Execution架构评估]] — 讲了：评估ToolExecutor与Execution
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-09-19 Wright重构任务交接提示词|2026-09-19 Wright重构任务交接提示词]] — 讲了：Wright 重构任务交接提示词 / 使用前须知(给你自己看的,不要粘给执行者) / Prompt 1 — 接上已有的质量关卡(最高优
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-09-19 交互邮箱与双线程设计|2026-09-19 交互邮箱与双线程设计]] — 讲了：核心全景：双线程与交互邮箱（Interactio / 逐步拆解关键代码细节 / 1.  prompt_permission  为
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-09-19 系统架构全景图|2026-09-19 系统架构全景图]] — 讲了：一、静态全景架构图（六大分层与物理模块） / 二、动态端到端协作图（三大核心流向） / 三、主客从属与生命周期速查表
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-09-23 工具调用权限把关流程|2026-09-23 工具调用权限把关流程]] — 讲了：全景流程图：一次工具调用是如何被“层层把关”的？ / 整个大过程分为 5 个核心环节（结合真实代码文件 / 环节 1：入口与调度中枢
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-09-23 测试体系的深层隐患|2026-09-23 测试体系的深层隐患]] — 讲了：隐患一：“温室 Mock”带来的假安全感（Fal / 测试直接 import 了具体的底层工具函数：
- [[02-Topics/AI/Agent与RAG/Wright项目/2026-10-07 Wright项目简历描述|2026-10-07 Wright项目简历描述]] — 讲了：Wright项目简历描述
