# Letta (formerly MemGPT) 深度学习与架构设计笔记

Letta（前身是 **MemGPT**）是一个革命性的 AI Agent 开发框架。它的核心设计理念是**将大语言模型（LLM）视为操作系统的 CPU**，并通过引入类似操作系统的虚拟内存管理机制，为 LLM 提供持久化、可自我演进的多级记忆系统。

---

## 一、 核心概念与项目现状

### 1.1 项目定位
* **MemGPT 阶段**：最初作为一个学术和开源项目推出，探讨如何解决 LLM 有限上下文窗口（Context Window）的限制。
* **Letta 阶段**：品牌重塑为 Letta，旨在为开发者提供构建生产级、状态化（Stateful）Agent 应用程序的基础设施。

### 1.2 仓库架构分工（当前状态）
由于项目快速迭代，Letta 的仓库进行了拆分：
* **`letta-ai/letta`（当前遗留仓库）**：自托管的 API 服务端，提供 Letta V1 API 和旧版的 SDK，目前处于维护状态（Legacy）。
* **`letta-ai/letta-code`**：最新的 Agent 核心，包含 CLI 命令行工具、频道集成（Slack 等）和 Agent 运行沙箱。
* **App Server & Constellation**：取代了 V1 API Server，作为最新的云端/本地 Agent 托管引擎。

---

## 二、 记忆系统三层架构 (Memory Architecture)

Letta 模拟了计算机的存储层级，设计了“工作内存-历史缓存-外部硬盘”的记忆架构：

```mermaid
graph TD
    subgraph LLM Context Window (上下文窗口 / RAM)
        SystemPrompt[System Prompt / 基础系统指令]
        CoreMemory[Core Memory / 核心工作内存]
        CoreMemory --> BlockPersona["Persona Block (角色设定)"]
        CoreMemory --> BlockHuman["Human Block (用户信息)"]
        Messages[In-Context Messages / 近期对话队列]
    end

    subgraph Relational / Vector DB (外部持久化存储 / 磁盘)
        RecallMemory[(Recall Memory / 召回对话历史库)]
        ArchivalMemory[(Archival Memory / 归档知识库)]
    end

    LLM[LLM / 决策核心] <--> |直接读写| CoreMemory
    LLM <--> |阅读当前上下文| Messages
    LLM <--> |调用工具检索| RecallMemory
    LLM <--> |调用工具插入/检索| ArchivalMemory
```

### 2.1 核心内存 (Core Memory - 相当于 RAM)
* **认知映射**：**工作记忆（Working Memory）**。
* **物理位置**：处于 LLM 的 Context Window（上下文）的最头部（System Message 的一部分）。LLM 随时可以直接感知。
* **组织结构**：由多个有字符上限的 `Block` 构成，默认主要有：
  * `<persona>`：Agent 自身的身份设定、性格和行为守则。
  * `<human>`：用户的个人画像、喜好、开发习惯和限制。
* **动态更新**：LLM 在运行过程中，可以通过调用内置的记忆工具（如 `core_memory_replace` 或 `core_memory_append`）主动修改这些 Block。
* **缓存代价**：因为 Core Memory 位于 Context 的头部，**只要它发生修改，下一次请求时大模型服务商的 Prompt Cache（如 KV 缓存）就会宣告失效**。因此，Letta 在程序中通过 Substring 匹配等方式，尽量避免在数据未发生实质变化时重构系统提示词。

### 2.2 召回内存 (Recall Memory - 相当于 L2/L3 缓存)
* **认知映射**：**情景记忆（Episodic Memory）**。
* **物理位置**：保存在外部关系型数据库中（Out-of-context），LLM 无法直接看到。
* **数据内容**：完整且无损的原始对话日志（包括 User 的发言、Assistant 的思考 thoughts、调用的 Tool 和返回的原始信息）。不进行任何主动的压缩，以防止细节丢失。
* **检索方式**：LLM 可以通过调用工具（如 `conversation_search`）对 Recall 数据库进行检索，将符合时间段或关键词的聊天原文片段临时载入当前的上下文窗口中。

### 2.3 归档内存 (Archival Memory - 相当于硬盘 / 外部存储)
* **认知映射**：**语义记忆（Semantic Memory）**。
* **物理位置**：保存在向量数据库（Vector DB）中，存储量理论上无限制。
* **数据内容**：
  * 用户上传并切片（Chunking）后的外部文件（在底层称为 `SourcePassage`）。
  * Agent 自身提炼总结并主动存入的长期知识和事实（在底层称为 `ArchivalPassage`）。
* **检索方式**：LLM 可以调用 `archival_memory_search` 进行向量相似度匹配（或者降级的文本模糊搜索），将匹配到的 Passage 载入上下文。

---

## 三、 各个内存的具体实现细节（写入/嵌入、修改、召回）

### 3.1 核心内存 (Core Memory)
* **写入与嵌入机制 (Ingestion & Embedding)**：
  * **非向量化**：虽然叫 "Memory"，但在工程上它并不使用特征向量（Embedding），而是直接以文本形式塞入 Prompt。
  * **动态拼接**：在每次推理前，Letta 会调用 `Memory.compile()`，循环遍历与当前 Agent 关联的所有 `Block`（如 `persona` 和 `human` 块），将它们的 label（标签）、value（具体内容值）、chars_limit（字符上限）和 metadata，用特定的 XML 标签进行嵌套格式化：
    ```xml
    <memory_blocks>
    The following memory blocks are currently engaged in your core memory unit:
    <persona>
    <description>Your Persona Description</description>
    <value>Your Persona Content...</value>
    </persona>
    ...
    </memory_blocks>
    ```
  * **上下文定位**：该 XML 字符串会被直接拼接放入 `system` 角色消息（System Message）的内容体中。该消息作为对话消息队列的 **Message[0]**，随整个上下文窗口被发送给 LLM。
* **修改机制 (Modification)**：
  * **LLM 发起修改**：大模型通过输出 JSON，调用内置工具 `core_memory_replace(label, old_content, new_content)` 或 `core_memory_append(label, content)`。
  * **数据落地**：后端拦截到调用后，在内存的 `Block` 中修改 value，并运行 SQL `UPDATE` 将其更新到关系型数据库的 `blocks` 表中。
  * **重构提示词**：接着调用 `rebuild_system_prompt_async()`，重新 compile 最新的 Block 内容，并在数据库的 `messages` 表中通过 SQL 更新系统消息（即 Message[0]）的 `content` 字段。下一次推理时，LLM 看到的 System Message 就已经发生了改变。
* **召回与检索机制 (Recall & Retrieval)**：
  * **无**：由于核心内存常驻在 LLM 上下文的最头部（In-context），LLM 始终可以直接读取并使用它，不需要任何“检索/召回”工具。

### 3.2 召回内存 (Recall Memory)
* **写入与嵌入机制 (Ingestion & Embedding)**：
  * **系统级捕获**：它是完全自动的。对话生命周期中发生的任何交互（用户消息、大模型输出、中间的思考、甚至 Tool 调用的参数和返回值），都会在回合结束前通过 `MessageManager.create_many_messages_async` 以 `MessageModel` 实体的形式被保存进数据库。
  * **异步向量化**：部分 Letta 版本会在写入后，于后台启动一个 `_embed_messages_background` 的异步协程，调用 Embedding API 为这些历史消息的纯文本生成特征向量，并存在表中的 vector 字段以备未来的语义搜索。
* **修改机制 (Modification)**：
  * **日志不可变性**：为了保证历史的客观真实，AI 无法修改已经写入 Recall Memory 的原始消息日志。
  * **上下文移出（被动压缩）**：当上下文窗口变满，系统触发 `summarize_messages_inplace`。在将最老的一部分消息使用大模型总结为 "Summary Message" 之后，这部分旧的原始消息会在发送给 LLM 的 in-context message 数组中被移除。它们依然完好存在于 Recall 数据库中，但不再占用 LLM 当前的 Context Window。
* **召回与检索机制 (Recall & Retrieval)**：
  * **工具驱动**：LLM 感觉当前上下文信息不足，可以通过调用 `conversation_search(query, roles, start_date, end_date, limit)` 工具来检索。
  * **检索过程**：
    1. **SQL 模式 (默认)**：直接在 `messages` 表中执行 `SELECT` 语句，过滤特定 `agent_id`。如果提供了 `query`，则使用 `LIKE / contains` 对内容做文本匹配，并且过滤掉 Tool 返回消息和调用了 `conversation_search` 自身的辅助消息（防止递归套娃和内容污染）。
    2. **向量模式 (Turbopuffer)**：如果配置了外部向量云服务，则将 `query` 文本生成 Embedding 向量，并发送给云端执行混合相关的向量检索。
  * **最终反馈**：查出来的多条历史消息，会被拼成 JSON 列表，作为 Tool Call 的 Response 返回给大模型。

### 3.3 归档内存 (Archival Memory)
* **写入与嵌入机制 (Ingestion & Embedding)**：
  * **写入入口**：有两种写入方式：
    1. **外部文件**：用户通过后台上传大文件，被切片成 Chunks（Passages），生成 `SourcePassage`。
    2. **AI 主动写入**：LLM 主动调用 `archival_memory_insert(content, tags)` 工具，向底层写入 `ArchivalPassage`。
  * **特征向量计算**：无论是哪种写入，`PassageManager` 都会通过 Embedding API（如 OpenAI Embeddings 接口）将文本转化为 1536 维等高维浮点数向量。
  * **持久化底层**：
    * 如果数据库是 **PostgreSQL**，利用 `pgvector` 扩展的 `Vector` 字段类型直接保存。
    * 如果数据库是 **SQLite**，通过 Python 侧自定义的序列化 adapter 将浮点数数组存入二进制字段。
* **修改机制 (Modification)**：
  * LLM 可以通过 `archival_memory_update` 修改归档记录的文本，这会触发后端对新文本的 Embedding 重新计算并做 SQL `UPDATE`；它也可以调用工具删除某条 Passage。
* **召回与检索机制 (Recall & Retrieval)**：
  * **语义检索**：LLM 调用 `archival_memory_search(query, tags, top_k, ...)`。
  * **检索执行流程**：
    1. **向量化 Query**：后端大模型收到 query 字符串，首先调用 Embedding Client 将其转换为与知识库相同维度的浮点数向量。
    2. **计算向量距离**：
       * 在 **Postgres** 中，使用 `<=>` 符号计算 cosine_distance，并使用 `ORDER BY` 排序：
         `SELECT * FROM archival_passages ORDER BY embedding <=> :query_embedding LIMIT :top_k`
       * 在 **SQLite** 中，使用 Letta 进程里注册的自定义 SQL 函数计算：
         `SELECT * FROM archival_passages ORDER BY cosine_distance(embedding, :query_embedding) LIMIT :top_k`
       * 在 **Turbopuffer** 中，使用 TPUF SDK 进行混合向量匹配打分（Hybrid Search）。
    3. **反馈载入**：返回最匹配的 Passage Chunks 和对应的 tags 给大模型作为 Tool Result，大模型阅读后将知识融入回复。

---

## 四、 底层工程实现与数据库选型

### 4.1 向量存储的适配与兼容
在 `letta/services/passage_manager.py` 中，Letta 支持多种向量数据库后端，并针对不同引擎进行了底层的适配：

* **PostgreSQL + pgvector (推荐后端)**：
  * 在数据库中定义 `embedding` 字段为 `Vector(MAX_EMBEDDING_DIM)` 类型。
  * 检索时直接利用 `pgvector` 的 SQL 扩展进行排序：
    ```python
    query = query.order_by(ArchivalPassage.embedding.cosine_distance(embedded_text).asc())
    ```
* **SQLite (本地嵌入开发)**：
  * 因为 SQLite 默认没有向量检索功能，Letta 会在初始化 SQLite 连接时，在 Python 进程中**自定义注册**一个名为 `cosine_distance` 的 SQL 函数。
  * 检索时调用自定义函数进行排序：
    ```python
    query = query.order_by(func.cosine_distance(ArchivalPassage.embedding, query_embedding_binary).asc())
    ```
* **Turbopuffer (TPUF - 云端向量库)**：
  * 整合了专业的云端向量数据库。支持 **RRF（Reciprocal Rank Fusion）** 算法进行向量搜索与全文检索（FTS）的**混合搜索（Hybrid Search）**，召回效果更佳。

### 4.2 召回内存与归档内存的物理隔离
* **Recall Memory**：映射为数据库中的 `messages` 表。通过 `agent_id` 过滤。所有会话（Session）的历史记录都存在一起。
* **Archival Memory**：映射为数据库中的 `archival_passages` 表。通过 `archive_id` 与具体的 Agent 实例绑定。
* **结论**：**所有的记忆都是跨 Session 存在的**。即使开启了新的连接 Session，由于它们属于同一个 Agent 物理实例，过去的所有历史对话和归档知识均可共享并搜索到。

---

## 四、 Agent 运行控制流 (Tool Execution Loop)

Letta 绝不是常规的“提问-回答”工作流，它是一个基于自循环的 Agent 操作系统。

### 4.1 心跳链式循环（Heartbeat Chaining）
在大脑决策（LLM Inference）到给用户回信之间，Letta 的程序实现了一个 `while True` 的**工具执行循环（Tool Execution Loop）**：
1. **大模型思考**：LLM 吐出 `thought`，并决定执行记忆操作（如 `archival_memory_insert`）。
2. **程序拦截与执行**：Letta Runtime 拦截了这一非 `send_message` 的调用，在本地/数据库中执行完毕。
3. **系统返回（Heartbeat）**：将执行结果伪装成“系统返回的消息”再次喂给 LLM。
4. **循环直到产出回复**：LLM 根据返回结果，选择继续查别的，或者调用 `send_message` 将内容发送给用户。只有遇到 `send_message` 或无工具调用时，循环才会退出。

### 4.2 语法与输出约束
为了保证 Agent 在连续循环中不掉链子（比如突然胡言乱语），程序做出了强力的类型和格式控制：
* **本地 LLM 侧**：使用 **GBNF 语法规则文件** 限制输出，在 Token 生成阶段就强制 LLM 只能输出预定格式的 JSON，无法输出杂乱文本。
* **云端 API 侧**：利用 `tool_choice="required"` 和 API 提供商的 `JSON Schema` 约束机制，确保返回百分之百的结构化数据。

### 4.3 上下文压缩与警告机制
* 当 Context Window 的 Token 使用率超标（如 80%）时，Letta 运行时会在消息队列尾部自动注入一条隐式的“系统压力警告”。
* LLM 收到警告后，会在下一个思考周期主动发起内存收缩，调用**摘要压缩机制（In-place Summarization）**：把最老的对话用 LLM 合并为一条“摘要消息”，并在当前 Context Window 中将其删除（原始对话仍妥善保存在 Recall 数据库中），从而腾出空间。

---

## 五、 横向对比：Letta 与 Claude-code

在本地项目 `Claude-code` 中，Anthropic 也设计了一套称为 `memdir` 的记忆系统。两者的实现存在明显的分水岭：

| 维度 | Letta (原 MemGPT 架构) | Claude-code (`memdir` 架构) |
| :--- | :--- | :--- |
| **存储物理介质** | 数据库（Postgres / SQLite） | 本地项目目录下的 Markdown 文件（配合 Git 管理） |
| **工作记忆 (Core)** | System Prompt 里的 Persona & Human 文本块 | System Prompt 自动加载索引文件 `MEMORY.md` |
| **检索方式** | 依靠 **向量距离** 进行数学上的相似度检索 | 依靠 **后台轻量级 Sonnet 側查询（Side Query）** 自动阅读本地文件的文件名和描述清单进行人类语义挑选 |
| **检索控制权** | **完全的 Agent 决策**。由 LLM 决定何时查、怎么查。 | **前置的工作流（Workflow-hybrid）**。在用户输入后、主模型启动前，程序强制在后台自动检索出相关文件，并作为 Attachment（附件）挂载发送给主模型。 |
| **写入与修改** | LLM 在运行循环中随时调用 API 写入数据库 | LLM 在运行时使用文件写入工具修改本地的 Markdown 文件 |

### 💡 架构设计启示
* **Letta** 倾向于构建一个**独立自足的、像人脑一样具有自律性的闭环 Agent**，依靠复杂的数据库基础设施支持复杂的语义多跳推理。
* **Claude-code** 则追求**极佳的工程实用性与极速的响应**。它通过“前置工作流自动筛选文件挂载”避免了 Agent 每次都得自己检索的昂贵延迟，并用纯文本 Markdown 绕过了臃肿的数据库体系，通过 Git 实现了版本控制和团队共享。
