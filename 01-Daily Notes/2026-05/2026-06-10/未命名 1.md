对，CLI 不急。你现在真正要升级的不是“入口形式”，而是 **Agent 内核从“会对话、会调工具”升级成“能长期、安全、可恢复地完成工程任务”**。

可以把目标理解成这样：

```text
现在：
用户问题 -> LLM 输出 JSON -> 调工具 -> 把结果塞回去 -> 继续

升级后：
任务目标 -> 会话状态机 -> 模型推理 -> 工具计划/权限/执行/回传
       -> 上下文管理 -> 失败恢复 -> 验证 -> 产出结果
```

你的 [agent.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/ReActMulti/agent.py:1) 已经有最核心的 ReAct loop，所以不是重写，而是把它一层层“长骨头”。

**一、先明确当前 Agent 的本质**
现在的主循环在 `Agent.run()`：

```text
1. 用户 prompt 进入 messages
2. LLM 生成一段 JSON
3. parse JSON
4. 如果 final_answer，结束
5. 如果 tool_calls，并行执行工具
6. 把 tool_results 作为 user message 塞回上下文
7. 下一轮
```

这就是一个典型 ReAct agent。

它已经具备几个很好的设计点：

- [llm.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/ReActMulti/llm.py:1) 把流式/非流式统一成事件流。
- [renderer.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/ReActMulti/renderer.py:1) 把展示层和主逻辑分离。
- [tools/base.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/ReActMulti/tools/base.py:1) 有 `Tool` / `ToolCall` / `ToolResult` 基本协议。
- `execute_tool_calls_parallel()` 已经有并行、保序、超时占位。
- `_fold_old_tool_results()` 已经开始做上下文压缩。

所以第一性原理不是“加更多工具”，而是：**让每一轮决策、执行、状态变化都可验证、可恢复、可控制。**

**二、第一层升级：把消息列表升级成 Session 状态**
现在 `self.messages` 是 Agent 里的一个 list。简单任务没问题，但长任务会乱。

应该升级成一个显式的 `SessionState`：

```text
SessionState
- messages
- current_task
- workspace_dir
- cwd
- tool_results
- background_tasks
- token_usage
- compact_history
- artifacts
- status
```

原理是：  
`messages` 只是给模型看的上下文，不应该承载全部真实状态。真实状态应该由系统自己维护，必要时再“投影”成 prompt 给模型。

否则以后你会遇到这类问题：

- 模型忘了当前 cwd。
- 旧工具结果折叠后，系统也失去原始数据。
- 后台任务 id 只存在上下文里，恢复会话就丢。
- token 统计、文件变更、权限决策混在对话里，不可控。

Claude Code 类系统一定有强状态层，原因就在这里。

**三、第二层升级：把工具从函数升级成能力对象**
你现在的 `Tool` 是：

```python
name
description
parameters
func
```

这已经很好，但还不够。工程 agent 的工具至少需要这些元信息：

```text
Tool
- name
- description
- input_schema
- output_schema
- read_only
- concurrency_safe
- requires_permission
- risk_level
- timeout_policy
- execute()
- render_result()
```

原理是：  
模型只负责“想调用什么”，系统负责判断“能不能调用、怎么调用、调用后怎么表示”。

例如：

- `read_file` 是 read-only，可以自动执行。
- `write_file` 会改 workspace，需要权限策略。
- `execute_command` 风险最高，需要展示命令、cwd、timeout。
- `web_search` 可以并行。
- `edit_file` 不能和另一个写文件工具随便并行。

你现在的并行执行是“所有工具都丢线程池”。下一步应该变成：

```text
读工具可以并行
写工具串行
命令工具按风险审批
同一文件的 edit/write 不能并行
未知工具直接 fail
```

这就是工具系统升级的核心逻辑。

**四、第三层升级：从“JSON 解析”升级成 Turn 协议**
现在模型输出靠 [prompt.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/ReActMulti/prompt.py:1) 约束成 JSON，然后 [util.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/ReActMulti/util.py:1) 解析。

这一步要强化成明确协议：

```json
{
  "reasoning": "...",
  "tool_calls": [
    {
      "id": "call_xxx",
      "name": "read_file",
      "arguments": {}
    }
  ],
  "final_answer": null
}
```

现在你的 `id` 是系统解析时补的，这可以继续。但长期最好让系统生成、系统追踪，模型只需给 name/arguments。

关键原则：

```text
每一轮只能是：
- 要么请求工具
- 要么给最终答案
- 不能两者都有
- 工具结果必须按 call_id 对账
- 解析失败不能崩，应该作为 observation 喂回模型
```

你已经做了 `TurnAbort`，这是对的。下一步是把这个协议独立出来，比如 `protocol.py`，让测试专门覆盖它。

**五、第四层升级：权限系统**
这是从 toy agent 到 coding agent 的分水岭。

现在文件工具被限制在 `workspace`，但命令工具可以跑 shell。要升级的话，应该引入 `PermissionPolicy`：

```text
allow
deny
ask
```

决策依据：

```text
工具名
参数
当前目录
是否写文件
是否访问网络
是否执行 shell
是否删除文件
是否改 git 状态
```

逻辑大概是：

```text
LLM 请求工具
-> ToolExecutor 收到 ToolCall
-> PermissionPolicy 检查
-> allow: 执行
-> deny: 返回 ToolResult.fail
-> ask: 暂停等待用户确认
```

即使你现在不做交互式 CLI，也可以先做一个非交互策略：

```text
read_file/list_files/web_search: allow
write_file/edit_file/execute_command: ask 或 deny
```

原理是：  
权限不应该写在 prompt 里，因为模型会忘、会绕、会误判。权限必须是系统硬边界。

**六、第五层升级：上下文管理**
你已经有 `_fold_old_tool_results()`，方向很对。但 Claude Code 类 agent 需要更系统的上下文策略。

上下文里有几类东西：

```text
1. 用户目标
2. 最近对话
3. 工具调用历史
4. 文件内容
5. 命令输出
6. 错误与修复尝试
7. 长期记忆/项目摘要
```

它们不能一视同仁。

升级逻辑：

```text
短期上下文：最近几轮完整保留
中期上下文：工具结果折叠成摘要
长期上下文：保存到 session/memory，需要时检索
大文件内容：只保留片段、行号、摘要
命令输出：保留 tail + returncode + 是否截断
```

你现在折叠旧工具结果：

```text
data -> "[旧工具结果已折叠以节省上下文]"
```

下一步可以变成：

```text
data -> {
  "summary": "...",
  "original_ref": "artifact/tool_result_xxx.json",
  "truncated": true
}
```

原理是：  
模型上下文是工作记忆，不是数据库。真实数据要落在系统状态或 artifact 里。

**七、第六层升级：工程任务闭环**
普通 ReAct agent 到 coding agent 的关键差别是：coding agent 不能只“回答”，它要能“验证”。

所以需要引入任务阶段：

```text
understand -> inspect -> plan -> act -> verify -> final
```

现在你的 agent 是自由循环，模型每轮自己决定做什么。升级后可以给它模式约束：

```text
Plan 模式：只能读、搜、分析，不能写和执行危险命令
Act 模式：可以改文件、跑测试
Verify 模式：必须检查 diff、运行测试或解释为什么没跑
Final 模式：总结结果
```

这不是为了形式，而是为了降低乱改代码的概率。

最小实现不复杂：

```text
AgentMode = "plan" | "act"
ToolExecutor 根据 mode 限制工具
```

例如：

```text
plan: list_files/read_file/grep/web_search
act: write_file/edit_file/execute_command
```

**八、第七层升级：失败恢复**
现在工具失败会变成 `ToolResult.fail`，这很好。但还需要更高层的失败分类。

失败类型大概有：

```text
LLM 输出非法
工具参数非法
工具不存在
权限拒绝
命令超时
命令退出码非 0
文件不存在
上下文过长
网络失败
达到最大步数
```

每类失败的处理不一样：

```text
LLM 输出非法 -> 要求模型重新输出合法 JSON
权限拒绝 -> 让模型换方案
命令失败 -> 把 stderr/stdout 给模型分析
上下文过长 -> compact 后重试
最大步数 -> 生成未完成报告
```

你现在的 `TurnAbort` 是第一块砖。下一步可以把错误类型标准化：

```text
AgentError
- kind
- message
- recoverable
- detail
```

原理是：  
失败不是异常，而是 agent 工作流的一部分。

**九、第八层升级：从单 Agent 到多 Agent**
这一步不要太早做，但你项目里已经有 [MultiAgent/agent_graph_engine.py](/Users/slyh/MyDir/Project/LLM/src/AIAgent/MultiAgent/agent_graph_engine.py:1)，后面可以接。

多 agent 的本质不是“开多个聊天机器人”，而是把复杂任务拆成角色和边界：

```text
Planner: 拆任务，不改文件
Researcher: 搜索/读代码
Coder: 改文件
Reviewer: 找 bug
Verifier: 跑测试
```

你的 ReActMulti 可以先作为一个基础执行 agent，然后图引擎调度它。

逻辑会变成：

```text
GraphEngine 决定下一个模块
每个模块内部用 ReAct Agent 完成局部任务
模块产出写入 RunState
下一个模块读取 RunState
```

这会比直接让一个模型无限循环稳定得多。

**最推荐的升级顺序**
我会这样排：

```text
1. 抽 SessionState
2. 抽 ToolExecutor
3. 强化 Tool 协议，加 schema/权限/并发属性
4. 增加 grep/glob/diff/apply_patch/git_status 这类 coding 工具
5. 做 PermissionPolicy
6. 做更正式的 context compact + artifact 保存
7. 做 plan/act/verify 模式
8. 做 session 持久化
9. 接 MultiAgent graph
10. 最后才考虑 CLI/Web/MCP/IDE
```

核心判断标准很简单：

```text
每升级一步，都问：
这个 Agent 是否更可控？
是否更能恢复？
是否更能验证自己的工作？
是否更不依赖模型“记得所有事”？
```

如果答案是 yes，就是在向 Claude Code 那类系统靠近。  
如果只是“多一个入口、多一个 UI、多一个模型”，那还不是本质升级。