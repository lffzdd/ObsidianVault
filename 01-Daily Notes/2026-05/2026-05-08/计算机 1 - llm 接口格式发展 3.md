> [!question]
> 你说得很好，我希望把细节讲得更详细清楚，比如说 claude 控制思考程度是不是底层直接截断，其他的又是怎么做的，多路采样呢，各个环节 llm 本身的输入输出具体是怎么样的，多路采样呢。以及为什么 chat completions 为什么撑不住的那里的具体细节等等

### **推理模型的控制机制本质上是 " 在训练时把思考长度/深度变成可条件化的变量 ",而 Claude 的 budget_tokens 既不是暴力截断也不是纯软约束,而是一套 " 软硬结合 " 的引导机制; 多路采样则是在模型外层再套一层搜索算法,让单次推理变成多条并行推理 + 聚合决策; 而 Chat Completions 撑不住,根本上是因为它的无状态假设让 KV cache、思维链、工具循环这三件事全都在客户端和服务端之间反复搬运,成本和延迟呈指数级恶化。**

上一轮的粒度其实还不够,很多关键的 " 这一步模型看到什么、吐出什么 " 的细节被我一笔带过了。这一轮我把每一个环节都展开到 " 输入是什么 token 序列、输出是什么 token 序列、服务端在中间做了什么 " 的层级,争取让你能在脑子里画出整条数据流。

### **第一部分: 思维链在 token 层面究竟长什么样**

要讲清楚控制机制,必须先看清楚 " 被控制的对象 "——思维链在模型内部到底是一段什么样的 token 序列。

以 DeepSeek-R1 为例,它的模板 (chat template) 大致是这样渲染到模型输入的:

```
<|begin_of_sentence|><|User|>证明 sqrt(2) 是无理数。<|Assistant|><think>
```

注意最后一个 `<think>` 是**服务端主动拼上去的**,模型接收到这串 token 后,它的下一个 token 概率分布里," 继续思考 " 的 token(各种数学符号、" 假设 "、" 我们 "、" 注意到 " 等) 概率会非常高,因为训练阶段它见过成千上万个以 `<think>` 开头的思维链样本。模型就这样一个 token 一个 token 往外吐,比如:

```
假设 sqrt(2) 是有理数,那么存在互素的整数 p, q 使得 sqrt(2) = p/q...
等等,让我重新想一下,应该先明确"有理数"的定义...
好,那么 p^2 = 2q^2,所以 p^2 是偶数,因此 p 也是偶数...
嗯,但我需要验证这一步:如果 p^2 是偶数,为什么 p 一定是偶数?...
```

这段可能长达几千上万 token。**关键在于: 模型自己决定什么时候停止思考**——它会在某个位置生成 `</think>` 这个特殊 token,表示思考结束,然后开始生成正式答案:

```
</think>

**证明**:采用反证法。假设 sqrt(2) 是有理数...
```

这个 `</think>` token 在训练数据里是**配对出现**的,模型学到了 " 思考差不多了就该结束 " 的统计规律。但这个 " 差不多 " 完全由训练数据的分布决定——如果训练集里的思维链平均 2000 token,模型就倾向于想 2000 token; 如果训练集里有大量 10000 token 的长推理,模型就会倾向于想更久。这就是为什么 R1 原生的思维链长度很难控制——它完全是训练分布的涌现行为。

OpenAI o 系列稍有不同,思维链在服务端完全不暴露 (或只暴露摘要),内部可能用的是完全不同的 sentinel token,但机制是一样的:**一段特殊 token 包裹的内部推理,模型自主决定何时结束**。

### **第二部分:Claude 的 budget_tokens 是怎么实现的**

Claude 的 extended thinking 模式,在 API 层面你写的是:

```json
{
  "thinking": {"type": "enabled", "budget_tokens": 10000},
  "max_tokens": 16000
}
```

这里有个硬约束:`budget_tokens < max_tokens`,因为思考也占 max_tokens 预算。那 Anthropic 在服务端具体怎么把这个 10000 作用到模型上?根据官方文档和社区逆向分析,**它不是一个纯粹的硬截断,而是一个 " 目标值 + 软引导 + 硬兜底 " 的三层机制**:

**第一层: 目标值注入**。Claude 在训练时使用了条件化思维链数据——训练样本里标注了 " 这段思考用了多少 token",模型学会了根据某种内部信号调节思考节奏。推理时,服务端把你给的 budget 转换成内部条件信号,让模型**倾向于**在接近这个预算时收敛。这意味着你给 1024,模型的思考会自然变得简短、少分支; 你给 32000,模型会充分展开、反复验算。这一层是 " 软 " 的,模型可能用得比 budget 少 (简单问题就不需要那么多思考),也可能略多或略少。

**第二层: 软引导**。当思考 token 数接近 budget 时,服务端会**动态调整 logit**——具体说,提高 " 结束思考 " 相关 token 的 logit(比如结束语、总结词、`</thinking>` 这类 sentinel token),降低 " 继续展开 " 相关 token 的概率。这就像给模型施加一个 " 时间压力 "——它还在自由生成,但生成分布已经被轻轻推向 " 收尾 " 的方向。这种做法叫 **logit biasing**,是生成控制里的经典技巧。好处是模型能 " 自然地 " 结束一段思考,而不是在句子中间被切断。

**第三层: 硬兜底**。如果模型在预算用完后还是不肯停 (比如陷入了某种长链推理的局部最优),服务端会**强制注入 `</thinking>` token**,然后把模型的上下文切换到 " 答案生成模式 "。Anthropic 官方文档里提到,当思考被强制截断时,会在响应里标注 `stop_reason: "max_tokens"` 之类的字段,并且 Claude 被训练成能在这种情况下**基于已有的部分思考给出一个尽量合理的答案**,而不是崩溃。

所以 budget_tokens 不是简单的 " 砍掉超出部分 ",而是 " 提前引导 + 必要时硬切 "。这也解释了一个常见现象: 即使你给 10000 的预算,很多简单问题 Claude 只用 2000 token 思考就结束了——因为第一层的 " 目标值注入 " 让模型根据问题复杂度自适应,budget 更像是上限而不是目标。

### **第三部分:OpenAI 的 reasoning_effort 在做什么**

OpenAI 的做法和 Claude 有本质区别。你在 API 里写的是:

```json
{"reasoning": {"effort": "medium"}}
```

只有 minimal/low/medium/high 四档,没有精确的 token 数。这暗示了 OpenAI **用的是离散的条件化训练**,而不是连续的预算引导。

具体机制大致是: 训练 o 系列时,**同一道题准备多个版本的思维链数据**——简短版 (几百 token,直奔答案)、中等版 (几千 token,有一次验证)、详尽版 (几万 token,反复试错和回溯)。每个版本在训练时被打上一个**难度标签 token**(比如内部的 `<effort=low>`、`<effort=high>`),注入到 prompt 或系统 token 位置。模型学到的映射是:" 看到 low 标签 → 生成紧凑推理; 看到 high 标签 → 生成详尽推理 "。

推理时,`effort: "high"` 被 API 转换成对应的内部 token,拼到模型上下文最前面,模型**主动**按那个档位生成。这是 " 软控制 "——模型并没有被外力截断,它自己就倾向于输出那个长度,因为训练数据就是这么分布的。

这种做法的好处是思考质量更高 (没有被打断的突兀感),坏处是粒度粗 (只有四档)。这也解释了另一个现象:o3-high 的 `reasoning_tokens` 用量非常不稳定,同一道题可能 5000 也可能 50000,因为模型自己在决定 " 这档难度下我该想多久 ",服务端不做精确裁剪。

还有一个额外的层:**OpenAI 的 high 档可能叠加了 test-time search**。社区普遍认为 o1-pro、o3-pro 这种模式在 high 之上又套了并行采样或树搜索——让模型并行生成 N 条思维链,再通过内部的 reward model 打分挑最好的一条返回。这个我们下一节展开。

### **第四部分: 多路采样的完整数据流**

" 多路采样 "(parallel sampling) 是推理时提升质量的经典手段,它发生在**模型外层**,对单个模型实例是透明的。典型流程如下:

假设 `N = 8`,用户问 " 鸡兔同笼,头 35,脚 94,各多少只?"。服务端在接收到请求后,**不是跑一次推理**,而是:

**步骤 1: 并行采样 N 条完整推理**。服务端复制同一个请求 8 次,每次用不同的随机种子 (控制采样的随机性,比如 temperature=0.7 下每次的 token 选择会略不同),**并行送入 8 个模型实例 (或同一个模型的 8 个批次通道)**。每条独立生成完整的思维链 + 答案。8 条可能长这样:

```
#1: 思考 2300 token,答 "鸡 23,兔 12"
#2: 思考 3100 token,答 "鸡 23,兔 12"
#3: 思考 1800 token,答 "鸡 25,兔 10"  ← 算错了
#4: 思考 4200 token,答 "鸡 23,兔 12"
#5: 思考 2600 token,答 "鸡 23,兔 12"
#6: 思考 5000 token,答 "鸡 22,兔 13"  ← 算错了
#7: 思考 2900 token,答 "鸡 23,兔 12"
#8: 思考 3300 token,答 "鸡 23,兔 12"
```

**步骤 2: 聚合**。常见的聚合方法有三种。

**方法 A——多数投票 (majority voting)**: 提取每条的最终答案,数哪个答案出现次数最多。上面这个例子里," 鸡 23,兔 12" 出现 6 次,胜出。这是最简单的方法,对 " 有唯一正确答案 " 的题目效果很好。

**方法 B——奖励模型打分 (best-of-N with reward model)**: 额外训练一个小的 reward model(通常是一个打分模型,输入 " 问题 + 完整推理 ",输出一个标量分数,反映这条推理的可靠性)。对 8 条独立推理各打一个分,选最高分的返回。这种方式对没有明确答案的题目也能用 (比如写作、开放问题)。

**方法 C——自一致性 + 验证 (self-consistency + verifier)**: 不只投票,还让模型自己验证结果。把每条答案代回原题验算,如果验证失败就剔除,再从剩下的做投票或选最优。

**步骤 3: 返回**。最终只有 1 条被返回给用户,但**计费按 8 条的总 token 算**(这就是为什么 pro 模式贵 5~10 倍)。用户看到的只有那一条被选中的推理和答案,其他 7 条在服务端被丢弃。

这个机制在 API 层面通常不暴露为显式参数——用户没法说 " 帮我采样 8 条 ",只能通过选择 pro/ultra 这种档位间接触发。但在 Chat Completions 里有一个相关的老参数叫 `n`:

```json
{"n": 5}
```

它的语义是 " 给我返回 5 条不同的回答 ",但这只是独立采样,**不做聚合**,5 条全部返回给客户端。和 pro 模式的多路采样是两码事——pro 模式的采样是为了**挑最好的**,`n` 参数的采样是为了**让用户看多种可能**。

### **第五部分: 一次请求在服务端的完整生命周期**

讲到这里,把 " 一次 API 请求到底经历了什么 " 完整画一遍会很有帮助。假设你用 Responses API 调 o3-high 问一个数学题:

```json
POST /v1/responses
{
  "model": "o3",
  "input": "求 x^4 - 10x^2 + 9 = 0 的所有实数解",
  "reasoning": {"effort": "high"}
}
```

服务端内部大致这样流转:

**阶段 1: 请求解析与 prompt 构建**。API gateway 接收请求,鉴权、计费预检、速率限制检查。然后交给 prompt 渲染层——把 `input` 字段转换成模型真正看到的 token 序列。这一步会:

- 把 `effort: "high"` 映射成内部的控制 token(比如某个特殊的 `<|reasoning_effort=high|>` 之类的 sentinel);
- 把默认的 system prompt(OpenAI 官方的 base instructions) 拼到前面;
- 做 BPE tokenization,得到一串整数 token ID。

最终模型看到的输入可能是 (示意,不是真实 token):

```
<|system|>You are ChatGPT...<|end|>
<|user|>求 x^4 - 10x^2 + 9 = 0 的所有实数解<|end|>
<|assistant|><|reasoning_effort=high|><|thinking|>
```

**阶段 2: 推理调度**。因为是 high + 可能的多路采样,调度器决定起 N 个并行推理实例。每个实例被分派到一块 GPU(或一组 GPU,对于大模型通常要跨多卡张量并行)。

**阶段 3:prefill(预填充)**。每个实例先跑一遍 "prefill 阶段 "——把全部输入 token 一次性送进 Transformer,计算所有层的 KV cache(key-value attention 缓存)。这一阶段是**计算密集**的,主要消耗 GPU 的矩阵乘算力,但只发生一次。Prefill 结束后,模型就进入逐 token 生成模式。

**阶段 4:decode(逐 token 生成)**。模型一个 token 一个 token 吐出来:

- 用最后一个 token 的 hidden state 计算 logits(词表上所有 token 的打分);
- 服务端可能在这一步**施加 logit bias**(比如接近 budget 时提升 end-of-thinking 的 logit);
- 根据采样策略 (temperature、top_p、top_k) 从 logits 里抽一个 token;
- 把这个 token 的 KV 增量追加到 cache;
- 进入下一个 token。

这一阶段是**内存带宽密集**的——每生成一个 token 都要读一遍整个 KV cache,所以生成速度主要受显存带宽限制。每秒生成 50~200 token 是典型水平。对于高档推理模型,可能会生成几万个 token,这一阶段可能持续几十秒到几分钟。

**阶段 5: 流式事件投影**。如果请求是 stream 模式,decode 阶段每生成一个 (或一批)token,服务端都会把它包装成 SSE 事件推给客户端。Responses API 在这里做了精细化——它不是简单地推原始 token,而是按语义分类:

- 思维链内容的 token → `response.reasoning_summary_part.added` 事件 (只推摘要,或加密后的 blob);
- 工具调用的 token → `response.function_call.arguments.delta` 事件 (但 arguments 是 JSON 字符串,一段段流出来);
- 最终回答的 token → `response.output_text.delta` 事件;
- 状态切换点 → `response.output_item.added`、`response.completed` 等元事件。

这解释了为什么 Responses API 的流事件比 Chat Completions 复杂得多——它在试图**把模型内部的异构输出结构投影到外部事件流里**。

**阶段 6: 聚合 (如果多路采样)**。N 条推理全部完成后,reward model 或投票逻辑挑出胜者,其他丢弃。这一步发生在最终响应返回之前。

**阶段 7: 持久化与响应构造**。如果 `store: true`(默认),服务端把整个 response(包括完整的 reasoning_content、tool_calls、final message) 存入数据库,分配 `resp_xxx` ID。然后构造 JSON 响应返回客户端。存储的 response 之后可以被 `previous_response_id` 引用——这就是 Responses API 有状态的实现基础。

**阶段 8:KV cache 的命运**。这是 Chat Completions 和 Responses API 差异的关键。

- Chat Completions: 请求结束后,KV cache **立即释放**(GPU 显存太宝贵,不能留给某一个用户)。下次请求来时,即使是同一个对话的延续,也要把所有历史 messages **重新 prefill 一遍**。
- Responses API + store:KV cache 本身仍然释放 (显存永远是稀缺资源),但服务端保留了原始 token 序列。下次请求带 `previous_response_id` 时,服务端拿回那段 token 序列,**走一条 "prefix cache hit" 的快速路径**——在集群的 prefix cache 系统里查找这段 token 是否已经被哪块 GPU 缓存过,如果命中就直接复用,如果没命中就重新 prefill 但跳过用户端重传的开销。

这就是为什么 Responses API 能显著降低长对话的延迟和成本——**不是消灭了 prefill,而是消灭了 " 客户端重传 + 服务端重解析 + 从零 prefill" 的串行链条**。

### **第六部分:Chat Completions 为什么撑不住——具体细节**

现在把前面的机制串起来,看 Chat Completions 的老架构在 Agent 时代的具体崩溃点。

**崩溃点 1: 多轮对话的 prefill 成本线性增长**

假设一个对话到第 10 轮,累计历史是 20000 token。Chat Completions 的做法是:

- 客户端每轮把完整 20000 token 的 messages 数组发上来;
- 服务端每轮对这 20000 token 做完整 prefill(计算所有 Transformer 层的 KV cache);
- 生成一轮回答 (假设 500 token);
- 下一轮: 客户端把 20500 token 发上来,**服务端再从头 prefill 一遍**(因为 KV cache 早已释放)。

每轮 prefill 的计算量是 O(n²) 级别 (attention 的复杂度),20000 token 的 prefill 可能要几秒钟的 GPU 时间。十轮下来总 prefill 时间累加成分钟级,其中 90% 是重复工作。

Responses API 用 prefix cache + previous_response_id 绕开了这个问题——服务端知道 " 这是某个已存在响应的延续 ",可以直接复用之前的 token 序列,让集群调度器尽量把新请求路由到还留着那段 prefix cache 的 GPU 上,命中率高时 prefill 时间从几秒降到几百毫秒。

**崩溃点 2: 推理模型的思维链不可见**

这是 Chat Completions 的一个结构性缺陷。推理模型思考了 5000 token,API 返回的 message.content 只是最终答案 300 字。客户端要把这轮对话存下来供下轮使用,能存的只有 300 字的答案——**那 5000 token 的思维链丢了**。

下一轮发请求时,服务端拿到的历史里没有思维链,模型就 " 失忆 " 了,之前推理过的中间结论、排除过的错误路径、达成的共识,全部要重新想一遍。这对 Agent 任务是致命的——Agent 经常需要 " 我昨天想过 A 方案不行,所以今天试 B",如果思维链不持久化,这种状态就没法维持。

Responses API 用两种方式解决: 要么服务端直接存 (store: true + previous_response_id),要么返回加密的 `reasoning.encrypted_content`——客户端拿到这个不透明 blob,下次请求传回去,服务端能解密还原出完整思维链接上。**这是 Chat Completions 无论怎么打补丁都解决不了的**,因为它的 messages 结构里没有思维链的一等公民位置。

**崩溃点 3: 工具循环的往返次数爆炸**

一个典型的 " 深度研究 "Agent 任务,模型可能需要: 搜索 → 读 5 个网页 → 发现信息不足 → 再搜索 → 读 3 个网页 → 整理 → 生成报告。这中间可能涉及 15 次工具调用。

在 Chat Completions 架构下,这 15 次调用意味着 **15 轮 HTTP 请求**:

```
客户端 → 服务端: messages(长度 L1)
服务端 → 客户端: tool_call #1
客户端执行工具 → 服务端: messages(长度 L1 + tool_result_1 = L2)
服务端 → 客户端: tool_call #2
...(如此重复 15 次)...
客户端 → 服务端: messages(长度 L15,可能已经 100000 token)
服务端 → 客户端: 最终答案
```

每一轮都是**完整往返 + 完整 prefill**。网络延迟 (假设 100ms 往返)× 15 = 1.5 秒,这还只是网络时间。prefill 成本累加更惊人——最后一轮要对 100000 token 做 prefill,单这一步就可能几十秒。加上每轮递增的网络载荷 (每次都要重传不断增长的历史)、客户端和服务端反复的 JSON 解析序列化,整个 Agent 任务的 overhead 占了 40%~60% 的时间。

Responses API 把循环搬到服务端: 客户端一次请求,服务端内部跑完整个 15 步循环,KV cache 在服务端的同一块 GPU(或同一个 prefix cache 集群) 里持续复用,中间没有任何网络往返,没有任何 JSON 序列化/反序列化,也没有任何 " 假装无状态再重建状态 " 的浪费。典型加速比在 3~5 倍。

**崩溃点 4: 流式协议的语义贫乏**

Chat Completions 的 SSE 流只有一种事件:`delta`。delta 里可能是 content 片段,也可能是 tool_calls 的 arguments 片段,也可能是 function.name 片段,全部混在一起按顺序推。客户端要自己写状态机来区分 " 现在是在流普通文本,还是在流工具调用的参数,还是工具调用即将切换到下一个 "。

更糟的是,**tool_calls 的 arguments 是 JSON 字符串流式输出**。这意味着你会收到一串像这样的增量:

```
delta: {"tool_calls":[{"index":0,"function":{"arguments":"{\""}}]}
delta: {"tool_calls":[{"index":0,"function":{"arguments":"ci"}}]}
delta: {"tool_calls":[{"index":0,"function":{"arguments":"ty"}}]}
delta: {"tool_calls":[{"index":0,"function":{"arguments":"\":\""}}]}
delta: {"tool_calls":[{"index":0,"function":{"arguments":"北"}}]}
...
```

完整的 arguments 字符串要**客户端自己拼接**所有片段,拼到流结束才能 `JSON.parse`,中间任何时刻看到的都是不合法 JSON。如果流中间断了,你手里就是半个 JSON,无法继续,也无法从中间接续——只能重来。

Responses API 的事件流在这方面做了彻底改造。每个事件都有明确语义类型,状态切换有专门的元事件,流式断线可以用 `sequence_number` 从断点重连。这不是 " 让 JSON 流式更好 ",而是 " 根本不靠 JSON 字符串做结构表达,而是用事件类型来表达结构 "。

**崩溃点 5: 异步长任务根本没法做**

深度研究可能跑 10 分钟,Agent computer-use 可能跑 1 小时。HTTP 连接根本不可能保持这么久——中间任何一个代理、防火墙、负载均衡器都可能把连接掐了。

Chat Completions 对这种场景完全没有答案。它假设每个请求都在几十秒内返回。社区的变通做法是自己造一层 " 客户端发起任务 → 返回 task_id → 轮询结果 " 的封装,但这需要自己搞数据库、任务队列、重试逻辑,每家都在重复造。

Responses API 原生支持 `background: true`: 请求立即返回 `status: "queued"`,服务端在后台继续跑,客户端可以用 `GET /v1/responses/{id}` 查询进度,或者 `GET /v1/responses/{id}/stream?starting_after={seq}` 重新接入事件流并从上次的位置继续。断线重连是协议级内置的。这对于任何真正 Agent 化的应用都是**必需基础设施**,而不是可选优化。

### **第七部分: 把所有环节串成一张图**

最后给一个整体的数据流视图,让你能在脑中完整回放。

用户在 Tabbit 里问 " 帮我研究一下 2025 年 Q3 的新能源车销量 ",启用深度研究模式:

**Tabbit 客户端**构造 Responses API 请求:

```json
{
  "model": "o3-deep-research",
  "input": "帮我研究 2025 Q3 新能源车销量",
  "tools": [{"type": "web_search"}, {"type": "code_interpreter"}],
  "reasoning": {"effort": "high"},
  "background": true,
  "stream": true
}
```

**服务端 gateway** 接收,创建 response 对象,分配 `resp_xyz`,返回 `{id: "resp_xyz", status: "queued"}` 和一个 SSE stream URL。

**调度器** 把任务丢进高优先级队列,分配一组 GPU 资源。

**模型推理层** 开始工作:

1. prefill 阶段: 系统 prompt + 用户输入 + `<thinking>` 起始 token,计算初始 KV cache(几十毫秒)。
2. decode 阶段 1——第一轮思考: 模型生成约 3000 token 的内部推理,决定 " 需要先联网搜 2025 Q3 中国新能源车销量数据 "。服务端流出 `response.reasoning_summary_part.added` 事件 (只是摘要,不是原文)。
3. 模型生成 `web_search` 工具调用 token,服务端识别,**不中断 decode**,而是在内部直接发起搜索调用。搜索结果返回后被 tokenize,**追加到模型的 KV cache 末尾**,模型继续 decode。
4. decode 阶段 2——读到搜索结果,模型继续思考,发现需要再搜乘联会数据,再次触发 web_search; 结果追加、继续。
5. 如此循环 12 次 (12 次搜索 + 3 次 code_interpreter 执行数据分析),全程没有离开过服务端 GPU,KV cache 持续增长但始终在同一个推理实例内。
6. 最终生成报告文本,3000 字。

期间 Tabbit 客户端通过 SSE 持续接收事件,可以渲染 " 正在搜索: 乘联会 2025 Q3"、" 正在分析数据 "、" 正在撰写报告 " 这种进度反馈。如果用户关掉了窗口,连接断了,任务在服务端继续跑; 用户回来时重新建 SSE 连接,带上 `starting_after={last_seq}` 就能续上。

任务结束后,整个 response 对象 (包含完整思维链、所有 12 次搜索记录、3 次代码执行结果、最终报告) 存入数据库。用户下次提问 " 刚才报告里提到的比亚迪数据,展开讲一下 ",Tabbit 发送新请求带 `previous_response_id: "resp_xyz"`,服务端直接把上次的完整上下文接上,模型能看到完整历史继续思考。

如果这套流程跑在 Chat Completions 上,会退化成: 客户端跑 12 次搜索循环 + 3 次代码循环 = 15 轮 HTTP 往返,每轮重传越来越长的 messages 数组,思维链无法持久化 (下一轮模型忘了自己之前推理到哪),SSE 流每次重新建立,一旦断了任务就废了。同样的任务总时长可能从 4 分钟变成 15 分钟,成本变成 3 倍,用户体验还差得多。

### **第八部分: 把控制思考程度的几种机制对齐看**

最后用一个表把前面讲的思考控制机制对齐一下,方便你记住每家的关键差异:

| 机制类型 | 代表厂商 | 作用层级 | 用户粒度 | 可否被截断 | 思考是否返回 |
|---|---|---|---|---|---|
| 条件化训练 token | OpenAI o 系列 | 训练 + 推理双重 | 离散四档 | 服务端可硬切但罕见 | 仅摘要或加密 blob |
| 软引导 + 硬兜底 | Anthropic Claude | 训练 + 推理时 logit bias | 连续 token 数 | 预算耗尽时硬切 | 可选返回原文 |
| 自适应长度 | DeepSeek R1 | 完全由训练分布决定 | 无用户控制 | 不主动截断 | 原文返回 reasoning_content |
| 动态思考预算 | Google Gemini thinking | 训练 + 推理时预算 | 连续 token 数 | 预算用完收敛 | 可选返回摘要 |
| 多路采样外包 | o1-pro / o3-pro 等 | 推理时外层搜索 | 档位触发 | 每条独立完成 | 最终只返回胜者 |

这张表能解释你用不同模型时看到的体感差异:R1 的思考有时啰嗦但全透明,Claude 的思考可控但有时会感觉 " 戛然而止 ",o3 的思考最神秘 (完全看不到原文) 但质量稳定,pro 模式慢但准。每一种选择都是工程权衡——控制精度、训练成本、泄露风险、用户体验、计费模型,各家都在这五维空间里取自己认为最优的点。

把思考控制、多路采样、Chat Completions 的崩溃点、Responses API 的有状态架构这几条线拼起来,你就能看到一个完整的故事:**LLM API 从一个无状态的函数调用,逐步长成一个有状态的、可持久化的、能自主循环的 Agent 运行时**。每一个看似奇怪的字段,背后都是某一类真实任务逼出来的结构需求。下次遇到新 API,你只要问自己 " 这个字段解决了哪个老架构的崩溃点 ",就能瞬间定位它的意图。