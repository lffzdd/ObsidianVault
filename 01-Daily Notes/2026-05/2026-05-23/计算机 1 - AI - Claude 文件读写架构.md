# Resume context (Codex History Viewer)

- Source: `C:\Users\Administrator\.claude\projects\f--Projects-AIlearning-claude-code\3b0fd719-b306-4c37-a312-c1da2410c509.jsonl`
- History Source: `claude`
- Start: `2026-05-22 17:29`
- CWD: `f:\Projects\AIlearning\claude-code`
- CLI: `2.1.145`
- Source type: `claude-vscode`

> IMPORTANT: This excerpt is copied from a past session. Read it and continue the work.

---

## Task

说说这个项目

## Recent messages

### Assistant
- Timestamp: `2026-05-22 17:29:20`

让我先看看这个项目的结构和内容。

### Assistant
- Timestamp: `2026-05-22 17:29:44`

这个项目是 **Claude Code 泄露源代码存档**，以下是核心要点：

---

## 项目概述

**背景：** 2026年3月31日，研究者 Chaofan Shou 发现 Anthropic 发布到 npm 的 Claude Code 包里附带了 `.map` 文件，该文件指向存储在 Anthropic R2 bucket 上的完整未混淆 TypeScript 源代码，由此泄露。

这个仓库就是对那份泄露源码的存档和整理。

---

## 技术栈

| 层次 | 技术 |
|------|------|
| 运行时 | Bun |
| 语言 | TypeScript (strict 模式) |
| 终端 UI | React + Ink |
| CLI 解析 | Commander.js |
| Schema 校验 | Zod v4 |
| 代码搜索 | ripgrep |
| API | Anthropic SDK |
| 协议 | MCP · LSP |
| 认证 | OAuth 2.0 · JWT |

---

## 核心架构

项目规模约 **1,900 个文件，51.2万+ 行代码**，主要子系统包括：

1. **工具系统** ([src/tools/](src/tools/)) — ~40 个工具，覆盖文件读写、shell 执行、搜索、Agent 派发等
2. **命令系统** ([src/commands/](src/commands/)) — ~85 个斜杠命令（`/review`、`/commit`、`/mcp` 等）
3. **查询引擎** ([src/QueryEngine.ts](src/QueryEngine.ts)) — ~46,000 行，核心 LLM 调用逻辑，含流式输出、工具循环、思考模式
4. **Bridge 系统** ([src/bridge/](src/bridge/)) — VS Code / JetBrains IDE 扩展的双向通信层
5. **权限系统** ([src/hooks/toolPermission/](src/hooks/toolPermission/)) — 每次工具调用前的权限检查
6. **MCP 服务** ([mcp-server/](mcp-server/)) — 随仓库附带的 MCP 探索服务器，可接入 Claude Code / Cursor 等客户端

---

## 仓库目的

- `src/` 是原始泄露代码，README 明确说明不应修改
- 附带了 `docs/` 文档目录、`prompts/` 提示词、`mcp-server/` 用于探索源码
- 官方声明：所有源代码版权归 Anthropic 所有，这是非官方存档，不提供再发行许可

你对哪个子系统比较感兴趣？

### User
- Timestamp: `2026-05-22 17:30:24`

我正在学习agent开发,我对他的框架感兴趣

### Assistant
- Timestamp: `2026-05-22 17:30:29`

让我深入探索这个项目的 agent 框架相关代码。

### Assistant
- Timestamp: `2026-05-22 17:32:42`

这个框架的设计非常精妙，以下是核心架构解读：

---

## Claude Code 的 Agent 框架设计

### 核心理念：一切 Agent 都运行相同的查询循环

最关键的设计决定是：**子 Agent 不是特殊的 RPC 调用**，它们在进程内运行与主线程完全相同的 `query()` 函数。

```
主线程                     子 Agent
  └── query()  ←────────────── runAgent() 内部也调用 query()
```

---

### 1. 三层核心抽象

```
Tool.ts        →  工具基础类型（每个工具是独立的微服务）
QueryEngine.ts →  管理单个对话的完整生命周期
query.ts       →  主执行循环（工具调用、MCP、压缩、钩子）
```

每个 `Tool` 必须实现：
- `call()` — 执行逻辑
- `checkPermissions()` — 权限检查
- `isConcurrencySafe()` — 是否可以并发
- `isReadOnly()` — 是否只读

---

### 2. 工具执行的并发模型

[src/services/tools/toolOrchestration.ts](src/services/tools/toolOrchestration.ts) 实现了一个聪明的分批策略：

```
只读工具可以并发：  [Read, Glob, Grep] → 并行执行
写入工具必须串行：  [Edit] → 单独执行
混合场景分批：      [Read, Read] | [Write] | [Read] | [Delete]
```

---

### 3. Agent 的四种协作模式

| 模式 | 触发方式 | 典型用途 |
|------|---------|---------|
| **后台 Worker** | `AgentTool({ run_in_background: true })` | 独立研究、并行任务 |
| **Fork Agent** | `AgentTool({ prompt })` 不指定类型 | 并行分支，共享提示缓存 |
| **Coordinator** | `CLAUDE_CODE_COORDINATOR_MODE=1` | 主线程作编排者管理多 Worker |
| **Team 模式** | `AgentTool({ name, team_name })` | 交互式多 Agent 团队 |

---

### 4. Agent 生命周期

```
AgentTool.call()
  ↓
权限过滤 + 工具解析
  ↓
createSubagentContext()    ← 克隆父上下文，可覆盖权限/工作目录
  ↓
query() 循环              ← 与主线程完全相同的函数！
  ↓
recordSidechainTranscript() → 消息写入磁盘 (.claude/transcripts/)
  ↓
enqueueAgentNotification()  → <task-notification> 注入主消息队列
  ↓
主线程下一回合看到结果，继续编排
```

---

### 5. Fork 机制（最精妙的设计）

[src/tools/AgentTool/forkSubagent.ts](src/tools/AgentTool/forkSubagent.ts) 的 Fork 机制：

- Fork Agent 继承父对话的**完整上下文**
- 所有 Fork 子进程产生**字节相同的 API 前缀** → 命中提示缓存
- 多个 Fork 并行时，缓存费用只付一次

---

### 6. 对学习 Agent 开发的启示

这个框架体现了几个重要的设计哲学：

1. **统一接口** — Agent 和主线程用同一套循环，没有特殊化
2. **权限即代码** — 每个工具自己管理权限，而不是集中控制
3. **生成器驱动** — 所有长时间操作都是 `async generator`，调用者消费消息流
4. **磁盘持久化** — Agent 消息写磁盘，支持引导和重放
5. **缓存对齐** — 设计 Fork 时特意保证前缀相同来省钱

---

你最感兴趣哪个部分？我可以深入讲解：
- **工具权限系统**的实现细节
- **多 Agent 编排**的消息流转
- **QueryEngine** 的流式输出机制
- 或者如何**仿照这个框架**设计你自己的 agent 系统

### User
- Timestamp: `2026-05-22 17:53:37`

文件读写工具呢?感觉单单给文件读写又不够,模型需要知道一些信息才知道怎么去读写哪个文件

### Assistant
- Timestamp: `2026-05-22 17:53:44`

很好的观察！让我去看看文件工具的实现细节。

### Assistant
- Timestamp: `2026-05-22 17:55:30`

你的直觉完全正确！这正是框架设计最有意思的地方。

---

## 文件工具的完整体系：先发现，再读写

单独的读写工具不够用，Claude Code 设计了一套**四阶段工具链**：

```
发现文件        →    精确定位    →    读取    →    修改
GlobTool           GrepTool       FileReadTool    FileEditTool
（按文件名找）      （按内容找）    （加载内容）    FileWriteTool
```

---

### 第一阶段：发现文件

**GlobTool** — 按文件名模式找文件

```
输入：pattern="**/*.ts", path="src/"
输出：["src/Tool.ts", "src/query.ts", ...]  按修改时间排序，最多100个
```

**GrepTool** — 按内容找文件（底层用 ripgrep）

```
输入：pattern="checkPermissions", glob="*.ts"
输出：
  files_with_matches 模式 → 只返回文件路径列表（省 token）
  content 模式           → 返回匹配行 + 上下文
  count 模式             → 每个文件的匹配数
```

两个工具配合：**Glob 找结构，Grep 找语义**。

---

### 第二阶段：读取（必须先读！）

**FileReadTool** 的关键设计：读取后会写入 `readFileState` 缓存。

```
readFileState: Map<filePath, {
  content,     // 文件内容
  timestamp,   // 读取时的 mtime
  offset?,     // 分页读取的起始行
  limit?,      // 读取了多少行
}>
```

大文件保护：
- 默认最多读 2000 行（超出提示用 offset/limit 分页）
- 最大 256KB / 25,000 token，超出报错

---

### 第三阶段：写入（强制依赖读取）

**FileWriteTool / FileEditTool 的 validateInput 会检查：**

```typescript
// 没读过 → 拒绝写入
if (!readFileState.has(filePath)) {
  throw new Error("Must read file before writing")
}

// 读后文件被改过 → 拒绝写入（防止覆盖别人的修改）
if (file.mtime > readFileState.get(filePath).timestamp) {
  throw new Error("File modified since last read")
}
```

这是个**强制读-写顺序**的架构约束，不是靠提示词引导，是代码层面强制的。

---

### FileEditTool 的 old_string/new_string 机制

这是最精妙的设计——**不用行号，用精确字符串定位**：

```
old_string: "function foo() {"
new_string: "function bar() {"
```

**为什么不用行号？**
因为模型读文件和实际执行之间，文件内容可能已经变了，行号会漂移。字符串匹配更稳定。

约束：
- `old_string` 在文件中必须唯一（否则报错，告诉你有几处匹配）
- `replace_all: true` 可以替换所有出现
- 自动处理引号变体（弯引号 vs 直引号）

---

### 整体设计哲学

你最初的问题——"模型怎么知道读哪个文件"——答案是：

| 问题 | 解决方案 |
|------|---------|
| 找哪个文件 | Glob + Grep 先探索 |
| 文件太大读不完 | offset/limit 分页，默认 2000 行 |
| 防止误写 | 代码强制 Read → Write 顺序 |
| 防止覆盖冲突 | mtime 时间戳校验 |
| 精确定位修改位置 | old_string 字符串匹配，而不是行号 |

你在自己做 agent 时，也可以借鉴这个模式：**文件系统工具不是孤立的，要设计成一组有状态依赖关系的工具链，并在代码层面强制执行操作顺序。**

想深入看哪部分的具体实现？比如 GrepTool 的分页机制，或者 FileEditTool 的冲突检测逻辑。

### User
- Timestamp: `2026-05-22 17:58:26`

那提示词里面怎么写的,让agent以正确实现读写

### Assistant
- Timestamp: `2026-05-22 17:58:30`

让我直接读这些工具的 prompt 文件。

### Assistant
- Timestamp: `2026-05-22 17:58:39`

现在让我看看系统提示里有没有更全局的引导。

### Assistant
- Timestamp: `2026-05-22 17:59:21`

现在有了完整的图景，来解读这个精妙的提示词设计。

---

## 三层提示词协同工作

### 第一层：系统提示 — 全局行为规则

[src/constants/prompts.ts:231](src/constants/prompts.ts#L231) 的 `getSimpleDoingTasksSection()` 里有一条关键规则：

```
In general, do not propose changes to code you haven't read.
If a user asks about or wants you to modify a file, read it first.
Understand existing code before suggesting modifications.
```

这是最顶层的行为约束——**先理解，再修改**，不用任何工具名，是纯粹的行为准则。

---

### 第二层：工具选择规则

[src/constants/prompts.ts:291](src/constants/prompts.ts#L291) 的 `getUsingYourToolsSection()` 明确告诉模型用哪个工具做哪件事：

```
To read files use Read instead of cat, head, tail, or sed
To edit files use Edit instead of sed or awk
To create files use Write instead of cat with heredoc or echo redirection
To search for files use Glob instead of find or ls
To search the content of files, use Grep instead of grep or rg
```

这解决了你说的"模型需要知道用什么工具"——**直接在系统提示里建立工具→任务的映射**。

---

### 第三层：每个工具自己的 prompt — 跨工具引用

这是最精妙的部分。各工具的 prompt 文件之间**通过代码导入相互引用**，保证名字永远一致：

**FileEditTool/prompt.ts**（[第5行](src/tools/FileEditTool/prompt.ts#L5)）：
```typescript
import { FILE_READ_TOOL_NAME } from '../FileReadTool/prompt.js'

// 生成的工具描述里写道：
"You must use your `Read` tool at least once in the conversation
before editing. This tool will error if you attempt an edit
without reading the file."
```

**FileWriteTool/prompt.ts**（[第7行](src/tools/FileWriteTool/prompt.ts#L7)）：
```typescript
import { FILE_READ_TOOL_NAME } from '../FileReadTool/prompt.js'

// 生成的工具描述里写道：
"If this is an existing file, you MUST use the Read tool first
to read the file's contents. This tool will fail if you did not
read the file first."
```

**GrepTool/prompt.ts**（[第10行](src/tools/GrepTool/prompt.ts#L10)）：
```typescript
// 告诉模型不要绕过专用工具：
"ALWAYS use Grep for search tasks.
NEVER invoke `grep` or `rg` as a Bash command."
```

---

### 三层的分工

```
系统提示（全局）        →  "先读再改" 的行为原则
  ↓
工具选择规则（全局）    →  "用 Read/Glob/Grep，别用 cat/find/grep"
  ↓
每个工具自己的描述      →  "必须先调用 Read，否则报错"
  ↓
代码层强制执行          →  validateInput 检查 readFileState，真的会报错
```

---

### 对你设计 agent 的启发

这个设计的核心是**多层冗余**：

| 层次 | 作用 | 解决什么问题 |
|------|------|-------------|
| 系统提示行为原则 | 引导思维方式 | 模型在选工具前就有了正确预期 |
| 工具选择规则 | 建立工具→任务映射 | 模型知道"搜文件用 Glob，不用 find" |
| 工具自身描述 | 说明前置依赖 | 模型在调用时看到"必须先 Read" |
| 代码强制校验 | 兜底保障 | 即使提示词失效，代码也会拒绝 |

**不要只靠提示词，也不要只靠代码校验**——两者都要，提示词引导模型做对，代码校验保证做错了有明确错误信息帮助模型纠正。

### User
- Timestamp: `2026-05-22 18:02:27`

说说整个框架的提示词架构

### Assistant
- Timestamp: `2026-05-22 18:02:34`

让我把提示词架构的关键文件都读完。

### Assistant
- Timestamp: `2026-05-22 18:04:39`

现在我有完整的图景了，来梳理整个提示词架构。

---

## Claude Code 的提示词完整架构

整体分为**四个注入点**，每个注入点有不同的生命周期和缓存策略：

```
┌─────────────────────────────────────────────────┐
│                  API 调用结构                    │
│                                                  │
│  system: [ 静态区 | BOUNDARY | 动态区 ]          │
│  messages: [ <system-reminder> | ... 对话 ... ]  │
│  tools: [ 每个工具的 description ]               │
└─────────────────────────────────────────────────┘
```

---

### 第一层：System Prompt — 行为蓝图

由 [src/constants/prompts.ts:444](src/constants/prompts.ts#L444) 的 `getSystemPrompt()` 组装，以 `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 为界分两段：

**静态区（跨 org 缓存，永不变）：**

```
1. Intro          "You are an interactive agent..."  + 安全指令
2. System         工具权限说明、hooks 说明、自动压缩提示
3. Doing tasks    行为准则（先读再改、不过度实现、安全编码）
4. Actions        可逆性与破坏性操作确认规则
5. Using tools    工具→任务映射（用 Read 代替 cat，用 Grep 代替 grep）
6. Tone & style   简洁、不用 emoji、用行号引用代码
7. Output style   简洁直接，不堆砌解释
```

**动态区（每段独立缓存，按需刷新）：**

```
8.  Session guidance   AgentTool 使用建议、Skill 调用方式
9.  Memory             auto memory 系统说明 + MEMORY.md 内容
10. Env info           CWD、Git 状态、平台、模型 ID
11. Language           用户语言偏好
12. MCP instructions   已连接 MCP server 的使用说明
13. Scratchpad         临时目录路径说明
```

动态区用 `systemPromptSection()` 缓存，`/clear` 或 `/compact` 时清空。极少数需要每轮刷新的用 `DANGEROUS_uncachedSystemPromptSection()`（会破坏 prompt cache，必须写明原因）。

---

### 第二层：System Context — 会话级快照

追加到 system prompt 末尾，由 [src/context.ts:116](src/context.ts#L116) 的 `getSystemContext()` 提供：

```
gitStatus: "Current branch: main
            Main branch: main
            Git user: xxx
            Status: M src/foo.ts
            Recent commits: abc123 fix: ..."
```

整个对话只生成一次（memoize），是快照而非实时数据，系统提示里也明说了：

> "Note that this status is a snapshot in time, and will not update during the conversation."

---

### 第三层：User Context — 每轮首条消息注入

由 [src/context.ts:155](src/context.ts#L155) 的 `getUserContext()` 提供，通过 [src/utils/api.ts:449](src/utils/api.ts#L449) 的 `prependUserContext()` 注入为**第一条 user 消息**，用 `<system-reminder>` 标签包裹：

```xml
<system-reminder>
As you answer the user's questions, you can use the following context:
# claudeMd
[CLAUDE.md 全部内容]

# currentDate
Today's date is 2026-05-22.

IMPORTANT: this context may or may not be relevant to your tasks.
You should not respond to this context unless it is highly relevant.
</system-reminder>
```

**为什么不放进 system prompt？** 因为 CLAUDE.md 是用户内容，每个项目不同，放进 user message 可以复用 system prompt 的 prompt cache。

**CLAUDE.md 的加载层级**（[src/utils/claudemd.ts:1](src/utils/claudemd.ts#L1)，加载顺序即优先级顺序，越晚加载越高）：

```
1. /etc/claude-code/CLAUDE.md        ← 企业管控级（最低优先级）
2. ~/.claude/CLAUDE.md               ← 用户全局
3. 项目根/CLAUDE.md
   项目根/.claude/CLAUDE.md
   项目根/.claude/rules/*.md         ← 项目级
4. 项目根/CLAUDE.local.md            ← 本地私有（最高优先级）
```

支持 `@include` 指令跨文件引用，支持从当前目录向上遍历到 git root。

---

### 第四层：Tool Descriptions — 工具级提示词

每个工具在 `tools` 数组里的 `description` 字段，是模型决定**调用哪个工具**时看到的。

关键设计：**工具间通过代码导入相互引用**，保证名字一致：

```typescript
// FileEditTool/prompt.ts
import { FILE_READ_TOOL_NAME } from '../FileReadTool/prompt.js'
// → 生成: "You must use your `Read` tool at least once..."

// GrepTool/prompt.ts
import { BASH_TOOL_NAME } from '../BashTool/toolName.js'
// → 生成: "ALWAYS use Grep. NEVER invoke `grep` as a Bash command."
```

---

### 完整数据流

```
每次 API 调用的实际内容：

system: [
  "You are an interactive agent..."     ← 静态，全局 cache
  "# Doing tasks ..."                   ← 静态，全局 cache
  "# Using your tools ..."              ← 静态，全局 cache
  ──────── BOUNDARY ────────
  "# Session-specific guidance ..."     ← 动态，org cache
  "# auto memory ..."                   ← 动态，org cache
  "# Environment ..."                   ← 动态，org cache
  "gitStatus: Current branch: main..."  ← system context，org cache
]

messages: [
  { role: "user",                       ← prependUserContext 注入
    content: "<system-reminder>
               # claudeMd [CLAUDE.md内容]
               # currentDate 2026-05-22
             </system-reminder>" }
  { role: "user", content: "用户消息" }
  { role: "assistant", ... }
  ...
]

tools: [
  { name: "Read",   description: "Reads a file..." }
  { name: "Edit",   description: "...must use Read first..." }
  { name: "Grep",   description: "ALWAYS use Grep..." }
  ...
]
```

---

### 对 Agent 开发的核心启示

| 设计决策 | 原因 |
|---------|------|
| 静态内容放 system，动态内容分段缓存 | 最大化 prompt cache 命中率，省钱降延迟 |
| CLAUDE.md 放 user message 而非 system | system 可跨用户复用缓存，用户内容不能混入 |
| 工具名用常量跨文件引用 | 工具改名时自动同步所有引用，不会出现提示词里写着旧名字 |
| system-reminder 标签明说"可能不相关" | 避免模型每次都对注入的上下文做无意义响应 |
| git status 只取快照并告知模型 | 防止模型误以为状态实时更新，避免幻觉 |

### Assistant
- Timestamp: `2026-05-22 18:26:35`

No response requested.

### User
- Timestamp: `2026-05-23 00:04:44`

我大概懂Agent开发的原理,也动手做过粗略的ReAct,但不知道搭建起这种要多久,自己的动手能力不够

### Assistant
- Timestamp: `2026-05-23 00:05:15`

这是个很诚实的自我评估。让我给你一个实际的拆解。

---

## Claude Code 的复杂度从哪来

你看到的 1900 文件、51万行，大部分不是"agent 框架"，而是**产品功能的积累**：

```
真正的 agent 框架核心：     ~5%
IDE 集成 (bridge/)：        ~15%
终端 UI (Ink/React)：       ~20%
各种工具实现 (~40个)：      ~30%
MCP、权限、插件、语音…：    ~30%
```

你已经做过 ReAct，说明你理解了最核心的东西。

---

## 从 ReAct 到"像 Claude Code 一样"的距离

| 阶段 | 要做什么 | 大概时间 |
|------|---------|---------|
| **你现在的位置** | 单工具 ReAct 循环 | ✅ 已完成 |
| **下一步** | 工具抽象（统一接口 + 描述 + schema） | 1-2 天 |
| **再下一步** | 工具并发（只读并行、写入串行） | 2-3 天 |
| **提示词架构** | 静态/动态分层、上下文注入 | 1 周迭代 |
| **多 agent** | 子 agent 派发、任务通知 | 2-3 周 |
| **生产级** | 权限、错误恢复、缓存、持久化 | 数月 |

你不需要走到"生产级"才有用的系统。**第二、三步就能做出很有价值的东西**。

---

## 具体建议

你现在最缺的可能不是理论，是**把脑子里的理解转化成代码结构**的经验。

最有效的练习路径：

1. **给你的 ReAct 加一个统一的 Tool 接口**，哪怕只有 `name`、`description`、`input_schema`、`execute()` 四个字段，然后实现 3-5 个工具

2. **加一个 `readFileState` 类似的状态对象**，让工具之间能共享状态（比如"读过哪些文件"）

3. **拆分提示词**：把系统提示分成"不变的行为规则"和"每轮注入的上下文"，感受一下缓存效果

这三步做完，你对 Claude Code 框架的理解就不只是看懂了，而是真的内化了。

---

你现在的 ReAct 是用什么语言/框架写的？可以基于你已有的代码一起看怎么演进。

### User
- Timestamp: `2026-05-23 00:09:39`

做到你说的第二三步.找下一段实习好找吗,我喜欢动手,不喜欢背八股和刷题那些,刷完又忘了

### Assistant
- Timestamp: `2026-05-23 00:10:12`

实话说。

---

## 现实情况

**传统大厂（字节、阿里、腾讯等）**：八股 + LeetCode 几乎是硬门槛，绕不开。他们面的人太多，需要标准化筛选。

**AI 创业公司 / 大厂的 AI 新团队**：情况完全不同，**项目能力优先**。你能说清楚"我做了一个 agent 框架，工具并发这样设计、提示词分层是这个思路"，比背出线程池原理更有说服力。

现在 agent 方向的需求是真实的，而且**能动手的人稀缺**——很多人看论文、用 LangChain 套壳，但真正理解底层机制、能自己搭的人不多。

---

## 你的优势怎么放大

关键是**让别人看见你做了什么**。

不需要多复杂，一个 GitHub 项目 + 一篇说清楚设计决策的文章，就能让面试官直接跳过"讲讲你做过什么"的环节，进入实质讨论。

你现在读 Claude Code 源码这件事本身就值得写出来——"我读了 Claude Code 的泄露源码，发现它的工具执行是这样设计并发的，我在自己的实现里借鉴了这个思路"——这种叙事很有说服力。

---

## 一个务实的路径

```
现在          做到步骤 2-3，推上 GitHub
              ↓
同时          写一篇文章，说清楚你的设计思路
              ↓
找实习时      投 AI 方向的公司/团队，简历直接附项目链接
              跳过或晚些再补八股
```

你喜欢动手这件事本身是加分项，问题只是**有没有东西证明这一点**。

你现在的 ReAct 代码放 GitHub 了吗？
