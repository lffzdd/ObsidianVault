```
<think>
The user wants me to explain how modern ReAct works at the model level, similar to the detailed explanation in their tool_call.md. They're asking whether modern ReAct still uses the plain-text "Action/Action Input/Observation" format, or whether it now uses `<tool_call>` and `<tool_response>` tags.

Let me think about this carefully.

The user's ReAct code is the "classic" ReAct approach - it's essentially prompt engineering. You put the ReAct format instructions in the system prompt, and the model generates plain text following that format. The Python code uses regex to parse "Action:" and "Action Input:" from the text, executes the tool, and feeds back "Observation:" results.

In modern practice, ReAct has evolved to leverage the model's native tool calling capabilities instead of relying on text parsing. The reasoning loop now works through the model's built-in `<tool_call>` mechanism for actions and `<tool_response>` tags for observations, rather than plain-text formatting that needs regex extraction.

Let me trace through what this actually looks like in practice with a concrete example. When a model like Qwen with thinking mode receives a query, it first generates reasoning within `<think>` blocks, then outputs structured tool calls that the API layer automatically parses and executes. The system then feeds back the tool results in the next round, and the cycle continues with the model's reasoning, tool calls, and observations forming the complete ReAct loop.

The fundamental difference from classic ReAct is that modern implementations use structured formats and native API support rather than plain text patterns—the model doesn't need to format "Thought: ..." or "Action: ..." as strings that get regex-parsed, since the protocol handles this directly.

</think>
```
# 现代 ReAct：从 Prompt Hack 到原生支持

## 一、先说结论

你写的那个 ReAct 代码是 **2022 年的经典做法**——纯 prompt 工程 + 正则解析。

现代做法中，ReAct 的三个核心概念已经被**模型原生支持的特殊 token** 取代了：

| ReAct 概念              | 经典做法（你的代码）                    | 现代做法（原生 tool calling + think）               |
| ----------------------- | --------------------------------------- | --------------------------------------------------- |
| **Thought**（思考）     | 纯文本 `Thought: 我需要查天气...`       | `<think>我需要查天气...</think>`                    |
| **Action**（行动）      | 纯文本 `Action: get_weather` + 正则解析 | `<tool_call>{"name":"get_weather",...}</tool_call>` |
| **Observation**（观察） | 你手动拼接 `Observation: 晴天25°C`      | `<tool_response>晴天25°C</tool_response>`           |

**ReAct 的"想→做→看"循环思想没变，但实现方式彻底变了。**

---

## 二、经典 ReAct：模型真正看到的文本

你的代码里，模型看到的是这样的：

### 第 1 次调用

```
<|im_start|>system
尽可能地回答用户的问题。你可以使用以下工具：
1. get_weather: 获取指定城市的天气。输入参数为城市名，例如：北京
2. calculate_length: 计算一段文本的字符长度。输入参数为任意字符串。

请严格遵循以下格式进行回答：

Question: 你需要回答的输入问题
Thought: 你应该始终思考接下来要做什么
Action: 你要采取的行动，必须是 [get_weather, calculate_length] 之一
Action Input: 采取该行动的输入参数
Observation: 动作的结果
... (Thought/Action/Action Input/Observation 可以重复 N 次)
Thought: 我现在知道最终答案了
Final Answer: 针对原始问题的最终答案

---开始---<|im_end|>
<|im_start|>user
Question:北京天气怎么样？
Thought:<|im_end|>
<|im_start|>assistant
```

模型续写 →  `我需要查询北京的天气信息。\nAction: get_weather\nAction Input: 北京\nObservation:`

但你的代码设了 `stop=["Observation:"]`，所以模型在生成到 `Observation:` 时被**强制截断**，实际返回的是：

```
我需要查询北京的天气信息。
Action: get_weather
Action Input: 北京
```

然后你的 Python 代码用**正则** `re.search(r"Action:\s*(\w+)", reply)` 把 `get_weather` 抠出来，执行真实函数，再手动拼接结果。

### 第 2 次调用

```
<|im_start|>system
...（同上）...<|im_end|>
<|im_start|>user
Question:北京天气怎么样？
Thought:<|im_end|>
<|im_start|>assistant
我需要查询北京的天气信息。
Action: get_weather
Action Input: 北京<|im_end|>
<|im_start|>user
Observation:北京天气晴朗，气温25°C
Thought:<|im_end|>
<|im_start|>assistant
```

模型续写 → `我现在知道最终答案了\nFinal Answer: 北京天气晴朗，气温25°C。`

### 经典做法的问题

```
❌ 工具描述 = 纯文本自然语言，没有 JSON Schema，参数类型、是否必填全靠模型"猜"
❌ 解析 = 正则表达式，模型输出格式稍微变一下就炸
❌ Thought = 混在正式输出里，用户会看到"我需要查询..."这种内心独白
❌ 一切都靠 prompt 约束，模型没在训练时见过这个格式，全靠 few-shot / 指令遵循能力
```

---

## 三、现代 ReAct：模型真正看到的文本

现在用 Qwen3 这类模型，同样的任务，模型看到的是：

### 第 1 次调用

```
<|im_start|>system
You are Qwen, a helpful assistant.

# Tools

You may call one or more functions to assist with the user query.

You are provided with function signatures within <tools></tools> XML tags:
<tools>
{"type":"function","function":{"name":"get_weather","description":"获取指定城市的天气","parameters":{"type":"object","properties":{"city":{"type":"string","description":"城市名"}},"required":["city"]}}}
{"type":"function","function":{"name":"calculate_length","description":"计算一段文本的字符长度","parameters":{"type":"object","properties":{"text":{"type":"string","description":"任意字符串"}},"required":["text"]}}}
</tools>

For each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:
<tool_call>
{"name": <function-name>, "arguments": <args-json-object>}
</tool_call><|im_end|>
<|im_start|>user
北京天气怎么样？<|im_end|>
<|im_start|>assistant
```

模型续写 →

```
<think>
用户想知道北京的天气情况。我有 get_weather 工具可以查询城市天气，
需要传入城市名参数。让我调用这个工具。
</think>
<tool_call>
{"name": "get_weather", "arguments": {"city": "北京"}}
</tool_call>
```

**注意：**
- `<think>...</think>` = 就是 ReAct 的 **Thought**，但这是模型训练时学会的原生行为
- `<tool_call>...</tool_call>` = 就是 ReAct 的 **Action + Action Input**，也是训练时学会的
- 没有正则，没有 `stop` 截断。API 层（llama.cpp / vLLM）检测到 `<tool_call>` 标记后自动解析成结构化数据返回

### 你的代码收到的 API 响应

```python
# response.choices[0].message 长这样：
{
    "role": "assistant",
    "content": None,                    # 正式回答为空，因为模型选择了调用工具
    "reasoning_content": "用户想知道北京的天气情况...",  # <think> 里的内容
    "tool_calls": [
        {
            "id": "call_abc123",
            "type": "function",
            "function": {
                "name": "get_weather",          # 从 <tool_call> JSON 解析出来的
                "arguments": "{\"city\": \"北京\"}"
            }
        }
    ]
}
```

你的代码执行 `get_weather("北京")`，得到结果，喂回去。

### 第 2 次调用

```
<|im_start|>system
You are Qwen, a helpful assistant.

# Tools

You may call one or more functions to assist with the user query.
...（同上，完整的工具描述）...<|im_end|>
<|im_start|>user
北京天气怎么样？<|im_end|>
<|im_start|>assistant
<think>
用户想知道北京的天气情况。我有 get_weather 工具可以查询城市天气，
需要传入城市名参数。让我调用这个工具。
</think>
<tool_call>
{"name": "get_weather", "arguments": {"city": "北京"}}
</tool_call><|im_end|>
<|im_start|>user
<tool_response>
北京天气晴朗，气温25°C
</tool_response><|im_end|>
<|im_start|>assistant
```

模型续写 →

```
<think>
工具返回了结果：北京天气晴朗，气温25°C。
我可以直接用这个信息回答用户了。
</think>
北京今天天气晴朗，气温约25°C，体感舒适，适合外出活动。
```

---

## 四、对比图：同一个 ReAct 循环，两种实现

```
经典 ReAct（你的代码）                     现代 ReAct（原生 tool calling + think）
========================                  ====================================

[system prompt 里写格式说明]              [system prompt 里放 <tools> JSON Schema]
        ↓                                          ↓
模型生成纯文本:                            模型生成带特殊标记的文本:
"Thought: 我需要查天气                     "<think>
 Action: get_weather                       用户想知道北京天气...
 Action Input: 北京                        </think>
 Observation:"                             <tool_call>
        ↓                                  {"name":"get_weather","arguments":{"city":"北京"}}
stop=["Observation:"] 截断                 </tool_call>"
        ↓                                          ↓
正则提取 Action/Input                      API 层自动解析 <tool_call> JSON
        ↓                                          ↓
Python 执行 get_weather("北京")            Python 执行 get_weather("北京")
        ↓                                          ↓
手动拼: "Observation:晴天25°C\nThought:"   API 拼: "<tool_response>晴天25°C</tool_response>"
        ↓                                          ↓
塞进 messages role="user"                  塞进 messages role="tool"
        ↓                                          ↓
第2次调用，模型输出 Final Answer            第2次调用，模型输出 <think>+正式回答
```

---

## 五、多步推理的完整文本流（现代版）

假设用户问："北京天气怎么样？另外帮我算一下'你好世界'这个字符串有多长"

### 第 1 次调用，模型看到：

```
<|im_start|>system
You are Qwen, a helpful assistant.

# Tools
...（get_weather + calculate_length 的 JSON Schema）...

For each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:
<tool_call>
{"name": <function-name>, "arguments": <args-json-object>}
</tool_call><|im_end|>
<|im_start|>user
北京天气怎么样？另外帮我算一下"你好世界"这个字符串有多长<|im_end|>
<|im_start|>assistant
```

模型续写 →

```
<think>
用户问了两个问题：
1. 北京天气怎么样 → 需要调用 get_weather
2. "你好世界"字符串有多长 → 需要调用 calculate_length
我可以同时调用两个工具。
</think>
<tool_call>
{"name": "get_weather", "arguments": {"city": "北京"}}
</tool_call>
<tool_call>
{"name": "calculate_length", "arguments": {"text": "你好世界"}}
</tool_call>
```

### API 层解析后，你的代码收到：

```python
{
    "role": "assistant",
    "content": None,
    "reasoning_content": "用户问了两个问题：\n1. 北京天气...\n2. ...",
    "tool_calls": [
        {
            "id": "call_001",
            "type": "function",
            "function": {"name": "get_weather", "arguments": "{\"city\":\"北京\"}"}
        },
        {
            "id": "call_002",
            "type": "function",
            "function": {"name": "calculate_length", "arguments": "{\"text\":\"你好世界\"}"}
        }
    ]
}
```

你的代码执行两个工具，分别得到结果。

### 第 2 次调用，模型看到的完整文本：

```
<|im_start|>system
You are Qwen, a helpful assistant.

# Tools
...（同上）...<|im_end|>
<|im_start|>user
北京天气怎么样？另外帮我算一下"你好世界"这个字符串有多长<|im_end|>
<|im_start|>assistant
<think>
用户问了两个问题：
1. 北京天气怎么样 → 需要调用 get_weather
2. "你好世界"字符串有多长 → 需要调用 calculate_length
我可以同时调用两个工具。
</think>
<tool_call>
{"name": "get_weather", "arguments": {"city": "北京"}}
</tool_call>
<tool_call>
{"name": "calculate_length", "arguments": {"text": "你好世界"}}
</tool_call><|im_end|>
<|im_start|>user
<tool_response>
北京天气晴朗，气温25°C
</tool_response><|im_end|>
<|im_start|>user
<tool_response>
4
</tool_response><|im_end|>
<|im_start|>assistant
```

模型续写 →

```
<think>
两个工具都返回了结果：
1. 北京天气：晴朗，25°C
2. "你好世界"的长度：4个字符
现在可以综合回答用户了。
</think>
关于你的两个问题：

1. **北京天气**：今天北京天气晴朗，气温约25°C，非常适合出行。
2. **字符串长度**："你好世界"共有 4 个字符。
```

---

## 六、那 `role="tool"` 到底是怎么回事？

你在 API 层写的代码是这样的：

```python
# 你写的代码（API 层）
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "北京天气怎么样？"},
    {"role": "assistant", "content": None, "tool_calls": [...]},  # 模型的工具调用
    {"role": "tool", "tool_call_id": "call_001", "content": "北京天气晴朗，气温25°C"},  # ← 这里
]
```

但模型层面**没有 `tool` 这个角色标记**。Chat Template 在渲染时做了转换：

```
role="tool" + content="北京天气晴朗，气温25°C"

        ↓ Chat Template 渲染

<|im_start|>user
<tool_response>
北京天气晴朗，气温25°C
</tool_response><|im_end|>
```

**`role="tool"` 在模型眼里就是一个 `user` 段，只不过内容用 `<tool_response>` 包裹了。**

模型训练时学会了：看到 `<tool_response>` 标签，就知道这是工具返回的结果，不是用户在说话。

### 完整映射表

| API 层（你写的代码）                         | Chat Template 渲染后（模型真正看到的）                       |
| -------------------------------------------- | ------------------------------------------------------------ |
| `{"role": "system", "content": "..."}`       | `<\|im_start\|>system\n...<\|im_end\|>`                      |
| `{"role": "user", "content": "..."}`         | `<\|im_start\|>user\n...<\|im_end\|>`                        |
| `{"role": "assistant", "content": "..."}`    | `<\|im_start\|>assistant\n...<\|im_end\|>`                   |
| `{"role": "assistant", "tool_calls": [...]}` | `<\|im_start\|>assistant\n<think>...</think>\n<tool_call>...</tool_call><\|im_end\|>` |
| `{"role": "tool", "content": "..."}`         | `<\|im_start\|>user\n<tool_response>\n...\n</tool_response><\|im_end\|>` |
| `tools=[...]`（API 参数）                    | 塞进 system 段的 `<tools>...</tools>` 文本                   |

---

## 七、现代 ReAct 代码长什么样？

```python
from openai import OpenAI

client = OpenAI(base_url="http://127.0.0.1:8080/v1", api_key="1234567890")

# ① 用 JSON Schema 定义工具（不是纯文本描述）
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "城市名"}
                },
                "required": ["city"],
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "calculate_length",
            "description": "计算一段文本的字符长度",
            "parameters": {
                "type": "object",
                "properties": {
                    "text": {"type": "string", "description": "任意字符串"}
                },
                "required": ["text"],
            },
        },
    },
]

available_tools = {
    "get_weather": lambda city: f"{city}天气晴朗，气温25°C",
    "calculate_length": lambda text: str(len(text)),
}


def run_agent(question: str, max_steps: int = 5):
    messages = [
        {"role": "user", "content": question},
    ]

    for step in range(max_steps):
        # ② 直接传 tools 参数，不需要在 prompt 里写格式说明
        response = client.chat.completions.create(
            model="qwen3",
            messages=messages,
            tools=tools,          # ← API 会把这个塞进 system prompt 的 <tools> 标签里
            temperature=0.7,
            # 不需要 stop=["Observation:"]，API 层自动处理
        )

        msg = response.choices[0].message

        # ③ 检查模型是否想调用工具
        if msg.tool_calls:
            # 把模型的决策记录到上下文（包括 think 过程 + tool_call）
            messages.append(msg)

            for tool_call in msg.tool_calls:
                name = tool_call.function.name
                args = eval(tool_call.function.arguments)  # 实际应该用 json.loads

                print(f"🔧 调用工具: {name}({args})")
                result = available_tools[name](**args)
                print(f"📋 结果: {result}")

                # ④ 用 role="tool" 反馈结果（Chat Template 会渲染成 <tool_response>）
                messages.append(
                    {
                        "role": "tool",
                        "tool_call_id": tool_call.id,
                        "content": str(result),
                    }
                )
        else:
            # ⑤ 模型没有调用工具，说明已经有了最终答案
            print(f"✅ 最终回答: {msg.content}")
            break

    else:
        print("⚠️ 达到最大步数")


run_agent("北京天气怎么样？另外帮我算一下'你好世界'有多长")
```

---

## 八、经典 vs 现代，本质区别一张图

```
经典 ReAct（2022）
=================

一切靠 prompt 约束，模型并没有在训练中见过这种格式

┌─ system prompt ──────────────────────────────┐
│ "请用以下格式回答：                              │
│  Thought: ...                                 │
│  Action: ...              ← 纯文本，祈祷模型遵守 │
│  Action Input: ...                            │
│  Observation: ..."                            │
└───────────────────────────────────────────────┘
                 ↓
模型生成纯文本 → 你用正则解析 → 执行工具 → 手动拼回 → 再调用
                 ↑
        任何一步格式出错就全崩


现代 ReAct（2024-2025）
=====================

模型在训练时就学会了 <think>、<tool_call>、<tool_response> 这些标记的含义

┌─ Chat Template 自动处理 ─────────────────────┐
│ tools 参数 → 渲染成 <tools>JSON Schema</tools>│
│ role="tool" → 渲染成 <tool_response>...</>    │
│ 模型输出 <tool_call> → API 解析成结构化数据     │
└───────────────────────────────────────────────┘
                 ↓
模型生成特殊标记 → API 自动解析 → 执行工具 → role="tool" → 再调用
                 ↑
        格式由训练保证，不依赖 prompt 遵循能力
```

### 核心变化

|                 | 经典                                 | 现代                                                        |
| --------------- | ------------------------------------ | ----------------------------------------------------------- |
| **Thought**     | prompt 里要求写 `Thought:`           | 模型自己生成 `<think>...</think>`                           |
| **Action**      | prompt 里要求写 `Action:` + 正则解析 | 模型自己生成 `<tool_call>...</tool_call>` + API 自动解析    |
| **Observation** | 你手动拼 `Observation: 结果`         | 你写 `role="tool"` + Chat Template 渲染成 `<tool_response>` |
| **格式保证**    | 靠 prompt 约束（不可靠）             | 靠训练（可靠）                                              |
| **工具描述**    | 自然语言随便写                       | JSON Schema 严格定义                                        |
| **解析方式**    | 正则表达式（脆弱）                   | API 层结构化解析（稳定）                                    |
| **多工具并行**  | 几乎不可能（格式太难控制）           | 模型可以一次输出多个 `<tool_call>`                          |

**本质：ReAct 的思想（Think → Act → Observe 循环）完全没变，但从"prompt hack + 正则解析"进化成了"训练级原生支持 + 结构化协议"。**