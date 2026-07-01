### **OpenAI 主要自建搜索基础设施（与 Bing/必应深度集成），Anthropic Claude 则采用自建 + Brave Search 的混合方案，目前两家头部厂商都没有直接调用 Tavily 或 Exa 这类第三方 AI 搜索 API。**

要把这件事说清楚，需要区分三个层次：**搜索引擎索引层**（爬虫、倒排索引、排序）、**LLM 优化层**（query 改写、结果摘要、引用对齐）、**API 暴露层**（开发者怎么调用）。Tavily 和 Exa 主要做的是后两层，把传统搜索能力包装成 " 对 LLM 友好 " 的接口；而 OpenAI 和 Anthropic 选择的路径并不一样。

Based on the searches, here's the full picture:

### **OpenAI:Bing 索引 + 自有爬虫 + 自研排序与摘要**

OpenAI 的 ChatGPT Search / SearchGPT 以及 API 中的 `web_search` 内置工具,**搜索索引层主要依赖 Microsoft Bing**。这一点由 OpenAI 工程副总裁在 AMA 中亲自确认过——Bing 是 ChatGPT 搜索功能的关键组成部分,如果一个网页没有被 Bing 收录,就不会出现在 ChatGPT 的搜索结果里,这和当年 Yahoo Search 用 Bing 作为后端的关系非常类似。这背后既有技术原因 (OpenAI 的基础设施全部跑在 Azure 上,直接对接 Bing API 最自然),也有商业原因 (微软对 OpenAI 数十亿美元的战略投资)。[The Keyword](https://www.thekeyword.co/news/openai-s-chatgpt-search-relies-on-bing-s-index) [The Verge](https://www.theverge.com/2023/5/23/23733189/chatgpt-bing-microsoft-default-search-openai-build)

但 OpenAI 并不是简单地把 Bing 的结果原样透传。他们在上层做了不少自研工作:

- **OAI-SearchBot**:OpenAI 自己运营了一个开放网络爬虫,用来补充 Bing 索引中缺失或时效性不足的内容,但它只是 Bing 结果的 " 补丁 ",而非替代品。
- **独立排序算法**:Seer Interactive 对 500+ 条引用做的实证研究显示,SearchGPT 的引用与 Bing Top 结果的匹配度约 87%——也就是说有相当一部分被 OpenAI 自家的排序/重排逻辑改写过,query 也常被改写简化 (例如把 " 哪些汽车贷款机构对信用差的人最友好 " 改写成 "best auto loan lenders bad credit")。
- **引用对齐与摘要生成**: 把搜索结果切片、做相关性筛选、按 token 预算注入 context、生成带 citation 的回答,这一整条 pipeline 都是 OpenAI 自己在做。[LinkedIn 分析](https://www.linkedin.com/posts/aashishpeepra_i-was-researching-what-openai-uses-internally-activity-7383047466612535298-eV7T) [Seer Interactive](https://www.seerinteractive.com/insights/87-percent-of-searchgpt-citations-match-bings-top-results)

所以准确说法是:**OpenAI 租用 Bing 的索引层,自己做 LLM 优化层和 API 暴露层**,完全没有用 Tavily 或 Exa。

### **Anthropic Claude:Brave Search 作为索引后端**

Anthropic 在 2025 年 3 月给 Claude 上线 web search 功能时,起初没有公开说明索引来自哪里,业内一度猜测他们是不是自建了搜索索引。很快 Simon Willison 等人通过两条证据链锁定了答案:

1. Anthropic 的 **Trust Center 子处理方列表**(subprocessor list) 在 2025 年 3 月 19 日新增了 "Brave Search" 条目。
2. 让 Claude 自己暴露内部 `web_search` 函数定义时,可以看到参数对象的 schema 名字赫然写着 **`BraveSearchParams`**。
3. 同样的 query 在 Claude 和 Brave 上跑,引用结果的 URL 高度一致。

到 2025 年 5 月,Anthropic 通过 API 正式上线 `web_search_20250305` 工具,**底层依然由 Brave 提供**。Anthropic 对开发者收 \$10 / 1000 次搜索,而 Brave Search API 自己零售价是 \$3 / \$5 / \$9 per thousand 不等——中间价差就是 Anthropic 做 query 改写、结果筛选、citation 对齐和模型集成的 " 加工费 "。[TechCrunch](https://techcrunch.com/2025/03/21/anthropic-appears-to-be-using-brave-to-power-web-searches-for-its-claude-chatbot/) [Simon Willison's Weblog](https://simonwillison.net/2025/Mar/21/anthropic-use-brave/) [Anthropic API 文档分析](https://simonwillison.net/2025/May/7/anthropic-api-search/)

值得一提的是,Brave 还给 Mistral Le Chat 提供搜索后端,所以 " 搞 LLM 厂商找 Brave 当索引供应商 " 已经成了一条成熟的行业路径,但从来不是 Tavily/Exa。

### **为什么不是 Tavily 或 Exa?**

要理解这个分工,得看清楚这几家究竟做的是什么:

| 公司 | 自建索引/爬虫 | 主要客户 | 定位 |
|---|---|---|---|
| Bing | 是 (全球级 Web 索引) | Microsoft、OpenAI | 通用搜索引擎 |
| Brave Search | 是 (自有独立索引,数百亿网页) | Brave 浏览器、Anthropic、Mistral | 隐私优先的独立索引 |
| Google | 是 | Google 自家 + Gemini | 通用搜索引擎 |
| Tavily | 否 (聚合 Bing/Google 等) | 中小开发者、LangChain 生态 | LLM-friendly 搜索 API |
| Exa | 是 (自建神经搜索索引,但规模远小于 Bing) | 开发者、研究类应用 | 语义/向量搜索 API |

**头部 LLM 厂商对搜索后端的核心要求是三点**: 索引规模 (几百亿网页起步)、刷新延迟 (分钟级)、QPS 容量 (亿级日调用)。Tavily 本质上是个聚合层 (它再去调 Bing/SerpAPI),把它接进 ChatGPT/Claude 等于多绕一跳、还多付一道钱;Exa 的索引规模和长尾覆盖目前也撑不住 ChatGPT 这种全球级流量。所以对头部模型厂来说,要么直接对接一线索引 (Bing、Brave、Google),要么干脆自建——而自建一个能和 Bing 抗衡的 Web 索引,基础设施投入是十亿美元级的、还要打几年爬虫合规战,这不是个好生意。

**Tavily/Exa 的真正定位**是服务**那些不想/不能自己接 Bing 企业 API 的中小开发者**: 它们把 " 搜索 + 抓取正文 + 切片 + 去广告 + 返回干净 markdown" 打包成一个 API,让 LangChain/LlamaIndex 用户开箱即用。这是一个不同的市场层级,和 OpenAI/Anthropic 的内置工具属于上下游而非竞争。[Brave Search API Guides](https://brave.com/search/api/guides/)

### **顺带说一下其他厂商**

- **Google Gemini**: 不用说,直接用自家 Google Search,内置 `google_search` 工具就是 grounding-on-Google。
- **xAI Grok**: 从 X(Twitter) 实时数据流 + 网络搜索,后者据信也是混合方案,部分依赖第三方索引。
- **Perplexity**: 自己做了爬虫和索引 (PerplexityBot),但也仍然混用 Bing/Google 的结果做兜底,定位介于 "AI 搜索引擎 " 和 "LLM 厂 " 之间。
- **国内厂商**(通义、Kimi、豆包、DeepSeek 等): 基本都是接自家集团的搜索能力 (阿里夸克、字节头条搜索) 或与百度/必应 API 合作,逻辑和 OpenAI-Bing 是同构的。

把这条链条还原回上一轮你关心的 " 内置工具机制 ": 当你在 API 里传一个 `tools: [{type: "web_search"}]`,模型生成 `<|tool_call|>web_search(query="...")<|/|>` 这样的特殊 token 后,服务端的 runtime 接管——OpenAI 的 runtime 去敲 Bing API,Anthropic 的 runtime 去敲 Brave Search API,拿到结果后再重新塞回模型的下一轮 context。**模型权重本身不知道也不在乎搜索是谁做的**,这层抽象正好让厂商可以随时换搜索后端而不用重训模型,这也是为什么 Anthropic 当初能在不发新模型的情况下,几周内就给 Claude 加上 web search 能力。