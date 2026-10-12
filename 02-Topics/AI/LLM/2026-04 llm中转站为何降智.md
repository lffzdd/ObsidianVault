# API 格式转换导致降智的完整拆解

我按数据流的顺序，把每一层的真实内容展示出来。

## 一、最核心的问题：Thinking（扩展思考）

### 1. Claude 原生 API 的 Thinking 请求

```json
// ======= 用户直接调用 Claude API =======
POST https://api.anthropic.com/v1/messages
{
  "model": "claude-opus-4-6",
  "max_tokens": 16000,
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000    // 允许模型用 10000 tokens 来思考
  },
  "messages": [
    {"role": "user", "content": "证明 √2 是无理数"}
  ]
}
```

### 2. LLM 内部实际看到的（推测的内部表示）

模型本身不认识 JSON，它看到的是内部标记格式，大致如下：

```
<|system|>You are Claude, made by Anthropic.<|end|>
<|human|>证明 √2 是无理数<|end|>
<|assistant|><|thinking|>
```

**↑ 注意这里**：thinking 已开启，所以输入在 `<|thinking|>` 之后截断，让模型自由生成思考内容。

### 3. LLM 的真实完整输出

```
<|thinking|>
我需要用反证法来证明这个经典命题。

假设 √2 是有理数，则存在互质的正整数 p, q 使得 √2 = p/q
两边平方：2 = p²/q²
所以 p² = 2q²
这意味着 p² 是偶数，所以 p 是偶数
设 p = 2k，代入得 4k² = 2q²，即 q² = 2k²
所以 q² 是偶数，q 也是偶数
但 p, q 都是偶数，与互质矛盾。

证毕。接下来我用严谨的数学语言组织输出...
<|/thinking|>
## 证明：√2 是无理数

**证法（反证法）：**

假设 √2 是有理数，则存在互质正整数 $p, q$ ...
<|end|>
```

### 4. API 层返回给用户的

```json
{
  "content": [
    {
      "type": "thinking",
      "thinking": "我需要用反证法来证明这个经典命题。\n\n假设 √2 是有理数..."
    },
    {
      "type": "text",
      "text": "## 证明：√2 是无理数\n\n**证法（反证法）：**..."
    }
  ]
}
```

### 5. 当 Thinking 关闭时，LLM 看到的

```
<|system|>You are Claude, made by Anthropic.<|end|>
<|human|>证明 √2 是无理数<|end|>
<|assistant|><|thinking|><|/thinking|>
```

**↑ 关键区别**：API 层直接 prefill 了一个**空的 thinking 块**，thinking 标签已经关闭了。模型从 `<|/thinking|>` 之后开始生成，**被迫跳过思考阶段直接输出答案**。

---

## 二、中转格式转换时发生了什么

用户通常用 OpenAI 兼容格式的客户端（ChatGPT-Next-Web、LobeChat 等），中转需要做 OpenAI → Claude 的格式转换。

### 场景 A：中转根本不支持 thinking

```
┌─────────────────────────────────────────────────────┐
│  用户客户端发出 (OpenAI 格式)                          │
├─────────────────────────────────────────────────────┤
│  POST /v1/chat/completions                          │
│  {                                                  │
│    "model": "claude-opus-4-6",                      │
│    "messages": [                                    │
│      {"role": "user", "content": "证明√2是无理数"}    │
│    ]                                                │
│  }                                                  │
│  // 注意：OpenAI 格式里根本没有 thinking 字段           │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  中转层转换 (one-api / new-api 等)                    │
├─────────────────────────────────────────────────────┤
│  转换为 Claude API 格式：                             │
│  {                                                  │
│    "model": "claude-opus-4-6",                      │
│    "max_tokens": 4096,                              │
│    // ❌ 没有 thinking 字段！因为用户没传，转换层也不加   │
│    "messages": [...]                                │
│  }                                                  │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  LLM 实际看到的                                      │
├─────────────────────────────────────────────────────┤
│  <|human|>证明 √2 是无理数<|end|>                     │
│  <|assistant|><|thinking|><|/thinking|>              │
│                                                     │
│  ← thinking 被自动填空封死了！模型被迫不思考直接回答      │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  LLM 输出 (无思考, 质量下降)                           │
├─────────────────────────────────────────────────────┤
│  <|/thinking|>                                      │
│  √2 是无理数，可以用反证法证明。                        │
│                                                     │
│  假设 √2 = p/q ...                                  │
│  ← 没有内部推理过程，直接"凭感觉"输出                    │
│  ← 对于复杂问题，容易出现逻辑跳跃、遗漏步骤              │
└─────────────────────────────────────────────────────┘
```

**这就是最大的降智来源**：对于简单问题（"你好"、"翻译这句话"），有没有 thinking 差别不大。但对于需要深度推理的任务（数学证明、复杂代码、多步逻辑），thinking 是质量的根本保障。没有 thinking，模型就像一个人被要求"不准在草稿纸上演算，直接写答案"。

### 场景 B：中转支持 thinking，但多轮对话丢失思考上下文

这是更隐蔽的问题。

第一轮没问题：

```json
// ===== 第一轮：用户提问，模型思考后回答 =====

// Claude API 返回：
{
  "content": [
    {"type": "thinking", "thinking": "这个问题需要用动态规划...状态转移方程是..."},
    {"type": "text", "text": "这道题可以用动态规划解决，代码如下..."}
  ]
}
```

第二轮，问题来了：

```json
// ===== 第二轮：用户追问 =====

// Claude 原生 API 要求的正确格式：
{
  "messages": [
    {"role": "user", "content": "用动态规划解这道题"},
    {"role": "assistant", "content": [
      {"type": "thinking", "thinking": "这个问题需要用动态规划...状态转移方程是..."},
      {"type": "text", "text": "这道题可以用动态规划解决..."}
    ]},  // ← 第一轮的 thinking 必须保留在历史中！
    {"role": "user", "content": "能优化空间复杂度吗？"}
  ]
}
```

但中转层在第一轮返回给客户端时，已经把格式转成了 OpenAI 格式：

```json
// 中转返回给客户端的 OpenAI 格式：
{
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "这道题可以用动态规划解决..."
      // ❌ thinking 内容在这里被丢弃了！OpenAI 格式没有 thinking 字段
    }
  }]
}
```

所以第二轮客户端发回来的历史是：

```json
// 客户端发送的第二轮请求（OpenAI 格式）：
{
  "messages": [
    {"role": "user", "content": "用动态规划解这道题"},
    {"role": "assistant", "content": "这道题可以用动态规划解决..."},
    //                    ↑ thinking 没了！
    {"role": "user", "content": "能优化空间复杂度吗？"}
  ]
}
```

中转转换后，LLM 在第二轮看到的：

```json
<|human|>用动态规划解这道题<|end|>
<|assistant|><|thinking|><|/thinking|>这道题可以用动态规划解决...<|end|>
                         ↑ 空的！模型看不到自己之前的推理过程
<|human|>能优化空间复杂度吗？<|end|>
<|assistant|><|thinking|>
```

模型在第二轮思考时，**看不到自己第一轮是怎么推导出状态转移方程的**，只能看到自己的最终回答。这就像你翻回自己的笔记，发现草稿纸被撕掉了，只剩下结论——你得重新推导一遍，而且可能推导出不一致的结果。

---

## 三、System Prompt 的格式差异

```
┌──────────────────────────────────────┐
│  OpenAI 格式                         │
├──────────────────────────────────────┤
│  "messages": [                       │
│    {                                 │
│      "role": "system",              │ ← system 是一条 message
│      "content": "You are helpful"    │
│    },                                │
│    {                                 │
│      "role": "user",                │
│      "content": "Hello"             │
│    }                                 │
│  ]                                   │
└──────────────────┬───────────────────┘
                   │ 中转转换
                   ▼
┌──────────────────────────────────────┐
│  Claude 格式                         │
├──────────────────────────────────────┤
│  "system": "You are helpful",        │ ← system 是独立顶层字段
│  "messages": [                       │
│    {                                 │
│      "role": "user",                │
│      "content": "Hello"             │
│    }                                 │
│  ]                                   │
└──────────────────────────────────────┘
```

这个转换看似简单，但如果中转处理不当（比如把 system 当普通 user message 塞进去，或者多个 system message 的合并出错），LLM 内部看到的消息结构就会畸形。

在 LLM 层面：

```json
// 正确的内部表示：
<|system|>You are helpful<|end|>
<|human|>Hello<|end|>

// 错误的转换可能产生：
<|human|>You are helpful<|end|>    ← system 被当成了 user 消息！
<|human|>Hello<|end|>             ← 连续两条 human，格式异常

// 或者更糟：
<|human|>System: You are helpful
Hello<|end|>                      ← 被拼成一条消息
```

Claude 训练时 system prompt 有专属位置和权重，放错位置 → 模型对指令的遵循程度降低。

---

## 四、Tool Use（工具调用）的格式灾难

```json
// OpenAI 的 function calling 格式：
"tools": [{
  "type": "function",
  "function": {
    "name": "get_weather",
    "parameters": { "type": "object", "properties": {"city": {"type": "string"}} }
  }
}]

// Claude 的 tool use 格式：
"tools": [{
  "name": "get_weather",
  "input_schema": { "type": "object", "properties": {"city": {"type": "string"}} }
}]
```

返回格式差异更大：

```json
// Claude LLM 实际输出：
<|assistant|>
<|tool_use|>{"id": "toolu_xxx", "name": "get_weather", "input": {"city": "北京"}}
<|/tool_use|>
<|end|>

// 需要转换为 OpenAI 格式：
"tool_calls": [{"id": "call_xxx", "type": "function", 
                "function": {"name": "get_weather", "arguments": "{\"city\":\"北京\"}"}}]
//                                                   ↑ OpenAI 的 arguments 是字符串不是对象！

// 然后用户返回的 tool result：
// OpenAI 格式：
{"role": "tool", "tool_call_id": "call_xxx", "content": "晴天 25°C"}

// 需要转回 Claude 格式：
{"role": "user", "content": [{"type": "tool_result", "tool_use_id": "toolu_xxx", "content": "晴天 25°C"}]}
//  ↑ Claude 的 tool_result 是 user role！不是独立的 tool role！
```

一旦转换层在这些细节上出错，LLM 看到的就是**损坏的工具调用上下文**，它会困惑、重复调用、或者放弃使用工具。

---

## 五、完整对比总结

```
直连 Claude API:

用户请求 ──→ Claude API ──→ LLM 输入（完美匹配训练格式）
                                │
                          LLM 输出（thinking + text）
                                │
Claude API ──→ 用户收到完整响应（thinking + text 分离）


经过中转格式转换:

用户请求(OpenAI格式)
    │
    ▼ ❌ 可能丢失：thinking 参数
中转层(OpenAI→Claude 转换)
    │ ❌ 可能错误：system prompt 位置
    │ ❌ 可能错误：tool 格式转换
    │ ❌ 可能篡改：max_tokens, temperature
    ▼ ❌ 可能注入：额外 system prompt
Claude API
    │
LLM 输入（可能畸形、缺 thinking、上下文被截断）
    │
LLM 输出（质量受损）
    │
    ▼ ❌ 可能丢失：thinking 内容
中转层(Claude→OpenAI 转换)
    │ ❌ 可能丢失：tool_use 格式细节
    │ ❌ 可能截断：streaming 不完整
    ▼
用户收到残缺响应
    │
    ▼ 下一轮对话时
❌ thinking 历史已丢失，模型失去推理链条
❌ 多轮越来越"蠢"
```

**核心结论**：格式转换的降智不是某一个单点问题，而是**每一层转换都有信息损失，且损失会在多轮对话中累积**。其中 thinking 的丢失是最致命的，因为它直接砍掉了模型的"内部推理能力"——这相当于把一个能深度思考的人变成了只能脱口而出的人。