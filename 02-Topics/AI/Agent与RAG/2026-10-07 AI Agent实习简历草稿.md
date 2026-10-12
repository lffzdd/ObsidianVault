# 姓名（待填）

**求职意向：AI Agent 开发实习生**  
电话：待填｜邮箱：待填｜GitHub：[lffzdd](https://github.com/lffzdd)

## 教育经历

**中南林业科技大学｜电子信息｜硕士**  
2024.09—2027.07（预计）

## 专业技能

- **Python 与后端：**熟悉 Python、FastAPI、Flask，理解面向对象、模块化设计、异步编程与并发处理，掌握 HTTP、RESTful API、SSE 等接口与通信基础。
- **Agent 与框架：**熟悉 ReAct、Function Calling、MCP、Skills、Memory 及多 Agent 协作，理解任务规划、上下文管理与工具权限边界；熟悉 LangChain、LangGraph 的基本使用。
- **RAG 与检索：**熟悉 Embedding、BM25、混合召回、RRF、Rerank、查询增强与多跳检索，理解 Agentic RAG、GraphRAG、Self-RAG、CRAG 等技术思路及 RAGAS 评估指标。
- **深度学习基础：**熟悉 PyTorch，理解 Transformer、Attention、位置编码、Tokenization、BPE 与自回归生成，具备模型训练和推理基础。
- **多模态与生成模型：**熟悉 ASR、TTS、OCR 等应用技术及 ComfyUI；了解 Stable Diffusion、FLUX、Qwen-Image、Wan，理解 LoRA、ControlNet 与 Inpainting 的用途及使用方式。
- **数据库与检索组件：**熟悉 MySQL、PostgreSQL、Redis，理解索引、事务、锁与缓存策略；了解 Elasticsearch、ChromaDB、Milvus 的基本使用及文本、向量检索原理。
- **工程与部署：**熟悉 Git、Linux、Docker、Nginx，具备环境配置、容器部署、接口联调与日志排查能力，理解 Git worktree、消息队列及常见设计模式。
- **开发与自动化工具：**熟练使用 Codex、Claude Code、Cursor，能够辅助代码开发、重构与调试；熟悉 n8n 工作流编排及 Firecrawl 网页采集。

## 实习经历

### 尚霖科技｜AI 应用开发实习生

2025.11—2026.07

**技术栈：**Python、FastAPI、Docker、PaddleOCR-VL、ComfyUI、RAGFlow  
**模型与组件实践：**Stable Diffusion、FLUX、Qwen-Image、Wan、LoRA、ControlNet、Elasticsearch、ChromaDB、Milvus、Nginx

- **多模态 OCR 服务与业务应用：**通过 Docker 部署 PaddleOCR-VL 文档解析模型，基于 FastAPI 封装图片上传与识别接口，串联表格内容提取、格式转换及业务标记统计流程；针对业务表格开发自动识别工具，将模型解析结果转化为可复核的统计结果，交付内部员工日常使用。
- **ComfyUI 多模型与自定义工作流：**部署 ComfyUI 及本地生成模型，围绕 Stable Diffusion、FLUX、Qwen-Image 组合并调试节点式工作流，覆盖文生图、图生图、Inpainting 局部重绘、背景替换与图像放大；通过提示词、采样参数及节点连接调整生成流程，为同事提供可复用的服装素材制作工作流，产出用于宣传册与内部搭配参考。
- **可控生成与视频工作流：**接入 LoRA 与 ControlNet，围绕风格适配、构图约束及局部编辑调试生成控制流程，将模型加载、条件输入、采样与后处理组合为自定义工作流；开展 Wan 视频生成模型与工作流实践，探索图像素材向视频内容的扩展。
- **企业知识库与检索组件：**基于 Docker 部署 RAGFlow，面向服装产品资料与新员工培训构建知识问答应用，完成资料导入、知识库配置及问答调试，支持产品信息、岗位职责与部门协作内容查询；涉及 Elasticsearch、ChromaDB、Milvus 等检索与向量存储组件实践，以及 Nginx 服务部署配置。

### 北京绿洲伟业｜AI 应用开发实习生

2025.07—2025.10

**技术栈：**SenseVoice、CosyVoice、LLM API、SSE、Redis、PostgreSQL、n8n、Firecrawl

- **语音模型集成与端到端交互：**参与边缘设备语音机器人开发，配置并集成 SenseVoice 语音识别模型与 CosyVoice 语音合成模型，对接云端 LLM API，打通“音频输入—ASR 文本转写—LLM 对话生成—TTS 合成播报”链路，完成硬件机器人语音对话原型演示。
- **情绪感知与流式语音响应：**接入用户情绪识别（Emotion Recognition）能力，为机器人交互提供情绪状态信息；基于 SSE 接收 LLM 增量文本响应，按句传递至 CosyVoice 进行分段语音合成，衔接文本生成与语音播报，使机器人能够在完整回复生成前开始响应。
- **实时消息缓冲与异步持久化：**采用 Producer–Consumer 生产者—消费者架构，以 Redis 消息队列缓冲流式对话文本，由独立消费者持续读取并写入 PostgreSQL；分离模型输出、消息缓冲与日志持久化职责，支持生成过程中的实时记录，并为增加实时分析处理环节预留消费入口。
- **LLM 驱动的资讯自动化：**基于 n8n 工作流编排与 Firecrawl API，搭建时尚网站资讯定时采集流程，串联网页内容提取、LLM 摘要生成与内容分类，完成外部 API 集成、处理节点配置及端到端联调，将原始网页资讯整理为摘要与分类结果，交付客户。

## 项目经历

### Wright｜个人编程 Agent

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
- **RAG 评估与验证：**基于 RAGAS 指标构建可审计的 LLM-as-a-Judge 评估流程，覆盖 Faithfulness、Answer Relevancy、Context Precision 与 Context Recall；提供基础检索与增强配置的对比入口，保留裁判判断及分数计算过程，并编写测试覆盖多跳依赖、失败重规划、记忆持久化与异常降级。e