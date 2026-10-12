非常好——理解 `AIMessage` 和 `ToolMessage`，就等于理解了 LangChain 工具调用系统的底层“对话协议”。  
我们一步讲透它们的设计动机、历史演化、以及它们在一次完整调用流程中的角色。

---

## 一、为什么需要 `AIMessage` 和 `ToolMessage`

LangChain 从最早期（2022）就以“消息流”（message passing）为核心抽象。  
最开始只有两种角色：

- `HumanMessage`：人类说的话
    
- `AIMessage`：模型说的话
    

随着 function calling（函数调用 / 工具调用）引入，**模型和工具之间的互动**也要纳入这个对话序列里。  
于是引入了更多“消息类型”：

|类型|谁发的|用来表示什么|
|---|---|---|
|`HumanMessage`|人类|用户输入|
|`AIMessage`|模型|模型输出（可能包含工具调用请求）|
|`ToolMessage`|工具|工具执行后的结果返回模型|
|`SystemMessage`|系统|额外的指令或上下文控制|
|`FunctionMessage`（旧名）|旧版兼容|function calling 早期版本的 ToolMessage|

---

## 二、`AIMessage`：模型的发言（带“意图”）

`AIMessage` 表示模型返回的一条消息。  
最简单时，它就只有文本内容：

```python
from langchain_core.messages import AIMessage
AIMessage(content="北京今天晴，气温30°C。")
```

但在有工具调用的情况下，模型的输出不仅是文本，还包含“我要调用哪个工具”的结构化信息。  
这时，`AIMessage` 会带上一个字段：

```python
AIMessage(
    content="我来帮您查询北京的天气情况。",
    tool_calls=[
        {
            "id": "call_00_84RtEoBvcF",
            "name": "get_weather",
            "args": {"city": "北京"}
        }
    ]
)
```

你可以把它理解为：

> 模型正在“说话 + 伸手要调用工具”。

`tool_calls` 是 LLM 在 function calling 协议下自动生成的。  
它告诉框架：

- **哪个工具要用**（`name`）
    
- **传什么参数**（`args`）
    
- **调用 ID**（`id`，用来和结果对应）
    

---

## 三、`ToolMessage`：工具的回应（给模型的回执）

当模型发出 `AIMessage` 后，LangChain 会检测它是否包含 `tool_calls`。  
如果有，就要执行对应的 Python 工具函数，并把执行结果封装成一条 `ToolMessage`。

```python
from langchain_core.messages import ToolMessage

ToolMessage(
    content="晴，30°C",              # 工具执行返回值
    tool_call_id="call_00_84RtEoBvcF"  # 对应上模型那条调用请求的 id
)
```

这条消息会被**再次送入模型**作为新输入。  
模型就能根据这条工具返回的内容，生成最终的自然语言回答，比如：

> “北京今天晴，气温30°C，适合出行。”

---

## 四、它们如何串联成“对话循环”

完整过程可以这样理解（时间线）：

```
1. HumanMessage      → 用户提问
2. AIMessage          → 模型说：“我需要调用 get_weather(city=北京)”
3. ToolMessage        → 工具返回结果 “晴 30°C”
4. AIMessage          → 模型据此生成最终回答
```

LangChain 框架内部会用这种消息序列作为上下文：

```python
[
    HumanMessage(content="请告诉我北京的天气情况"),
    AIMessage(content="我来帮您查一下", tool_calls=[...]),
    ToolMessage(content="晴 30°C", tool_call_id="..."),
    AIMessage(content="北京今天晴，气温30°C")
]
```

这就是“多轮工具对话”的核心。

---

## 五、为什么不用普通字符串？

因为：

1. LangChain 在执行链条时，需要知道消息的**角色**（人 / 模型 / 工具），以及是否有 **工具调用指令**。
    
2. 这些消息可以序列化、追踪、记录，便于在日志（LangSmith）里重放整个对话。
    
3. 对模型输入输出进行结构化包装，可以同时兼容 OpenAI、Anthropic、DeepSeek 等不同 API 的格式差异。
    

---

## 六、可视化一眼看懂：

```
┌───────────────────────────────────────────────┐
│  HumanMessage: 用户说「查一下北京天气」          │
└──────────────┬────────────────────────────────┘
               │
               ▼
┌───────────────────────────────────────────────┐
│  AIMessage: 模型说「调用 get_weather(city=北京)」│
│   tool_calls=[{ name:"get_weather", ... }]    │
└──────────────┬────────────────────────────────┘
               │
               ▼
┌───────────────────────────────────────────────┐
│  ToolMessage: 工具返回「晴 30°C」               │
└──────────────┬────────────────────────────────┘
               │
               ▼
┌───────────────────────────────────────────────┐
│  AIMessage: 模型基于工具结果生成最终回答        │
│   content="北京今天晴，气温30°C。"              │
└───────────────────────────────────────────────┘
```

---

## 七、小结

|类名|来源|作用|
|---|---|---|
|`HumanMessage`|人类输入|提问、命令|
|`AIMessage`|模型输出|模型的自然语言回答，或包含工具调用|
|`ToolMessage`|工具执行返回|把工具结果喂回模型|
|`SystemMessage`|系统控制信息|设置角色或上下文|
|`FunctionMessage`|旧版兼容|function calling 的历史遗留名|

---

下一步我可以教你写一个 **“自动循环执行器”**，它能自动识别 `AIMessage.tool_calls`、执行工具、生成 `ToolMessage`、再次调用模型，直到获得最终答案（也就是你自己的迷你 Agent）。  
要我带你写这个吗？