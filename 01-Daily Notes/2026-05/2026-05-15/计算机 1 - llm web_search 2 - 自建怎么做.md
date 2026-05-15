### **如果你是个人开发者或中小团队自建 Agent，最稳的方案是 "Tavily / Exa 这类 LLM-native 搜索 API + 必要时自建抓取兜底 " 的混合架构; 只有在月调用量超过百万级、或对召回有特殊垂直要求时,才值得直接对接 Bing / Brave 原生 API 自己做加工层。**

下面我把这件事拆成六个维度详细讲清楚——**需求拆解、可选方案矩阵、各方案深度对比、混合架构设计、工程落地细节、成本与风险测算**——帮你根据自己的场景做决策,而不是给一个 " 标准答案 "。

## **一、先把 "web search" 这个需求拆成五个子问题**

很多人写 Agent 时把 web search 想成 " 调一个 API、拿到几个链接 " 就完事,这是低估了它的复杂度。一个生产级 Agent 的搜索环节实际上由五个子问题串起来,每一个都决定最终回答质量:

**子问题 1:Query 改写 (Query Rewriting / Reformulation)**。用户原始 query 往往口语化、含糊、带指代,例如 " 帮我找下昨天那个芯片公司的财报 "——直接送进搜索引擎几乎不可能命中。需要 LLM 先做一轮改写,生成 1\~3 个适合搜索引擎的 keyword query,可能还要拆成多个并行子查询 (multi-query retrieval)。这一步是你**自己**用 LLM 做的,任何搜索 API 都不替你做。

**子问题 2: 索引检索 (Indexing & Retrieval)**。给定一个 query,从全网几百亿网页里捞出最相关的 N 条。这一步需要倒排索引 + 排序模型,工程量是十亿美元级的,**没人会自建**,只能调 API。

**子问题 3: 正文抓取与清洗 (Fetching & Cleaning)**。搜索 API 通常只返回 title + URL + snippet(几十到几百字的摘要),要做深度回答必须把网页正文完整拉下来,还要去广告、去导航、去 cookie 弹窗、提取主体文本。这一步是 Tavily / Exa 等厂商相比纯搜索 API(Bing / Brave / SerpAPI) 的核心增值点。

**子问题 4: 结果筛选与压缩 (Reranking & Compression)**。即便拿到 10 条正文,加起来可能有 10 万 token,远超模型 context 窗口或者会让推理成本爆炸。需要 reranker(可以用 Cohere Rerank、bge-reranker、或直接让 LLM 打分) 挑出真正相关的片段,再做摘要压缩。这一步是你自己实现的。

**子问题 5: 引用对齐 (Citation Grounding)**。最后生成回答时,每个事实点要能溯源到具体 URL 和段落,否则就是幻觉。这需要在 prompt 工程或后处理里加引用校验逻辑。

把这五个子问题摆出来,你就能看出**不同方案各自接管了哪几步、留给你做哪几步**,这是选型的根本依据。

## **二、可选方案矩阵: 六类供应商各司其职**

我把市面上能给 Agent 用的搜索方案分成六类,每一类背后的商业逻辑、适配场景、坑在哪都不同:

**第一类——一线通用搜索引擎 API**:Bing Web Search API(已迁移到 Azure Marketplace,定价约 \$15 / 1k queries,2025 年微软曾宣布逐步关停后又延期)、Google Custom Search JSON API(\$5 / 1k,每天 1 万次硬上限,且只能搜你预先指定的站点集合)、Yandex / Baidu(地域性强)。**优点是召回最全、最权威**; 缺点是返回的 snippet 短、不带正文,而且 Bing API 政策反复、Google CSE 限制严苛,对独立开发者不太友好。

**第二类——独立索引型 API**:Brave Search API(\$3\~9 / 1k,自有几百亿网页索引,Anthropic Claude 在用)、Mojeek(英国独立搜索引擎,索引相对小)、Marginalia(开源小众索引)。**核心价值是 " 独立于 Google/Bing 的第二份索引 "**,在隐私、抗审查、避免单点依赖上有优势,Brave 是这一类里规模和质量最好的。

**第三类——LLM-native 搜索 API**:Tavily(\$0.008 / search 起,免费 1000 次/月)、Exa(原 Metaphor,神经搜索 + 关键词混合,\$5 / 1k 起)、You.com API、Serper、SerpAPI(\$50/月起,本质是 Google 抓取代理)、Linkup、Jina Reader / Jina Search。**这一类的核心增值**是: 返回干净的 markdown 正文 (不只是 snippet)、自动去广告、支持按时间/域名/语言过滤、原生支持 " 搜索 + 抓取 " 一步到位、给 LLM 用的接口设计 (比如直接返回适合塞 context 的格式)。Tavily 和 Exa 是这个赛道的两个头部。

**第四类——纯抓取/Reader API**:Jina Reader(`r.jina.ai/{url}`,免费额度大方)、Firecrawl(\$19/月起,支持 crawl 整站和 JS 渲染)、Browserless、ScrapingBee。**它们不做搜索,只做 " 给一个 URL,返回干净的 markdown"**,通常和搜索 API 配合使用。

**第五类——MCP / 集成层**:Model Context Protocol(Anthropic 推出的) 生态里已经有大量封装好的搜索 server,比如官方的 brave-search MCP server、social-search MCP、各种垂直数据源 MCP。**好处是统一协议、即插即用**,坏处是多一层抽象、调试链路变长。

**第六类——自建路径**: 用 SearXNG(开源元搜索,聚合 Google/Bing/DuckDuckGo 等结果,可自部署)+ 自己的爬虫 + 自己的 reranker。**只在你有特殊需求时才考虑**,例如要搜内网/私域、要做垂直领域 (法律、医学、专利)、或者月调用量已经大到自建比买 API 便宜。

## **三、深度对比:Tavily vs Exa vs Brave vs Bing vs 自建**

我挑出对自建 Agent 最有现实意义的五个方案,从七个维度做一次硬对比。

| 维度 | Tavily | Exa | Brave Search API | Bing Web Search | SearXNG 自建 |
|---|---|---|---|---|---|
| 索引来源 | 聚合 (Bing+Google+ 自爬) | 自建神经索引 + 关键词 | 自建独立索引 | 自建一线索引 | 聚合公开搜索引擎 |
| 单价 (\$/1k) | 8\~25 | 5\~25 | 3\~9 | 15 | 服务器成本 |
| 返回正文 | 是,markdown | 是,markdown | 否 (只 snippet) | 否 (只 snippet) | 否 |
| 延迟 (P50) | 1\~3s | 0.8\~2s | 0.3\~0.6s | 0.2\~0.5s | 1\~5s |
| LLM 友好度 | 极高 | 极高 | 中 | 中 | 低 |

**详细展开每一家的特点**:

**Tavily** 是这个领域目前对独立开发者最友好的选择。它的核心卖点是 `search` 和 `extract` 两个端点合二为一——你传一个 query,它内部自动调底层搜索 (Bing/Google 的聚合)、抓取 top N 网页、用自己的小模型做相关性筛选和摘要,返回一个直接能塞进 LLM context 的 JSON,字段包括 `query`、`answer`(可选的 quick answer)、`results: [{title, url, content, raw_content, score}]`。它还有 `search_depth: "basic" | "advanced"` 两档模式,advanced 会做更深的抓取和去重但更慢更贵。LangChain、LlamaIndex、CrewAI 都有 first-class 集成。**最适合**: 刚起步、想快速跑通 Agent loop、不想自己折腾抓取和清洗的项目。**短板**: 聚合层之上还有一层加工,极端 query 下偶尔会丢掉某些 Bing/Google 原本能召回的结果; 价格在头部里偏中等,大规模用不算便宜。

**Exa**(原名 Metaphor) 走的是另一条路:**自建神经搜索索引**,把 " 语义相似 " 作为一等公民。它有三种搜索模式——`neural`(向量召回)、`keyword`(传统倒排)、`auto`(让模型自己挑)。`neural` 模式特别擅长 " 找类似 X 的网页 ",比如 " 找一些和这篇博客风格类似的技术文章 ",这是 Bing/Google 的关键词匹配做不到的。它也支持 `contents` 端点直接返回 markdown 正文,以及 `findSimilar` 端点 (给一个 URL 找相似页面)。**最适合**: 研究类、推荐类、长尾发现类需求,比如做学术 Agent、做内容推荐 Agent。**短板**: 自建索引规模仍然小于 Bing,新闻/时效性强的 query 召回会差一些,需要和别的源混用。

**Brave Search API** 是 " 严肃做事 " 的选择。它有真·独立索引 (不是元搜索),价格在所有一线索引里最便宜 (\$3\~9 / 1k),延迟也最低 (亚秒级),而且明确没有 Google/Bing 的政治风险 (独立公司、明确的隐私立场)。Anthropic 选它做 Claude 的 web search 后端就很说明问题。**短板**是它就是个纯搜索 API——返回 title + url + description,不返回正文,你需要自己配 Jina Reader 或 Firecrawl 抓取; 另外它的索引虽然独立但仍小于 Bing,英文内容很好,中文/日文/小语种召回明显弱。**最适合**: 量大、对成本敏感、愿意自己组装抓取层的中等规模团队。

**Bing Web Search API** 是 " 曾经的金标准 ",但 2025 年以来不太友好——微软先是宣布 2025 年 8 月 11 日关停 Bing Search APIs,后来部分延期、并把流量推向 "Grounding with Bing Search for Azure AI Foundry" 这个新产品 (只能在 Azure AI Foundry 里用,绑定他们的 Agent 框架)。如果你的 Agent 跑在 Azure 上、且使用 Azure OpenAI 服务,Grounding with Bing 是个一站式选择;**否则不推荐新项目从 Bing API 起步**,政策不确定性太高。

**SearXNG 自建** 是给 " 我就是想完全掌控、零供应商绑定 " 的人准备的。SearXNG 是开源元搜索引擎,可以一键 Docker 部署,聚合 Google、Bing、DuckDuckGo、Brave、Wikipedia 等几十个上游,返回去重后的统一结果。**优点**: 零 API 费用、可以无限调用、可以自己改排序逻辑、可以加私有数据源;**缺点**: 你需要自己处理上游反爬 (Google 会很快封 IP,得配代理池,这又是一个长期维护负担)、自己做正文抓取、自己做 reranker; 真实运营成本很可能比直接用 Tavily 还高。**只在你有强政策约束 (比如不能让数据出境)、或月调用量超过 500 万次时才值得**。

## **四、给个人/中小团队的推荐架构: 三段式混合**

基于上面的对比,我推荐一个**三段式架构**,在工程复杂度和效果之间取最优:

**第一段:Query 路由层 (自己写,~50 行代码)**。用一个轻量 LLM(GPT-4o mini / Claude Haiku / DeepSeek) 分析用户原始 query,做三件事:(a) 判断是否真的需要联网搜索 (很多 query 用模型自身知识就能答),(b) 改写成 1\~3 个搜索友好的 sub-query,(c) 给每个 sub-query 打一个**意图标签**——是 " 事实查询 "(factual,如 " 埃菲尔铁塔多高 ")、" 时效查询 "(news,如 " 今天英伟达股价 ")、" 探索查询 "(exploratory,如 " 找些类似 Pieter Levels 的独立开发者博客 ")、还是 " 垂直查询 "(vertical,如学术/法律/代码)。

**第二段: 搜索分发层 (配置驱动)**。根据意图标签把 query 分发到不同后端:

- **factual / news → Tavily**(`search_depth: "advanced"`,`include_answer: true`,直接拿到带正文的结果,简单粗暴)
- **exploratory → Exa**(`type: "neural"`,利用语义检索找长尾)
- **vertical → 专业源**(arXiv API、PubMed、GitHub Search、Stack Overflow 直接调,效果远超通用搜索)
- **可选兜底 → Brave Search**(当 Tavily 结果稀疏时再来一遍,提高召回)

这种分发逻辑让你**用便宜的 API 做大头、用 Exa 这类贵但精的 API 做特定场景**,平均成本比单一方案低 30\~50%。

**第三段: 抓取与压缩层**。对于 Tavily/Exa 已经返回正文的结果直接用; 对于 Brave 等只返回 URL 的,用 Jina Reader(`https://r.jina.ai/{url}`,免费额度可观、零配置) 抓取。然后跑一个 reranker——简单做法是让 LLM 给每个片段打 0\~10 分相关性,复杂做法是接 Cohere Rerank API(\$2 / 1k docs) 或自部署 bge-reranker-v2-m3。最后按 token 预算贪心选片段塞进 context。

这个架构的好处是**每一段都可以独立替换**——明天 Tavily 涨价了,把搜索分发层里那一行 client 换成 Linkup 就行,LLM 路由层和压缩层完全不用动。这是面向长期演进的工程纪律。

## **五、工程落地细节: 十个容易踩坑的地方**

把架构画出来只是开始,真正决定 Agent 体验的是细节。这里列十个我和同行踩过的典型坑:

**1. 不要在 main loop 里同步调搜索**。一次搜索 + 抓取 + rerank 容易到 5\~10 秒,会让 Agent 的 ReAct 循环慢得难以接受。要么在 LLM 流式输出时并行触发搜索,要么用 async/await + asyncio.gather 并行多 query,延迟能从 8s 压到 2s。

**2. Snippet 不等于正文,但有时只用 snippet 反而更好**。当用户问的是 "X 是什么 " 这种事实题,top 3 snippet 拼起来已经够了,强行抓正文反而引入噪声且翻倍成本。我的经验法则是:short factual query 只用 snippet,long-form research query 才抓正文。

**3. 给搜索结果加缓存**。同一个 query 在 1 小时内被问到的概率很高 (尤其在多用户产品里),用 Redis 把 (query, search_depth) → results 缓存 1\~24 小时,缓存命中率经常能到 30%+,直接省钱。

**4. 注意 robots.txt 和反爬**。Tavily/Exa/Jina Reader 都已经处理了 robots,但你自己用 requests 抓页面时务必尊重 robots.txt,而且 User-Agent 不要伪装成 Googlebot——既不道德也很容易被封。Firecrawl 这类商业服务还会处理 Cloudflare 等反爬,自己抓的话要做好心理准备。

**5. 时效性陷阱**。搜索 API 默认按相关性排,很多时候返回的是几年前的 " 经典 " 页面而非最新信息。一定要用 `time_range` 或 `published_after` 这类参数显式过滤——Tavily 有 `days` 参数,Exa 有 `start_published_date`,Brave 有 `freshness`。

**6. 中文搜索是另一个世界**。Brave、Exa、Tavily 在英文上表现都不错,但中文召回明显弱。如果你的 Agent 主要服务中文用户,Brave/Exa 几乎不能单用,Tavily 因为聚合了 Bing 还可以,但仍建议混入中文垂直源 (知乎搜索、微信公众号搜索 via 搜狗、B 站搜索 API 等)。或者考虑国内服务 (博查、智谱搜索等)。

**7. 引用必须带原文片段做校验**。让 LLM 生成回答时,prompt 里要求每句事实陈述后面跟 `[source_id]`,然后写一个后处理步骤检查 source_id 对应的原文里是不是真的包含这个事实 (可以用 embedding 相似度或 LLM judge)。这一步能把幻觉率从 15% 压到 3% 以下,对生产级 Agent 非常关键。

**8. 不要让 LLM 自己决定 " 再搜一次 " 超过 3 轮**。Agent 容易陷入 " 搜不到就改 query 再搜 " 的死循环,既费钱又惹用户烦。硬性设一个 max_search_iterations = 3,超了就用现有信息生成回答并诚实说明 " 信息有限 "。

**9. 区分 " 搜索 token" 和 " 输出 token" 的成本**。一个看起来简单的 web search Agent,真实成本结构是:LLM 调用占 60\~70%,搜索 API 占 15\~25%,抓取/rerank 占 10\~15%。优化重点应该放在**减少 LLM 上下文**(更狠的压缩) 而不是搜索调用本身。

**10. Observability 从第一天就上**。每一次搜索的 query、命中的 URL、rerank 分数、最终用了哪几个片段、生成的引用,全部记日志。Langfuse、LangSmith、Helicone 都是不错的现成方案。没有这套观测,搜索质量回退你根本发现不了。

## **六、成本测算与最终决策表**

假设你做一个有 1000 DAU 的 Agent 产品,每个用户日均触发 5 次搜索,月搜索量 = 1000 × 5 × 30 = 15 万次。我们对比三种方案的月成本:

**方案 A: 纯 Tavily(advanced)**:15 万 × \$0.015 = \$2,250 / 月。**优点是零工程量**,一周就能上线,运维零负担。

**方案 B:Tavily + Exa + Jina Reader 混合 (我推荐的三段式)**: 假设 70% 走 Tavily basic(\$0.008)、20% 走 Exa(\$0.005)、10% 走 Brave + Jina(\$0.003 + 免费),加权 = 15 万 × \$0.0066 ≈ \$990 / 月,**省 56%**; 代价是约 3\~5 人天的工程投入和持续维护。

**方案 C:SearXNG 自建 + Jina Reader**: 服务器 (2 核 4G + 代理池) 约 \$200 / 月,Jina Reader 免费额度足够,**总成本 \$200 / 月**; 代价是要花 1\~2 人月做反爬对抗和质量调优,而且每次 Google 改反爬策略你都得跟。**只在月搜索量超过 100 万次或有合规约束时才划算**。

**我对你的最终决策建议**:

- **如果你是周末项目 / MVP / DAU < 100**: 直接上 Tavily,免费额度 1000 次/月就够你跑通整个 PoC,跑通了再优化。**别把时间浪费在搜索基础设施上,把时间花在 Agent 的核心逻辑——规划、记忆、工具编排——上**,这才是产品差异化的来源。
- **如果你是认真在做产品 / DAU 在 100\~10000**: 走方案 B 的三段式混合架构,Tavily 做主力 + Exa 做语义补充 + 必要的垂直源 (arXiv/GitHub 等),前期 3 人天,长期省钱且效果更好。
- **如果你已经规模化 / DAU > 10000 或服务企业客户**: 开始考虑直接对接 Brave Search API 自己做加工层,同时保留 Tavily 作为 fallback; 并且认真考虑接入 Cohere Rerank 或自建 reranker。
- **如果你做的是中文为主的产品**:Tavily(因为聚合了 Bing)+ 国内搜索 (博查 / 智谱)+ 知乎/微信公众号垂直源,英文那套主流方案不能直接套用。
- **如果你做的是垂直领域 (法律/医学/学术/代码)**:**少用通用搜索,多接专业 API**——arXiv、Semantic Scholar、PubMed、GitHub Search、Stack Exchange、CourtListener 等,效果比任何通用搜索都好,而且大多免费。

回到上一轮我们聊的 " 内置工具机制 ": 你自己写 Agent 和 OpenAI/Anthropic 的内置 web_search 最大的区别在于——**他们把整个搜索 pipeline 都黑盒在服务端,你失去了所有调优空间**; 而你自己实现意味着 query 改写、source 选择、rerank 策略、引用格式全部可控。这种可控性恰恰是垂直 Agent 战胜通用 Agent 的杀手锏——OpenAI 的 web_search 永远不会知道你的用户更关心 Hacker News 还是更关心知乎,但你的 Agent 可以。所以**自建搜索层不是为了重新发明轮子,而是为了把 " 通用搜索 " 驯化成 " 你的产品独有的搜索 "**——这才是它的真正价值,也是为什么这件事值得你认真做一遍。