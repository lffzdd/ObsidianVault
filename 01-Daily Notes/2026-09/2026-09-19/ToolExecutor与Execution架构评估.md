# 评估ToolExecutor与Execution架构

> 现在ToolExecutor名不副实，只是调度者，Execution才是真正的执行者，但是把执行者传给Tool，虽然是通过runtime，但是这个方向没问题吗，这个架构没问题吗

<details><summary>4 previous messages</summary>

> 我先沿着 `ToolExecutor → runtime → Tool → Execution` 的实际调用链看一下，重点判断执行能力的归属和依赖方向是否合理，不改代码。
>
> <details><summary>Explored 4 files, 3 searches, 2 lists, ran 2 commands</summary>
>
> <details><summary>Explored a search, a list</summary>
>
> - Searched for `ExecutionBackend|ToolExecutor|AuthorizedExecution|PreparedInvocation` in `MEMORY.md`
> - Listed files in `src`
>
> </details>
>
> - Ran `sed -n '1,48p' /Users/williamlao/.codex/memories/MEMORY.md; pwd; rg --files -g AGENTS.md -g '*execution*' -g '*permission*' -g '*base*' src/wright`
>
> - Ran `git status --short; sed -n '1,260p' src/wright/tool_capabilities.py; sed -n '1,260p' src/wright/engine/executor.py`
>
> <details><summary>Explored 4 files, 2 searches, a list</summary>
>
> - Listed files in `execution`
> - Read `./src/wright/engine/executor.py`
> - Searched for `execution|tool.call|def ` in `executor.py`
> - Read `./src/wright/tools/base.py`
> - Searched for `runtime.execution|def call|def describe_access` in `file_tools.py`
> - Read `./src/wright/execution/base.py`
> - Read `./src/wright/execution/authorized.py`
>
> </details>
>
> </details>
>
> 调用链已经清楚了：Tool 收到的是本次调用专属的 `AuthorizedExecution`，底层 backend 由 `ToolExecutor` 持有。关键要区分“执行一次工具调用”和“执行文件／进程操作”这两个层次；目前它们的分工比名字表达得更清楚。
>
> <details><summary>Explored 5 files, 3 searches</summary>
>
> - Read `./src/wright/execution/protocols.py`
> - Read `./src/wright/tools/file_tools.py`
> - Read `./src/wright/engine/executor.py`
> - Searched for `^(async )?def test_|^class Test` in `test_plan2_execution.py`
> - Read `./src/wright/tools/base.py`
> - Read `./src/wright/execution/protocols.py`
> - Searched for `^def |^class |describe_access|InvocationGrant` in `resolver.py`
> - Searched for `subprocess|Path\(|open\(|execution|Authorized` in `file_tools.py`
>
> </details>
</details>

**这个依赖方向是合理的，但命名确实容易让人误解。** 看了当前代码，我不认为“把 execution 通过 runtime 交给 Tool”本身是架构问题。

你说 `ToolExecutor` 主要在调度，这个观察对；不过这里有两个不同层次的“执行”：

| 组件                    | 实际职责                                   |
| --------------------- | -------------------------------------- |
| `ToolExecutor`        | 管理一次工具调用：校验、组织授权、并发、超时、日志、调用 Tool、收集结果 |
| `Tool`                | 实现具体工具语义，例如如何编辑文件、如何组织搜索结果             |
| `AuthorizedExecution` | 检查本次调用可以进行哪些操作、访问哪些目标                  |
| ´`ExecutionBackend`   | 完成实际的文件读写、搜索、进程启动                      |

**Tool 需要的是完成工作所需的受限能力。给它这些能力，并没有把调用管理权交给它。** 类似给业务服务传一个数据库接口：业务服务决定业务步骤，数据库接口负责实际存取。

当前代码也确实按这个方向实现：

- [`ToolExecutor`](./src/wright/engine/executor.py:363) 根据本次调用的 grant 创建 `AuthorizedExecution`，放入 runtime。
- 随后调用 [`tool.call(arguments, runtime)`](./src/wright/engine/executor.py:448)，调用结束时关闭这份执行能力。
- Tool 通过 [`AuthorizedExecution`](./src/wright/execution/authorized.py:46) 访问 backend，文件操作会检查操作类型、目标路径和授权范围。

例如编辑文件，Tool 负责“检查旧内容、定位替换、生成新内容”；execution 负责“读取和写入指定文件”。如果为了让 `ToolExecutor` 成为唯一的“执行者”，把这些逻辑都搬进去，它就需要理解每一种工具，反而破坏扩展边界。

**但合理的关键不是“经过 runtime”，而是传进去的对象有什么权力。** runtime 只是载体。这里应始终保持：

- Tool 获得本次调用的受限能力，不能自行扩大授权。
- backend 不需要认识 Tool、Agent 或工具注册表。
- 工具语义留在 Tool，环境操作留在 backend，调用生命周期留在调度层。

当前主链符合这个分工。不过，这属于可信工具代码之间的权限约束，并不构成对任意 Python 插件代码的沙箱。

我认为值得改进的是两个表达问题：

1. **`ToolExecutor` 可以叫 `ToolRunner` 或 `ToolInvocationRunner`。** 它也负责调用生命周期，所以“只是调度者”略微低估了它的职责；原名并非错误，只是和 `ExecutionBackend` 放在一起不够清楚。
2. **Tool 面向的执行接口可以更明确。** 当前 [`ToolRuntime.execution`](./src/wright/tools/base.py:45) 直接标注具体类 `AuthorizedExecution`。以后可抽出工具侧的能力 Protocol，由授权包装器实现；这能减轻实现耦合，但现在不需要为了形式再增加一层。

所以，我支持保留当前的责任分工。需要澄清的是“谁管理工具调用、谁实现工具语义、谁提供环境操作”，没有理由仅因为执行能力传给了 Tool 就反转调用方向。本次只检查了源码，没有修改或运行测试。

> 你说ToolExecutor不只是orchestra，也负责了整个周期，这个确实，如果把它改名了，这个架构就是正确清晰的了吗，还有因为backend我看成是执行的角色而且在executor里它排在tool签名，所以老是下意识认为它在tool上层，但实际上确实它就是在tool下层，这个没问题，execution才在tool上层？层级搞得我有些混乱了

**改名能减少误解，但不能单靠改名证明架构正确。你现在混淆的核心，是把“谁创建、持有对象”和“谁依赖谁”当成了同一种层级。**

还有，我上一条需要说得更明确：**如果你说的 `execution` 是 `runtime.execution`，也就是 `AuthorizedExecution`，它同样在 Tool 下层。** 在 Tool 上层的是管理调用生命周期的 `ToolExecutor`。

可以先沿着一次实际调用来看：

```text
ToolExecutor                  管理这次工具调用
    ↓ 调用
Tool                          实现工具语义
    ↓ 使用
AuthorizedExecution           检查本次授权，转发操作
    ↓ 使用
ExecutionBackend              提供环境操作接口
    ↑ 实现
LocalExecutionBackend         完成本地文件、进程操作
```

这里的“上下层”表示调用和服务关系：上面的组件使用下面提供的能力。最后的“实现”箭头另有含义：`LocalExecutionBackend` 实现 `ExecutionBackend` 接口。

**为什么 backend 明明由 executor 持有、传入，却仍然在 Tool 下层？**

因为这是另一条关系——**组装关系**。当前 `ToolExecutor` 同时承担了部分组装工作：

1. 持有 backend。
2. 取得本次调用的 grant。
3. 用 backend 和 grant 创建 `AuthorizedExecution`。
4. 将它放入 runtime，交给 Tool 使用。

“谁把依赖传给 Tool”与“Tool 使用谁提供的服务”完全可以不同。比如程序启动时创建数据库连接并传给业务服务，数据库仍然是业务服务使用的底层能力。构造参数的位置、对象的创建顺序，都不代表架构层级。

还有一个容易造成错觉的点：**`AuthorizedExecution` 能拒绝 Tool 的操作，不意味着它在 Tool 上层。** 它是在底层接口处守住授权边界。数据库也可以拒绝业务层的越权查询，但数据库不会因此变成业务层的上层。

至于“改名后是不是就正确清晰了”，我的判断是：

- **责任分工本身合理。** 调用生命周期、工具语义、授权检查、环境操作，这几个职责有明确归属。
- **改名能让这种分工更容易被读出来。** 比如 `ToolInvocationRunner` 比 `ToolExecutor` 更明确地指向“运行一次工具调用”。
- **仍有一个独立的耦合点：当前 executor 兼做调用管理和依赖组装。** 这不等于架构错误，但它解释了为什么你在 executor 里会同时看到 session、backend、runtime、权限组件。以后若组装逻辑继续膨胀，可以将它移到专门的组装位置，Runner 继续管理调用生命周期。

你可以固定用这三个问题判断位置：**谁管理这次调用？Runner。谁定义工具怎么工作？Tool。谁提供受限的环境操作？AuthorizedExecution 和 backend。** `execution` 这个词覆盖了不同层次的工作，才让它们看起来像在争夺同一个“执行者”角色。

> executor这个角色，现在承担的工作有点不单一，是不是要分离开来，做两个角色出来一起去承担

:codex-annotation{index="1"} **是，值得把“依赖组装”和“调用生命周期管理”分开。但应当分成两个职责，不一定要新建两个长期存在的对象。**

当前 `ToolExecutor` 同时回答了两类问题：

- **这个运行环境怎么搭起来？** 从 Session、Services 中提取能力，选择 backend，准备共享资源和回调。
- **这次工具调用怎么完成？** 校验参数、组织授权、调度执行、处理取消、记录结果、释放本次调用的资源。

前者通常发生在创建或绑定 Run 时；后者发生在每次工具调用时。**它们的变化原因和生命周期不同，这是值得拆分的依据。**

我建议保留两个明确角色：

| 角色 | 负责什么 |
|---|---|
| 应用层组装逻辑 | 根据 Session、Services、配置，准备 backend、基础能力、资源和绑定好的回调 |
| `ToolInvocationRunner` | 使用这些依赖，管理每次工具调用从准备到结束的完整流程 |

但有一个关键边界：**不要把所有“创建对象”的代码都搬到组装层。**

例如，根据本次 grant 创建 `AuthorizedExecution`、生成本次 `ToolRuntime`，以及结束后关闭授权包装器，都属于**本次调用的生命周期**，应该继续由 Runner 管理。只有 Runner 此时知道最终授权、调用身份和取消信号。

因此可以这样划分：

```text
应用层组装
    准备 backend、基础能力、共享资源、回调
    ↓ 注入
ToolInvocationRunner
    校验 → 授权 → 创建本次 runtime → 调用 Tool → 收尾
```

当前已经有 `assemble_tool_capabilities()`，所以不需要凭空再造一套工厂体系。可以沿着现有入口，把它对 Session、Services 的解释和组装工作移到应用层，让 Runner 接收组装结果。`bind_run()` 中同类的重新组装也应一起调整，否则只是挪了构造函数里的代码。

**而校验、授权、调度、日志、收尾虽然动作很多，仍然可以属于一个职责：保证一次工具调用正确地走完整个生命周期。** 单一职责不是“只做一个动作”，不需要再把这些步骤逐个拆成管理器。

我的建议是：**分离应用层组装与调用运行这两个角色，保留调用生命周期的统一所有权。** 这样既能减轻 executor 对应用内部结构的了解，也不会把一次调用的控制权拆散。

> 之前是认为其中一起角色是负责并行等orchestra的，看来你认为不需要，orchestra可以被包含在这一套里面，要把组装抽离出去？

**对，我目前的判断就是：并行编排可以留在工具调用生命周期管理里，优先抽离依赖组装。**

并行编排和调用管理联系很紧：哪些调用能并行，要看最终参数；什么时候启动超时，要看实际执行起点；取消、失败、结果顺序，也都需要统一协调。因此，它们放在同一个角色里是合理的，**编排本身就是管理调用生命周期的一部分。**

组装则回答另一类问题：怎样从 `Session`、`Services`、配置中得到 backend、能力和资源。Runner 使用这些依赖就够了，不必了解它们如何从应用中拼出来。

所以我建议的边界是：

- **应用层组装：** 准备运行条件。
- **ToolInvocationRunner：** 管理一批工具调用，包括单次调用的准备、执行、收尾，以及调用之间的并行关系。

但这不意味着“编排永远不该独立”。如果以后出现复杂调度需求，例如跨批次排队、优先级、全局并发配额、调用依赖图，那么就值得拆成：

- `ToolCallRunner`：负责一次调用。
- `ToolCallOrchestrator`：组织多个调用，委托 Runner 执行。

**目前还没必要仅仅因为存在并行逻辑，就把它独立成一个角色。** 现在更明确的职责边界，是应用依赖的组装与工具调用的运行。

> 但是从Agent角度，把tool统一丢给一个角色去处理是很舒服的

**对，这个体验应该保留。Agent 把一批 `tool_calls` 交给一个入口，拿回结果，不需要参与工具侧的内部协调。**

这里要区分：**对外一个入口，内部可以有多个职责。** 即使以后拆出单次调用 Runner 和批次 Orchestrator，Agent 也只面对 Orchestrator，不必分别调用它们。

不过按现在的复杂度，我更倾向于：

- 应用层负责组装依赖、创建 `ToolExecutor`。
- Agent 只调用 `ToolExecutor.execute(tool_calls)`。
- `ToolExecutor` 统一管理批次编排和单次调用生命周期。

这样抽离组装后，Agent 的使用方式完全不需要变，也不会承担新的协调工作。

而且从这个角度看，**`ToolExecutor` 这个名字其实可以保留**：它对 Agent 承诺的是“把这些工具调用执行完”。内部委托 Tool 和 backend 完成具体操作，并不影响这个名字成立。我前面偏向改成 `ToolInvocationRunner`，主要考虑职责表达；结合你强调的 Agent 视角，原名也很自然。

> 什么意思

:codex-annotation{index="1"} 我的意思是：**目前只把“依赖组装”从 `ToolExecutor` 移出去，暂时不把它内部的“批次编排”和“单次调用管理”再拆成两个角色。**

拆完以后：

- **应用层**：准备 backend、能力、资源，创建 `ToolExecutor`。
- **ToolExecutor**：接收一批工具调用，负责授权、并行、执行、超时和结果收集。
- **Agent**：照旧把 `tool_calls` 交给 `ToolExecutor`。

我说“按现在的复杂度”，是在表达**暂时没有看到需要独立调度器的充分理由**，不是说已经证明内部绝对不需要再拆。而你说的“Agent 只面对一个入口”，无论内部以后怎么拆，都可以保留。

> 这个应用层是？

:codex-annotation{index="1"} 我指的是 **Wright 自己负责启动和连接各组件的那层代码**，也就是 `src/wright/app/` 所代表的职责，不是特指 Web、TUI 这些界面。

它知道当前用哪个 Session、有哪些 Services、配置是什么，所以由它准备工具运行所需的依赖，再交给 `ToolExecutor`：

```text
应用层：知道整个运行环境，负责组装
    ├─ 创建并配置 ToolExecutor
    └─ 将 ToolExecutor 交给 Agent

Agent：决定调用哪些工具
    ↓
ToolExecutor：管理这些工具调用
```

具体到这次讨论，就是把 `ToolExecutor` 里“从 Session、Services 提取能力、选择 backend、准备资源”的工作移到这个组装位置。Executor 接收准备好的依赖。

**“应用层”是职责位置，不一定需要新建一个叫 Application 的类。** 我前面还没有追踪完整的 Agent 创建入口，所以具体该放进 `app/` 下哪个现有文件，还是增加一个组装模块，需要再沿创建链确认。
