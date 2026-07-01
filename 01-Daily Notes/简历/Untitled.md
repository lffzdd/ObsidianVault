**Ready for review**

Select text to add comments on the plan

# 为 ReActMulti 实现长期记忆系统（对标 Claude Code memdir）

## Context

当前 ReActMulti agent 是一个无记忆的 REPL：`Agent.run()` 把一个 user turn 跑到 `final_answer`，会话历史只活在内存 `SessionState` 里，进程退出即丢失。用户希望移植 Claude Code 的 `memdir` 长期记忆能力，让跨会话的「用户画像 / 工作偏好 / 项目背景 / 外部资源指针」能持久化、并在未来对话里自动被召回。

已确认的范围决策：

- **全自动**：自动召回（每个 user turn 用小查询选相关记忆并注入）+ 会话结束自动提取落盘
    - 显式 `save_memory` / `search_memory` 工具三者都做。
- **存储在仓库内**：默认 `src/AIAgent/ReActMulti/memory_store/`（可用环境变量覆盖）。
- **仅主 Agent 有记忆**：子 Agent 保持现有的纯净隔离上下文，不读写记忆、不带记忆工具。

设计严格沿用现有协作者（collaborator）模式：记忆逻辑独立成 `MemoryManager`，`Agent` 只在主循环里「喊它一声」，与 `ContextCompactor` / `ToolExecutor` 的接法一致。

记忆只存「无法从当前项目状态推导」的四类信息（user / feedback / project / reference）， 代码结构、git 历史、修 bug 配方等一律不存——这是 memdir 的核心约束，原样移植。

## 新增模块：`src/AIAgent/ReActMulti/memory/` 包

### `paths.py`

- `memory_dir() -> Path`：默认包内 `memory_store/`，环境变量 `REACT_MEMORY_DIR` 可覆盖。
- `ensure_memory_dir()`：幂等创建（`mkdir(parents=True, exist_ok=True)`）。
- `MEMORY_INDEX = "MEMORY.md"`，`entrypoint_path()`。
- 在仓库 `.gitignore`（[src/AIAgent/ReActMulti/.gitignore](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/.gitignore)）按需追加 `memory_store/` 忽略规则（除非用户想提交记忆——默认忽略，计划里留一行说明）。

### `types.py`（从 memoryTypes.ts 移植 + 浓缩翻译）

- `MEMORY_TYPES = ("user", "feedback", "project", "reference")`，`parse_memory_type()`。
- 静态提示词文本块（中文/精炼）：`TYPES_SECTION`、`WHAT_NOT_TO_SAVE`、 `WHEN_TO_ACCESS`（含「记忆会过期，落地前先核对当前文件」漂移告诫）、 `TRUSTING_RECALL`（「记忆提到 X 存在 ≠ X 现在存在」，引用文件/函数前先核实）、 `FRONTMATTER_EXAMPLE`（`name` / `description` / `type` 三字段）。

### `store.py`（落盘 + 索引，纯文件操作，可单测）

- `parse_frontmatter(text) / dump_frontmatter(...)`：解析/生成 `---` YAML 头（只需 name/description/type，用极简手写解析，避免引 PyYAML）。
- `MemoryHeader` dataclass：`filename / path / mtime / description / type`。
- `scan_memory_files(dir) -> list[MemoryHeader]`：扫 `*.md`、跳过 `MEMORY.md`、读前 ~30 行 取 frontmatter、按 mtime 倒序（对标 memoryScan.ts）。
- `format_manifest(headers) -> str`：`- [type] file.md (mtime): description` 清单。
- `write_memory_file(name, description, type, content)`：sanitize name → `<slug>.md`， 写 frontmatter + 正文；同名即覆盖（兼作 update）。
- `rebuild_index()`：扫全部记忆文件重新生成 `MEMORY.md`（`- [name](file.md) — description` 一行一条）。每次写记忆后调用，自愈、不漂。
- `read_entrypoint() -> str`：读 `MEMORY.md`，套行数/字节上限截断（移植 `truncateEntrypointContent`，200 行 / 25KB）。
- `read_memories_for_surfacing(paths) -> str`：读选中记忆全文（带截断），拼成注入块。

### `recall.py`（自动召回的「小模型选 → 主模型用」两段式）

- `find_relevant_memories(query, dir, llm, already_surfaced) -> list[Path]`： `scan_memory_files` → 生成 manifest → 调 `llm` 做一次 side-query 选最多 5 条（系统提示 对标 `SELECT_MEMORIES_SYSTEM_PROMPT`）→ 校验文件名合法 → 返回路径。
    - **复用现有 `LLMClient`**：构造 `[{system},{user}]` 消息，`for ev in llm(msgs)` 抽 `ContentDone.content`，`json.loads`。`LLMClient` 默认 `response_format=json_object` 已保证返回合法 JSON。整段 try/except 失败返回 `[]`（召回失败绝不阻塞主流程）。
    - 可选：`MemoryManager` 接受 `selector_llm`，main 可注入更便宜的模型 （`OPENAI_MEMORY_MODEL`）省钱；默认复用主 `llm`。
- `build_recall_block(query, dir, llm) -> str`：拼 `MEMORY.md` 索引 + 选中记忆全文， 包进 `<system-reminder>…</system-reminder>`；无内容返回 `""`。

### `prompt.py`

- `build_memory_instructions(dir) -> str`：组合 `types.py` 各文本块 → 追加进 system prompt 的静态指令段（讲清四种类型、两步保存、何时存取、引用前核实）。**指令是静态的进 system prompt；MEMORY.md 内容和相关记忆走 per-turn 注入保证新鲜**（与 memdir 一致）。

### `extract.py`（会话结束自动提取，对标 extractMemories.ts）

- `extract_and_save(session_state, dir, llm)`：从 `session_state.message_records` 取 user/assistant 文本拼有界 transcript + 现有 manifest，side-query 让 LLM 输出 `{"memories":[{name, description, type, content, action: create|update|skip}]}`； 对非 skip 项 `write_memory_file` + `rebuild_index`。全程 best-effort try/except， 任何失败都不影响 `run()` 的返回。

### `manager.py` → `MemoryManager`（协作者，Agent 的注入点）

- 持有 `memory_dir` + `llm`（+ 可选 selector_llm）。
- `instructions() -> str`、`recall_block(query) -> str`、`extract(session_state)`。

## 工具：`tools/memory_tools.py`（仅挂给主 Agent）

- `save_memory_tool`：参数 `name / description / type / content` → `write_memory_file` + `rebuild_index`，返回写入路径。`check_permission` 默认 allow（agent 写自己的记忆，低风险）。
- `search_memory_tool`：参数 `query` → 返回 manifest（read-only，`is_concurrency_safe=True`）。
- 记忆目录由 `paths.memory_dir()` 解析，**不经过 `ToolRuntime`**（记忆在 workspace 之外， 也不受 file_tools 的 `_safe_path` workspace 限制约束）。沿用现有 `Tool` / `ToolResult` 契约。

## 集成改动（少量、定点）

### `prompt.py`（顶层）— [src/AIAgent/ReActMulti/prompt.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/prompt.py)

- `build_system_prompt(tools_json, memory_section="")`：在模板尾部追加 `memory_section`。

### `agent.py` — [src/AIAgent/ReActMulti/agent.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/agent.py)

- `Agent.__init__` 新增 `memory: MemoryManager | None = None`（子 Agent 传 None）。
- 构造 system prompt 时，若 `memory` 存在则把 `memory.instructions()` 作为 `memory_section` 传入 `build_system_prompt`。
- `run()`：`append_message(user prompt)` 之后，若 `memory` 存在，取 `block = memory.recall_block(prompt)`，非空则 `append_message({"role":"user", "content": block})`（system-reminder 包裹，role=user 兼容各 OpenAI 端点； `append_message` 会自动把它计入 `context_tokens`）。
- `turn.kind == "final"` 分支 `mark_completed()` 之后、`return` 之前：若 `memory` 存在调 `memory.extract(self.session_state)`（best-effort）。

### `main.py` — [src/AIAgent/ReActMulti/main.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/main.py)

- `ensure_memory_dir()`；构造 `memory_manager = MemoryManager(memory_dir(), llm_client)`。
- 工具装配保证子 Agent 不沾记忆： `tools = build_agent_tools(llm, base_tools + mcp_tools, …)` 后再 `tools = [*tools, save_memory_tool, search_memory_tool]`。 （记忆工具**不**进 `base_tools`，所以 `spawn_agent` 内部为子 Agent 重建工具集时拿不到。）
- 主 `Agent(..., memory=memory_manager)`。

## 关键复用点

- `LLMClient.__call__` 事件流 drain 取 `ContentDone.content` + 默认 json_object —— 直接做 selector / extractor 的 side-query，无需新 client（[llm.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/llm.py)）。
- `SessionState.append_message` 是 wire 唯一入口且自动维护 `context_tokens`，召回注入走它 （[session.py:107](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/session.py#L107)）。
- `Tool` / `ToolResult` / `ToolRuntime` 契约与 file_tools 写法照搬 （[tools/base.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/tools/base.py)、[tools/file_tools.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/tools/file_tools.py)）。
- `estimate_tokens`（[util.py:15](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/util.py#L15)）用于召回/索引块的截断预算。
- 协作者注入模式照搬 `ContextCompactor`（[context.py](vscode-webview://0lctl8qrt7cmo8r76bvgcljq0128hfafa32mm9rsvcd9t3fi16cb/src/AIAgent/ReActMulti/context.py)）。

## 已知取舍

- 召回块以 role=user 的 system-reminder 持久化进 wire，多轮会累积。v1 靠「top-3 + 截断」 控量；后续可让召回消息变 ephemeral（下一轮前移除）或纳入 compactor 折叠范围。
- selector 默认复用主模型；省钱可经 `OPENAI_MEMORY_MODEL` 注入便宜模型。

## 验证

1. **单测**（放 `tests/memory/`）：frontmatter parse↔dump round-trip；`scan_memory_files` 排序 + 跳过 MEMORY.md；`rebuild_index` 生成的索引行；`read_entrypoint` 截断。
2. **手动 REPL**（`python -m ...ReActMulti.main`）：
    - 说「记住我用 bun 不用 npm」→ 确认 `memory_store/` 下生成带 frontmatter 的 `.md` 且 `MEMORY.md` 出现索引行。
    - 新问一句相关问题 → 终端/日志可见召回注入，回答用上该偏好。
    - 重启进程再问 → 记忆仍在（验证持久化 + 自动召回）。
    - 跑一个普通任务到 final → 确认 `extract_and_save` 在结束时落盘了值得记的内容。
3. **隔离回归**：让主 Agent `spawn_agent` 一个子任务 → 确认子 Agent 工具集里没有 `save_memory` / `search_memory`，且子 Agent 系统提示不含记忆指令段。