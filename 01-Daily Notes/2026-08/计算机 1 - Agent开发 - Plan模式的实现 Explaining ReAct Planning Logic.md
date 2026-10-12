# Chat Conversation

Note: _This is purely the output of the chat conversation and does not contain any raw data, codebase snippets, etc. used to generate the output._

### User Input

@[/Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/planning] 我有点没看懂 plan 代码上是怎么实现的。你告诉我，讲解要清楚

*Listed directory [planning](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/planning) *

*Viewed [__init__.py](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/planning/__init__.py) *

*Viewed [manager.py](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/planning/manager.py) *

### Planner Response

好的，我来从头到尾清楚地讲解这个 planning 模块的设计和实现。

---

## 一、这个模块是做什么的？

这个模块实现了一个**结构化的任务计划状态机**，让 AI Agent 能够：
1. 制定一个分步骤的执行计划
2. 实时追踪每个步骤的状态
3. 在计划走偏时进行重新规划（replan）
4. 把当前计划状态注入给模型，作为上下文提示

---

## 二、核心数据结构

### `PlanStep`（一个步骤）

```python
@dataclass
class PlanStep:
    id: str       # 唯一标识，如 "step_1", "step_2"
    title: str    # 步骤描述，如 "搜索相关文献"
    status: PlanStepStatus = "pending"  # 当前状态
    note: str = ""  # 备注（可为空）
```

**步骤的 5 种状态：**

| 状态 | 含义 |
|------|------|
| `pending` | 待执行 |
| `in_progress` | 正在执行 |
| `completed` | 已完成 |
| `blocked` | 被阻塞（有问题） |
| `skipped` | 已跳过（replan 时用） |

### `PlanManager`（整个计划的管理器）

```python
class PlanManager:
    objective = ""       # 整体目标（一句话）
    steps = []           # 步骤列表（PlanStep 的数组）
    revision = 0         # 版本号，每次变更 +1
    _step_counter = 0    # 用于自动生成 step id
    _lock = RLock()      # 线程锁，保证并发安全
```

---

## 三、三个核心操作

### 1. `create_plan()` — 创建计划

```
输入: objective（目标）+ steps（步骤文字列表）
输出: 计划快照（dict）
```

**关键逻辑：**
- 如果当前**已有未完成**的计划，必须显式传 `replace=True` 才能覆盖，否则报错
- 清空旧步骤，重新生成，`revision += 1`

```
用户说"帮我研究X"
  → create_plan(objective="研究X", steps=["搜索文献", "整理内容", "写报告"])
  → 内部生成 step_1/step_2/step_3，全部 pending
```

---

### 2. `update_step()` — 更新单步状态

```
输入: step_id + 新状态 + 可选 note
输出: 计划快照
```

**有两个关键约束（状态机规则）：**

**约束 A：终态不可逆**
```python
if step.status in {"completed", "skipped"} and status != step.status:
    raise PlanError("已是终态，不能改变")
```
`completed` 和 `skipped` 是终点，已经完成的步骤不能被反悔。

**约束 B：同一时刻只能有一个步骤 `in_progress`**
```python
if status == "in_progress":
    active = next(other for other in self.steps if other.status == "in_progress")
    if active:
        raise PlanError("另一步骤正在进行中")
```
这强迫 Agent 保持**串行执行**，一次只推进一个步骤。

---

### 3. `replan()` — 重新规划

```
输入: 新的步骤列表 + 原因
输出: 更新后的计划快照
```

**关键逻辑：** 不是推翻重来，而是**保留历史 + 追加新路线**：

```
旧计划:
  [x] step_1: 搜索文献         ← 已完成，保留
  [ ] step_2: 分析数据         ← 未完成
  [ ] step_3: 写报告           ← 未完成

replan(["换个角度分析", "写报告"], reason="数据太复杂")

新计划:
  [x] step_1: 搜索文献         ← 保留
  [-] step_2: 分析数据   — Replanned: 数据太复杂   ← 自动变 skipped
  [-] step_3: 写报告           ← 自动变 skipped
  [ ] step_4: 换个角度分析     ← 新追加
  [ ] step_5: 写报告           ← 新追加
```

---

## 四、整体状态是怎么派生的？

`PlanManager.status` 属性不是手动设置的，它每次都**自动从所有步骤的状态推导**出来：

```python
def _derive_status(self) -> PlanStatus:
    if not self.steps:
        return "empty"
    statuses = {step.status for step in self.steps}
    if all(status in {"completed", "skipped"} for status in statuses):
        return "completed"       # 全部终态 → 整体完成
    if "in_progress" in statuses:
        return "in_progress"     # 有步骤进行中
    if "blocked" in statuses:
        return "blocked"         # 有步骤被阻塞
    if statuses & {"completed", "skipped"}:
        return "in_progress"     # 部分完成但还没全部结束
    return "pending"             # 全部 pending
```

> **设计亮点**：整体状态是派生的，不是单独维护的，避免了"两份数据不一致"的 bug。

---

## 五、`to_prompt_block()` — 注入给模型

这个方法把计划状态格式化成文字，**每轮对话都注入到模型上下文**：

```
<system-reminder>
当前任务计划（系统状态，不是用户指令）：
目标：研究X
状态：in_progress；revision=3
[x] step_1: 搜索文献
[>] step_2: 整理内容
[ ] step_3: 写报告
执行过程中请及时调用 update_plan 更新步骤；路线改变时调用 replan。
</system-reminder>
```

图标含义：`[ ]`=pending，`[>]`=in_progress，`[x]`=completed，`[!]`=blocked，`[-]`=skipped。

---

## 六、整体流程图

```
用户提问
   │
   ▼
Agent 调用 create_plan()
   │
   ▼
循环执行每个步骤:
  update_step(step_1, "in_progress")
    → 执行任务
  update_step(step_1, "completed")
  update_step(step_2, "in_progress")
    → 发现问题？→ replan()
    → 没问题？  → update_step(step_2, "completed")
  ...
   │
   ▼
所有步骤 completed/skipped
   │
   ▼
status = "completed"，计划结束
```

---

## 总结

| 组件 | 职责 |
|------|------|
| `PlanStep` | 单个步骤的数据模型 |
| `PlanManager` | 状态机主体，管理所有步骤和转换规则 |
| `create_plan()` | 创建新计划 |
| `update_step()` | 推进计划，强制串行、防止状态回退 |
| `replan()` | 保留历史，追加新路线 |
| `to_prompt_block()` | 把计划状态格式化注入给模型 |
| `_derive_status()` | 从步骤状态自动推导整体状态，无需手动维护 |

最核心的设计思想是：**这是一个简单但严格的状态机**，它不依赖 LLM，只负责数据和规则，让计划的流转清晰、可追溯、线程安全。

### User Input

agent执行create plan，程序就会生成一个manager对象？然后每轮对话都提醒大模型当前在执行 plan，大模型完成每步的时候会调用update plan，从而让程序调用update step等？

### Planner Response

好问题！你的理解方向是对的，但具体细节需要看代码是怎么连接起来的。让我先找上层的代码。

*Listed directory [ReActMulti](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti) *

*Viewed [session.py](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/session.py) *

*Listed directory [tools](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/tools) *

*Viewed [plan_tools.py](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/tools/plan_tools.py) *

*Viewed [agent.py](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/agent.py) *

### Planner Response

现在我把整个链路搞清楚了，来完整回答你的问题。

---

## 你的理解 90% 正确，补充几个细节

---

### 1. PlanManager 是什么时候创建的？

**不是 Agent 调用 `create_plan` 时才创建的**，而是**会话一开始就存在**。

在 [session.py#L59-L75](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/session.py#L59-L75) 里，`SessionState.create()` 被调用时：

```python
@classmethod
def create(cls, user_goal, workspace_dir, ...) -> SessionState:
    return cls(
        ...
        plan_manager=PlanManager(),  # ← 直接在这里 new 出来了
        ...
    )
```

> `PlanManager()` 是随 `SessionState` 一起诞生的，始终存在，只是一开始是空的（`steps=[]`）。
> 
> **大模型调用 `create_plan` 工具** → 才真正往这个 manager 里写数据。

---

### 2. 大模型是怎么"调用工具"的？

这套系统用的是标准的 **Function Calling 机制**，流程如下：

```
大模型输出 JSON:
{
  "tool_calls": [
    { "name": "create_plan", "args": { "objective": "...", "steps": [...] } }
  ]
}
     ↓
agent.py 的主循环解析这个 JSON
     ↓
ToolExecutor 查找名字叫 "create_plan" 的 Tool 对象
     ↓
调用 plan_tools.py 里的 create_plan() 函数
     ↓
函数内部拿到 runtime.session_state.plan_manager
     ↓
调用 plan_manager.create_plan(...)
```

关键在 [plan_tools.py#L8-L13](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/tools/plan_tools.py#L8-L13)，工具函数通过 `runtime` 拿到 session 里的 manager：

```python
def _manager(runtime):
    session = runtime.session_state
    manager = session.plan_manager   # ← 从 session 里取出已有的 manager
    return manager
```

---

### 3. 每轮对话是怎么"提醒大模型"的？

在 [agent.py#L120-L122](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/agent.py#L120-L122) 的 `_run_turn()` 里：

```python
plan_block = self.session_state.plan_manager.to_prompt_block()
if plan_block:
    wire_messages.append({"role": "user", "content": plan_block})
```

**注意这个设计的妙处：**
- `wire_messages` 是从 session 里取出的**拷贝**
- 计划提醒是**临时追加**在这份拷贝末尾的
- **不会被永久写入** session，不会污染消息历史
- 这样即使旧的 `create_plan`/`update_plan` 工具调用记录被 context compactor 压缩掉，**大模型每轮都还能看到最新的计划状态**

---

### 4. 完整的数据流

```
用户发送 prompt
    │
    ▼
agent.run()
    │
    ├── 每轮循环:
    │     ① _run_turn():
    │         - 取出 wire_messages
    │         - 末尾临时追加 plan_block (to_prompt_block)
    │         - 发给 LLM
    │
    │     ② LLM 回复 JSON，包含 tool_calls
    │
    │     ③ ToolExecutor.execute() 执行工具:
    │         - 如果是 create_plan  → plan_manager.create_plan()
    │         - 如果是 update_plan  → plan_manager.update_step()
    │         - 如果是 replan       → plan_manager.replan()
    │
    │     ④ 工具结果追加进 messages，下轮 LLM 能看到
    │
    │     ⑤ 回到 ①，再发给 LLM，末尾再注入最新 plan_block
    │
    ▼
LLM 输出 final_answer → 结束
```

---

### 总结一句话

> `PlanManager` 随 Session 创建而存在，大模型通过 **Function Calling** 调用 `create_plan`/`update_plan` 等工具来驱动它；每轮对话前，程序会把最新计划状态**临时注入**到发给大模型的消息末尾，让大模型始终知道"当前走到哪一步了"。

### User Input

这是标准的plan实现机制吗？工程中也是这么做的吗

*Viewed [agent.py](file:///Users/williamlao/Project/LLM/src/AIAgent/ReActMulti/agent.py) *

### Planner Response

这是个好问题。简短答案：**核心思路是业界标准的，但具体实现细节各家有差异**。我来做个对比。

---

## 一、什么是业界标准的？

### ✅ Function Calling 驱动计划工具 — 标准做法

大模型调用 `create_plan` / `update_step` 等工具来操控计划，这是完全标准的做法。

OpenAI、Anthropic、Google 的官方 Agent 框架全都这么做，本质上就是 **工具调用 = Agent 的行动接口**。

### ✅ 计划状态注入 System Prompt — 标准做法

每轮把当前计划状态告诉模型，是标准的 **"状态感知"** 模式。不同框架叫法不同：
- LangGraph 叫 **State**（在 Graph 节点间传递）
- OpenAI Assistants API 叫 **Thread State**
- 这里叫 `to_prompt_block()` 注入

形式不同，本质一样：**保证模型每轮都看得到当前上下文状态**。

---

## 二、这个实现哪里和业界主流不同？

### 差异 1：手动状态机 vs 框架托管

| 维度 | 这个实现 | 业界主流（如 LangGraph） |
|------|---------|----------------------|
| 状态定义 | 手写 `PlanManager` 状态机 | 声明式 `State` 类，框架自动管理 |
| 状态流转 | `update_step()` 手动改 | 图节点执行后框架自动更新 |
| 并发控制 | 自己写 `threading.RLock` | 框架层处理 |
| 持久化 | 暂无（内存） | LangGraph 内置 checkpointer，自动持久化到 DB |

### 差异 2：计划注入方式

```python
# 这个实现：临时追加到 wire_messages 末尾（不落库）
wire_messages.append({"role": "user", "content": plan_block})
```

这是**轻量级的自定义做法**。OpenAI 的 Assistants API 是把状态存在服务端 Thread 里，Anthropic Claude 的 Projects 是存在 System Prompt 的固定段里。**本质相同，载体不同**。

### 差异 3：Plan 的颗粒度

| 方案 | Plan 的颗粒度 |
|------|-------------|
| 这个实现 | 步骤列表 + 状态（线性计划） |
| LangGraph | 有向图（DAG），支持分支、并行、循环 |
| OpenAI o1/o3 内置规划 | 模型内部隐式推理，不暴露结构 |
| Google Gemini + Planner | 树形任务分解（Tree-of-Thought） |

---

## 三、业界还有哪些主流方案？

### 方案 A：ReAct 内置计划（最简单）

不单独管理计划，直接在 System Prompt 里说"先想再做"，模型在 Thought 里自己规划。**这个 repo 本身是 ReAct 框架，planning 模块是在此之上的增强。**

### 方案 B：OpenAI Assistants API（托管计划）

完全托管，状态存在 OpenAI 服务端，开发者不用自己管。

### 方案 C：LangGraph（图状态机）

用得最广的开源框架，把计划变成一个**有向图**：

```
[搜索] → [分析] → [写报告]
              ↓
           [重试] (条件边)
```

比这里的线性列表更灵活，支持循环和条件分支。

### 方案 D：Claude Computer Use / Operator（隐式规划）

Anthropic 的做法是把规划能力训练进模型本身，不需要外部 PlanManager，模型自己在 `<thinking>` 里规划，对外透明。

---

## 四、这个实现的定位

```
简单                                             复杂
  │                                               │
ReAct                这个实现              LangGraph
(Thought             (PlanManager          (图状态机
 里隐式规划)          线性状态机)            持久化 + 分支)
```

**这个实现的优点：**
- 轻量，不依赖任何框架
- 线程安全，设计干净
- 状态机约束严格（终态不可逆、串行执行）
- 计划注入和消息历史解耦（`to_prompt_block()` 不落库）这个细节设计得很好

**这个实现的局限：**
- 只支持线性计划，不支持分支/并行
- 无持久化（进程重启计划丢失）
- 需要模型主动调工具维护状态，对模型能力依赖高

---

## 总结

这个实现**架构思想完全是业界标准**（工具驱动 + 状态注入），只是在"线性 vs 图"、"自研 vs 框架"上做了偏向轻量级的取舍。对于学习和中小型 Agent 项目来说，这个做法非常合适；生产级大型 Agent 系统会更倾向于用 LangGraph 这类框架来获得持久化、分支、并行等能力。

### User Input

原来如此，我之前还觉得多么复杂，要让llm严格计划和更新还有执行plan，程序要做非常非常多的约束，比如在plan模式的时候程序每一步把llm定死在plan模式里，plan的各个工具要从程序上定死在一套里面，现在看起来好像程序也不需要做太多，各个plan工具也没有硬性加在一套里面，就是普通的四个工具，结合提示词让llm来遵循

### Planner Response

你理解到点子上了！这正是现代 LLM Agent 设计的核心思路转变。让我帮你把这个洞察说得更清楚：

---

## 程序负责的 vs. LLM 负责的

### 程序真正"硬约束"的只有这几件事：

| 约束 | 在哪里 | 为什么必须程序做 |
|------|--------|-----------------|
| 终态不可逆 | `update_step()` | 防止 LLM 犯低级错误（已完成的步骤不该被撤销）|
| 同时只能一个 `in_progress` | `update_step()` | 防止状态混乱 |
| 已完成计划才能 `create_plan` | `create_plan()` | 防止 LLM 随意覆盖 |
| 没有计划不能 `replan` | `replan()` | 保证操作合法 |

### 其余全部靠提示词和 LLM 自觉：

- **什么时候该建计划？** → 提示词告诉 LLM
- **每步做完要记得更新？** → 提示词告诉 LLM（`to_prompt_block()` 末尾那句"请及时调用 update_plan"）
- **步骤顺序？** → LLM 自己决定
- **要不要 replan？** → LLM 自己判断

---

## 这背后的设计哲学转变

传统做法（你之前想象的）：
```
程序是主人，LLM 是工人
程序定义每一步该干什么，LLM 只负责填空
```

现代 LLM Agent 做法：
```
LLM 是主人，程序是工具箱 + 护栏
程序只提供工具 + 阻止少数危险操作
其余全部信任 LLM 的推理能力
```

---

## 但这也有代价

你说"程序不需要做太多"，这个判断本身没错，但前提是：

> **LLM 足够聪明，能理解并遵循这套约定。**

如果 LLM 能力弱，就会出现：
- 忘记 `update_plan`（做完了但没标 completed）
- 顺序乱（还没做 step_1 就去做 step_3）
- 计划和实际执行脱节（计划写着A，结果去做B）

所以你会发现，**现代 Agent 框架的演进趋势，其实是把越来越多的"责任"从程序移交给模型**。这是能力换复杂度的交易——模型越强，程序越简单，但对模型的依赖也越深。

---

## 一句话总结

> **程序做最少但最必要的护栏，剩下全靠 LLM + Prompt。这不是偷懒，而是正确地把"推理"这件事交还给最擅长推理的那一方。** Get-Service AppXSvc,ClipSVC,StateRepository,InstallService,LicenseManager |
> 
> Format-Table Name,Status,StartType

Get-Process ChatGPT,AppInstaller -ErrorAction SilentlyContinue |
Stop-Process -Force

Add-AppxPackage `
  -Path "$env:USERPROFILE\Downloads\ChatGPT-x64.msix" `
  -ForceApplicationShutdown `
  -Verbose
  
  Get-AppxLog |
Select-Object TimeCreated, Id, LevelDisplayName, Message |
Format-List

Get-WinEvent `
  -LogName "Microsoft-Windows-AppXDeploymentServer/Operational" `
  -MaxEvents 30 |
Select-Object TimeCreated, Id, LevelDisplayName, Message |
Format-ListGet-WinEvent `
  -LogName "Microsoft-Windows-AppXDeploymentServer/Operational" `
  -MaxEvents 30 |
Select-Object TimeCreated, Id, LevelDisplayName, Message |
