### **大模型 API 调用格式之所以让人困惑，是因为它从早期单纯的"文本补全"接口，逐步演化成涵盖对话、工具调用、多模态、流式与异步推理的复合协议，每一次能力扩展都在原有结构上"打补丁"，最终形成了今天这套层层嵌套、看似杂乱却有迹可循的 JSON 规范。**

要真正理解今天 OpenAI、Anthropic、Google、阿里、字节、DeepSeek 等厂商各种 API 的"奇怪格式"，最好的方法不是去背字段，而是顺着历史的脉络,从最底层的"模型到底是什么"开始,一层层往上看每一层为什么要长成那个样子。下面我会按照"模型本质 → 推理过程 → 文本补全 API → 对话 API → 系统角色与多轮 → 函数/工具调用 → 多模态 → 流式与异步 → 各厂商差异 → 新一代 Responses/Realtime API"这个顺序,从底向上把整条线讲清楚。

### **第一层:模型本质——一个"下一个 token 概率分布"的函数**

要理解 API 的形状,必须先承认一个事实:今天所有的大语言模型,无论是 GPT、Claude、Gemini、Qwen、DeepSeek、Llama,本质上都是同一种数学对象——一个条件概率函数 $P(x_{t+1} \mid x_1, x_2, \ldots, x_t)$。给它一串 token,它输出下一个 token 的概率分布。仅此而已。

模型本身**不知道什么是"对话"**,**不知道什么是"系统提示"**,**不知道什么是"用户"和"助手"**,更不知道什么是"函数调用"。它眼里只有一件事:一串整数(token id)进去,一个概率向量出来。

所以从模型的角度看,最自然、最底层的接口应该是这样的:

```
输入: [token_id_1, token_id_2, ..., token_id_n]
输出: [概率分布 over 词表]
```

或者退一步,把 token 化封装掉,变成:

```
输入: 一段字符串
输出: 一段字符串(模型把概率最大的 token 一个个采样出来,再解码成文字)
```

**这就是最原始的"文本补全"(Text Completion)接口的雏形**。它和模型的工作方式一一对应,没有任何多余的概念。如果历史就停在这里,API 会非常干净。但需求很快就超出了"补全一段话"的范畴,于是格式开始膨胀。

### **第二层:Text Completion 时代——GPT-3 与 `prompt` 字段**

2020 年 GPT-3 发布时,OpenAI 提供的就是这种最朴素的接口,核心字段只有一个 `prompt`:

```json
POST /v1/completions
{
  "model": "text-davinci-003",
  "prompt": "请把下面这句话翻译成英文:今天天气真好。",
  "max_tokens": 100,
  "temperature": 0.7,
  "top_p": 1,
  "stop": ["\n\n"]
}
```

返回:

```json
{
  "choices": [
    {"text": "The weather is really nice today.", "finish_reason": "stop"}
  ]
}
```

这套格式之所以"好理解",是因为它和模型本质几乎一一对应:你给一段文本,它接着写。`temperature`、`top_p`、`max_tokens`、`stop` 这些参数都是采样器的直接控制开关,没有任何抽象。

但这个时代的"提示工程"非常痛苦。你想让模型扮演助手回答问题,得自己拼字符串:

```
以下是用户和一位乐于助人的 AI 助手的对话。

用户: 你好
助手: 你好!有什么可以帮你的吗?
用户: 帮我写一首诗
助手:
```

模型续写"助手:"后面的内容。多轮对话要靠开发者**手动拼接历史**,管理 `\n用户:` 和 `\n助手:` 这种分隔符,还要小心别让模型"越界"继续生成"用户:"开头的下一轮。这种做法又脆弱又容易越狱,而且每个开发者都在重复造轮子。

### **第三层:Chat Completions 的诞生——把"角色"作为一等公民**

2023 年 3 月,OpenAI 发布 ChatGPT API(`gpt-3.5-turbo`),同时推出了一个**全新的接口形态**:`/v1/chat/completions`。这是大模型 API 历史上最重要的一次结构性变化。它把原来的 `prompt: "字符串"` 替换成了 `messages: [...]` 数组:

```json
POST /v1/chat/completions
{
  "model": "gpt-3.5-turbo",
  "messages": [
    {"role": "system",    "content": "你是一个乐于助人的助手。"},
    {"role": "user",      "content": "帮我写一首关于春天的诗。"},
    {"role": "assistant", "content": "春风拂面柳丝长……"},
    {"role": "user",      "content": "再来一首关于秋天的。"}
  ]
}
```

为什么要这么改?**原因就一个:把原本由开发者手工拼接的"对话格式"下沉到模型训练里**。OpenAI 在训练 ChatGPT 时使用了一种叫 **ChatML** 的内部格式(类似 `<|im_start|>system\n...\n<|im_end|>`),模型已经学会识别角色边界。API 层面把 `messages` 数组传进去,服务器内部自动把它渲染成模型真正能看懂的 token 序列。这样做有三大好处:

第一,**安全性提升**:用户输入被限制在 `user` 角色里,无法伪造 `system` 指令越狱(虽然提示注入仍然存在,但难度大大增加)。第二,**多轮对话标准化**:开发者不再操心分隔符,只需要把历史消息按顺序塞进数组。第三,**为后续扩展留空间**:`role` 字段一旦存在,以后就可以加 `function`、`tool`、`developer` 等新角色,而不用重新设计 API。

返回结构也相应变化,从 `choices[].text` 变成 `choices[].message`:

```json
{
  "choices": [{
    "message": {"role": "assistant", "content": "秋风萧瑟枫叶红……"},
    "finish_reason": "stop"
  }]
}
```

**这是你今天看到几乎所有大模型 API 都长得很像的根本原因**——它们全都抄了这套 `messages: [{role, content}, ...]` 的形状,因为开发者已经习惯了。

```markdown
## **那为什么要单独拆出 `chat/completions`？​**

原因是多方面叠加的结果。

**首先是微调方式变了。​** GPT-3.5-turbo 这类模型是专门用对话格式的数据做了指令微调（Instruction Tuning）和 RLHF 的，它"期待"的输入就是带角色标记的结构化格式。如果你用旧的 `completions` 接口给它发一个裸 prompt，效果会明显变差，因为模型在训练时从没见过这种输入形式。所以接口分离，**本质上是在告诉开发者：这个模型要用这种方式喂**。

**其次是产品层面的清晰切割。​** OpenAI 当时面临一个现实问题：旧的 `text-davinci-003` 等模型继续走 `/completions`，新的对话优化模型走 `/chat/completions`，两条产品线并行，用不同的 endpoint 可以非常干净地隔离定价、限流、版本管理和文档体系，避免混用导致的各种歧义。

**第三是结构化输入带来了额外的语义价值。​** 虽然最终都是拼成一段文本，但在 API 层面保持结构化的 `messages`，让中间件、SDK、应用框架可以在不解析自由文本的情况下操作对话历史——比如截断最早的几轮、替换 system prompt、注入工具调用结果等。这是纯字符串 `prompt` 做不到的。

---

所以总结来说，`chat/` 这个前缀更多是一个**接口契约和使用意图的声明**，而非技术上的必要分层。它在说："这个 endpoint 背后的模型是为对话场景微调的，请用对话格式来调用它。" 你完全可以在理解层面把它看作是"带角色标记的补全"，这个理解反而更接近底层真相。
```

### **第四层:Function Calling——让模型"开口说结构化数据"**

2023 年 6 月,OpenAI 又加了一个里程碑式的特性:**Function Calling**。这是为了解决一个非常实际的问题:模型只会生成文本,但实际应用经常需要它"调用外部 API"——查天气、订机票、搜数据库。早期做法是让模型生成一段 JSON 字符串,然后开发者用正则或 `json.loads` 去解析,失败率非常高。

OpenAI 的方案是在 API 里加一个 `functions` 参数(后来改名为 `tools`),让你把可调用的函数 schema 提前声明:

```json
{
  "model": "gpt-4",
  "messages": [{"role": "user", "content": "北京今天多少度?"}],
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "查询某城市的天气",
      "parameters": {
        "type": "object",
        "properties": {
          "city": {"type": "string", "description": "城市名"}
        },
        "required": ["city"]
      }
    }
  }]
}
```

模型如果决定"我需要调函数",它返回的就不再是普通文本,而是一个特殊结构:

```json
{
  "message": {
    "role": "assistant",
    "content": null,
    "tool_calls": [{
      "id": "call_abc123",
      "type": "function",
      "function": {
        "name": "get_weather",
        "arguments": "{\"city\": \"北京\"}"
      }
    }]
  },
  "finish_reason": "tool_calls"
}
```

**注意几个让人困惑的细节**:`content` 是 `null`,因为模型没说话,只"动手"了;`arguments` 是一个 JSON **字符串**而不是 JSON 对象(历史包袱,因为底层模型是逐 token 生成的,JSON 只能作为字符串流式吐出);返回了一个 `tool_call_id`,你下一轮要把函数执行结果作为一条新消息塞回去:

```json
{"role": "tool", "tool_call_id": "call_abc123", "content": "{\"temp\": 22, \"weather\": \"晴\"}"}
```

这就是为什么你看到 `messages` 数组里突然多了 `role: "tool"`、`tool_call_id`、`tool_calls` 这一堆字段——它们都是为了**把"模型 ↔ 外部世界"的循环用同一个 messages 数组表达出来**。每一轮你都把完整历史(包括模型的工具调用决策、工具返回结果)再喂回去,让模型基于完整上下文生成最终回答。

后来 `functions` 字段被废弃,统一改成 `tools`(因为以后还会有 `tools` 类型不只是 function,比如 `code_interpreter`、`file_search`、`web_search` 等内置工具),这就是你看到"好像一直在变"的一个典型例子——**每一次能力扩展,字段都在分裂、改名、嵌套加深**。

### **第五层:多模态——`content` 从字符串变成数组**

2023 年底 GPT-4V 上线,模型可以"看图"了。问题来了:`content` 字段原来是一个字符串,怎么塞进图片?OpenAI 的解决办法是把 `content` 从 `string` 变成 `string | array`:

```json
{
  "role": "user",
  "content": [
    {"type": "text", "image_url": "看看这张图里是什么?"},
    {"type": "image_url", "image_url": {"url": "https://example.com/cat.jpg"}}
  ]
}
```

这是另一处让初学者抓狂的地方:**同样一个 `content` 字段,可能是字符串,可能是对象数组,数组里每个元素又有 `type` 来区分** `text`/`image_url`/`input_audio`/`file` 等。这种"多态字段"在 JSON 里非常不优雅,但它是为了**向后兼容**——老代码写 `content: "你好"` 仍然能跑,新代码写数组形式才能传图片。

Anthropic 的 Claude API 也走了类似的路线,但更彻底:它从一开始就把 `content` 设计成数组,每个块叫做 `content block`,有 `text`、`image`、`tool_use`、`tool_result` 等多种类型。Google 的 Gemini API 又叫得不一样,它把消息叫 `contents`,每条里有 `parts` 数组,字段名是 `text`、`inlineData`、`fileData`、`functionCall`、`functionResponse`。**三家都在表达同一个抽象"一条消息由多个不同类型的块组成",但字段名各不相同**,这就是跨厂商迁移痛苦的根源。

### **第六层:流式输出——SSE 与增量 delta**

模型生成是 token by token 的,等它写完几百字再返回,用户体验非常差。所以 API 必须支持**流式输出**,标准做法是 Server-Sent Events(SSE):

```
POST /v1/chat/completions
{
  "model": "gpt-4",
  "messages": [...],
  "stream": true
}
```

返回不再是一个完整 JSON,而是一连串 `data: {...}` 行:

```
data: {"choices":[{"delta":{"role":"assistant","content":""}}]}
data: {"choices":[{"delta":{"content":"你"}}]}
data: {"choices":[{"delta":{"content":"好"}}]}
data: {"choices":[{"delta":{"content":"!"}}]}
data: {"choices":[{"delta":{},"finish_reason":"stop"}]}
data: [DONE]
```

注意每个 chunk 里是 `delta`(增量)而不是完整内容,客户端要自己拼接。流式模式下工具调用更复杂:`function.arguments` 这个 JSON 字符串是**一段段流出来的**,你必须把所有 `arguments` 片段拼起来才能 `JSON.parse`。如果你曾经被"为什么 arguments 突然不是合法 JSON"坑过,根源就在这里。

### **第七层:厂商分化与"OpenAI 兼容协议"**

到了 2024 年,大模型百花齐放,几乎每家都做了自己的 API。但开发者已经被 OpenAI 的 `messages` 格式驯化,工具链(LangChain、LlamaIndex、各种 SDK)也都围绕它建立。结果就是:**绝大多数后来者主动提供"OpenAI 兼容接口"**,只要把 `base_url` 换一下,代码不用改就能跑。

国内的 DeepSeek、Moonshot、Qwen(DashScope)、智谱、Minimax、字节豆包,几乎都提供了 `chat/completions` 兼容端点。但每家又会**额外加自己的"私货字段"**:DeepSeek 加了 `reasoning_content`(暴露思维链),Qwen 加了 `enable_search`(联网),Anthropic 自己的原生 API 用 `system` 作为顶层参数而不是 messages 里的一个 role,Google Gemini 把 system 叫 `systemInstruction`。这些差异就是让你觉得"格式一直在变"的直接原因。

下面这个表对比了主流厂商在几个关键字段上的差异:

| 概念 | OpenAI | Anthropic Claude | Google Gemini |
|---|---|---|---|
| 接口路径 | `/v1/chat/completions` | `/v1/messages` | `/v1/models/{m}:generateContent` |
| 消息字段 | `messages` | `messages` + 顶层 `system` | `contents` + `systemInstruction` |
| 内容块字段 | `content`(string 或 array) | `content`(始终 array) | `parts` |
| 工具调用 | `tool_calls` / `role:tool` | `tool_use` / `tool_result` 块 | `functionCall` / `functionResponse` |
| 最大输出 | `max_tokens` | `max_tokens`(必填) | `maxOutputTokens` |

### **第八层:推理模型与"思维链"的暴露**

2024 年 9 月 OpenAI 发布 o1,2025 年 DeepSeek-R1、Claude 3.7 Sonnet thinking、Gemini 2.5 Pro thinking 陆续登场,**推理模型**成为新主流。这类模型会先"思考"很长一段(几千到几万 token 的内部推理),再输出最终答案。API 层面又出现了新字段:OpenAI 加了 `reasoning_effort`(low/medium/high)控制思考深度,响应里有 `reasoning_tokens` 计数;DeepSeek 在 message 里多出 `reasoning_content` 字段,把思维链直接暴露给开发者;Anthropic 引入 `thinking` 块类型,可以选择是否返回。

这又是一次"在老结构上打补丁"的典型——**思维链既不是普通 content,也不是 tool_call,只能新开一个字段**。结果就是同一个概念在不同家有完全不同的表达。

### **第九层:Responses API 与 Agent 化的未来**

2025 年 3 月,OpenAI 发布了**新一代 Responses API**(`/v1/responses`),试图取代 Chat Completions。它的核心变化是:**把"一次对话"从无状态的 messages 数组,升级为有状态的 response 对象**。你可以引用上一次的 `response_id` 继续对话,服务器端帮你管历史;内置工具(`web_search`、`file_search`、`computer_use`)可以直接在一次调用里完成多步推理和工具调用循环,不用客户端手动 round-trip。同时它原生支持异步、后台任务、复用上下文等高级特性。

可以预见,未来几年 API 还会继续演化:从"一次问答"到"一个 Agent 任务",从客户端编排循环到服务端编排循环。**但底层的两个核心抽象不会变:消息列表(messages)和工具调用(tool calls)**。无论字段怎么改名、怎么嵌套,你只要抓住这两个核心,就能快速看懂任何新 API。

### **回头看:为什么"莫名其妙"**

把整条线串起来,你就能理解今天 API 看上去乱的根本原因:

它本质上是一个**十年间不断打补丁的协议**。最底层是模型的 token 概率函数,最自然的接口是 `prompt → completion`;为了支持对话,套了一层 `messages: [{role, content}]`;为了支持函数调用,加了 `tools` 和 `role: "tool"`;为了支持多模态,把 `content` 从字符串扩成数组;为了支持流式,加了 `delta` 增量;为了支持推理模型,加了 `reasoning_content` 和 `reasoning_effort`;为了支持 Agent,又出现了 Responses API 这种有状态结构。**每一层都在解决一个真实问题,每一层都不能破坏向后兼容,所以字段越叠越多、嵌套越来越深**。

而厂商之间的差异,本质上是大家在抄 OpenAI 这套 de facto 标准时,各自加了一些"既要兼容、又要差异化"的私货,加上少数几家(主要是 Anthropic 和 Google)有自己更早的设计哲学,导致字段命名上无法统一。

理解了这条历史脉络,以后你再看任何一份 API 文档——不管是新出的厂商,还是老厂商的新版本——都可以问自己三个问题:它的 messages 怎么组织?它的 tool calling 怎么表达?它对多模态和流式的扩展放在哪里?把这三个核心抓住了,所谓"奇怪的格式"就只是同一个抽象的不同方言而已。