这个问题问得非常到位。
`AIMessage`、`ToolMessage`、`HumanMessage` 等确实是 **LangChain 的“中间抽象层”**，本质是为了让开发者不用直接处理底层协议。
但你问的是——**模型（LLM）真正看到的是什么？它实际输入输出的 token 到底是什么样？**

我们就来揭掉这层封装，看看“裸协议”长什么样。

---

## 一、LangChain 只是「语法糖」

LangChain 内部会把一组 `Message` 转换成 LLM 能理解的原始格式。
不同模型（OpenAI、DeepSeek、Anthropic、Claude）有不同的接口协议，但本质都类似于：

```json
[
  {"role": "system", "content": "你是一个乐于助人的助手"},
  {"role": "user", "content": "请告诉我北京的天气"},
  {"role": "assistant", "content": [
      {"type": "text", "text": "我来帮您查询北京的天气情况。"},
      {"type": "tool_call", "id": "call_00", "function": {"name": "get_weather", "arguments": "{\"city\": \"北京\"}"}}
  ]}
]
```

这段 JSON 才是模型真正接收到的“输入文本流”。
LangChain 只是负责帮你生成它、解析它、再包回 `AIMessage` 这种结构体。

---

## 二、模型返回的原始输出（function calling 协议）

很准确的问题。我们要区分两个层次：
1️⃣ **模型“生成”的内容（LLM输出的token流）**
2️⃣ **API服务器封装后的响应格式（外层JSON）**。

---

### 一、外层JSON是服务器包装的，不是模型输出的

当你调用 OpenAI、DeepSeek、Anthropic 这些 API 时，HTTP 响应的确是一个 JSON，比如：

```json
{
  "id": "cmpl-xyz",
  "object": "chat.completion",
  "created": 1730977000,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "我来帮您查询北京的天气情况。",
        "tool_calls": [
          {
            "id": "call_00_84RtEoBvcF",
            "type": "function",
            "function": {
              "name": "get_weather",
              "arguments": "{\"city\": \"北京\"}"
            }
          }
        ]
      },
      "finish_reason": "tool_calls"
    }
  ]
}
```

这一整段 JSON 是 **服务器封装的响应结构**，由 API 服务端拼出来的，
方便客户端程序解析出：

- 本次对话 ID
- 使用的模型
- 模型生成的文本（content）
- 工具调用请求（tool_calls）
- token 计数（usage）

模型本身**不会**生成这个 JSON。
这些键名（`id`、`object`、`choices`）是 OpenAI/DeepSeek 的服务器逻辑，不在模型权重里。

---

### 二、模型真正“生成”的是什么？

模型内部生成的，永远是 **一串 token 序列**——也就是文本。
举个例子：

- 如果是普通聊天模式：
	它生成的 token 流可能是

	```
    "我", "来", "帮", "您", "查", "询", "北", "京", "的", "天", "气", "情", "况", "。"
    ```

	服务器会把这些 token 拼成 `"我来帮您查询北京的天气情况。"`

- 如果启用了 *function calling* 协议，模型被训练成在需要时输出特定格式文本，比如：

	```xml
    <function_call>
    {"name": "get_weather", "arguments": {"city": "北京"}}
    </function_call>
    ```

	或者直接输出一段 JSON：

	```json
    {"name": "get_weather", "arguments": {"city": "北京"}}
    ```

	**模型确实在“生成这段 JSON 文本”**，但注意：
	
	- 这段 JSON 是模型在生成文本时“模拟”出的结构；
	- 它并不是 API 响应的外层 JSON；
	- 服务器在收到模型生成的这段结构化文本后，会解析它，再包装成标准响应。

---

### 三、服务器端的转换逻辑

流程可以这样看：

```
你发请求 →
  API服务包装输入消息 → 转成模型的prompt
  ↓
  模型逐token生成输出文本（可能是普通文字，也可能是JSON结构）
  ↓
  服务器解析这些token流：
      - 如果符合function calling格式，就提取 name / args
      - 如果只是文本，就放进content字段
  ↓
  服务器组装成标准化响应JSON → 返回给你
```

换句话说：

|层级|谁生成的|内容形式|举例|
|---|---|---|---|
|模型|LLM权重|纯文本token|`"我来帮您查询北京的天气情况。"` 或 `"{"name":"get_weather","arguments":{"city":"北京"}}"`|
|服务器|API程序|JSON封装|上面那个 `"choices": [...]` 的结构|
|客户端（如LangChain）|Python对象|`AIMessage`、`ToolMessage`等|方便你操作|

---

### 四、那 function calling 的 JSON 是怎么训练出来的？

LLM 在训练时，开发者会向它展示：

> 当用户提问需要调用函数时，你要以这种格式输出：
>
> ```json
> {"name": "get_weather", "arguments": {"city": "北京"}}
> ```

所以模型学会了“在遇到调用情境时生成类似 JSON 的文本”。
这些 JSON-like 文本并非程序解析层原生的“结构体”，只是模型输出的特殊格式字符串。
API 服务器再解析成 JSON 结构，告诉你“模型想调用哪个工具”。

---

### 五、总结一句话

> **模型输出的永远是文本（token 序列）；JSON 是服务器对这串文本进行结构化封装和解释的结果。**

---

如果你愿意，我可以给你演示一个**完全去掉 LangChain 和 API 层封装**的例子，
用 `curl` 或 Python 的 `requests` 模拟底层 HTTP 请求，让你看到
模型输出的原始 token 流（以及其中的“伪 JSON”片段）。

要不要我们下一步就这么做？