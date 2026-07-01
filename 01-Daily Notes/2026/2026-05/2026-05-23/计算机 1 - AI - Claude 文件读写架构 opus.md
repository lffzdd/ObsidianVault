# Claude Code Session

- File: `<PATH>
- History Source: `claude`
- Location: `Active`
- Start: `2026-05-23 11:26`
- CWD: `<PATH>
- CLI: `2.1.145`
- Source: `claude-vscode`

---

## [#1] User
- Timestamp: `2026-05-23 11:26:08`

说说这个项目,我在学习AIAgent开发,对这个项目的架构很感兴趣

## [tool] Glob
- Call ID: `toolu_018rSC6UB83d8nUE32Hw4bjz`
- Timestamp: `2026-05-23 11:26:12`

### Arguments
```json
{
  "pattern": "*"
}
```

## [tool] Glob
- Call ID: `toolu_01Ax34tPFsHFTGScjcEMrpKE`
- Timestamp: `2026-05-23 11:26:12`

### Arguments
```json
{
  "pattern": "**/*.md"
}
```

## [tool output]
- Call ID: `toolu_018rSC6UB83d8nUE32Hw4bjz`
- Timestamp: `2026-05-23 11:26:12`

### Output
```
.git\description
.git\hooks\applypatch-msg.sample
.git\hooks\commit-msg.sample
.git\hooks\fsmonitor-watchman.sample
.git\hooks\post-update.sample
.git\hooks\pre-applypatch.sample
.git\hooks\pre-commit.sample
.git\hooks\pre-merge-commit.sample
.git\hooks\pre-push.sample
.git\hooks\pre-rebase.sample
.git\hooks\pre-receive.sample
.git\hooks\prepare-commit-msg.sample
.git\hooks\push-to-checkout.sample
.git\hooks\sendemail-validate.sample
.git\hooks\update.sample
.git\info\exclude
.git\objects\pack\pack-90900755a51dadbc30d790dd8bb36b336866d0a7.idx
.git\objects\pack\pack-90900755a51dadbc30d790dd8bb36b336866d0a7.rev
.git\packed-refs
.git\refs\remotes\origin\HEAD
.git\logs\refs\remotes\origin\HEAD
.git\HEAD
.git\refs\heads\main
.git\logs\HEAD
.git\logs\refs\heads\main
.DS_Store
.github\FUNDING.yml
CONTRIBUTING.md
Dockerfile
LICENSE
README.md
agent.md
Skill.md
biome.json
bun.lock
bunfig.toml
docker\.dockerignore
docker\docker-compose.yml
docker\Dockerfile
docker\entrypoint.sh
docs\architecture.md
docs\bridge.md
docs\commands.md
docs\exploration-guide.md
docs\subsystems.md
docs\tools.md
gitpretty-apply.sh
mcp-server\.gitignore
mcp-server\.npmignore
mcp-server\Dockerfile
mcp-server\README.md
mcp-server\api\index.ts
mcp-server\api\vercelApp.ts
mcp-server\package-lock.json
mcp-server\package.json
mcp-server\railway.json
mcp-server\server.json
mcp-server\src\http.ts
mcp-server\src\index.ts
mcp-server\src\index.ts.new
mcp-server\src\server.ts
mcp-server\tsconfig.json
package-lock.json
package.json
prompts\00-overview.md
prompts\01-install-bun-and-deps.md
prompts\02-runtime-shims.md
prompts\03-build-config.md
prompts\04-fix-mcp-server.md
prompts\05-env-and-auth.md
prompts\06-ink-react-terminal-ui.md
prompts\07-tool-system.md
prompts\08-command-system.md
prompts\09-query-engine.md
prompts\10-context-and-prompts.md
prompts\11-mcp-integration.md
prompts\12-services-layer.md
prompts\13-bridge-ide.md
prompts\14-dev-runner.md
prompts\15-production-bundle.md
prompts\16-testing.md
scripts\build-bundle.ts
scripts\build-web.ts
scripts\build.sh
scripts\bun-plugin-shims.ts
scripts\ci-build.sh
scripts\dev.ts
scripts\package-npm.ts
scripts\test-auth.ts
scripts\test-commands.ts
scripts\test-mcp.ts
scripts\test-services.ts
scripts\tsconfig.json
scripts\types.d.ts
server.json
src\QueryEngine.ts
src\Task.ts
src\Tool.ts
src\assistant\sessionHistory.ts
src\bootstrap\state.ts
(Results are truncated. Consider using a more specific path or pattern.)
```

### Output
```
CONTRIBUTING.md
README.md
agent.md
Skill.md
docs\architecture.md
docs\bridge.md
docs\commands.md
docs\exploration-guide.md
docs\subsystems.md
docs\tools.md
mcp-server\README.md
prompts\00-overview.md
prompts\01-install-bun-and-deps.md
prompts\02-runtime-shims.md
prompts\03-build-config.md
prompts\04-fix-mcp-server.md
prompts\05-env-and-auth.md
prompts\06-ink-react-terminal-ui.md
prompts\07-tool-system.md
prompts\08-command-system.md
prompts\09-query-engine.md
prompts\10-context-and-prompts.md
prompts\11-mcp-integration.md
prompts\12-services-layer.md
prompts\13-bridge-ide.md
prompts\14-dev-runner.md
prompts\15-production-bundle.md
prompts\16-testing.md
```

## [tool] Read
- Call ID: `toolu_01CEVKq4ZXYzQEhaD18gnYWF`
- Timestamp: `2026-05-23 11:26:18`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\README.md"
}
```

### Output
```
1	<div align="center">
2	
3	# Claude Code — Leaked Source
4	
5	**The full source code of Anthropic's Claude Code CLI, leaked on March 31, 2026**
6	
7	[![[01-Daily Notes/2026-05/2026-05-22/attachments/84d7b1fba8619e4319db98430d689814_MD5.svg]]](#tech-stack)
8	[![[01-Daily Notes/2026-05/2026-05-22/attachments/b0aa050e04f029031650501c26be73ec_MD5.svg]]](#tech-stack)
9	[![[01-Daily Notes/2026-05/2026-05-22/attachments/373db9265b412f7ce92cfbda1a6f8f9e_MD5.svg]]](#tech-stack)
10	[![[01-Daily Notes/2026-05/2026-05-22/attachments/e21f4df9c4f46a026b6b35243aa69cfc_MD5.svg]]](#directory-structure)
11	[![[01-Daily Notes/2026-05/2026-05-22/attachments/885185c9a6ebf707e7b6450a6a48e0d7_MD5.svg]]](#-explore-with-mcp-server)
12	[![[01-Daily Notes/2026-05/2026-05-22/attachments/125f6a6b21d76eb9b88714edc2055f17_MD5.svg]]](https://www.npmjs.com/package/warrioraashuu-codemaster)
13	[![[01-Daily Notes/2026-05/2026-05-22/attachments/86dc5416bf8c5a528141aebe25d43b09_MD5.svg]]](https://twitter.com/intent/follow?screen_name=warrioraashuu)
14	
15	> The original unmodified leaked source is preserved in the [`backup` branch](https://github.com/codeaashu/claude-code/tree/backup).
16	
17	</div>
18	
19	---
20	<!-- <div align="center">
21	<table>
22	<tr>
23	  
24	`Spend less time reading code, more time building.`
25	  
26	<td width="120" align="center">
27	<img src="https://repoxray.devdisplay.org/logo.png" width="80" height="80" alt="RepoXray Logo"/>
28	</td>
29	<td>
30	<h2 align="center">Understand Any Codebase<br>X-Ray Any Repo, in Seconds.<br>
31	<a href="https://repoxray.devdisplay.org/"><strong>repoxray.devdisplay.org</strong></a></h2>
32	</td>
33	</tr>
34	</table>
35	</div>
36	
37	--- -->
38	
39	## Table of Contents
40	
41	- [How It Leaked](#how-it-leaked)
42	- [What Is Claude Code?](#what-is-claude-code)
43	- [Documentation](#-documentation)
44	- [Explore with MCP Server](#-explore-with-mcp-server)
45	- [Directory Structure](#directory-structure)
46	- [Architecture](#architecture)
47	  - [Tool System](#1-tool-system)
48	  - [Command System](#2-command-system)
49	  - [Service Layer](#3-service-layer)
50	  - [Bridge System](#4-bridge-system)
51	  - [Permission System](#5-permission-system)
52	  - [Feature Flags](#6-feature-flags)
53	- [Key Files](#key-files)
54	- [Tech Stack](#tech-stack)
55	- [Design Patterns](#design-patterns)
56	- [GitPretty Setup](#gitpretty-setup)
57	- [Contributing](#contributing)
58	- [Disclaimer](#disclaimer)
59	
60	---
61	
62	## How It Leaked
63	
64	[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) discovered that the published npm package for Claude Code included a `.map` file referencing the full, unobfuscated TypeScript source — downloadable as a zip from Anthropic's R2 storage bucket.
65	
66	> **"Claude code source code has been leaked via a map file in their npm registry!"**
67	>
68	> — [@Fried_rice, March 31, 2026](https://x.com/Fried_rice/status/2038894956459290963)
69	
70	---
71	
72	## What Is Claude Code?
73	
74	Claude Code is Anthropic's official CLI tool for interacting with Claude directly from the terminal — editing files, running commands, searching codebases, managing git workflows, and more. This repository contains the leaked `src/` directory.
75	
76	| | |
77	|---|---|
78	| **Leaked** | 2026-03-31 |
79	| **Language** | TypeScript (strict) |
80	| **Runtime** | [Bun](https://bun.sh) |
81	| **Terminal UI** | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
82	| **Scale** | ~1,900 files · 512,000+ lines of code |
83	
84	---
85	
86	## � Documentation
87	
88	For in-depth guides, see the [`docs/`](docs/) directory:
89	
90	| Guide | Description |
91	|-------|-------------|
92	| **[Architecture](docs/architecture.md)** | Core pipeline, startup sequence, state management, rendering, data flow |
93	| **[Tools Reference](docs/tools.md)** | Complete catalog of all ~40 agent tools with categories and permission model |
94	| **[Commands Reference](docs/commands.md)** | All ~85 slash commands organized by category |
95	| **[Subsystems Guide](docs/subsystems.md)** | Deep dives into Bridge, MCP, Permissions, Plugins, Skills, Tasks, Memory, Voice |
96	| **[Exploration Guide](docs/exploration-guide.md)** | How to navigate the codebase — study paths, grep patterns, key files |
97	
98	Also see: [CONTRIBUTING.md](CONTRIBUTING.md) · [MCP Server README](mcp-server/README.md)
99	
100	---
101	
102	## �🔍 Explore with MCP Server
103	
104	This repo ships an [MCP server](https://modelcontextprotocol.io/) that lets any MCP-compatible client (Claude Code, Claude Desktop, VS Code Copilot, Cursor) explore the full source interactively.
105	
106	### Install from npm
107	
108	The MCP server is published as [`warrioraashuu-codemaster`](https://www.npmjs.com/package/warrioraashuu-codemaster) on npm — no need to clone the repo:
109	
110	```bash
111	# Claude Code
112	claude mcp add warrioraashuu-codemaster -- npx -y warrioraashuu-codemaster
113	```
114	
115	### One-liner setup (from source)
116	
117	```bash
118	git clone https://github.com/codeaashu/claude-code.git ~/claude-code \
119	  && cd ~/claude-code/mcp-server \
120	  && npm install && npm run build \
121	  && claude mcp add claude-code-explorer -- node ~/claude-code/mcp-server/dist/index.js
122	```
123	
124	<details>
125	<summary><strong>Step-by-step setup</strong></summary>
126	
127	```bash
128	# 1. Clone the repo
129	git clone https://github.com/codeaashu/claude-code.git
130	cd claude-code/mcp-server
131	
132	# 2. Install & build
133	npm install && npm run build
134	
135	# 3. Register with Claude Code
136	claude mcp add claude-code-explorer -- node <PATH>
137	```
138	
139	Replace `/absolute/path/to/claude-code` with your actual clone path.
140	
141	</details>
142	
143	<details>
144	<summary><strong>VS Code / Cursor / Claude Desktop config</strong></summary>
145	
146	**VS Code** — add to `.vscode/mcp.json`:
147	```json
148	{
149	  "servers": {
150	    "claude-code-explorer": {
151	      "type": "stdio",
152	      "command": "node",
153	      "args": ["${workspaceFolder}/mcp-server/dist/index.js"],
154	      "env": { "CLAUDE_CODE_SRC_ROOT": "${workspaceFolder}/src" }
155	    }
156	  }
157	}
158	```
159	
160	**Claude Desktop** — add to your config file:
161	```json
162	{
163	  "mcpServers": {
164	    "claude-code-explorer": {
165	      "command": "node",
166	      "args": ["/absolute/path/to/claude-code/mcp-server/dist/index.js"],
167	      "env": { "CLAUDE_CODE_SRC_ROOT": "/absolute/path/to/claude-code/src" }
168	    }
169	  }
170	}
171	```
172	
173	**Cursor** — add to `~/.cursor/mcp.json` (same format as Claude Desktop).
174	
175	</details>
176	
177	### Available tools & prompts
178	
179	| Tool | Description |
180	|------|-------------|
181	| `list_tools` | List all ~40 agent tools with source files |
182	| `list_commands` | List all ~50 slash commands with source files |
183	| `get_tool_source` | Read full source of any tool (e.g. BashTool, FileEditTool) |
184	| `get_command_source` | Read source of any slash command (e.g. review, mcp) |
185	| `read_source_file` | Read any file from `src/` by path |
186	| `search_source` | Grep across the entire source tree |
187	| `list_directory` | Browse `src/` directories |
188	| `get_architecture` | High-level architecture overview |
189	
190	| Prompt | Description |
191	|--------|-------------|
192	| `explain_tool` | Deep-dive into how a specific tool works |
193	| `explain_command` | Understand a slash command's implementation |
194	| `architecture_overview` | Guided tour of the full architecture |
195	| `how_does_it_work` | Explain any subsystem (permissions, MCP, bridge, etc.) |
196	| `compare_tools` | Side-by-side comparison of two tools |
197	
198	**Try asking:** *"How does the BashTool work?"* · *"Search for where permissions are checked"* · *"Show me the <PATH> command source"*
199	
200	### Custom source path / Remove
201	
202	```bash
203	# Custom source location
204	claude mcp add claude-code-explorer -e CLAUDE_CODE_SRC_ROOT=/path/to/src -- node <PATH>
205	
206	# Remove
207	claude mcp remove claude-code-explorer
208	```
209	
210	---
211	
212	## Directory Structure
213	
214	```
215	src/
216	├── main.tsx                 # Entrypoint — Commander.js CLI parser + React/Ink renderer
217	├── QueryEngine.ts           # Core LLM API caller (~46K lines)
218	├── Tool.ts                  # Tool type definitions (~29K lines)
219	├── commands.ts              # Command registry (~25K lines)
220	├── tools.ts                 # Tool registry
221	├── context.ts               # System/user context collection
222	├── cost-tracker.ts          # Token cost tracking
223	│
224	├── tools/                   # Agent tool implementations (~40)
225	├── commands/                # Slash command implementations (~50)
226	├── components/              # Ink UI components (~140)
227	├── services/                # External service integrations
228	├── hooks/                   # React hooks (incl. permission checks)
229	├── types/                   # TypeScript type definitions
230	├── utils/                   # Utility functions
231	├── screens/                 # Full-screen UIs (Doctor, REPL, Resume)
232	│
233	├── bridge/                  # IDE integration (VS Code, JetBrains)
234	├── coordinator/             # Multi-agent orchestration
235	├── plugins/                 # Plugin system
236	├── skills/                  # Skill system
237	├── server/                  # Server mode
238	├── remote/                  # Remote sessions
239	├── memdir/                  # Persistent memory directory
240	├── tasks/                   # Task management
241	├── state/                   # State management
242	│
243	├── voice/                   # Voice input
244	├── vim/                     # Vim mode
245	├── keybindings/             # Keybinding configuration
246	├── schemas/                 # Config schemas (Zod)
247	├── migrations/              # Config migrations
248	├── entrypoints/             # Initialization logic
249	├── query/                   # Query pipeline
250	├── ink/                     # Ink renderer wrapper
251	├── buddy/                   # Companion sprite (Easter egg 🐣)
252	├── native-ts/               # Native TypeScript utils
253	├── outputStyles/            # Output styling
254	└── upstreamproxy/           # Proxy configuration
255	```
256	
257	---
258	
259	## Architecture
260	
261	### 1. Tool System
262	
263	> `src/tools/` — Every tool Claude can invoke is a self-contained module with its own input schema, permission model, and execution logic.
264	
265	| Tool | Description |
266	|---|---|
267	| **File I/O** | |
268	| `FileReadTool` | Read files (images, PDFs, notebooks) |
269	| `FileWriteTool` | Create / overwrite files |
270	| `FileEditTool` | Partial modification (string replacement) |
271	| `NotebookEditTool` | Jupyter notebook editing |
272	| **Search** | |
273	| `GlobTool` | File pattern matching |
274	| `GrepTool` | ripgrep-based content search |
275	| `WebSearchTool` | Web search |
276	| `WebFetchTool` | Fetch URL content |
277	| **Execution** | |
278	| `BashTool` | Shell command execution |
279	| `SkillTool` | Skill execution |
280	| `MCPTool` | MCP server tool invocation |
281	| `LSPTool` | Language Server Protocol integration |
282	| **Agents & Teams** | |
283	| `AgentTool` | Sub-agent spawning |
284	| `SendMessageTool` | Inter-agent messaging |
285	| `TeamCreateTool` / `TeamDeleteTool` | Team management |
286	| `TaskCreateTool` / `TaskUpdateTool` | Task management |
287	| **Mode & State** | |
288	| `EnterPlanModeTool` / `ExitPlanModeTool` | Plan mode toggle |
289	| `EnterWorktreeTool` / `ExitWorktreeTool` | Git worktree isolation |
290	| `ToolSearchTool` | Deferred tool discovery |
291	| `SleepTool` | Proactive mode wait |
292	| `CronCreateTool` | Scheduled triggers |
293	| `RemoteTriggerTool` | Remote trigger |
294	| `SyntheticOutputTool` | Structured output generation |
295	
296	### 2. Command System
297	
298	> `src/commands/` — User-facing slash commands invoked with `/` in the REPL.
299	
300	| Command | Description | | Command | Description |
301	|---|---|---|---|---|
302	| `/commit` | Git commit | | `/memory` | Persistent memory |
303	| `/review` | Code review | | `/skills` | Skill management |
304	| `/compact` | Context compression | | `/tasks` | Task management |
305	| `/mcp` | MCP server management | | `/vim` | Vim mode toggle |
306	| `/config` | Settings | | `/diff` | View changes |
307	| `/doctor` | Environment diagnostics | | `/cost` | Check usage cost |
308	| `/login` / `/logout` | Auth | | `/theme` | Change theme |
309	| `/context` | Context visualization | | `/share` | Share session |
310	| `/pr_comments` | PR comments | | `/resume` | Restore session |
311	| `/desktop` | Desktop handoff | | `/mobile` | Mobile handoff |
312	
313	### 3. Service Layer
314	
315	> `src/services/` — External integrations and core infrastructure.
316	
317	| Service | Description |
318	|---|---|
319	| `api/` | Anthropic API client, file API, bootstrap |
320	| `mcp/` | Model Context Protocol connection & management |
321	| `oauth/` | OAuth 2.0 authentication |
322	| `lsp/` | Language Server Protocol manager |
323	| `analytics/` | GrowthBook feature flags & analytics |
324	| `plugins/` | Plugin loader |
325	| `compact/` | Conversation context compression |
326	| `extractMemories/` | Automatic memory extraction |
327	| `teamMemorySync/` | Team memory synchronization |
328	| `tokenEstimation.ts` | Token count estimation |
329	| `policyLimits/` | Organization policy limits |
330	| `remoteManagedSettings/` | Remote managed settings |
331	
332	### 4. Bridge System
333	
334	> `src/bridge/` — Bidirectional communication layer connecting IDE extensions (VS Code, JetBrains) with the CLI.
335	
336	Key files: `bridgeMain.ts` (main loop) · `bridgeMessaging.ts` (protocol) · `bridgePermissionCallbacks.ts` (permission callbacks) · `replBridge.ts` (REPL session) · `jwtUtils.ts` (JWT auth) · `sessionRunner.ts` (session execution)
337	
338	### 5. Permission System
339	
340	> `src/hooks/toolPermission/` — Checks permissions on every tool invocation.
341	
342	Prompts the user for approval/denial or auto-resolves based on the configured permission mode: `default`, `plan`, `bypassPermissions`, `auto`, etc.
343	
344	### 6. Feature Flags
345	
346	Dead code elimination at build time via Bun's `bun:bundle`:
347	
348	```typescript
349	import { feature } from 'bun:bundle'
350	
351	const voiceCommand = feature('VOICE_MODE')
352	  ? require('./commands/voice/index.js').default
353	  : null
354	```
355	
356	Notable flags: `PROACTIVE` · `KAIROS` · `BRIDGE_MODE` · `DAEMON` · `VOICE_MODE` · `AGENT_TRIGGERS` · `MONITOR_TOOL`
357	
358	---
359	
360	## Key Files
361	
362	| File | Lines | Purpose |
363	|------|------:|---------|
364	| `QueryEngine.ts` | ~46K | Core LLM API engine — streaming, tool loops, thinking mode, retries, token counting |
365	| `Tool.ts` | ~29K | Base types/interfaces for all tools — input schemas, permissions, progress state |
366	| `commands.ts` | ~25K | Command registration & execution with conditional per-environment imports |
367	| `main.tsx` | — | CLI parser + React/Ink renderer; parallelizes MDM, keychain, and GrowthBook on startup |
368	
369	---
370	
371	## Tech Stack
372	
373	| Category | Technology |
374	|---|---|
375	| Runtime | [Bun](https://bun.sh) |
376	| Language | TypeScript (strict) |
377	| Terminal UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
378	| CLI Parsing | [Commander.js](https://github.com/tj/commander.js) (extra-typings) |
379	| Schema Validation | [Zod v4](https://zod.dev) |
380	| Code Search | [ripgrep](https://github.com/BurntSushi/ripgrep) (via GrepTool) |
381	| Protocols | [MCP SDK](https://modelcontextprotocol.io) · LSP |
382	| API | [Anthropic SDK](https://docs.anthropic.com) |
383	| Telemetry | OpenTelemetry + gRPC |
384	| Feature Flags | GrowthBook |
385	| Auth | OAuth 2.0 · JWT · macOS Keychain |
386	
387	---
388	
389	## Design Patterns
390	
391	<details>
392	<summary><strong>Parallel Prefetch</strong> — Startup optimization</summary>
393	
394	MDM settings, keychain reads, and API preconnect fire in parallel as side-effects before heavy module evaluation:
395	
396	```typescript
397	// main.tsx
398	startMdmRawRead()
399	startKeychainPrefetch()
400	```
401	
402	</details>
403	
404	<details>
405	<summary><strong>Lazy Loading</strong> — Deferred heavy modules</summary>
406	
407	OpenTelemetry (~400KB) and gRPC (~700KB) are loaded via dynamic `import()` only when needed.
408	
409	</details>
410	
411	<details>
412	<summary><strong>Agent Swarms</strong> — Multi-agent orchestration</summary>
413	
414	Sub-agents spawn via `AgentTool`, with `coordinator/` handling orchestration. `TeamCreateTool` enables team-level parallel work.
415	
416	</details>
417	
418	<details>
419	<summary><strong>Skill System</strong> — Reusable workflows</summary>
420	
421	Defined in `skills/` and executed through `SkillTool`. Users can add custom skills.
422	
423	</details>
424	
425	<details>
426	<summary><strong>Plugin Architecture</strong> — Extensibility</summary>
427	
428	Built-in and third-party plugins loaded through the `plugins/` subsystem.
429	
430	</details>
431	
432	---
433	
434	## GitPretty Setup
435	
436	<details>
437	<summary>Show per-file emoji commit messages in GitHub's file UI</summary>
438	
439	```bash
440	# Apply emoji commits
441	bash ./gitpretty-apply.sh .
442	
443	# Optional: install hooks for future commits
444	bash ./gitpretty-apply.sh . --hooks
445	
446	# Push as usual
447	git push origin main
448	```
449	
450	</details>
451	
452	---
453	
454	## Contributing
455	
456	Contributions to documentation, the MCP server, and exploration tooling are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
457	
458	> **Note:** The `src/` directory is the original leaked source and should not be modified.
459	
460	---
461	
462	## Disclaimer
463	
464	This repository archives source code leaked from Anthropic's npm registry on **2026-03-31**. All original source code is the property of [Anthropic](https://www.anthropic.com). This is not an official release and is not licensed for redistribution. Contact [aashuu ✦](https://x.com/warrioraashuu) for any comments.
465	
466	---
467	
468	<a href="https://www.star-history.com/?repos=codeaashu%2Fclaude-code&type=date&legend=bottom-right">
469	 <picture>
470	   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/image?repos=codeaashu/claude-code&type=date&theme=dark&legend=bottom-right" <PATH>
471	   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/image?repos=codeaashu/claude-code&type=date&legend=bottom-right" <PATH>
472	   <img alt="Star History Chart" src="https://api.star-history.com/image?repos=codeaashu/claude-code&type=date&legend=bottom-right" <PATH>
473	 </picture>
474	</a>
475	
476	
477	
478	
```

## [tool] Read
- Call ID: `toolu_01P35mJ4kaBaQyK5A9zeGP33`
- Timestamp: `2026-05-23 11:26:18`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\docs\\architecture.md"
}
```

### Output
```
1	# Architecture
2	
3	> Deep-dive into how Claude Code is structured internally.
4	
5	---
6	
7	## High-Level Overview
8	
9	Claude Code is a terminal-native AI coding assistant built as a single-binary CLI. The architecture follows a pipeline model:
10	
11	```
12	User Input → CLI Parser → Query Engine → LLM API → Tool Execution Loop → Terminal UI
13	```
14	
15	The entire UI layer is built with **React + Ink** (React for the terminal), making it a fully reactive CLI application with components, hooks, state management, and all the patterns you'd expect in a React web app — just rendered to the terminal.
16	
17	---
18	
19	## Core Pipeline
20	
21	### 1. Entrypoint (`src/main.tsx`)
22	
23	The CLI parser is built with [Commander.js](https://github.com/tj/commander.js) (`@commander-js/extra-typings`). On startup, it:
24	
25	- Fires parallel prefetch side-effects (MDM settings, Keychain, API preconnect) before heavy module imports
26	- Parses CLI arguments and flags
27	- Initializes the React/Ink renderer
28	- Hands off to the REPL launcher (`src/replLauncher.tsx`)
29	
30	### 2. Initialization (`src/entrypoints/`)
31	
32	| File | Role |
33	|------|------|
34	| `cli.tsx` | CLI session orchestration — the main path from launch to REPL |
35	| `init.ts` | Config, telemetry, OAuth, MDM policy initialization |
36	| `mcp.ts` | MCP server mode entrypoint (Claude Code as an MCP server) |
37	| `sdk/` | Agent SDK — programmatic API for embedding Claude Code |
38	
39	Startup performs parallel initialization: MDM policy reads, Keychain prefetch, feature flag checks, then core init.
40	
41	### 3. Query Engine (`src/QueryEngine.ts`, ~46K lines)
42	
43	The heart of Claude Code. Handles:
44	
45	- **Streaming responses** from the Anthropic API
46	- **Tool-call loops** — when the LLM requests a tool, execute it and feed the result back
47	- **Thinking mode** — extended thinking with budget management
48	- **Retry logic** — automatic retries with backoff for transient failures
49	- **Token counting** — tracks input/output tokens and cost per turn
50	- **Context management** — manages conversation history and context windows
51	
52	### 4. Tool System (`src/Tool.ts` + `src/tools/`)
53	
54	Every capability Claude can invoke is a **tool**. Each tool is self-contained with:
55	
56	- **Input schema** (Zod validation)
57	- **Permission model** (what needs user approval)
58	- **Execution logic** (the actual implementation)
59	- **UI components** (how invocation/results render in the terminal)
60	
61	Tools are registered in `src/tools.ts` and discovered by the Query Engine during tool-call loops.
62	
63	See [Tools Reference](tools.md) for the complete catalog.
64	
65	### 5. Command System (`src/commands.ts` + `src/commands/`)
66	
67	User-facing slash commands (`/commit`, `/review`, `/mcp`, etc.) that can be typed in the REPL. Three types:
68	
69	| Type | Description | Example |
70	|------|-------------|---------|
71	| **PromptCommand** | Sends a formatted prompt to the LLM with injected tools | `/review`, `/commit` |
72	| **LocalCommand** | Runs in-process, returns plain text | `/cost`, `/version` |
73	| **LocalJSXCommand** | Runs in-process, returns React JSX | `/doctor`, `/install` |
74	
75	Commands are registered in `src/commands.ts` and invoked via `/command-name` in the REPL.
76	
77	See [Commands Reference](commands.md) for the complete catalog.
78	
79	---
80	
81	## State Management
82	
83	Claude Code uses a **React context + custom store** pattern:
84	
85	| Component | Location | Purpose |
86	|-----------|----------|---------|
87	| `AppState` | `src/state/AppStateStore.ts` | Global mutable state object |
88	| Context Providers | `src/context/` | React context for notifications, stats, FPS |
89	| Selectors | `src/state/` | Derived state functions |
90	| Change Observers | `src/state/onChangeAppState.ts` | Side-effects on state changes |
91	
92	The `AppState` object is passed into tool contexts, giving tools access to conversation history, settings, and runtime state.
93	
94	---
95	
96	## UI Layer
97	
98	### Components (`src/components/`, ~140 components)
99	
100	- Functional React components using Ink primitives (`Box`, `Text`, `useInput()`)
101	- Styled with [Chalk](https://github.com/chalk/chalk) for terminal colors
102	- React Compiler enabled for optimized re-renders
103	- Design system primitives in `src/components/design-system/`
104	
105	### Screens (`src/screens/`)
106	
107	Full-screen UI modes:
108	
109	| Screen | Purpose |
110	|--------|---------|
111	| `REPL.tsx` | Main interactive REPL (the default screen) |
112	| `Doctor.tsx` | Environment diagnostics (`/doctor`) |
113	| `ResumeConversation.tsx` | Session restore (`/resume`) |
114	
115	### Hooks (`src/hooks/`, ~80 hooks)
116	
117	Standard React hooks pattern. Notable categories:
118	
119	- **Permission hooks** — `useCanUseTool`, `src/hooks/toolPermission/`
120	- **IDE integration** — `useIDEIntegration`, `useIdeConnectionStatus`, `useDiffInIDE`
121	- **Input handling** — `useTextInput`, `useVimInput`, `usePasteHandler`, `useInputBuffer`
122	- **Session management** — `useSessionBackgrounding`, `useRemoteSession`, `useAssistantHistory`
123	- **Plugin/skill hooks** — `useManagePlugins`, `useSkillsChange`
124	- **Notification hooks** — `src/hooks/notifs/` (rate limits, deprecation warnings, etc.)
125	
126	---
127	
128	## Configuration & Schemas
129	
130	### Config Schemas (`src/schemas/`)
131	
132	Zod v4-based schemas for all configuration:
133	
134	- User settings
135	- Project-level settings
136	- Organization/enterprise policies
137	- Permission rules
138	
139	### Migrations (`src/migrations/`)
140	
141	Handles config format changes between versions — reads old configs and transforms them to the current schema.
142	
143	---
144	
145	## Build System
146	
147	### Bun Runtime
148	
149	Claude Code runs on [Bun](https://bun.sh) (not Node.js). Key implications:
150	
151	- Native JSX/TSX support without a transpilation step
152	- `bun:bundle` feature flags for dead-code elimination
153	- ES modules with `.js` extensions (Bun convention)
154	
155	### Feature Flags (Dead Code Elimination)
156	
157	```typescript
158	import { feature } from 'bun:bundle'
159	
160	// Code inside inactive feature flags is completely stripped at build time
161	if (feature('VOICE_MODE')) {
162	  const voiceCommand = require('./commands/voice/index.js').default
163	}
164	```
165	
166	Notable flags:
167	
168	| Flag | Feature |
169	|------|---------|
170	| `PROACTIVE` | Proactive agent mode (autonomous actions) |
171	| `KAIROS` | Kairos subsystem |
172	| `BRIDGE_MODE` | IDE bridge integration |
173	| `DAEMON` | Background daemon mode |
174	| `VOICE_MODE` | Voice input/output |
175	| `AGENT_TRIGGERS` | Triggered agent actions |
176	| `MONITOR_TOOL` | Monitoring tool |
177	| `COORDINATOR_MODE` | Multi-agent coordinator |
178	| `WORKFLOW_SCRIPTS` | Workflow automation scripts |
179	
180	### Lazy Loading
181	
182	Heavy modules are deferred via dynamic `import()` until first use:
183	
184	- OpenTelemetry (~400KB)
185	- gRPC (~700KB)
186	- Other optional dependencies
187	
188	---
189	
190	## Error Handling & Telemetry
191	
192	### Telemetry (`src/services/analytics/`)
193	
194	- [GrowthBook](https://www.growthbook.io/) for feature flags and A/B testing
195	- [OpenTelemetry](https://opentelemetry.io/) for distributed tracing and metrics
196	- Custom event tracking for usage analytics
197	
198	### Cost Tracking (`src/cost-tracker.ts`)
199	
200	Tracks token usage and estimated cost per conversation turn. Accessible via the `/cost` command.
201	
202	### Diagnostics (`/doctor` command)
203	
204	The `Doctor.tsx` screen runs environment checks: API connectivity, authentication, tool availability, MCP server status, and more.
205	
206	---
207	
208	## Concurrency Model
209	
210	Claude Code uses a **single-threaded event loop** (Bun/Node.js model) with:
211	
212	- Async/await for I/O operations
213	- React's concurrent rendering for UI updates
214	- Web Workers or child processes for CPU-intensive tasks (gRPC, etc.)
215	- Tool concurrency safety — each tool declares `isConcurrencySafe()` to indicate if it can run in parallel with other tools
216	
217	---
218	
219	## See Also
220	
221	- [Tools Reference](tools.md) — Complete catalog of all 40 agent tools
222	- [Commands Reference](commands.md) — Complete catalog of all slash commands
223	- [Subsystems Guide](subsystems.md) — Bridge, MCP, permissions, skills, plugins, and more
224	- [Exploration Guide](exploration-guide.md) — How to navigate this codebase
225	
```

## [tool] Read
- Call ID: `toolu_01WvyXVFXAqKctVuCMoGUxug`
- Timestamp: `2026-05-23 11:26:23`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\prompts\\00-overview.md"
}
```

## [tool] Glob
- Call ID: `toolu_01Ubtf38Rp3GXKwm6pLz5EkA`
- Timestamp: `2026-05-23 11:26:23`

### Arguments
```json
{
  "pattern": "src/*.ts"
}
```

## [tool output]
- Call ID: `toolu_01WvyXVFXAqKctVuCMoGUxug`
- Timestamp: `2026-05-23 11:26:23`

### Output
```
1	# Build-Out Prompt Index
2	
3	Run these prompts **in order** in separate chat sessions. Each one is self-contained.
4	
5	| # | File | What It Does | Depends On |
6	|---|------|-------------|------------|
7	| 01 | `01-install-bun-and-deps.md` | Install Bun runtime, install all dependencies | — |
8	| 02 | `02-runtime-shims.md` | Create `bun:bundle` runtime shim + `MACRO` globals so code runs without Bun's bundler | 01 |
9	| 03 | `03-build-config.md` | Create esbuild-based build system that bundles the CLI to a single runnable file | 01, 02 |
10	| 04 | `04-fix-mcp-server.md` | Fix TypeScript errors in `mcp-server/` and make it build | 01 |
11	| 05 | `05-env-and-auth.md` | Set up `.env` file, API key config, OAuth stubs | 01 |
12	| 06 | `06-ink-react-terminal-ui.md` | Verify and fix the Ink/React terminal rendering pipeline | 01, 02, 03 |
13	| 07 | `07-tool-system.md` | Audit and wire up the 40+ tool implementations (BashTool, FileEditTool, etc.) | 01–03 |
14	| 08 | `08-command-system.md` | Audit and wire up the 50+ slash commands (<PATH> <PATH> etc.) | 01–03, 07 |
15	| 09 | `09-query-engine.md` | Get the core LLM call loop (QueryEngine) functional — streaming, tool calls, retries | 01–03, 05, 07 |
16	| 10 | `10-context-and-prompts.md` | Wire up system prompt construction, context gathering, memory system | 01–03 |
17	| 11 | `11-mcp-integration.md` | Get MCP client/server integration working — registry, tool discovery | 01–04 |
18	| 12 | `12-services-layer.md` | Wire up analytics, policy limits, remote settings, session memory | 01–03, 05 |
19	| 13 | `13-bridge-ide.md` | Stub out or implement the VS Code / JetBrains bridge layer | 01–03, 09 |
20	| 14 | `14-dev-runner.md` | Create `npm run dev` / `bun run dev` script that launches the CLI in dev mode | 01–03 |
21	| 15 | `15-production-bundle.md` | Create production build: minified bundle, platform-specific packaging | 03 |
22	| 16 | `16-testing.md` | Add test infrastructure (vitest), write smoke tests for core subsystems | All |
23	
24	## Quick Start
25	
26	1. Open a new Copilot chat
27	2. Paste the contents of `01-install-bun-and-deps.md`
28	3. Follow the instructions / let the agent run
29	4. Repeat for `02`, `03`, etc.
30	
31	## Notes
32	
33	- Prompts 07–13 can be run somewhat in **parallel** (they touch different subsystems)
34	- If a prompt fails, fix the issue before moving to the next one
35	- Each prompt is designed to be **independently verifiable** — it tells you how to confirm it worked
36	
```

### Output
```
src\QueryEngine.ts
src\Task.ts
src\Tool.ts
src\commands.ts
src\context.ts
src\cost-tracker.ts
src\costHook.ts
src\history.ts
src\ink.ts
src\projectOnboardingState.ts
src\query.ts
src\setup.ts
src\tasks.ts
src\tools.ts
```

## [tool] Write
- Call ID: `toolu_0145T3TpvXaPCL79sKQzY5vE`
- Timestamp: `2026-05-23 11:26:32`

### Arguments
```json
{
  "file_path": "<PATH>\Users\\Administrator\\.claude\\projects\\f--Projects-AIlearning-claude-code\\memory\\user_role.md",
  "content": "---\nname: user-role\ndescription: User is learning AI Agent development; interested in architecture/design of agent systems (using leaked Claude Code source as study material)\nmetadata:\n  type: user\n---\n\nUser is studying AI Agent development. The `<PATH>\Projects\\AIlearning\\claude-code` repo is a learning artifact (the 2026-03-31 leaked Claude Code source). They are particularly interested in **architecture** — tool systems, query loops, command systems, agent orchestration — rather than just using the CLI.\n\nFrame responses around architectural insight: pipelines, abstractions, design patterns, why-it's-built-this-way. Reference concrete files (e.g. `src/QueryEngine.ts`, `src/Tool.ts`) so they can navigate to source. Chinese is fine for explanation (user wrote in Chinese).\n"
}
```

### Output
```
File created successfully at: <PATH>
```

## [tool] Write
- Call ID: `toolu_01R7dK81HzBQ1mknsU8zJW6K`
- Timestamp: `2026-05-23 11:26:33`

### Arguments
```json
{
  "file_path": "<PATH>\Users\\Administrator\\.claude\\projects\\f--Projects-AIlearning-claude-code\\memory\\MEMORY.md",
  "content": "- [User role](user_role.md) — Learning AI Agent development; uses leaked Claude Code source as architecture study material; responds well to Chinese + file-path references\n"
}
```

### Output
```
File created successfully at: <PATH>
```

## [#2] Assistant
- Timestamp: `2026-05-23 11:27:00`

# 这个项目是什么

**Claude Code 的泄露源码**(2026-03-31 通过 npm 包里的 `.map` 文件泄露)。Anthropic 官方 CLI 工具的完整 TypeScript 实现 —— 约 1900 个文件、51 万行代码。对你学 AI Agent 来说,这是**生产级 Agent 系统**的最佳教材之一。

# 核心架构(从 Agent 视角看)

最关键的一条主线在 [docs/architecture.md:11](docs/architecture.md#L11):

```
User Input → CLI Parser → Query Engine → LLM API → Tool Execution Loop → Terminal UI
```

把它拆成 Agent 开发者关心的 5 个抽象层:

### 1. Query Engine — Agent 的心脏 [src/QueryEngine.ts](src/QueryEngine.ts)
约 4.6 万行,实现了一个 Agent 最核心的 **tool-call loop**:
- 流式调用 LLM API
- LLM 返回 tool_use → 执行 tool → 把结果塞回对话 → 再次调用 LLM
- 配合 thinking mode(扩展思考预算)、自动重试、token 统计、上下文管理

这就是 ReAct/Tool-Use Agent 范式在工业级代码里的样子,**先吃透这个文件**,你就理解了一半 Agent 系统。

### 2. Tool 抽象 [src/Tool.ts](src/Tool.ts) + [src/tools/](src/tools/)
每个工具都是自包含模块,四件套:
- **Input schema**(Zod 校验)
- **Permission model**(哪些操作需要用户授权)
- **Execution logic**
- **UI 渲染组件**(在终端里怎么显示调用过程)

约 40 个工具,从 `FileEditTool`、`BashTool` 到 `AgentTool`(子 Agent)、`SendMessageTool`(Agent 间通信)、`TeamCreateTool`(团队级并行)。**这是 Agent 能力边界的设计模板**。

### 3. 多 Agent 编排 [src/coordinator/](src/coordinator/)
单 Agent 之外,还有 sub-agent spawn(`AgentTool`)、团队协作(`TeamCreateTool`/`SendMessageTool`)、任务管理(`tasks/`)。学 multi-agent 模式可以重点看这里。

### 4. Command 系统 [src/commands.ts](src/commands.ts)
slash 命令(`/review`、`/commit` 等)分三类 —— 见 [docs/architecture.md:67](docs/architecture.md#L67):
- **PromptCommand**:本质是把模板化 prompt + 工具集塞进 LLM
- **LocalCommand**:本地执行返回文本
- **LocalJSXCommand**:本地执行返回 React UI

这是把"用户意图"映射到"Agent 行为"的一种解耦设计。

### 5. Bridge / MCP / Permission(支撑系统)
- [src/bridge/](src/bridge/) — 跟 IDE(VS Code、JetBrains)双向通信
- [src/services/mcp/](src/services/mcp/) — Model Context Protocol,外部工具的标准化接入
- [src/hooks/toolPermission/](src/hooks/toolPermission/) — 每次工具调用前的权限决策

# 推荐学习路径

仓库自带的学习材料相当友好:

1. **[docs/architecture.md](docs/architecture.md)** —— 先通读
2. **[docs/exploration-guide.md](docs/exploration-guide.md)** —— 给你 grep 模式和阅读顺序
3. **[docs/subsystems.md](docs/subsystems.md)** —— Bridge / MCP / Skills / Memory 深入
4. **[prompts/00-overview.md](prompts/00-overview.md)** —— 16 步的"从零搭建可运行版本"指南,如果你想跑起来逐步调试

要不要我从某个具体子系统切入(比如 Query Engine 的 tool-call 循环、或者多 Agent 编排)做深入拆解?

## [#3] User
- Timestamp: `2026-05-23 11:32:14`

- Opened file: `\session.md`

对tools架构做深入拆解

## [tool] Read
- Call ID: `toolu_01LQFP8LB2FrnCrNzqLX1XxW`
- Timestamp: `2026-05-23 11:32:19`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\src\\Tool.ts"
}
```

## [tool] Glob
- Call ID: `toolu_017FAbk5xG5pG92u4Y3p5f5d`
- Timestamp: `2026-05-23 11:32:19`

### Arguments
```json
{
  "pattern": "src/tools/*"
}
```

### Output
```
src\tools\utils.ts
```

## [tool output]
- Call ID: `toolu_01LQFP8LB2FrnCrNzqLX1XxW`
- Timestamp: `2026-05-23 11:32:19`

### Output
```
1	import type {
2	  ToolResultBlockParam,
3	  ToolUseBlockParam,
4	} from '@anthropic-ai/sdk/resources/index.mjs'
5	import type {
6	  ElicitRequestURLParams,
7	  ElicitResult,
8	} from '@modelcontextprotocol/sdk/types.js'
9	import type { UUID } from 'crypto'
10	import type { z } from 'zod/v4'
11	import type { Command } from './commands.js'
12	import type { CanUseToolFn } from './hooks/useCanUseTool.js'
13	import type { ThinkingConfig } from './utils/thinking.js'
14	
15	export type ToolInputJSONSchema = {
16	  [x: string]: unknown
17	  type: 'object'
18	  properties?: {
19	    [x: string]: unknown
20	  }
21	}
22	
23	import type { Notification } from './context/notifications.js'
24	import type {
25	  MCPServerConnection,
26	  ServerResource,
27	} from './services/mcp/types.js'
28	import type {
29	  AgentDefinition,
30	  AgentDefinitionsResult,
31	} from './tools/AgentTool/loadAgentsDir.js'
32	import type {
33	  AssistantMessage,
34	  AttachmentMessage,
35	  Message,
36	  ProgressMessage,
37	  SystemLocalCommandMessage,
38	  SystemMessage,
39	  UserMessage,
40	} from './types/message.js'
41	// Import permission types from centralized location to break import cycles
42	// Import PermissionResult from centralized location to break import cycles
43	import type {
44	  AdditionalWorkingDirectory,
45	  PermissionMode,
46	  PermissionResult,
47	} from './types/permissions.js'
48	// Import tool progress types from centralized location to break import cycles
49	import type {
50	  AgentToolProgress,
51	  BashProgress,
52	  MCPProgress,
53	  REPLToolProgress,
54	  SkillToolProgress,
55	  TaskOutputProgress,
56	  ToolProgressData,
57	  WebSearchProgress,
58	} from './types/tools.js'
59	import type { FileStateCache } from './utils/fileStateCache.js'
60	import type { DenialTrackingState } from './utils/permissions/denialTracking.js'
61	import type { SystemPrompt } from './utils/systemPromptType.js'
62	import type { ContentReplacementState } from './utils/toolResultStorage.js'
63	
64	// Re-export progress types for backwards compatibility
65	export type {
66	  AgentToolProgress,
67	  BashProgress,
68	  MCPProgress,
69	  REPLToolProgress,
70	  SkillToolProgress,
71	  TaskOutputProgress,
72	  WebSearchProgress,
73	}
74	
75	import type { SpinnerMode } from './components/Spinner.js'
76	import type { QuerySource } from './constants/querySource.js'
77	import type { SDKStatus } from './entrypoints/agentSdkTypes.js'
78	import type { AppState } from './state/AppState.js'
79	import type {
80	  HookProgress,
81	  PromptRequest,
82	  PromptResponse,
83	} from './types/hooks.js'
84	import type { AgentId } from './types/ids.js'
85	import type { DeepImmutable } from './types/utils.js'
86	import type { AttributionState } from './utils/commitAttribution.js'
87	import type { FileHistoryState } from './utils/fileHistory.js'
88	import type { Theme, ThemeName } from './utils/theme.js'
89	
90	export type QueryChainTracking = {
91	  chainId: string
92	  depth: number
93	}
94	
95	export type ValidationResult =
96	  | { result: true }
97	  | {
98	      result: false
99	      message: string
100	      errorCode: number
101	    }
102	
103	export type SetToolJSXFn = (
104	  args: {
105	    jsx: React.ReactNode | null
106	    shouldHidePromptInput: boolean
107	    shouldContinueAnimation?: true
108	    showSpinner?: boolean
109	    isLocalJSXCommand?: boolean
110	    isImmediate?: boolean
111	    <PATH> Set to true to clear a local JSX command (e.g., from its onDone callback) */
112	    clearLocalJSX?: boolean
113	  } | null,
114	) => void
115	
116	// Import tool permission types from centralized location to break import cycles
117	import type { ToolPermissionRulesBySource } from './types/permissions.js'
118	
119	// Re-export for backwards compatibility
120	export type { ToolPermissionRulesBySource }
121	
122	// Apply DeepImmutable to the imported type
123	export type ToolPermissionContext = DeepImmutable<{
124	  mode: PermissionMode
125	  additionalWorkingDirectories: Map<string, AdditionalWorkingDirectory>
126	  alwaysAllowRules: ToolPermissionRulesBySource
127	  alwaysDenyRules: ToolPermissionRulesBySource
128	  alwaysAskRules: ToolPermissionRulesBySource
129	  isBypassPermissionsModeAvailable: boolean
130	  isAutoModeAvailable?: boolean
131	  strippedDangerousRules?: ToolPermissionRulesBySource
132	  <PATH> When true, permission prompts are auto-denied (e.g., background agents that can't show UI) */
133	  shouldAvoidPermissionPrompts?: boolean
134	  <PATH> When true, automated checks (classifier, hooks) are awaited before showing the permission dialog (coordinator workers) */
135	  awaitAutomatedChecksBeforeDialog?: boolean
136	  <PATH> Stores the permission mode before model-initiated plan mode entry, so it can be restored on exit */
137	  prePlanMode?: PermissionMode
138	}>
139	
140	export const getEmptyToolPermissionContext: () => ToolPermissionContext =
141	  () => ({
142	    mode: 'default',
143	    additionalWorkingDirectories: new Map(),
144	    alwaysAllowRules: {},
145	    alwaysDenyRules: {},
146	    alwaysAskRules: {},
147	    isBypassPermissionsModeAvailable: false,
148	  })
149	
150	export type CompactProgressEvent =
151	  | {
152	      type: 'hooks_start'
153	      hookType: 'pre_compact' | 'post_compact' | 'session_start'
154	    }
155	  | { type: 'compact_start' }
156	  | { type: 'compact_end' }
157	
158	export type ToolUseContext = {
159	  options: {
160	    commands: Command[]
161	    debug: boolean
162	    mainLoopModel: string
163	    tools: Tools
164	    verbose: boolean
165	    thinkingConfig: ThinkingConfig
166	    mcpClients: MCPServerConnection[]
167	    mcpResources: Record<string, ServerResource[]>
168	    isNonInteractiveSession: boolean
169	    agentDefinitions: AgentDefinitionsResult
170	    maxBudgetUsd?: number
171	    <PATH> Custom system prompt that replaces the default system prompt */
172	    customSystemPrompt?: string
173	    <PATH> Additional system prompt appended after the main system prompt */
174	    appendSystemPrompt?: string
175	    <PATH> Override querySource for analytics tracking */
176	    querySource?: QuerySource
177	    <PATH> Optional callback to get the latest tools (e.g., after MCP servers connect mid-query) */
178	    refreshTools?: () => Tools
179	  }
180	  abortController: AbortController
181	  readFileState: FileStateCache
182	  getAppState(): AppState
183	  setAppState(f: (prev: AppState) => AppState): void
184	  <PATH>
185	   * Always-shared setAppState for session-scoped infrastructure (background
186	   * tasks, session hooks). Unlike setAppState, which is no-op for async agents
187	   * (see createSubagentContext), this always reaches the root store so agents
188	   * at any nesting depth can register/clean up infrastructure that outlives
189	   * a single turn. Only set by createSubagentContext; main-thread contexts
190	   * fall back to setAppState.
191	   */
192	  setAppStateForTasks?: (f: (prev: AppState) => AppState) => void
193	  <PATH>
194	   * Optional handler for URL elicitations triggered by tool call errors (-32042).
195	   * In print/SDK mode, this delegates to structuredIO.handleElicitation.
196	   * In REPL mode, this is undefined and the queue-based UI path is used.
197	   */
198	  handleElicitation?: (
199	    serverName: string,
200	    params: ElicitRequestURLParams,
201	    signal: AbortSignal,
202	  ) => Promise<ElicitResult>
203	  setToolJSX?: SetToolJSXFn
204	  addNotification?: (notif: Notification) => void
205	  <PATH> Append a UI-only system message to the REPL message list. Stripped at the
206	   *  normalizeMessagesForAPI boundary — the Exclude<> makes that type-enforced. */
207	  appendSystemMessage?: (
208	    msg: Exclude<SystemMessage, SystemLocalCommandMessage>,
209	  ) => void
210	  <PATH> Send an OS-level notification (iTerm2, Kitty, Ghostty, bell, etc.) */
211	  sendOSNotification?: (opts: {
212	    message: string
213	    notificationType: string
214	  }) => void
215	  nestedMemoryAttachmentTriggers?: Set<string>
216	  <PATH>
217	   * CLAUDE.md paths already injected as nested_memory attachments this
218	   * session. Dedup for memoryFilesToAttachments — readFileState is an LRU
219	   * that evicts entries in busy sessions, so its .has() check alone can
220	   * re-inject the same CLAUDE.md dozens of times.
221	   */
222	  loadedNestedMemoryPaths?: Set<string>
223	  dynamicSkillDirTriggers?: Set<string>
224	  <PATH> Skill names surfaced via skill_discovery this session. Telemetry only (feeds was_discovered). */
225	  discoveredSkillNames?: Set<string>
226	  userModified?: boolean
227	  setInProgressToolUseIDs: (f: (prev: Set<string>) => Set<string>) => void
228	  <PATH> Only wired in interactive (REPL) contexts; SDK/QueryEngine don't set this. */
229	  setHasInterruptibleToolInProgress?: (v: boolean) => void
230	  setResponseLength: (f: (prev: number) => number) => void
231	  <PATH> Ant-only: push a new API metrics entry for OTPS tracking.
232	   *  Called by subagent streaming when a new API request starts. */
233	  pushApiMetricsEntry?: (ttftMs: number) => void
234	  setStreamMode?: (mode: SpinnerMode) => void
235	  onCompactProgress?: (event: CompactProgressEvent) => void
236	  setSDKStatus?: (status: SDKStatus) => void
237	  openMessageSelector?: () => void
238	  updateFileHistoryState: (
239	    updater: (prev: FileHistoryState) => FileHistoryState,
240	  ) => void
241	  updateAttributionState: (
242	    updater: (prev: AttributionState) => AttributionState,
243	  ) => void
244	  setConversationId?: (id: UUID) => void
245	  agentId?: AgentId // Only set for subagents; use getSessionId() for session ID. Hooks use this to distinguish subagent calls.
246	  agentType?: string // Subagent type name. For the main thread's --agent type, hooks fall back to getMainThreadAgentType().
247	  <PATH> When true, canUseTool must always be called even when hooks auto-approve.
248	   *  Used by speculation for overlay file path rewriting. */
249	  requireCanUseTool?: boolean
250	  messages: Message[]
251	  fileReadingLimits?: {
252	    maxTokens?: number
253	    maxSizeBytes?: number
254	  }
255	  globLimits?: {
256	    maxResults?: number
257	  }
258	  toolDecisions?: Map<
259	    string,
260	    {
261	      source: string
262	      decision: 'accept' | 'reject'
263	      timestamp: number
264	    }
265	  >
266	  queryTracking?: QueryChainTracking
267	  <PATH> Callback factory for requesting interactive prompts from the user.
268	   * Returns a prompt callback bound to the given source name.
269	   * Only available in interactive (REPL) contexts. */
270	  requestPrompt?: (
271	    sourceName: string,
272	    toolInputSummary?: string | null,
273	  ) => (request: PromptRequest) => Promise<PromptResponse>
274	  toolUseId?: string
275	  criticalSystemReminder_EXPERIMENTAL?: string
276	  <PATH> When true, preserve toolUseResult on messages even for subagents.
277	   * Used by in-process teammates whose transcripts are viewable by the user. */
278	  preserveToolUseResults?: boolean
279	  <PATH> Local denial tracking state for async subagents whose setAppState is a
280	   *  no-op. Without this, the denial counter never accumulates and the
281	   *  fallback-to-prompting threshold is never reached. Mutable — the
282	   *  permissions code updates it in place. */
283	  localDenialTracking?: DenialTrackingState
284	  <PATH>
285	   * Per-conversation-thread content replacement state for the tool result
286	   * budget. When present, query.ts applies the aggregate tool result budget.
287	   * Main thread: REPL provisions once (never resets — stale UUID keys
288	   * are inert). Subagents: createSubagentContext clones the parent's state
289	   * by default (cache-sharing forks need identical decisions), or
290	   * resumeAgentBackground threads one reconstructed from sidechain records.
291	   */
292	  contentReplacementState?: ContentReplacementState
293	  <PATH>
294	   * Parent's rendered system prompt bytes, frozen at turn start.
295	   * Used by fork subagents to share the parent's prompt cache — re-calling
296	   * getSystemPrompt() at fork-spawn time can diverge (GrowthBook cold→warm)
297	   * and bust the cache. See forkSubagent.ts.
298	   */
299	  renderedSystemPrompt?: SystemPrompt
300	}
301	
302	// Re-export ToolProgressData from centralized location
303	export type { ToolProgressData }
304	
305	export type Progress = ToolProgressData | HookProgress
306	
307	export type ToolProgress<P extends ToolProgressData> = {
308	  toolUseID: string
309	  data: P
310	}
311	
312	export function filterToolProgressMessages(
313	  progressMessagesForMessage: ProgressMessage[],
314	): ProgressMessage<ToolProgressData>[] {
315	  return progressMessagesForMessage.filter(
316	    (msg): msg is ProgressMessage<ToolProgressData> =>
317	      msg.data?.type !== 'hook_progress',
318	  )
319	}
320	
321	export type ToolResult<T> = {
322	  data: T
323	  newMessages?: (
324	    | UserMessage
325	    | AssistantMessage
326	    | AttachmentMessage
327	    | SystemMessage
328	  )[]
329	  // contextModifier is only honored for tools that aren't concurrency safe.
330	  contextModifier?: (context: ToolUseContext) => ToolUseContext
331	  <PATH> MCP protocol metadata (structuredContent, _meta) to pass through to SDK consumers */
332	  mcpMeta?: {
333	    _meta?: Record<string, unknown>
334	    structuredContent?: Record<string, unknown>
335	  }
336	}
337	
338	export type ToolCallProgress<P extends ToolProgressData = ToolProgressData> = (
339	  progress: ToolProgress<P>,
340	) => void
341	
342	// Type for any schema that outputs an object with string keys
343	export type AnyObject = z.ZodType<{ [key: string]: unknown }>
344	
345	<PATH>
346	 * Checks if a tool matches the given name (primary name or alias).
347	 */
348	export function toolMatchesName(
349	  tool: { name: string; aliases?: string[] },
350	  name: string,
351	): boolean {
352	  return tool.name === name || (tool.aliases?.includes(name) ?? false)
353	}
354	
355	<PATH>
356	 * Finds a tool by name or alias from a list of tools.
357	 */
358	export function findToolByName(tools: Tools, name: string): Tool | undefined {
359	  return tools.find(t => toolMatchesName(t, name))
360	}
361	
362	export type Tool<
363	  Input extends AnyObject = AnyObject,
364	  Output = unknown,
365	  P extends ToolProgressData = ToolProgressData,
366	> = {
367	  <PATH>
368	   * Optional aliases for backwards compatibility when a tool is renamed.
369	   * The tool can be looked up by any of these names in addition to its primary name.
370	   */
371	  aliases?: string[]
372	  <PATH>
373	   * One-line capability phrase used by ToolSearch for keyword matching.
374	   * Helps the model find this tool via keyword search when it's deferred.
375	   * 3–10 words, no trailing period.
376	   * Prefer terms not already in the tool name (e.g. 'jupyter' for NotebookEdit).
377	   */
378	  searchHint?: string
379	  call(
380	    args: z.infer<Input>,
381	    context: ToolUseContext,
382	    canUseTool: CanUseToolFn,
383	    parentMessage: AssistantMessage,
384	    onProgress?: ToolCallProgress<P>,
385	  ): Promise<ToolResult<Output>>
386	  description(
387	    input: z.infer<Input>,
388	    options: {
389	      isNonInteractiveSession: boolean
390	      toolPermissionContext: ToolPermissionContext
391	      tools: Tools
392	    },
393	  ): Promise<string>
394	  readonly inputSchema: Input
395	  // Type for MCP tools that can specify their input schema directly in JSON Schema format
396	  // rather than converting from Zod schema
397	  readonly inputJSONSchema?: ToolInputJSONSchema
398	  // Optional because TungstenTool doesn't define this. TODO: Make it required.
399	  // When we do that, we can also go through and make this a bit more type-safe.
400	  outputSchema?: z.ZodType<unknown>
401	  inputsEquivalent?(a: z.infer<Input>, b: z.infer<Input>): boolean
402	  isConcurrencySafe(input: z.infer<Input>): boolean
403	  isEnabled(): boolean
404	  isReadOnly(input: z.infer<Input>): boolean
405	  <PATH> Defaults to false. Only set when the tool performs irreversible operations (delete, overwrite, send). */
406	  isDestructive?(input: z.infer<Input>): boolean
407	  <PATH>
408	   * What should happen when the user submits a new message while this tool
409	   * is running.
410	   *
411	   * - `'cancel'` — stop the tool and discard its result
412	   * - `'block'`  — keep running; the new message waits
413	   *
414	   * Defaults to `'block'` when not implemented.
415	   */
416	  interruptBehavior?(): 'cancel' | 'block'
417	  <PATH>
418	   * Returns information about whether this tool use is a search or read operation
419	   * that should be collapsed into a condensed display in the UI. Examples include
420	   * file searching (Grep, Glob), file reading (Read), and bash commands like find,
421	   * grep, wc, etc.
422	   *
423	   * Returns an object indicating whether the operation is a search or read operation:
424	   * - `isSearch: true` for search operations (grep, find, glob patterns)
425	   * - `isRead: true` for read operations (cat, head, tail, file read)
426	   * - `isList: true` for directory-listing operations (ls, tree, du)
427	   * - All can be false if the operation shouldn't be collapsed
428	   */
429	  isSearchOrReadCommand?(input: z.infer<Input>): {
430	    isSearch: boolean
431	    isRead: boolean
432	    isList?: boolean
433	  }
434	  isOpenWorld?(input: z.infer<Input>): boolean
435	  requiresUserInteraction?(): boolean
436	  isMcp?: boolean
437	  isLsp?: boolean
438	  <PATH>
439	   * When true, this tool is deferred (sent with defer_loading: true) and requires
440	   * ToolSearch to be used before it can be called.
441	   */
442	  readonly shouldDefer?: boolean
443	  <PATH>
444	   * When true, this tool is never deferred — its full schema appears in the
445	   * initial prompt even when ToolSearch is enabled. For MCP tools, set via
446	   * `_meta['anthropic/alwaysLoad']`. Use for tools the model must see on
447	   * turn 1 without a ToolSearch round-trip.
448	   */
449	  readonly alwaysLoad?: boolean
450	  <PATH>
451	   * For MCP tools: the server and tool names as received from the MCP server (unnormalized).
452	   * Present on all MCP tools regardless of whether `name` is prefixed (mcp__server__tool)
453	   * or unprefixed (CLAUDE_AGENT_SDK_MCP_NO_PREFIX mode).
454	   */
455	  mcpInfo?: { serverName: string; toolName: string }
456	  readonly name: string
457	  <PATH>
458	   * Maximum size in characters for tool result before it gets persisted to disk.
459	   * When exceeded, the result is saved to a file and Claude receives a preview
460	   * with the file path instead of the full content.
461	   *
462	   * Set to Infinity for tools whose output must never be persisted (e.g. Read,
463	   * where persisting creates a circular Read→file→Read loop and the tool
464	   * already self-bounds via its own limits).
465	   */
466	  maxResultSizeChars: number
467	  <PATH>
468	   * When true, enables strict mode for this tool, which causes the API to
469	   * more strictly adhere to tool instructions and parameter schemas.
470	   * Only applied when the tengu_tool_pear is enabled.
471	   */
472	  readonly strict?: boolean
473	
474	  <PATH>
475	   * Called on copies of tool_use input before observers see it (SDK stream,
476	   * transcript, canUseTool, PreToolUse/PostToolUse hooks). Mutate in place
477	   * to add legacy/derived fields. Must be idempotent. The original API-bound
478	   * input is never mutated (preserves prompt cache). Not re-applied when a
479	   * hook/permission returns a fresh updatedInput — those own their shape.
480	   */
481	  backfillObservableInput?(input: Record<string, unknown>): void
482	
483	  <PATH>
484	   * Determines if this tool is allowed to run with this input in the current context.
485	   * It informs the model of why the tool use failed, and does not directly display any UI.
486	   * @param input
487	   * @param context
488	   */
489	  validateInput?(
490	    input: z.infer<Input>,
491	    context: ToolUseContext,
492	  ): Promise<ValidationResult>
493	
494	  <PATH>
495	   * Determines if the user is asked for permission. Only called after validateInput() passes.
496	   * General permission logic is in permissions.ts. This method contains tool-specific logic.
497	   * @param input
498	   * @param context
499	   */
500	  checkPermissions(
501	    input: z.infer<Input>,
502	    context: ToolUseContext,
503	  ): Promise<PermissionResult>
504	
505	  // Optional method for tools that operate on a file path
506	  getPath?(input: z.infer<Input>): string
507	
508	  <PATH>
509	   * Prepare a matcher for hook `if` conditions (permission-rule patterns like
510	   * "git *" from "Bash(git *)"). Called once per hook-input pair; any
511	   * expensive parsing happens here. Returns a closure that is called per
512	   * hook pattern. If not implemented, only tool-name-level matching works.
513	   */
514	  preparePermissionMatcher?(
515	    input: z.infer<Input>,
516	  ): Promise<(pattern: string) => boolean>
517	
518	  prompt(options: {
519	    getToolPermissionContext: () => Promise<ToolPermissionContext>
520	    tools: Tools
521	    agents: AgentDefinition[]
522	    allowedAgentTypes?: string[]
523	  }): Promise<string>
524	  userFacingName(input: Partial<z.infer<Input>> | undefined): string
525	  userFacingNameBackgroundColor?(
526	    input: Partial<z.infer<Input>> | undefined,
527	  ): keyof Theme | undefined
528	  <PATH>
529	   * Transparent wrappers (e.g. REPL) delegate all rendering to their progress
530	   * handler, which emits native-looking blocks for each inner tool call.
531	   * The wrapper itself shows nothing.
532	   */
533	  isTransparentWrapper?(): boolean
534	  <PATH>
535	   * Returns a short string summary of this tool use for display in compact views.
536	   * @param input The tool input
537	   * @returns A short string summary, or null to not display
538	   */
539	  getToolUseSummary?(input: Partial<z.infer<Input>> | undefined): string | null
540	  <PATH>
541	   * Returns a human-readable present-tense activity description for spinner display.
542	   * Example: "Reading src/foo.ts", "Running bun test", "Searching for pattern"
543	   * @param input The tool input
544	   * @returns Activity description string, or null to fall back to tool name
545	   */
546	  getActivityDescription?(
547	    input: Partial<z.infer<Input>> | undefined,
548	  ): string | null
549	  <PATH>
550	   * Returns a compact representation of this tool use for the auto-mode
551	   * security classifier. Examples: `ls -la` for Bash, `/tmp/x: new content`
552	   * for Edit. Return '' to skip this tool in the classifier transcript
553	   * (e.g. tools with no security relevance). May return an object to avoid
554	   * double-encoding when the caller JSON-wraps the value.
555	   */
556	  toAutoClassifierInput(input: z.infer<Input>): unknown
557	  mapToolResultToToolResultBlockParam(
558	    content: Output,
559	    toolUseID: string,
560	  ): ToolResultBlockParam
561	  <PATH>
562	   * Optional. When omitted, the tool result renders nothing (same as returning
563	   * null). Omit for tools whose results are surfaced elsewhere (e.g., TodoWrite
564	   * updates the todo panel, not the transcript).
565	   */
566	  renderToolResultMessage?(
567	    content: Output,
568	    progressMessagesForMessage: ProgressMessage<P>[],
569	    options: {
570	      style?: 'condensed'
571	      theme: ThemeName
572	      tools: Tools
573	      verbose: boolean
574	      isTranscriptMode?: boolean
575	      isBriefOnly?: boolean
576	      <PATH> Original tool_use input, when available. Useful for compact result
577	       * summaries that reference what was requested (e.g. "Sent to #foo"). */
578	      input?: unknown
579	    },
580	  ): React.ReactNode
581	  <PATH>
582	   * Flattened text of what renderToolResultMessage shows IN TRANSCRIPT
583	   * MODE (verbose=true, isTranscriptMode=true). For transcript search
584	   * indexing: the index counts occurrences in this string, the highlight
585	   * overlay scans the actual screen buffer. For count ≡ highlight, this
586	   * must return the text that ends up visible — not the model-facing
587	   * serialization from mapToolResultToToolResultBlockParam (which adds
588	   * system-reminders, persisted-output wrappers).
589	   *
590	   * Chrome can be skipped (under-count is fine). "Found 3 files in 12ms"
591	   * isn't worth indexing. Phantoms are not fine — text that's claimed
592	   * here but doesn't render is a count≠highlight bug.
593	   *
594	   * Optional: omitted → field-name heuristic in transcriptSearch.ts.
595	   * Drift caught by test/utils/transcriptSearch.renderFidelity.test.tsx
596	   * which renders sample outputs and flags text that's indexed-but-not-
597	   * rendered (phantom) or rendered-but-not-indexed (under-count warning).
598	   */
599	  extractSearchText?(out: Output): string
600	  <PATH>
601	   * Render the tool use message. Note that `input` is partial because we render
602	   * the message as soon as possible, possibly before tool parameters have fully
603	   * streamed in.
604	   */
605	  renderToolUseMessage(
606	    input: Partial<z.infer<Input>>,
607	    options: { theme: ThemeName; verbose: boolean; commands?: Command[] },
608	  ): React.ReactNode
609	  <PATH>
610	   * Returns true when the non-verbose rendering of this output is truncated
611	   * (i.e., clicking to expand would reveal more content). Gates
612	   * click-to-expand in fullscreen — only messages where verbose actually
613	   * shows more get a hover/click affordance. Unset means never truncated.
614	   */
615	  isResultTruncated?(output: Output): boolean
616	  <PATH>
617	   * Renders an optional tag to display after the tool use message.
618	   * Used for additional metadata like timeout, model, resume ID, etc.
619	   * Returns null to not display anything.
620	   */
621	  renderToolUseTag?(input: Partial<z.infer<Input>>): React.ReactNode
622	  <PATH>
623	   * Optional. When omitted, no progress UI is shown while the tool runs.
624	   */
625	  renderToolUseProgressMessage?(
626	    progressMessagesForMessage: ProgressMessage<P>[],
627	    options: {
628	      tools: Tools
629	      verbose: boolean
630	      terminalSize?: { columns: number; rows: number }
631	      inProgressToolCallCount?: number
632	      isTranscriptMode?: boolean
633	    },
634	  ): React.ReactNode
635	  renderToolUseQueuedMessage?(): React.ReactNode
636	  <PATH>
637	   * Optional. When omitted, falls back to <FallbackToolUseRejectedMessage <PATH>
638	   * Only define this for tools that need custom rejection UI (e.g., file edits
639	   * that show the rejected diff).
640	   */
641	  renderToolUseRejectedMessage?(
642	    input: z.infer<Input>,
643	    options: {
644	      columns: number
645	      messages: Message[]
646	      style?: 'condensed'
647	      theme: ThemeName
648	      tools: Tools
649	      verbose: boolean
650	      progressMessagesForMessage: ProgressMessage<P>[]
651	      isTranscriptMode?: boolean
652	    },
653	  ): React.ReactNode
654	  <PATH>
655	   * Optional. When omitted, falls back to <FallbackToolUseErrorMessage <PATH>
656	   * Only define this for tools that need custom error UI (e.g., search tools
657	   * that show "File not found" instead of the raw error).
658	   */
659	  renderToolUseErrorMessage?(
660	    result: ToolResultBlockParam['content'],
661	    options: {
662	      progressMessagesForMessage: ProgressMessage<P>[]
663	      tools: Tools
664	      verbose: boolean
665	      isTranscriptMode?: boolean
666	    },
667	  ): React.ReactNode
668	
669	  <PATH>
670	   * Renders multiple parallel instances of this tool as a group.
671	   * @returns React node to render, or null to fall back to individual rendering
672	   */
673	  <PATH>
674	   * Renders multiple tool uses as a group (non-verbose mode only).
675	   * In verbose mode, individual tool uses render at their original positions.
676	   * @returns React node to render, or null to fall back to individual rendering
677	   */
678	  renderGroupedToolUse?(
679	    toolUses: Array<{
680	      param: ToolUseBlockParam
681	      isResolved: boolean
682	      isError: boolean
683	      isInProgress: boolean
684	      progressMessages: ProgressMessage<P>[]
685	      result?: {
686	        param: ToolResultBlockParam
687	        output: unknown
688	      }
689	    }>,
690	    options: {
691	      shouldAnimate: boolean
692	      tools: Tools
693	    },
694	  ): React.ReactNode | null
695	}
696	
697	<PATH>
698	 * A collection of tools. Use this type instead of `Tool[]` to make it easier
699	 * to track where tool sets are assembled, passed, and filtered across the codebase.
700	 */
701	export type Tools = readonly Tool[]
702	
703	<PATH>
704	 * Methods that `buildTool` supplies a default for. A `ToolDef` may omit these;
705	 * the resulting `Tool` always has them.
706	 */
707	type DefaultableToolKeys =
708	  | 'isEnabled'
709	  | 'isConcurrencySafe'
710	  | 'isReadOnly'
711	  | 'isDestructive'
712	  | 'checkPermissions'
713	  | 'toAutoClassifierInput'
714	  | 'userFacingName'
715	
716	<PATH>
717	 * Tool definition accepted by `buildTool`. Same shape as `Tool` but with the
718	 * defaultable methods optional — `buildTool` fills them in so callers always
719	 * see a complete `Tool`.
720	 */
721	export type ToolDef<
722	  Input extends AnyObject = AnyObject,
723	  Output = unknown,
724	  P extends ToolProgressData = ToolProgressData,
725	> = Omit<Tool<Input, Output, P>, DefaultableToolKeys> &
726	  Partial<Pick<Tool<Input, Output, P>, DefaultableToolKeys>>
727	
728	<PATH>
729	 * Type-level spread mirroring `{ ...TOOL_DEFAULTS, ...def }`. For each
730	 * defaultable key: if D provides it (required), D's type wins; if D omits
731	 * it or has it optional (inherited from Partial<> in the constraint), the
732	 * default fills in. All other keys come from D verbatim — preserving arity,
733	 * optional presence, and literal types exactly as `satisfies Tool` did.
734	 */
735	type BuiltTool<D> = Omit<D, DefaultableToolKeys> & {
736	  [K in DefaultableToolKeys]-?: K extends keyof D
737	    ? undefined extends D[K]
738	      ? ToolDefaults[K]
739	      : D[K]
740	    : ToolDefaults[K]
741	}
742	
743	<PATH>
744	 * Build a complete `Tool` from a partial definition, filling in safe defaults
745	 * for the commonly-stubbed methods. All tool exports should go through this so
746	 * that defaults live in one place and callers never need `?.() ?? default`.
747	 *
748	 * Defaults (fail-closed where it matters):
749	 * - `isEnabled` → `true`
750	 * - `isConcurrencySafe` → `false` (assume not safe)
751	 * - `isReadOnly` → `false` (assume writes)
752	 * - `isDestructive` → `false`
753	 * - `checkPermissions` → `{ behavior: 'allow', updatedInput }` (defer to general permission system)
754	 * - `toAutoClassifierInput` → `''` (skip classifier — security-relevant tools must override)
755	 * - `userFacingName` → `name`
756	 */
757	const TOOL_DEFAULTS = {
758	  isEnabled: () => true,
759	  isConcurrencySafe: (_input?: unknown) => false,
760	  isReadOnly: (_input?: unknown) => false,
761	  isDestructive: (_input?: unknown) => false,
762	  checkPermissions: (
763	    input: { [key: string]: unknown },
764	    _ctx?: ToolUseContext,
765	  ): Promise<PermissionResult> =>
766	    Promise.resolve({ behavior: 'allow', updatedInput: input }),
767	  toAutoClassifierInput: (_input?: unknown) => '',
768	  userFacingName: (_input?: unknown) => '',
769	}
770	
771	// The defaults type is the ACTUAL shape of TOOL_DEFAULTS (optional params so
772	// both 0-arg and full-arg call sites type-check — stubs varied in arity and
773	// tests relied on that), not the interface's strict signatures.
774	type ToolDefaults = typeof TOOL_DEFAULTS
775	
776	// D infers the concrete object-literal type from the call site. The
777	// constraint provides contextual typing for method parameters; `any` in
778	// constraint position is structural and never leaks into the return type.
779	// BuiltTool<D> mirrors runtime `{...TOOL_DEFAULTS, ...def}` at the type level.
780	// eslint-disable-next-line @typescript-eslint/no-explicit-any
781	type AnyToolDef = ToolDef<any, any, any>
782	
783	export function buildTool<D extends AnyToolDef>(def: D): BuiltTool<D> {
784	  // The runtime spread is straightforward; the `as` bridges the gap between
785	  // the structural-any constraint and the precise BuiltTool<D> return. The
786	  // type semantics are proven by the 0-error typecheck across all 60+ tools.
787	  return {
788	    ...TOOL_DEFAULTS,
789	    userFacingName: () => def.name,
790	    ...def,
791	  } as BuiltTool<D>
792	}
793	
794	
795	
```

## [tool] Glob
- Call ID: `toolu_01VSzoQri5uJkopaBy7GJRNp`
- Timestamp: `2026-05-23 11:32:24`

### Arguments
```json
{
  "pattern": "src/tools/**/*.ts",
  "head_limit": "60"
}
```

### Output
```
<tool_use_error>InputValidationError: Glob failed due to the following issue:
An unexpected parameter `head_limit` was provided</tool_use_error>
```

## [tool] Read
- Call ID: `toolu_012CJxZmfjkhiN8BsY5hr6JJ`
- Timestamp: `2026-05-23 11:32:25`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\src\\tools.ts",
  "limit": 100
}
```

### Output
```
1	// biome-ignore-all assist/source/organizeImports: ANT-ONLY import markers must not be reordered
2	import { toolMatchesName, type Tool, type Tools } from './Tool.js'
3	import { AgentTool } from './tools/AgentTool/AgentTool.js'
4	import { SkillTool } from './tools/SkillTool/SkillTool.js'
5	import { BashTool } from './tools/BashTool/BashTool.js'
6	import { FileEditTool } from './tools/FileEditTool/FileEditTool.js'
7	import { FileReadTool } from './tools/FileReadTool/FileReadTool.js'
8	import { FileWriteTool } from './tools/FileWriteTool/FileWriteTool.js'
9	import { GlobTool } from './tools/GlobTool/GlobTool.js'
10	import { NotebookEditTool } from './tools/NotebookEditTool/NotebookEditTool.js'
11	import { WebFetchTool } from './tools/WebFetchTool/WebFetchTool.js'
12	import { TaskStopTool } from './tools/TaskStopTool/TaskStopTool.js'
13	import { BriefTool } from './tools/BriefTool/BriefTool.js'
14	// Dead code elimination: conditional import for ant-only tools
15	<PATH> eslint-disable custom-rules/no-process-env-top-level, @typescript-eslint/no-require-imports */
16	const REPLTool =
17	  process.env.USER_TYPE === 'ant'
18	    ? require('./tools/REPLTool/REPLTool.js').REPLTool
19	    : null
20	const SuggestBackgroundPRTool =
21	  process.env.USER_TYPE === 'ant'
22	    ? require('./tools/SuggestBackgroundPRTool/SuggestBackgroundPRTool.js')
23	        .SuggestBackgroundPRTool
24	    : null
25	const SleepTool =
26	  feature('PROACTIVE') || feature('KAIROS')
27	    ? require('./tools/SleepTool/SleepTool.js').SleepTool
28	    : null
29	const cronTools = feature('AGENT_TRIGGERS')
30	  ? [
31	      require('./tools/ScheduleCronTool/CronCreateTool.js').CronCreateTool,
32	      require('./tools/ScheduleCronTool/CronDeleteTool.js').CronDeleteTool,
33	      require('./tools/ScheduleCronTool/CronListTool.js').CronListTool,
34	    ]
35	  : []
36	const RemoteTriggerTool = feature('AGENT_TRIGGERS_REMOTE')
37	  ? require('./tools/RemoteTriggerTool/RemoteTriggerTool.js').RemoteTriggerTool
38	  : null
39	const MonitorTool = feature('MONITOR_TOOL')
40	  ? require('./tools/MonitorTool/MonitorTool.js').MonitorTool
41	  : null
42	const SendUserFileTool = feature('KAIROS')
43	  ? require('./tools/SendUserFileTool/SendUserFileTool.js').SendUserFileTool
44	  : null
45	const PushNotificationTool =
46	  feature('KAIROS') || feature('KAIROS_PUSH_NOTIFICATION')
47	    ? require('./tools/PushNotificationTool/PushNotificationTool.js')
48	        .PushNotificationTool
49	    : null
50	const SubscribePRTool = feature('KAIROS_GITHUB_WEBHOOKS')
51	  ? require('./tools/SubscribePRTool/SubscribePRTool.js').SubscribePRTool
52	  : null
53	<PATH> eslint-enable custom-rules/no-process-env-top-level, @typescript-eslint/no-require-imports */
54	import { TaskOutputTool } from './tools/TaskOutputTool/TaskOutputTool.js'
55	import { WebSearchTool } from './tools/WebSearchTool/WebSearchTool.js'
56	import { TodoWriteTool } from './tools/TodoWriteTool/TodoWriteTool.js'
57	import { ExitPlanModeV2Tool } from './tools/ExitPlanModeTool/ExitPlanModeV2Tool.js'
58	import { TestingPermissionTool } from './tools/testing/TestingPermissionTool.js'
59	import { GrepTool } from './tools/GrepTool/GrepTool.js'
60	import { TungstenTool } from './tools/TungstenTool/TungstenTool.js'
61	// Lazy require to break circular dependency: tools.ts -> TeamCreateTool/TeamDeleteTool -> ... -> tools.ts
62	<PATH> eslint-disable @typescript-eslint/no-require-imports */
63	const getTeamCreateTool = () =>
64	  require('./tools/TeamCreateTool/TeamCreateTool.js')
65	    .TeamCreateTool as typeof import('./tools/TeamCreateTool/TeamCreateTool.js').TeamCreateTool
66	const getTeamDeleteTool = () =>
67	  require('./tools/TeamDeleteTool/TeamDeleteTool.js')
68	    .TeamDeleteTool as typeof import('./tools/TeamDeleteTool/TeamDeleteTool.js').TeamDeleteTool
69	const getSendMessageTool = () =>
70	  require('./tools/SendMessageTool/SendMessageTool.js')
71	    .SendMessageTool as typeof import('./tools/SendMessageTool/SendMessageTool.js').SendMessageTool
72	<PATH> eslint-enable @typescript-eslint/no-require-imports */
73	import { AskUserQuestionTool } from './tools/AskUserQuestionTool/AskUserQuestionTool.js'
74	import { LSPTool } from './tools/LSPTool/LSPTool.js'
75	import { ListMcpResourcesTool } from './tools/ListMcpResourcesTool/ListMcpResourcesTool.js'
76	import { ReadMcpResourceTool } from './tools/ReadMcpResourceTool/ReadMcpResourceTool.js'
77	import { ToolSearchTool } from './tools/ToolSearchTool/ToolSearchTool.js'
78	import { EnterPlanModeTool } from './tools/EnterPlanModeTool/EnterPlanModeTool.js'
79	import { EnterWorktreeTool } from './tools/EnterWorktreeTool/EnterWorktreeTool.js'
80	import { ExitWorktreeTool } from './tools/ExitWorktreeTool/ExitWorktreeTool.js'
81	import { ConfigTool } from './tools/ConfigTool/ConfigTool.js'
82	import { TaskCreateTool } from './tools/TaskCreateTool/TaskCreateTool.js'
83	import { TaskGetTool } from './tools/TaskGetTool/TaskGetTool.js'
84	import { TaskUpdateTool } from './tools/TaskUpdateTool/TaskUpdateTool.js'
85	import { TaskListTool } from './tools/TaskListTool/TaskListTool.js'
86	import uniqBy from 'lodash-es/uniqBy.js'
87	import { isToolSearchEnabledOptimistic } from './utils/toolSearch.js'
88	import { isTodoV2Enabled } from './utils/tasks.js'
89	// Dead code elimination: conditional import for CLAUDE_CODE_VERIFY_PLAN
90	<PATH> eslint-disable custom-rules/no-process-env-top-level, @typescript-eslint/no-require-imports */
91	const VerifyPlanExecutionTool =
92	  process.env.CLAUDE_CODE_VERIFY_PLAN === 'true'
93	    ? require('./tools/VerifyPlanExecutionTool/VerifyPlanExecutionTool.js')
94	        .VerifyPlanExecutionTool
95	    : null
96	<PATH> eslint-enable custom-rules/no-process-env-top-level, @typescript-eslint/no-require-imports */
97	import { SYNTHETIC_OUTPUT_TOOL_NAME } from './tools/SyntheticOutputTool/SyntheticOutputTool.js'
98	export {
99	  ALL_AGENT_DISALLOWED_TOOLS,
100	  CUSTOM_AGENT_DISALLOWED_TOOLS,
```

## [tool] Glob
- Call ID: `toolu_01U1x6oJW7L2h7BtmWyJ16pF`
- Timestamp: `2026-05-23 11:32:28`

### Arguments
```json
{
  "pattern": "src/tools/BashTool/*.ts"
}
```

### Output
```
src\tools\BashTool\bashCommandHelpers.ts
src\tools\BashTool\bashPermissions.ts
src\tools\BashTool\bashSecurity.ts
src\tools\BashTool\commandSemantics.ts
src\tools\BashTool\commentLabel.ts
src\tools\BashTool\destructiveCommandWarning.ts
src\tools\BashTool\modeValidation.ts
src\tools\BashTool\pathValidation.ts
src\tools\BashTool\prompt.ts
src\tools\BashTool\readOnlyValidation.ts
src\tools\BashTool\sedEditParser.ts
src\tools\BashTool\sedValidation.ts
src\tools\BashTool\shouldUseSandbox.ts
src\tools\BashTool\toolName.ts
src\tools\BashTool\utils.ts
```

## [tool] Read
- Call ID: `toolu_01EuZ4LC4ZGK7ApL6X7cjjcf`
- Timestamp: `2026-05-23 11:32:30`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\src\\tools.ts",
  "offset": 100,
  "limit": 120
}
```

### Output
```
100	  CUSTOM_AGENT_DISALLOWED_TOOLS,
101	  ASYNC_AGENT_ALLOWED_TOOLS,
102	  COORDINATOR_MODE_ALLOWED_TOOLS,
103	} from './constants/tools.js'
104	import { feature } from 'bun:bundle'
105	// Dead code elimination: conditional import for OVERFLOW_TEST_TOOL
106	<PATH> eslint-disable custom-rules/no-process-env-top-level, @typescript-eslint/no-require-imports */
107	const OverflowTestTool = feature('OVERFLOW_TEST_TOOL')
108	  ? require('./tools/OverflowTestTool/OverflowTestTool.js').OverflowTestTool
109	  : null
110	const CtxInspectTool = feature('CONTEXT_COLLAPSE')
111	  ? require('./tools/CtxInspectTool/CtxInspectTool.js').CtxInspectTool
112	  : null
113	const TerminalCaptureTool = feature('TERMINAL_PANEL')
114	  ? require('./tools/TerminalCaptureTool/TerminalCaptureTool.js')
115	      .TerminalCaptureTool
116	  : null
117	const WebBrowserTool = feature('WEB_BROWSER_TOOL')
118	  ? require('./tools/WebBrowserTool/WebBrowserTool.js').WebBrowserTool
119	  : null
120	const coordinatorModeModule = feature('COORDINATOR_MODE')
121	  ? (require('./coordinator/coordinatorMode.js') as typeof import('./coordinator/coordinatorMode.js'))
122	  : null
123	const SnipTool = feature('HISTORY_SNIP')
124	  ? require('./tools/SnipTool/SnipTool.js').SnipTool
125	  : null
126	const ListPeersTool = feature('UDS_INBOX')
127	  ? require('./tools/ListPeersTool/ListPeersTool.js').ListPeersTool
128	  : null
129	const WorkflowTool = feature('WORKFLOW_SCRIPTS')
130	  ? (() => {
131	      require('./tools/WorkflowTool/bundled/index.js').initBundledWorkflows()
132	      return require('./tools/WorkflowTool/WorkflowTool.js').WorkflowTool
133	    })()
134	  : null
135	<PATH> eslint-enable custom-rules/no-process-env-top-level, @typescript-eslint/no-require-imports */
136	import type { ToolPermissionContext } from './Tool.js'
137	import { getDenyRuleForTool } from './utils/permissions/permissions.js'
138	import { hasEmbeddedSearchTools } from './utils/embeddedTools.js'
139	import { isEnvTruthy } from './utils/envUtils.js'
140	import { isPowerShellToolEnabled } from './utils/shell/shellToolUtils.js'
141	import { isAgentSwarmsEnabled } from './utils/agentSwarmsEnabled.js'
142	import { isWorktreeModeEnabled } from './utils/worktreeModeEnabled.js'
143	import {
144	  REPL_TOOL_NAME,
145	  REPL_ONLY_TOOLS,
146	  isReplModeEnabled,
147	} from './tools/REPLTool/constants.js'
148	export { REPL_ONLY_TOOLS }
149	<PATH> eslint-disable @typescript-eslint/no-require-imports */
150	const getPowerShellTool = () => {
151	  if (!isPowerShellToolEnabled()) return null
152	  return (
153	    require('./tools/PowerShellTool/PowerShellTool.js') as typeof import('./tools/PowerShellTool/PowerShellTool.js')
154	  ).PowerShellTool
155	}
156	<PATH> eslint-enable @typescript-eslint/no-require-imports */
157	
158	<PATH>
159	 * Predefined tool presets that can be used with --tools flag
160	 */
161	export const TOOL_PRESETS = ['default'] as const
162	
163	export type ToolPreset = (typeof TOOL_PRESETS)[number]
164	
165	export function parseToolPreset(preset: string): ToolPreset | null {
166	  const presetString = preset.toLowerCase()
167	  if (!TOOL_PRESETS.includes(presetString as ToolPreset)) {
168	    return null
169	  }
170	  return presetString as ToolPreset
171	}
172	
173	<PATH>
174	 * Get the list of tool names for a given preset
175	 * Filters out tools that are disabled via isEnabled() check
176	 * @param preset The preset name
177	 * @returns Array of tool names
178	 */
179	export function getToolsForDefaultPreset(): string[] {
180	  const tools = getAllBaseTools()
181	  const isEnabled = tools.map(tool => tool.isEnabled())
182	  return tools.filter((_, i) => isEnabled[i]).map(tool => tool.name)
183	}
184	
185	<PATH>
186	 * Get the complete exhaustive list of all tools that could be available
187	 * in the current environment (respecting process.env flags).
188	 * This is the source of truth for ALL tools.
189	 */
190	<PATH>
191	 * NOTE: This MUST stay in sync with https://console.statsig.com/4aF3Ewatb6xPVpCwxb5nA3/dynamic_configs/claude_code_global_system_caching, in order to cache the system prompt across users.
192	 */
193	export function getAllBaseTools(): Tools {
194	  return [
195	    AgentTool,
196	    TaskOutputTool,
197	    BashTool,
198	    // Ant-native builds have bfs/ugrep embedded in the bun binary (same ARGV0
199	    // trick as ripgrep). When available, find/grep in Claude's shell are aliased
200	    // to these fast tools, so the dedicated Glob/Grep tools are unnecessary.
201	    ...(hasEmbeddedSearchTools() ? [] : [GlobTool, GrepTool]),
202	    ExitPlanModeV2Tool,
203	    FileReadTool,
204	    FileEditTool,
205	    FileWriteTool,
206	    NotebookEditTool,
207	    WebFetchTool,
208	    TodoWriteTool,
209	    WebSearchTool,
210	    TaskStopTool,
211	    AskUserQuestionTool,
212	    SkillTool,
213	    EnterPlanModeTool,
214	    ...(process.env.USER_TYPE === 'ant' ? [ConfigTool] : []),
215	    ...(process.env.USER_TYPE === 'ant' ? [TungstenTool] : []),
216	    ...(SuggestBackgroundPRTool ? [SuggestBackgroundPRTool] : []),
217	    ...(WebBrowserTool ? [WebBrowserTool] : []),
218	    ...(isTodoV2Enabled()
219	      ? [TaskCreateTool, TaskGetTool, TaskUpdateTool, TaskListTool]
```

## [tool] Read
- Call ID: `toolu_01Q7fa8WDPjPz5BNXcaWu9DS`
- Timestamp: `2026-05-23 11:32:34`

### Arguments
```json
{
  "file_path": "<PATH>\Projects\\AIlearning\\claude-code\\src\\tools\\BashTool\\bashPermissions.ts",
  "limit": 80
}
```

### Output
```
1	import { feature } from 'bun:bundle'
2	import { APIUserAbortError } from '@anthropic-ai/sdk'
3	import type { z } from 'zod/v4'
4	import { getFeatureValue_CACHED_MAY_BE_STALE } from '../../services/analytics/growthbook.js'
5	import {
6	  type AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS,
7	  logEvent,
8	} from '../../services/analytics/index.js'
9	import type { ToolPermissionContext, ToolUseContext } from '../../Tool.js'
10	import type { PendingClassifierCheck } from '../../types/permissions.js'
11	import { count } from '../../utils/array.js'
12	import {
13	  checkSemantics,
14	  nodeTypeId,
15	  type ParseForSecurityResult,
16	  parseForSecurityFromAst,
17	  type Redirect,
18	  type SimpleCommand,
19	} from '../../utils/bash/ast.js'
20	import {
21	  type CommandPrefixResult,
22	  extractOutputRedirections,
23	  getCommandSubcommandPrefix,
24	  splitCommand_DEPRECATED,
25	} from '../../utils/bash/commands.js'
26	import { parseCommandRaw } from '../../utils/bash/parser.js'
27	import { tryParseShellCommand } from '../../utils/bash/shellQuote.js'
28	import { getCwd } from '../../utils/cwd.js'
29	import { logForDebugging } from '../../utils/debug.js'
30	import { isEnvTruthy } from '../../utils/envUtils.js'
31	import { AbortError } from '../../utils/errors.js'
32	import type {
33	  ClassifierBehavior,
34	  ClassifierResult,
35	} from '../../utils/permissions/bashClassifier.js'
36	import {
37	  classifyBashCommand,
38	  getBashPromptAllowDescriptions,
39	  getBashPromptAskDescriptions,
40	  getBashPromptDenyDescriptions,
41	  isClassifierPermissionsEnabled,
42	} from '../../utils/permissions/bashClassifier.js'
43	import type {
44	  PermissionDecisionReason,
45	  PermissionResult,
46	} from '../../utils/permissions/PermissionResult.js'
47	import type {
48	  PermissionRule,
49	  PermissionRuleValue,
50	} from '../../utils/permissions/PermissionRule.js'
51	import { extractRules } from '../../utils/permissions/PermissionUpdate.js'
52	import type { PermissionUpdate } from '../../utils/permissions/PermissionUpdateSchema.js'
53	import { permissionRuleValueToString } from '../../utils/permissions/permissionRuleParser.js'
54	import {
55	  createPermissionRequestMessage,
56	  getRuleByContentsForTool,
57	} from '../../utils/permissions/permissions.js'
58	import {
59	  parsePermissionRule,
60	  type ShellPermissionRule,
61	  matchWildcardPattern as sharedMatchWildcardPattern,
62	  permissionRuleExtractPrefix as sharedPermissionRuleExtractPrefix,
63	  suggestionForExactCommand as sharedSuggestionForExactCommand,
64	  suggestionForPrefix as sharedSuggestionForPrefix,
65	} from '../../utils/permissions/shellRuleMatching.js'
66	import { getPlatform } from '../../utils/platform.js'
67	import { SandboxManager } from '../../utils/sandbox/sandbox-adapter.js'
68	import { jsonStringify } from '../../utils/slowOperations.js'
69	import { windowsPathToPosixPath } from '../../utils/windowsPaths.js'
70	import { BashTool } from './BashTool.js'
71	import { checkCommandOperatorPermissions } from './bashCommandHelpers.js'
72	import {
73	  bashCommandIsSafeAsync_DEPRECATED,
74	  stripSafeHeredocSubstitutions,
75	} from './bashSecurity.js'
76	import { checkPermissionMode } from './modeValidation.js'
77	import { checkPathConstraints } from './pathValidation.js'
78	import { checkSedConstraints } from './sedValidation.js'
79	import { shouldUseSandbox } from './shouldUseSandbox.js'
80	
```

## [tool] Glob
- Call ID: `toolu_01Y26Dcmcr2A7hUo2mNpi7R9`
- Timestamp: `2026-05-23 11:32:35`

### Arguments
```json
{
  "pattern": "src/tools/FileEditTool/*"
}
```

### Output
```
src\tools\FileEditTool\FileEditTool.ts
src\tools\FileEditTool\UI.tsx
src\tools\FileEditTool\constants.ts
src\tools\FileEditTool\prompt.ts
src\tools\FileEditTool\types.ts
src\tools\FileEditTool\utils.ts
```

## [#4] Assistant
- Timestamp: `2026-05-23 11:34:20`

# Tools 架构深入拆解

源码三个核心入口:[src/Tool.ts](src/Tool.ts)(接口定义,793 行)、[src/tools.ts](src/tools.ts)(注册中心)、[src/tools/](src/tools/)(40+ 工具实现)。下面按"由抽象到具象"剖开。

---

## 1. Tool 接口 —— Agent 工具的"完整生命周期"模型

[src/Tool.ts:362-695](src/Tool.ts#L362-L695) 这个 `Tool` interface 是整个系统的核心契约。它不只是"一个函数",而是定义了一个工具从**声明 → 校验 → 鉴权 → 执行 → 渲染 → 中断**的全生命周期。把它分成 6 组方法看:

### 组 A:身份与元数据
```
name           # API 调用名,LLM 看到的就是它
aliases?       # 重命名时的兼容别名
searchHint?    # ToolSearch 关键词匹配(给 LLM 检索用)
isMcp / isLsp  # 来源标签
shouldDefer?   # 是否延迟加载(应对工具集膨胀,见 §6)
alwaysLoad?    # 永不延迟
```

### 组 B:Schema 与输入处理
```
inputSchema    # Zod schema(LLM 调用参数的契约)
inputJSONSchema? # MCP 工具可直接给 JSONSchema
backfillObservableInput?  # 观察者看到的输入派生字段(不影响 prompt cache)
```
这里有个**关键细节**:`backfillObservableInput` 修改的是输入的副本,**绝不动原始输入** —— 因为原始 input 参与了 prompt cache key 的计算,改一个字符就会让缓存失效。这是 LLM 应用工程里常被忽视的优化点。

### 组 C:执行与控制(三个布尔特性 + 一个执行函数)
```
call(args, context, canUseTool, parentMessage, onProgress)  # 实际执行
isConcurrencySafe(input)  # 能否与其他工具并行执行
isReadOnly(input)         # 只读?(plan mode 用)
isDestructive?(input)     # 不可逆?(删除/覆盖/发送)
isOpenWorld?(input)       # 是否访问外部世界(网络等)
interruptBehavior?()      # 用户中断时:'cancel' 丢弃 | 'block' 等结果
```
**`isConcurrencySafe` 是 Agent 并行加速的关键开关**。Claude Code 把"多个独立工具调用"打包并行执行(见 [docs/architecture.md:215](docs/architecture.md#L215)),但写操作不能乱并发 —— 工具自己声明能否并行,系统据此调度。

### 组 D:权限与鉴权(两阶段)
```
validateInput?(input, context)   # 第一道:输入是否在业务规则内?(返回错误码给 LLM)
checkPermissions(input, context) # 第二道:用户是否授权?(可能弹窗)
preparePermissionMatcher?(input) # 给 hooks 用的 pattern matcher
toAutoClassifierInput(input)     # 喂给安全分类器的紧凑表示
```
两层**职责严格分离**:`validateInput` 失败 → 直接把错误反馈给 LLM 让它换参数;`checkPermissions` 失败 → 走用户授权 UI。这种分层让"模型问题"和"用户问题"不会混在一起,Agent 能正确地自我修正。

### 组 E:Prompt 文档化
```
prompt({tools, agents, ...})     # 工具的完整使用说明(注入到 system prompt)
description(input, options)      # 单次调用的简短描述
userFacingName(input)            # UI 显示名
getActivityDescription(input)    # 转 spinner 文案("Reading src/foo.ts")
```
**`prompt()` 是真正决定 LLM 怎么用这个工具的地方** —— 不是 schema,而是这段自然语言。每个工具一个 `prompt.ts` 文件,这就是为什么 [BashTool 的目录](src/tools/BashTool/) 里有 `prompt.ts`、`bashSecurity.ts`、`bashPermissions.ts` 这么多专属文件。

### 组 F:终端 UI 渲染管线(7 个 render*)
```
renderToolUseMessage          # 调用瞬间(input 还在流式传入,所以 Partial)
renderToolUseProgressMessage  # 运行中的进度
renderToolResultMessage       # 完成结果
renderToolUseRejectedMessage  # 被拒绝
renderToolUseErrorMessage     # 报错
renderToolUseQueuedMessage    # 排队中
renderGroupedToolUse          # 并行调用的聚合显示
```
每个工具自己负责"我在终端里长什么样" —— 这是 Claude Code 把 UI 跟工具耦合的代价/收益。**Web Agent 通常不会这么做**(UI 在前端,工具在后端);但终端 CLI 因为 Ink+React 全在一起,这种把"UI 渲染"作为工具的一等公民的做法极大降低了新工具的接入复杂度。

---

## 2. `buildTool` 工厂 —— 默认值收敛(优雅的 fail-closed)

看 [src/Tool.ts:783-792](src/Tool.ts#L783-L792):

```typescript
export function buildTool<D extends AnyToolDef>(def: D): BuiltTool<D> {
  return {
    ...TOOL_DEFAULTS,          // 安全默认值
    userFacingName: () => def.name,
    ...def,                    // 覆盖
  } as BuiltTool<D>
}
```

默认值哲学是**"对错误一侧 fail-closed"**:
- `isConcurrencySafe: false` ← 默认不并行,要安全先证明
- `isReadOnly: false` ← 默认认为会写
- `isDestructive: false` ← 但破坏性要显式声明
- `toAutoClassifierInput: ''` ← 默认跳过安全分类(显式承认风险)

这是好的 Agent 工具系统设计原则:**新工具忘记声明时,不会成为安全漏洞**。

---

## 3. 单工具的目录结构 —— BashTool 范例

看 [src/tools/BashTool/](src/tools/BashTool/) 这 15 个文件就懂 Claude Code 工具的"工程组织":

```
BashTool.ts              主入口(implements Tool)
prompt.ts                给 LLM 看的工具说明书
bashPermissions.ts       权限决策(配合 classifier)
bashSecurity.ts          AST 级安全检查(parse bash → 检测危险模式)
shouldUseSandbox.ts      何时启用沙箱
pathValidation.ts        路径约束(防止越权访问)
sedValidation.ts         sed 命令特殊校验(因为它能编辑文件)
modeValidation.ts        permission mode 校验
commandSemantics.ts      命令语义分析
destructiveCommandWarning.ts  破坏性命令警告
```

**关键洞察**:一个看似"运行 shell 命令"的简单工具,实际上包含了 **bash AST 解析** + **安全分类器** + **沙箱判定** + **路径约束** + **sed 特殊处理**。Agent 工具的复杂度不在执行,而在**"在不可信输入下保持安全和确定性"**。

学 Agent 开发可以反复研读 [bashSecurity.ts](src/tools/BashTool/bashSecurity.ts) 和 [bashPermissions.ts](src/tools/BashTool/bashPermissions.ts) —— 这是工业级"LLM 调命令行"的标准答案。

---

## 4. 注册中心 —— [src/tools.ts](src/tools.ts) 的条件加载魔法

[src/tools.ts:14-52](src/tools.ts#L14-L52) 大量条件 `require`:

```typescript
const SleepTool = feature('PROACTIVE') || feature('KAIROS')
  ? require('./tools/SleepTool/SleepTool.js').SleepTool : null
const cronTools = feature('AGENT_TRIGGERS') ? [...] : []
const RemoteTriggerTool = feature('AGENT_TRIGGERS_REMOTE') ? ... : null
```

工具集是**按编译时 feature flag + 运行时环境**动态拼装的。再看 [src/tools.ts:193-220](src/tools.ts#L193-L220) 的 `getAllBaseTools()`:工具进入注册表的条件可以是:
- `feature('FLAG')` 编译时开关 → Bun 死代码消除
- `process.env.USER_TYPE === 'ant'` 内部员工独享
- `isTodoV2Enabled()` 运行时 GrowthBook 实验组
- `hasEmbeddedSearchTools()` 二进制能力探测

**收益**:同一份代码可以编出"开源版/内部版/实验版/精简版",而**不会把无关工具的 schema 喂给 LLM**(浪费 token + 干扰决策)。这对 Agent 开发是一个反复出现的实际问题。

---

## 5. ToolUseContext —— 工具运行时的"宿主环境"

[src/Tool.ts:158-300](src/Tool.ts#L158-L300) 的 `ToolUseContext` 是每次工具调用都会收到的一个**巨型上下文对象**。挑几个关键字段:

| 字段 | 作用 |
|------|------|
| `abortController` | 中断信号(用户按 ESC 时整条传播) |
| `getAppState() / setAppState()` | 工具可读写全局状态 |
| `readFileState: FileStateCache` | 文件状态缓存(防止 Read→Edit race) |
| `messages` | 完整对话历史(工具可看上下文) |
| `setToolJSX` | 工具可往 UI 推 React 节点 |
| `requestPrompt` | 工具可中途问用户(如 `AskUserQuestion`) |
| `mcpClients / mcpResources` | MCP 连接句柄 |
| `agentId / agentType` | 是哪个 sub-agent 在调用 |
| `queryTracking` | 调用链追踪(Agent 嵌套深度) |
| `localDenialTracking` | 异步子 Agent 的拒绝计数 |
| `contentReplacementState` | 工具结果预算管理 |

**这是 Agent 工具与宿主系统的耦合面**。注意它**不只是"输入参数"**,而是"我跟整个 Agent runtime 的双向接口"。设计自己 Agent 时,这个 Context 对象的设计直接决定工具能做多复杂的事。

---

## 6. ToolSearch —— 应对工具集爆炸的"延迟加载"

40+ 工具全塞进 system prompt,token 成本巨大且会干扰 LLM 决策。解法在 [src/Tool.ts:441-449](src/Tool.ts#L441-L449):

```typescript
shouldDefer?: boolean   // 不放进初始 prompt
alwaysLoad?: boolean    // 强制放进初始 prompt
searchHint?: string     // 提供关键词供检索
```

延迟工具只暴露**名字**(在 system-reminder 里),LLM 想用时必须先调 `ToolSearch` 工具拿 schema 才能调用。

你这次对话开头就看到了:
```
<system-reminder>
The following deferred tools are now available via ToolSearch...
CronCreate, CronDelete, WebFetch, TodoWrite, ...
</system-reminder>
```

**这是工业级 Agent 处理"工具长尾"的标准方案** —— 类似计算机里的"二级页表":常用工具直接拿,冷门工具按需检索。

---

## 7. 权限决策的 5 阶段流水线

把权限检查拆成多个独立切面,执行顺序大致是:

```
LLM 发起 tool_use
   ↓
1. validateInput(input, ctx)       ← 工具自己的输入合法性
   ↓ (失败 → 错误反馈给 LLM)
2. PreToolUse hooks                 ← 用户配置的 hook(可改 input/拒绝)
   ↓
3. checkPermissions(input, ctx)    ← 工具自己的权限逻辑
   ↓
4. 通用 permission 规则匹配         ← alwaysAllow/Deny/Ask 规则
   ↓ (Ask → 弹 UI 给用户)
5. canUseTool(...)                  ← 最终用户决策(REPL 或 Bridge 路由)
   ↓
6. call(args, ctx, canUseTool, ...) ← 真正执行
   ↓
7. PostToolUse hooks                ← 后处理
```

**为什么分这么多层?** 因为每层的"语义"和"失败处理"不一样:
- validateInput 失败 → 告诉 LLM 改参数(model-facing)
- checkPermissions 失败 → 弹 UI 让用户决策(user-facing)
- hooks 失败 → 系统级强约束(policy-level)

对 Agent 开发者:**不要把所有检查塞进同一个 if-else**,把它分成"模型可修复 / 用户可授权 / 系统硬约束"三层,系统能优雅得多。

---

## 8. 同构抽象:MCP 工具 / LSP 工具 / Agent 工具 Tool

注意接口里这些字段:
```typescript
isMcp?: boolean
isLsp?: boolean
mcpInfo?: { serverName; toolName }
isTransparentWrapper?(): boolean  # REPL 这种"透明转发"的工具
```

**所有外部能力 —— MCP server 的工具、LSP server 的能力、子 Agent(`AgentTool`)、技能(`SkillTool`)、工作流(`WorkflowTool`)—— 都被包装成同一个 `Tool` 接口**。

这是 Claude Code 工具系统最值得学的设计:**把"调用外部服务"和"内置功能"统一成同一个抽象**。LLM 不知道也不需要知道 `BashTool` 是本地的、`mcp__github__create_issue` 是远程的、`AgentTool` 是会递归调 LLM 的 —— 它们调用方式完全一样。

---

## 推荐你接下来读什么

1. [src/Tool.ts](src/Tool.ts) —— 通读一遍,作为工具系统的"宪法"
2. [src/tools/FileEditTool/FileEditTool.ts](src/tools/FileEditTool/FileEditTool.ts) —— 中等复杂度,看一个"完整、典型"的工具
3. [src/tools/BashTool/](src/tools/BashTool/) —— 看复杂工具如何拆分
4. [src/tools/AgentTool/](src/tools/AgentTool/) —— 看"工具调用 = spawn 子 Agent"的实现
5. [src/tools/ToolSearchTool/](src/tools/ToolSearchTool/) —— 看延迟加载机制怎么落地

要不要我接着拆 `AgentTool`(子 Agent 怎么 spawn)或者 `QueryEngine` 的 tool-call loop(LLM 怎么和这堆工具往返)?这两个是 Agent 架构最值得继续深入的地方。
