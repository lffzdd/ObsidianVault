Searched for "class CoreMemory"
Listed directory schemas
Viewed memory.py:1-200
Viewed block.py:1-100

Letta（继承自 MemGPT 的设计理念）采用了一种**多级/分层记忆架构**（Multi-tier Memory Architecture）。这套架构的设计灵感来源于操作系统的内存管理机制（如 CPU 缓存、主存 RAM 和外部硬盘），旨在让 LLM（大语言模型）在有限的上下文窗口（Context Window）中，能够处理无限长的时间线并拥有持久的记忆。

以下是 Letta 记忆系统的核心三层架构：

```mermaid
graph TD
    subgraph Context Window (上下文窗口 - RAM)
        CoreMemory[Core Memory / 核心内存]
        CoreMemory --> BlockPersona["Persona Block (角色设定)"]
        CoreMemory --> BlockHuman["Human Block (用户信息)"]
        Messages[Recent Messages / 近期对话上下文]
    end

    subgraph External Storage (外部持久化存储 - 硬盘)
        RecallMemory[(Recall Memory / 对话历史库)]
        ArchivalMemory[(Archival Memory / 归档知识库)]
    end

    LLM[LLM / 决策大脑] <--> |直接读写| CoreMemory
    LLM <--> |Recent Messages| Messages
    LLM <--> |调用工具检索/追加| RecallMemory
    LLM <--> |调用工具插入/向量检索| ArchivalMemory
```

---

### 1. 核心内存 (Core Memory - 相当于 RAM)
* **特点**：直接放入 LLM 的上下文窗口或 System Prompt 中，是 LLM 在对话时**唯一能够直接感知和读写**的区域。
* **组成 (Blocks)**：它是由多个模块化、有字符上限限制的块（`Block`）组成的。最典型的两个 Block 是：
  * **Persona**：Agent 自身的角色设定（我是谁、我的目标和性格）。
  * **Human**：当前对话用户的背景信息（用户的名字、喜好、习惯等）。
* **动态更新**：当 LLM 发现关于用户或自身设定的新信息时，它可以通过调用内置的记忆工具（例如 `core_memory_append` 或 `core_memory_replace`）在对话中**实时修改**这些 Block。修改后的内容会立即应用到下一次对话的上下文里。

### 2. 召回内存 (Recall Memory - 相当于历史对话数据库)
* **特点**：存储在外部数据库（如 PostgreSQL 或 SQLite）中。它是一个**只读且完整的历史日志**，完整地保留了 Agent 自创建以来的每一条消息（包括用户的输入、Agent 的思考过程 `thought`、工具调用以及工具返回的结果）。
* **检索方式**：LLM 无法直接看到所有的历史，但当它需要寻找过去的某段对话细节时，可以通过调用检索工具（如 `conversation_search` 或基于时间的 `conversation_search_date`），从 Recall Memory 中把匹配的内容临时读取出来，加载到当前的上下文窗口中。

### 3. 归档内存 (Archival Memory - 相当于硬盘 / 向量数据库)
* **特点**：用于存储任意、大量的结构化或非结构化知识、背景事实或长篇文档（这些无法全部装入 Core Memory）。通常由外部的关系型数据库或向量数据库（Vector DB）支持。
* **读写方式**：
  * **写入**：LLM 可以通过调用内置工具（如 `archival_memory_insert`）将新的知识/片段插入 Archival Memory 中。
  * **搜索**：LLM 使用类似 `archival_memory_search` 的工具，通过语义向量检索（Embedding Search）或全文检索（Full-text Search）来查找相关知识，并将检索到的片段临时载入上下文。

---

### 记忆架构的调度中枢：Tool Execution Loop
Letta 能够实现这套记忆管理的关键在于它的**工具执行循环（Tool Execution Loop）**：
1. 当收到用户输入时，Agent 的 LLM 开始思考。
2. 如果 LLM 觉得需要查找记忆，或者需要更新核心内存，它会**先发起一个或多个 Tool Call**（如 `core_memory_replace` 或 `archival_memory_search`），而不会直接将回复输出给用户。
3. 系统在后台自动执行这些记忆工具，并把更新后的核心内存或检索到的结果追加回上下文。
4. LLM 拿到最新的上下文后，再做出最终的解答，从而实现了**自我纠正、记忆自我演进和深度反思**。