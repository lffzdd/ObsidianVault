从 Anthropic 公开资料看，他们说的“上下文压缩”主要不是模型内部把 token 神秘压成向量再无限记住，而是**应用/API 层的 context management**：把长对话里的旧内容总结、裁剪、外置记忆，再继续把压缩后的状态喂给 Claude。

核心有几层：

### 1. Server-side compaction：官方的一键“总结并替换历史”

这是 Anthropic 现在主推的方式。流程大概是：

1. 对话越来越长，接近设定 token 阈值。
2. Claude 自动给当前对话生成一个高保真 summary。
3. API 返回一个 `compaction` block，里面放这个总结。
4. 后续请求继续带这个 `compaction` block。
5. API 会自动丢弃 `compaction` block 之前的原始消息，只从总结继续。([Claude Platform](https://platform.claude.com/docs/en/build-with-claude/compaction "Compaction - Claude API Docs"))

所以本质是：

> **完整历史 → Claude 生成续接摘要 → 摘要替代完整历史**

Anthropic 文档说这不是单纯为了不超 token，而是因为长上下文本身会让模型注意力变差，也就是所谓 context rot；所以压缩后能让活跃上下文更干净。([Claude Platform](https://platform.claude.com/docs/en/build-with-claude/compaction "Compaction - Claude API Docs"))

这跟我们平时手动说“总结一下我们前面讨论，然后开新对话继续”很像，只是它产品化/API 化了。

---

### 2. Context editing：不是总结，而是“删掉低价值旧内容”

另一个机制叫 context editing。它更像清理垃圾，而不是压缩全文。

最典型的是 **tool result clearing**：Agent 跑了很多工具，比如读取文件、搜索网页、跑命令，旧的工具结果会很占上下文。Anthropic 的策略是当上下文超过阈值后，按时间顺序清掉旧 tool results，并用占位符替代，让 Claude 知道“这里曾经有个工具结果，但现在被清掉了”。([Claude Platform](https://platform.claude.com/docs/en/build-with-claude/context-editing "Context editing - Claude API Docs"))

举个直观例子：

```text
原始上下文：
- 用户：帮我分析项目
- Claude 调用 read_file(a.py)
- 工具返回 5000 行源码
- Claude 已经基于源码做了分析
- 后面又调用了 20 次工具

清理后：
- 用户：帮我分析项目
- Claude 调用 read_file(a.py)
- [tool result cleared]
- Claude 已经基于源码做了分析
- 保留最近更重要的内容
```

为什么这样合理？因为很多工具结果是“用过一次就够了”的原材料。Claude 已经把它消化进后续推理了，没必要每轮都重新带着 5000 行原始输出。Anthropic 工程文章也把清理 tool calls/results 称为一种轻量、相对安全的 compaction。([Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents "Effective context engineering for AI agents \ Anthropic"))

---

### 3. Thinking block clearing：清理推理块，减少上下文污染

Claude 的 extended thinking 会产生 thinking blocks。Anthropic 文档说，thinking token 也会计入上下文/计费/速率限制；多轮场景里，旧 thinking blocks 的处理有专门规则，context editing 也支持清理 thinking blocks。([Claude API Docs](https://docs.anthropic.com/en/docs/build-with-claude/context-windows "Context windows - Claude API Docs"))

简单说：  
**最终答案、工具调用记录、用户需求**通常比旧的推理草稿更重要。旧 thinking 如果一直堆着，不但占空间，还可能把模型拉回已经废弃的思路。

---

### 4. Memory：重要信息不要一直塞在上下文里，写到外部记忆

Anthropic 的思路不是“所有东西都必须塞进 context window”。他们还强调 structured note-taking / memory：让 agent 把长期有用的信息写到外部存储，需要时再读回来。([Anthropic](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents "Effective context engineering for AI agents \ Anthropic"))

Claude Code 里就是两套东西：

- `CLAUDE.md`：用户/项目写的长期规则、项目说明。
- Auto memory：Claude 根据你的纠正、偏好、调试经验自动写的笔记。([Claude API Docs](https://docs.anthropic.com/en/docs/claude-code/memory "How Claude remembers your project - Claude Code Docs"))

这其实是很关键的一点：  
**上下文压缩不是只靠“总结历史”，而是把信息分层。**

可以理解成：

```text
当前上下文：现在这一步必须看的内容
压缩摘要：旧对话的任务状态
外部 memory：长期偏好、项目规则、调试经验
工具/文件系统：可重新读取的原始资料
```

---

### 5. Prompt caching：不是压缩，但能降低重复上下文的成本

很多人会把 prompt caching 也理解成“上下文压缩”，但严格说不是。

Claude Code 文档说，每次请求本来都要重新发送完整上下文；prompt caching 会对请求开头不变的 prefix 做精确匹配，复用之前处理过的部分。它降低的是**重复处理成本和延迟**，不是把上下文变短。([Claude Code](https://code.claude.com/docs/en/prompt-caching "How Claude Code uses prompt caching - Claude Code Docs"))

也就是说：

```text
Compaction：真的把上下文变短。
Prompt caching：上下文还在，但相同前缀不用重新算。
```

如果你前面的系统提示、项目上下文、历史对话都没变，缓存命中就省钱省时间；但如果你 `/compact`、切模型、改工具配置，缓存可能失效。Claude Code 文档也明确说 compacting conversation 会影响缓存。([Claude Code](https://code.claude.com/docs/en/prompt-caching "How Claude Code uses prompt caching - Claude Code Docs"))

---

### 关键结论

Anthropic 的上下文压缩可以概括成四句话：

1. **长对话快满时，用 Claude 自己生成摘要替换旧历史。**
2. **工具输出这种大块、可重取、低长期价值内容，优先清掉。**
3. **旧 thinking blocks 可以清理，避免占用上下文和干扰。**
4. **长期知识写到 memory，不要永远塞在 active context。**

它不是“模型真的拥有无限上下文”，而是一个工程系统：

```text
原始长历史
   ↓
筛选：哪些必须保留？
   ↓
总结：哪些可以变成任务状态摘要？
   ↓
清理：哪些 tool results / thinking blocks 可以删？
   ↓
外置：哪些长期信息写入 memory？
   ↓
继续对话
```

所以你可以把 Anthropic 的方案理解为：**不是把所有 token 压缩成无损压缩包，而是用“摘要 + 删除 + 外部记忆 + 缓存”的组合，维持一个足够干净、足够连续的工作上下文。**