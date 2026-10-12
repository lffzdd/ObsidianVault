全部通过。顺带一个值得玩味的观察：测试逻辑本身 0.5s 就完了，但进程总耗时 3.4s——slow 工具的孤儿线程把解释器退出拖住了（非 daemon 线程会在退出时被 join），这就是"线程杀不死"的又一次现身。

## 我顺手改了这些

- **A-4 内层超时钳制**（[agent.py:73-78](src/AIAgent/ReActMulti/agent.py#L73-L78)）：`_execute_tool_call` 里把模型传的 `timeout` 参数钳到 `tool_timeout` 以内，保证工具内部的超时机制（如 execute_command 转后台）永远先于外层线程预算触发。
- **过期文档全清**：`_call_stream` 的 docstring 还在说"中途断流会重试"（代码已改成首事件定界），重写为现状；llm.py 模块头的 `call_llm` 改成 `LLMClient`；`is_retryable` 加了一句"APITimeoutError/RateLimitError 已被父类隐式覆盖,不是遗漏"；agent.py 两段大 docstring 修掉粘贴变形空格，并把已失效的"回调全在主线程"承诺改写成如实版本（注明 `on_output` 例外）。
- **`run()` 补了签名** `-> str | None` 和一行 docstring。
- **测试固化**：原 test.py 里的 subprocess 草稿挪到 [extra/subprocess_stream_scratch.py](src/AIAgent/ReActMulti/extra/subprocess_stream_scratch.py)，新写了 [test.py](src/AIAgent/ReActMulti/test.py) 三个离线回归测试（超时不蒸发不炸穿、内层 timeout 被钳制、工具异常变 fail），`python -m src.AIAgent.ReActMulti.test` 全过。

② 至此验收完毕。

## ③：上下文管理（token 预算 → 历史压缩）

这是 agent 工程里真正的硬课题：你的循环每轮把**完整历史**原样发出去，长任务跑下去上下文必爆。先立一个关键认知，再拆小步。

**关键认知：爆上下文必须事前防，不能事后救。** 思考题先抛给你——为什么不能等 API 报 "context length exceeded" 再压缩？两个原因：那是 400 类错误，你自己写的 `is_retryable` 会正确地拒绝重试（重试一万次也是超），那一轮就废了；更糟的是历史里塞着的大体积工具结果不会自己消失，下一轮还得爆。所以需要一个**事前的仪表盘**，这就引出 3a。

**3a：接住 UsageEvent（最便宜的一步，先做）**

你的传输层早就把 `UsageEvent` 吐出来了，`_run_turn` 一直默默扔掉。两件事：

1. `_run_turn` 的事件分发加一个 `UsageEvent` 分支，存到 Agent 上。存什么有讲究：`usage.prompt_tokens` 的**最近一次值** ≈ 当前上下文体积（这是个滞后一轮的精确读数——比本地用 tiktoken 估算省事，而且换任何模型都准）；`completion_tokens` 做**累计**，那是成本。
2. `Renderer` 加一个 `on_usage` 回调（ABC 上给个空默认实现，别强迫 SilentRenderer 跟着改），ConsoleRenderer 里每轮打一行 `tokens: 当前上下文 X / 累计输出 Y`。

**3b：预算与水位线**

`Agent.__init__` 加 `context_limit: int`（模型属性，从构造传入，比如 64k/128k）。每轮开头检查：`最近 prompt_tokens > context_limit * 0.75` 就触发压缩。为什么留 25% 余量？三笔账：读数滞后一轮、本轮工具结果还要追加进去、模型回答本身也占窗口。

**3c：折叠旧工具结果（先做的压缩策略，纯数据操作）**

观察你的 messages 结构会发现：体积大头是 `tool_results` 消息，而且它们**过期最快**——模型读完结果、写出下一步推理后，原始的 8000 字符命令输出就基本没用了，结论已经沉淀在后续 assistant 消息里。所以第一刀：触发水位线时，把"距今超过 K 轮"的 tool_results 消息内容替换成占位符（如 `{"tool_results_folded": "已折叠 N 条结果以节省上下文"}`）。三条不变量：system prompt 永不动；最近 K 轮（比如 3 轮）原样保留；消息**条数和角色序列不变**，只换内容——这样对账结构不被破坏。Claude Code 自己的 microcompact 就是这个思路。

**3d（选做进阶）：LLM 摘要压缩**

3c 不够用时的重武器：花一次 LLM 调用，把 system 之后、最近 M 条之前的全部历史总结成一条消息（要求摘要保留：任务目标、已确认的结论、未完成事项），整段替换。这就是 Claude Code 的 /compact。设计上的新问题：压缩用的 prompt 放哪、压缩本身失败了怎么办（提示：你已经有 TurnAbort 的先例——想想"压缩失败"是数据还是中止）。

建议节奏：3a 一刻钟，3b 半小时，3c 是本步的主菜，3d 看兴致。3a+3b 写完可以先跑一次真任务看看仪表盘读数，再动 3c。写完叫我看。