**Wright｜个人编程 Agent**  
2026—至今｜[项目仓库](https://github.com/lffzdd/wright)｜[早期实现](https://github.com/williamlao-org/LLM)

**技术栈：**Python、Function Calling、MCP、RAG、NetworkX、RAGAS、SQLite、Git worktree

围绕代码编写、文件修改与命令执行开发个人 Agent，提供 CLI、TUI 与 Web 交互入口；项目演进涵盖前期 Agentic RAG、GraphRAG 与分层记忆原型，以及后续工具编排、多 Agent 协作和执行权限体系。

- **Agent 执行与任务规划：**构建 ReAct 多轮推理与工具执行循环，由自定义 JSON 协议迁移至原生 Function Calling；实现 Plan 创建、步骤跟踪与重新规划，结合完成检查验证未完成任务、工具状态及文件产物，形成规划、执行与反馈调整流程。
    
- **MCP 与 Skills 动态扩展：**统一封装文件、Shell 与 MCP 工具，管理外部服务连接及调用生命周期；通过 Skills 元数据发现、正文按需加载与 `tool_search` 工具发现机制，动态暴露工具定义，控制工具和指令占用的上下文规模。
    
- **混合检索与查询增强：**在前期 RAG 原型中实现多格式文档解析、PDF OCR 回退、递归分块与向量索引；组合 Dense Retrieval、BM25、RRF 排名融合与 Rerank，接入 Query Rewriting、HyDE 和 Multi-Query，支持查询扩展、多路召回及候选精排。
    
- **自适应多跳检索与纠错：**实现 Adaptive Plan-and-Execute，将复杂问题拆解为带依赖的子查询，传递前置事实与实体信息；结合 Self-RAG／CRAG 思路逐跳评估证据相关性与充分性，支持查询改写、局部重试、剩余计划重规划及提前终止，通过步数与重试上限控制执行成本。
    
- **GraphRAG 图检索：**实现 LLM 实体关系抽取、Gleaning 补充抽取及合并去重，基于 NetworkX 与 Leiden／Louvain 完成社区检测和摘要生成；构建实体邻域 Local Search 与社区摘要 Map-Reduce Global Search，并将图检索封装为 Agent 工具，支持关系查询与全局主题问答。
    
- **上下文与分层记忆：**围绕长任务上下文管理，实现 Token 预算裁剪、滚动摘要与结构化工作记忆原型；构建 Core Memory 常驻注入、Semantic Memory 事实提炼和 Episode Memory 执行经历记录与召回，探索基于重要性、访问频次及闲置时间的记忆淘汰；编程 Agent 支持历史工具结果压缩、完整会话保存与检查点恢复。
    
- **多 Agent 协作与执行控制：**实现独立上下文的子 Agent 并发委派、任务树及状态跟踪，共享步数与 Token 预算，支持取消向后代传播；统一处理工具访问声明、范围解析、策略审批与单次执行授权，并通过 Git worktree 为 Web 会话建立独立工作目录。
    
- **RAG 评估与验证：**基于 RAGAS 指标构建可审计的 LLM-as-a-Judge 评估流程，覆盖 Faithfulness、Answer Relevancy、Context Precision 与 Context Recall；提供基础检索与增强配置的对比入口，保留裁判判断及分数计算过程，并编写测试覆盖多跳依赖、失败重规划、记忆持久化与异常降级。