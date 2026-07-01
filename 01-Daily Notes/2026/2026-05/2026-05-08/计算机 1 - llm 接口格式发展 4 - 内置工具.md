### **内置工具并不是把 web_search 这种功能"塞进"模型权重里,而是 OpenAI 在服务端构建了一整套"工具运行时 + 训练时让模型熟悉这些工具调用习惯"的双层架构;模型本身仍然只是文本生成器,真正干活的是服务端那一圈围绕模型转的搜索引擎、代码沙箱、文件检索系统、虚拟机集群,它们和模型之间通过一种预先约定好的特殊 token 序列对话。**

这是一个非常好的细节问题,因为表面上你看到 API 里写 `tools: [{"type": "web_search"}]`,模型就好像"会上网"了,但实际上模型自己根本不会建立网络连接、不会读 DNS、不会发 HTTP。它只是生成了一段特殊 token,然后服务端某个角落里的另一套系统接管了真正的工作。下面我把这整套机制从底到上拆开讲。

### **第一层:模型本身的能力边界没有变**

最关键的事实先摆在前面:**给模型加上 web_search 这种内置工具,模型权重一个比特都没改**。

GPT-5、o3、GPT-4o 这些模型的 Transformer 参数是固定的,你用 Chat Completions(没有内置工具)调用它,和用 Responses API(带 web_search)调用它,**底层跑的是同一个模型实例**。区别完全在模型周围的"脚手架"里。

这一点非常重要,因为它澄清了一个常见误解:很多人以为内置工具是某种"模型微调"或者"模型升级",其实不是。内置工具是**服务端运行时的能力**,模型只是被训练成"知道在合适的时候说出特定的 token,触发服务端的对应行为"。

类比一下:模型像一个只会说话的人,他不会自己上网,但他知道对着对讲机说"小张,帮我查一下 2025 Q3 销量"——对讲机另一端有一个真正能上网的助手小张,查完把结果念给他听,他继续基于这些信息说话。`web_search` 工具就是模型和"小张"之间约定好的暗号。

### **第二层:训练阶段——让模型学会"说暗号"**

模型怎么"知道"该在什么时候说什么暗号?完全是训练出来的。

OpenAI(以及其他做内置工具的厂商)在模型的后训练阶段,会专门构造**带工具调用的训练数据**。一条典型样本大概长这样(示意,真实格式是 OpenAI 内部的 sentinel token):

```
<|user|>2025 年特斯拉 Q3 销量是多少?<|end|>
<|assistant|><|thinking|>
这个问题需要最新数据,我的训练截止时间无法覆盖,
应该调用 web_search 工具查询。
<|/thinking|>
<|tool_call|>web_search
{"query": "Tesla Q3 2025 delivery numbers"}
<|/tool_call|>
<|tool_result|>web_search
特斯拉公布 2025 年第三季度交付量为 462,890 辆,同比增长 6.4%...
<|/tool_result|>
<|thinking|>
拿到了数据,可以回答用户了。
<|/thinking|>
根据特斯拉官方公布的数据,2025 年第三季度交付量约为 46.3 万辆...
<|end|>
```

注意这条样本里:**工具调用、工具结果、思考、回答全部都是 token**——对模型来说,它们都只是同一个序列里前后衔接的 token。模型在监督微调阶段看过成千上万这种样本,学到的统计规律是:

- "用户问了需要实时信息的问题" → 接 web_search 调用 token;
- "用户问了需要计算的问题" → 接 code_interpreter 调用 token;
- "用户引用了自己上传的文档" → 接 file_search 调用 token;
- "工具返回了结果" → 基于结果继续推理;
- "信息够了" → 生成最终回答。

强化学习阶段会进一步打磨——模型如果调用了不必要的工具(比如问"1+1 等于几"它也去搜索),会被低分惩罚;模型如果该调用工具却没调(比如问"今天的新闻"它就直接编),也会被低分惩罚。多轮迭代后,模型对"什么时候调哪个工具、传什么参数"的判断会变得很准。

**这里有一个非常重要的细节**:这种训练让模型对**那些特定工具名**有了"亲和力"。你看 API 文档里内置工具的名字是固定的几个——`web_search`、`code_interpreter`、`file_search`、`image_generation`、`computer_use`、`mcp`——这不是 API 设计师拍脑袋起的名字,而是**模型在训练数据里反复见过这几个特定字符串**。模型对 `web_search` 这个名字的"调用直觉"远比你自己定义的 `my_search_function` 强,因为前者它见过几百万次,后者它只是临场看到 schema 才学会。

这也解释了一个有趣的现象:**你完全可以用 Chat Completions 的自定义 function calling 模拟出 web_search**——自己定义一个 function 叫 web_search,自己实现搜索后台,自己把结果塞回 messages。功能上完全等价,但你会发现模型调用你这个自定义 web_search 的频率、参数质量、和后续整合的流畅度,都不如用内置 web_search 时好。原因就是内置版本走的是模型"母语级"的工具调用通道。

### **第三层:推理时的握手协议**

训练完的模型上线后,推理时的握手大致是这样的。我们沿用 Tabbit 用户问"2025 Q3 新能源车销量"的例子:

**第 1 步:请求注入**。客户端 Responses API 请求里写了 `tools: [{"type": "web_search"}]`。服务端 prompt 渲染层做两件事:

第一件,把可用工具的描述拼到 system prompt 里,大致像这样(实际格式是模型专属的 sentinel,这里用伪代码示意):

```
<|system|>You are ChatGPT. You have access to the following tools:

web_search(query: string, recency_days?: number) -> SearchResults
  使用此工具搜索互联网获取最新信息。

code_interpreter(code: string) -> ExecutionResult
  在 Python 沙箱中执行代码。

调用工具时使用 <|tool_call|>tool_name\n{json_args}\n<|/tool_call|> 格式。
<|end|>
<|user|>2025 Q3 新能源车销量<|end|>
<|assistant|>
```

注意工具描述是**用自然语言 + token 协议**告诉模型的——模型并没有什么神秘的"工具调用接口",它只是看到 system prompt 里描述了几个可用工具,然后基于训练时养成的习惯决定要不要用。

**第 2 步:模型 decode 到工具调用 token**。模型开始生成,可能先吐一段 thinking,然后吐:

```
<|tool_call|>web_search
{"query": "2025 Q3 中国新能源车销量"}
<|/tool_call|>
```

**关键时刻来了**:推理引擎(vLLM、TensorRT-LLM、或者 OpenAI 自研的 stack)在 decode 过程中**持续监测输出 token 流**。一旦检测到 `<|/tool_call|>` 这个结束 sentinel,推理引擎就知道:"模型完成了一次工具调用请求,先暂停 decode"。

这个暂停**不释放 KV cache**——模型保持着完整的内部状态,只是不再生成新 token,等待工具结果。

**第 3 步:工具执行**。推理引擎把刚才生成的 tool_call 内容(`web_search` + `{"query": "..."}`)发给**工具调度器**。工具调度器是服务端的一个独立服务,它知道:

- `web_search` 路由到搜索后端(可能是 OpenAI 自己运营的搜索集群,或者跟 Bing、Google 等第三方有合作);
- `code_interpreter` 路由到 Python 沙箱集群(Docker/gVisor 隔离的容器);
- `file_search` 路由到向量数据库(OpenAI 的 Vector Store 服务);
- `computer_use` 路由到一组运行 Linux 桌面的虚拟机(每个用户隔离);
- `image_generation` 路由到图像模型(DALL-E 系列或后继者);
- `mcp` 路由到用户在请求里指定的 MCP server。

以 web_search 为例,搜索后端拿到 query 后:发起搜索 API 调用 → 拿到 top N 个 URL → 并发 fetch 这些网页 → 用一个小模型或规则做 HTML 清洗和摘要 → 把结果打包成结构化数据返回。这一步可能持续 2~10 秒。

**第 4 步:结果回灌**。搜索结果回到推理引擎,引擎把它包装成 token 序列,**追加到模型的 KV cache 末尾**:

```
<|tool_result|>web_search
[
  {"title": "乘联会:2025年9月新能源车销量...", "url": "...", "snippet": "..."},
  {"title": "比亚迪Q3财报:销量同比增长...", "url": "...", "snippet": "..."},
  ...
]
<|/tool_result|>
```

注意这一步**不需要重新 prefill**——KV cache 是从模型暂停的那个 token 位置直接延伸的,新追加的 token 只需要做一次 forward(对新增 token 计算 attention),复杂度 O(n) 而不是 O(n²)。这是内置工具相比客户端循环最大的性能优势。

**第 5 步:模型继续 decode**。引擎恢复 decode,模型基于"看到了搜索结果"这个新状态继续生成。它可能:

- 觉得信息够了,直接生成最终回答;
- 觉得还要再搜,吐出第二个 `<|tool_call|>web_search`;
- 觉得需要分析数据,吐出 `<|tool_call|>code_interpreter`,执行 Python 计算各品牌占比;
- 思考一会儿,先吐 thinking,再决定下一步。

整个循环在**同一个推理实例的同一个 GPU 上**进行,KV cache 持续累积,中间没有任何 HTTP 往返、没有任何 JSON 序列化反序列化、没有任何 prompt 重渲染。

**第 6 步:终止条件**。模型最终生成 `<|end|>` 或者达到 max_tokens,或者达到工具调用次数上限(防止无限循环),整轮结束。完整的 output 数组(包含每次工具调用的元数据、思维链摘要、最终消息)被打包返回。

### **第四层:每种内置工具的具体实现**

不同工具的服务端实现差异巨大,值得分开看。

**web_search**。最简单的一类——本质是搜索引擎 + 网页抓取 + 摘要管道。OpenAI 公开的资料显示他们和 Bing 等搜索服务有合作,但具体细节是内部基础设施。模型看到的不是原始 HTML,而是清洗后的摘要 + 引用 URL。返回的 token 里通常带有 source 标记,让模型在最终回答里能正确生成引用脚注。这也是为什么 ChatGPT 的网页搜索回答天然带 `[1]` `[2]` 这种引用——是工具结果格式里就埋了引用 anchor。

**code_interpreter**。一个 stateful 的 Python 沙箱,每次会话分配一个隔离容器,文件系统可读写但和外部完全隔离(不通外网,除了模型新装包的特殊通道),内存和 CPU 有配额。模型生成 Python 代码 → 沙箱执行 → stdout/stderr/异常 + 任何生成的文件(图、CSV)返回。这是真正的代码执行,不是模拟——所以模型可以"算"出正确的 sqrt(2),可以画出真实的 matplotlib 图,可以处理你上传的 Excel 文件。沙箱内还预装了 pandas、numpy、scikit-learn、matplotlib 等几百个常用包。

**file_search**。OpenAI 的 Vector Store 服务。你提前上传文件,服务端做切块 → embedding → 存向量库。模型生成 `file_search({"query": "..."})` 时,服务端在指定 vector store 里做语义检索,返回 top-K 相关切块。模型基于这些切块生成回答,本质上是把 RAG(检索增强生成)流程内置到了 API 里,用户不用自己维护向量库基础设施。

**image_generation**。模型决定要生成图时,吐出 `image_generation({"prompt": "...", "size": "1024x1024"})`,服务端调用图像生成模型(基于 GPT-Image-1 或后继),返回图像的 URL 或 base64。这里有个有趣的点:返回给文本模型的不是图像本身的像素,而是一个"图像引用 token + 文字描述"——文本模型自己不需要"看"那张图,它只需要知道"我生成了一张图,可以在最终回答里引用它"。

**computer_use**。最复杂的一个。服务端维护一组 Linux 虚拟机集群,每个 session 分配一台。模型可以发出 `computer_use({"action": "screenshot"})`、`computer_use({"action": "click", "x": 234, "y": 567})`、`computer_use({"action": "type", "text": "hello"})` 等指令。screenshot 返回的图像被多模态模型"看到",形成下一步决策的依据。这套机制让模型能像人一样使用桌面应用——浏览器、编辑器、各种 GUI 程序。Anthropic 的 Claude Computer Use 工作方式类似,但具体协议有差异。

**mcp**(Model Context Protocol)。这是 2024 年底 Anthropic 推出、后被 OpenAI 也采纳的开放协议。和前面几个"OpenAI 自己维护实现"的内置工具不同,mcp 是一个**桥接协议**——你在 API 请求里指定一个 MCP server URL(可能是你自己运行的,也可能是第三方服务),服务端连接到那个 MCP server,把它声明的所有工具暴露给模型。模型调用时,服务端转发到你的 MCP server。这相当于"内置工具的可扩展插槽",让生态可以自己长出来。

### **第五层:为什么不是简单 function calling 的语法糖**

可能你会想:既然这些工具最终都是"模型说出特定 token → 服务端执行 → 结果回灌",那它和你自己用 Chat Completions 写 function calling 有什么本质区别?为什么需要单独叫"内置工具"?

差异在四个层面。

**差异 1:模型的训练偏置**。前面提过,内置工具的名字和参数模式,模型在训练阶段反复见过,有"母语级"的调用直觉。自定义 function 模型是临场看 schema 学的,理解深度不一样。这种差异在简单场景看不出来,在复杂任务(比如需要连续 10 次工具调用、动态调整策略)里会显现——内置工具的调用决策准确性显著更高。

**差异 2:循环在哪里跑**。自定义 function calling 是**客户端跑循环**——模型说要调函数,服务端把这个意图返回给客户端,客户端执行,把结果塞回 messages,再发请求。每次循环都是一次完整 HTTP 往返 + 完整 prefill。内置工具是**服务端跑循环**——所有循环发生在模型的同一段 KV cache 里,中间不出 GPU,不重新 prefill,不重传 messages。前面讲过的性能差异本质上都源于此。

**差异 3:服务端基础设施**。自定义 function 需要客户端自己实现工具后端——你要自己接搜索 API,自己跑代码沙箱,自己维护向量库,自己管虚拟机集群。内置工具是 OpenAI 已经替你跑好了这些基础设施,你按 token 付费就能用。对小型开发者和个人用户,这是巨大的便利。

**差异 4:可观测性和安全边界**。内置工具是 OpenAI 维护的,他们能在工具层做安全过滤(屏蔽危险网站、禁止恶意代码、隔离沙箱)。自定义 function 的安全责任完全在客户端,出了问题(比如 prompt injection 让模型调用了不该调的内部 API)只能自己背锅。

### **第六层:Anthropic 和 Google 的做法对比**

这套机制不是 OpenAI 独有的,各家有类似但细节不同的实现。

**Anthropic Claude**。和 OpenAI 思路接近,在 API 里提供 `web_search`、`code_execution`、`computer_use`、`bash`(让模型在沙箱里跑 bash 命令)、`text_editor`(让模型修改服务端维护的文件)等内置工具。Anthropic 在 tool_use 块的设计上更"内容块化"——工具调用和工具结果是 messages content 数组里的独立 block,结构上比 OpenAI 的 tool_calls 字段更对称。Computer Use 是 Anthropic 最早商用的,Claude 3.5 Sonnet 时代就上线了。

**Google Gemini**。提供 `google_search`(grounded 搜索,直接对接 Google Search)、`code_execution`、`url_context`(让模型直接读某个 URL 的内容)等。Gemini 的 grounding 体系做得很重——返回结果里会带详细的 source 元数据,每段生成文本都能追溯到具体的搜索结果块。这反映了 Google 自己作为搜索公司的优势。Gemini Live API 还集成了实时的视频和语音工具,可以让模型"看着"摄像头实时对话。

**国内厂商**。阿里通义、字节豆包、智谱、Moonshot 等也都在做类似事情。阿里的 Qwen-Agent 框架、字节的 Doubao Pro 都内置了搜索、代码、文档分析等工具。各家细节不同,但抽象层次都是"训练时让模型熟悉特定工具语法 + 推理时服务端拦截执行 + 结果回灌继续 decode"这套模式。

### **第七层:一个常被忽略的微妙之处——上下文污染**

最后讲一个细节,是工程上的现实痛点。

内置工具看起来很美好,但它有一个隐性成本:**工具结果会占用模型的上下文窗口**。一次 web_search 可能返回 10 个网页的摘要,合起来几千 token;连续 10 次搜索就是几万 token 全部塞进 KV cache。这有两个后果。

第一,**长任务会撞上下文上限**。即使是 200K context 的 Claude 或 GPT-5,深度研究跑久了一样会塞满,塞满之后要么截断早期内容(失忆),要么压缩(信息损失)。Responses API 在这里做了一些工程优化——会自动对历史工具结果做摘要压缩,但不是无损的。

第二,**注意力被稀释**。Transformer 的 attention 在长上下文下质量是下降的,模型可能"看到了"早期搜索结果,但在 100K token 后已经记不清细节了。这也是为什么 deep research 模式会引入显式的"中间报告生成"环节——让模型周期性地把当前进展浓缩成结构化笔记,再丢掉原始搜索结果,腾出上下文空间继续工作。这种"分阶段消化"在 API 层面对用户透明,但在 OpenAI 内部是关键的工程技巧。

理解了这一层,你就能体会到内置工具的本质:**它不是给模型加了一项新能力,而是给"模型 + 服务端运行时"这个组合系统加了一项新能力**。模型本身仍然只是文本生成器,但服务端围绕它构建了一整套工具运行时、工具调度器、KV cache 管理器、上下文压缩器,让这个组合系统对外表现得像一个能上网、能编程、能操作电脑的智能体。下次你看到 `tools: [{"type": "web_search"}]` 这么一行简单的配置,背后其实是 OpenAI 几千张 GPU、上万台 VM、整套搜索基础设施、若干训练阶段精心构造的数据集协同工作的结果——这才是"内置工具"四个字真正的分量。