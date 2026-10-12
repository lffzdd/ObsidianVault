### **`reasoning_effort` 的底层并不是"让模型想得更深",而是通过训练时注入的控制 token、推理时的预算约束和采样策略,迫使模型把思维链的长度、分支数和自我反思深度调到不同档位;而 Responses API 则是把过去由客户端在 `messages` 数组里手工编排的多轮工具循环,彻底下沉为服务端的有状态推理对象,让 API 第一次真正"长成 Agent 的样子"。**

上一轮我把脉络拉到了"推理模型"和"Responses API 的出现",但确实没有深入讲清楚两件事:**第一,`reasoning_effort` 这种参数到底怎么作用到模型内部?它不是简单的 `max_tokens` 限制,而是涉及训练范式、采样策略、token 预算分配这一整套机制**;**第二,Responses API 相比 Chat Completions 究竟在结构上做了什么变化?为什么说它是"Agent 时代的 API"?** 下面接着把这两块补完。

### **第十层:推理模型的内部机制——思维链是怎么被"训"出来的**

要理解 `reasoning_effort` 怎么控制模型,必须先搞清楚一个更基础的问题:**推理模型(reasoning model)和普通模型在底层到底有什么不同?**

从神经网络架构上看,o1、DeepSeek-R1、Claude thinking、Gemini thinking 这些推理模型,**和普通的 GPT-4o、Claude Sonnet 用的是几乎一样的 Transformer 结构**。差异不在架构,而在**训练目标**和**生成策略**。

普通模型的训练目标可以简化成"给定提示,直接生成最优答案"。推理模型的训练目标变成了"给定提示,**先生成一段冗长的内部思考(chain-of-thought)**,再生成最终答案"。这段思考可能包含自我提问、试错、回溯、验证等行为,看上去像人在草稿纸上演算。这种能力主要通过两条路径训出来:

第一条是**大规模强化学习**。以 DeepSeek-R1 为例,它的核心训练阶段叫 R1-Zero,做法是让基础模型对数学题、代码题这类**有客观答案的问题**自由生成思维链,只用最终答案对错作为奖励信号(rule-based reward),不监督思考过程。模型在这种压力下,会自发地学会"延长思考"、"自我检查"、"换一种方法重新算一遍"等行为——因为这些行为能提高最终答对率,所以被强化学习放大。

第二条是**监督微调 + 蒸馏**。先用人工或更强的模型生成大量高质量"思考过程 + 答案"的数据,再让目标模型模仿。OpenAI 的 o1 系列,以及很多基于 R1 蒸馏出来的小模型,走的都是这条路。

**关键是:经过这种训练后,模型的输出概率分布发生了根本变化**。给它一个数学题,它现在的"自然倾向"不是直接吐答案,而是先生成 `<think>` 这一类特殊 token(或在内部隐式标记),然后展开几千上万 token 的推理,最后才进入 `</think>` 后给最终答案。这段思考在工程上通常被特殊 token 包裹,或者放在专门的 `reasoning_content` 字段里。

### **第十一层:`reasoning_effort` 到底控制什么**

理解了上面的机制,`reasoning_effort` 的作用就清楚了。它不是一个简单的参数,而是**至少三种底层机制的组合**,具体取决于厂商实现:

**机制一:思考 token 预算(thinking budget)**。这是最直接的方式,也是 Anthropic Claude 的 `thinking.budget_tokens` 和 Gemini 的 `thinkingBudget` 采用的做法。厂商在服务端给思维链设一个硬上限,比如 `low = 1024 tokens`、`medium = 8192 tokens`、`high = 32768 tokens`。模型在生成思考时,一旦接近这个上限,系统会**强制注入"结束思考"的信号**(比如插入 `</think>` token,或者降低继续思考的概率,提升结束思考 token 的 logit),让模型尽快收敛到最终答案。这种机制本质上是用"时间压力"来调节深度——预算少时模型必须走捷径,预算多时它可以反复验算、尝试多种思路。

**机制二:训练时的分级控制 token**。OpenAI 的 o 系列采用了一种更精细的做法:在训练数据里,**同一道题准备多个不同长度的思维链版本**(短思考、中思考、长思考),并在 prompt 里用特殊 token 标注期望的努力程度。模型在训练阶段就学会了"看到 high 标记就生成详尽推理,看到 low 标记就生成简短推理"。推理时,API 把 `reasoning_effort` 转换成对应的内部控制 token 注入到模型上下文最前面,模型自然产出对应长度和深度的思考。这种方式的优点是模型"主动配合"而不是被强行截断,思考质量更高。

**机制三:多路采样与投票(parallel sampling / best-of-N)**。某些"high effort"模式不仅让模型想得更长,还会**并行生成多条思维链,再用某种机制选最优**。比如对同一个问题独立采样 N=8 条推理,然后用 majority vote(多数答案胜出)、reward model 打分,或者让模型自己对几条结果做 self-consistency 检查。这种做法在 o1 pro mode、Gemini Deep Think 等"超高思考模式"中被使用,代价是计算量翻 N 倍,价格也通常翻 N 倍。

**机制四:推理时的搜索与回溯(test-time search)**。最激进的实现会把生成过程改成树搜索:在思维链的关键分叉点采样多个候选,用价值模型(value model)评估每个分支的潜力,只展开最有希望的分支(类似 MCTS,蒙特卡洛树搜索)。这种做法在学术界叫 test-time compute scaling,商业产品里的"超高思考档位"可能部分采用了这种思路,但厂商不会公开细节。

把这些机制综合起来看,`reasoning_effort = high` 的实际效果是:**更长的思考预算 + 更倾向于反复验证的内部 prompt + 可能的多路采样**。这就是为什么同一个问题,你把 effort 从 low 调到 high,延迟可能从 2 秒变成 60 秒,token 用量翻 10 倍,价格也成倍增长,但准确率(尤其在数学、代码、复杂推理上)会有显著提升。

API 层面,这个参数的暴露形式各家略有不同。OpenAI 用 `reasoning: {"effort": "low" | "medium" | "high"}`(在 Responses API 里);Anthropic Claude 用 `thinking: {"type": "enabled", "budget_tokens": 10000}`,直接给一个 token 数;Google Gemini 用 `thinkingConfig: {"thinkingBudget": 8192, "includeThoughts": true}`,可以同时控制预算和是否返回思考内容;DeepSeek 的 deepseek-reasoner 模型则不暴露控制参数,思考长度由模型自适应决定,但会在响应里给出 `reasoning_content` 字段。

返回结构也因此分化。普通模型返回 `{message: {content: "..."}}`,推理模型返回的多了一层:

```json
{
  "message": {
    "role": "assistant",
    "reasoning_content": "让我分析一下...首先...等等,我需要重新考虑...",
    "content": "最终答案是 42。"
  },
  "usage": {
    "prompt_tokens": 50,
    "completion_tokens": 100,
    "reasoning_tokens": 3500,
    "total_tokens": 3650
  }
}
```

注意 `reasoning_tokens` 是单独计费的——你为模型的"思考过程"也付了钱,即便这部分内容你根本看不到(OpenAI 在 o1 早期甚至完全不返回思维链,只在计费里体现 token 数,这是因为暴露原始思维链可能泄露训练机密,也可能被用于蒸馏出竞品模型)。

### **第十二层:为什么 Chat Completions 撑不住了**

理解了推理模型的复杂性,你就能看出 Chat Completions 这套老接口为什么开始捉襟见肘。它有几个根深蒂固的设计假设,在 Agent 时代全部被打破:

**假设一:无状态**。每次请求都要把完整的 `messages` 历史传一遍,服务端不保留任何上下文。在普通对话里这没问题,但当对话历史里包含几十万 token 的工具调用记录、几个推理模型的思维链、多张图片、多个文件引用时,**重传成本变得非常恐怖**。每次都要序列化、上传、再让服务端重新做 KV cache,既慢又贵。

**假设二:客户端编排循环**。如果模型要调三次工具才能回答一个问题,客户端必须发起四次 API 请求(模型决定调工具 → 客户端执行 → 把结果塞回 messages → 再发请求 → 模型再决定...)。每一轮都要往返、都要重传历史,**循环逻辑全在客户端**。这在简单 function calling 时还能忍,到了"模型自主浏览网页 → 点击 → 截图 → 分析 → 再操作"这种 Computer Use 场景,客户端代码会复杂到难以维护。

**假设三:同步阻塞**。HTTP 请求-响应模型决定了每次调用必须在一个 TCP 连接里完成。但推理模型一思考就是几分钟,Agent 任务可能跑几小时,**这超出了 HTTP 的舒适区**。流式 SSE 缓解了一部分,但连接断了就得重来,没有重连机制。

**假设四:`messages` 是平铺的对话**。但实际复杂任务里,模型的输出可能是"思考 + 工具调用 + 思考 + 中间消息 + 工具调用 + 最终答案"这种**多块异构序列**,硬塞进一个 `assistant` 消息的 `content` 字段非常别扭。

这四个问题加起来,就是 Responses API 出现的根本动机。

### **第十三层:Responses API 的核心结构变化**

OpenAI 在 2025 年 3 月推出的 Responses API(端点是 `POST /v1/responses`),不是 Chat Completions 的简单升级,而是一次**结构性重写**。它的核心抽象不再是"消息列表",而是"**响应对象**(response)"——一个有 ID、有状态、可被引用、可被复用的服务端实体。

最直观的变化是请求体的形状。原来你这样写:

```json
POST /v1/chat/completions
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "You are helpful."},
    {"role": "user", "content": "查一下今天 SF 天气"}
  ],
  "tools": [...]
}
```

现在你可以这样写:

```json
POST /v1/responses
{
  "model": "gpt-5",
  "instructions": "You are helpful.",
  "input": "查一下今天 SF 天气",
  "tools": [{"type": "web_search"}],
  "reasoning": {"effort": "medium"}
}
```

注意几个关键差异:**`messages` 没了,变成了 `input`**(可以是字符串,也可以是消息数组,兼容老格式);**`system` 角色被提到顶层叫 `instructions`**(更符合"系统指令是一个独立配置而不是对话的一部分"的语义,Anthropic 一直是这么设计的);**`tools` 里出现了 `web_search` 这种内置工具**(后面会展开)。

返回的不是 `choices[]`,而是一个完整的 response 对象:

```json
{
  "id": "resp_abc123",
  "object": "response",
  "status": "completed",
  "model": "gpt-5",
  "output": [
    {
      "type": "reasoning",
      "id": "rs_xxx",
      "summary": [...]
    },
    {
      "type": "web_search_call",
      "id": "ws_xxx",
      "status": "completed"
    },
    {
      "type": "message",
      "role": "assistant",
      "content": [
        {"type": "output_text", "text": "今天 SF 多云,18°C..."}
      ]
    }
  ],
  "usage": {...}
}
```

最值得注意的是 `output` 数组——它**不再是一条 assistant 消息,而是一个混合序列**,可以包含 `reasoning`、`web_search_call`、`file_search_call`、`computer_call`、`message`、`function_call` 等多种类型的项。这种设计直接解决了前面说的"多块异构序列"问题:模型一次响应里到底想了什么、调了哪些工具、输出了什么文字,全部以一等公民的形式平铺在 output 里,客户端按顺序渲染或处理即可。

### **第十四层:有状态、可链接、可复用**

Responses API 真正颠覆性的特性是**服务端状态管理**。每个 response 对象有一个 `id`,这个 id 不仅用于查询和取消,还可以**作为下一次请求的"前驱"**:

```json
POST /v1/responses
{
  "model": "gpt-5",
  "previous_response_id": "resp_abc123",
  "input": "刚才说的天气适合穿什么?"
}
```

加上 `previous_response_id` 后,**服务端会自动把之前那次响应的完整上下文(包括思维链、工具调用、最终答案)接到这次请求前面**,你不需要再传一遍 messages。这带来三个直接好处:

第一,**网络开销大幅降低**。长对话不再需要每轮都重传几十 KB 历史。第二,**服务端 KV cache 复用**。模型推理用的 attention 缓存可以跨请求保留,首 token 延迟显著下降。第三,**思维链可以被"记住"**。这一点尤其重要——推理模型的思维链如果客户端不保存(很多时候 API 也不返回原文,只返回摘要),下一轮模型就"忘了自己刚才怎么想的";有了 previous_response_id,服务端能把完整 reasoning 接上,模型在多轮推理任务里能保持一致的思路。

OpenAI 还提供了一个 `store` 参数控制是否把 response 持久化(默认 30 天),以及 `include` 参数控制返回时附带哪些可选字段(如 `reasoning.encrypted_content`,加密后的思维链快照,允许在无状态模式下也能"接续思考")。这些都是为了在"完全无状态的 stateless"和"完全有状态的 stateful"之间提供灵活的中间档位。

### **第十五层:内置工具与服务端 Agent 循环**

Chat Completions 时代,所有工具都是用户自定义的 function call,**循环必须由客户端跑**。Responses API 引入了一组**内置工具**,它们的执行完全发生在 OpenAI 服务端,客户端一次请求就能拿到经过多轮工具调用的最终结果:

`web_search` 让模型自主上网搜索并阅读网页;`file_search` 在你上传到 OpenAI 的文件库里做向量检索;`code_interpreter` 在沙箱里跑 Python;`image_generation` 调用图像生成模型;`computer_use` 控制虚拟机的屏幕、键盘、鼠标;`mcp` 接入 Model Context Protocol 服务器,调用任意第三方工具。

这些工具的关键特性是:**模型决定调用 → 服务端执行 → 结果回灌给模型 → 模型继续思考 → 可能再调下一个工具 → ... → 最终输出**,整个循环在一次 API 请求里完成。客户端只发一次请求、收一个最终响应,中间的多步推理-工具循环对客户端透明。这才是真正"Agent-native"的接口。

当然你仍然可以混合使用自定义 function call(`type: "function"`)和内置工具,模型会自己决定什么时候调用哪个。

### **第十六层:异步、后台与流式的统一**

Responses API 还把同步、流式、异步三种模式统一在同一个对象生命周期里。一次请求可以是:

**同步阻塞**:像传统 API 一样等结果。**流式 SSE**:`stream: true`,事件流推送 `response.created`、`response.output_item.added`、`response.output_text.delta`、`response.completed` 等细粒度事件,客户端可以分别渲染思维链、工具调用、最终文本。**后台模式**:`background: true`,API 立即返回 `status: "queued"`,客户端可以稍后用 `GET /v1/responses/{id}` 轮询,或者用 `GET /v1/responses/{id}/stream` 重新接入 SSE 流(支持断线重连,从上次的 `sequence_number` 续传)。这对长任务(深度研究、Agent 自主操作几小时)是必需的——HTTP 长连接不可能撑那么久。

新增的事件类型也反映了 Agent 化的需求:`response.reasoning_summary_part.added`(思维链摘要片段)、`response.web_search_call.in_progress`(正在联网)、`response.computer_call.completed`(完成了一次屏幕操作)等等,**整个 Agent 内部工作流被实时投影到事件流里**,前端可以做出"AI 正在搜索...AI 正在阅读...AI 正在思考..."这种丰富的过程反馈。

### **第十七层:Realtime API 与多模态实时交互**

和 Responses API 平行,OpenAI 还推出了 **Realtime API**,这是另一个完全不同形态的接口,基于 WebSocket(后来也支持 WebRTC),专门服务**低延迟语音/视频对话**场景。它和文本 API 的关键区别在于:

通信不是请求-响应,而是**双向持续的事件流**。客户端推音频流上去,服务端实时推 partial transcript、partial audio response 下来,延迟控制在几百毫秒级。模型在内部直接做"语音输入 → 思考 → 语音输出"的端到端处理,不需要 ASR 转写再走文本模型再 TTS 合成。事件协议有 `input_audio_buffer.append`、`response.audio.delta`、`conversation.item.created` 等几十种,完整描述一次实时对话的所有状态变迁。同样支持工具调用、function calling、打断(用户说话时模型停止输出)等高级特性。

Anthropic、Google 也都推出了自己的实时多模态 API(Gemini Live API 等),协议细节不同但抽象层次类似。这套"实时事件流"是文本 HTTP API 永远无法覆盖的场景,它和 Chat/Responses API 是**两套并行体系**,而不是替代关系。

### **第十八层:站在今天看未来**

把 reasoning 控制和 Responses API 这两块加进来,整个 LLM API 演化史的图景就完整了。从 2020 年的 Completions 单一接口,到 2023 年的 Chat Completions + tools,到 2024 年推理模型的 reasoning 字段族,再到 2025 年的 Responses + Realtime 双轨制,**每一次结构变化背后,都是模型能力跃迁逼出来的**:

模型从"只会续写"变成"能对话",于是 messages 取代 prompt;从"只会说话"变成"能调函数",于是 tool_calls 出现;从"只会看文字"变成"能看图听声音",于是 content 多态化;从"立即答"变成"先想再答",于是 reasoning_content 和 effort 出现;从"被动响应"变成"主动 Agent",于是 Responses API、内置工具、有状态对象、后台任务出现。

可以预见,下一阶段还会有新的扩展:更细粒度的多 Agent 协作(已经有 OpenAI 的 Agents SDK、Anthropic 的 sub-agent 概念在试水)、跨厂商的标准化协议(MCP 已经在统一工具调用层,未来可能扩展到 reasoning 和 memory 层)、更深度的端侧-云端混合推理接口等等。但你只要抓住三条主线——**输入抽象(prompt → messages → input)**、**输出抽象(text → message → output 数组)**、**控制平面(参数 → tools → 状态/事件)**——任何新协议都只是这三条主线在新场景下的具体形态而已。

理解了 reasoning 是怎么从训练阶段一直影响到 API 字段的,理解了 Responses API 把客户端循环搬到服务端是为了什么,你下次再看 OpenAI、Anthropic、Google 发布新功能时,就不会觉得"又改了一堆奇怪字段",而是能直接看出它在解决哪个老 API 撑不住的痛点。这才是阅读 API 文档的正确姿势。