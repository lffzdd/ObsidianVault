---
type: 入口笔记
---
# LLM

## 当前理解

LLM 本质是对模板化 token 序列做续写：role/content、im_start 这类标记是训练出来的格式，不是所有模型都一样；enable_think 等参数只是改模板。中转站“降智”多半来自模板、参数或模型被替换。

## 笔记（按时间）

- [[02-Topics/AI/LLM/2025-11-05 role，content字段是训练好的|2025-11-05 role，content字段是训练好的]] — 讲了：一、 role  是训练时的固定结构，不可随意改 / 二、API 层也强制限制 / 三、为什么要固定这几个？
- [[02-Topics/AI/LLM/2025-11-05 模型的输入是Json，输出不一定|2025-11-05 模型的输入是Json，输出不一定]] — 讲了：一、接口层：JSON 包裹，文本在里面 / 二、LangChain 等上层框架怎么处理这个输 / 三、模型“输出 JSON”的两种含义
- [[02-Topics/AI/LLM/2025-11-06 enable_think参数|2025-11-06 enable_think参数]] — 讲了：一、层次结构分解：从外到内 / 1.   客户端调用层 / 2.   vLLM 模板渲染层
- [[02-Topics/AI/LLM/2025-11-06 llm模板|2025-11-06 llm模板]] — 讲了：一、模板是什么？ / 二、模板存在的意义 / 1.   统一格式
- [[02-Topics/AI/LLM/2025-11-06 不是所有llm都采用im_xxx|2025-11-06 不是所有llm都采用im_xxx]] — 讲了：不是所有llm都采用im_xxx
- [[02-Topics/AI/LLM/2026-01-27 知识蒸馏|2026-01-27 知识蒸馏]] — 讲了：1. 核心矛盾：越强越慢 vs. 越快越弱 / 2. 历史背景：Hinton 的神来之笔 (20 / 3. 原理详解：温度 (Temperature)
- [[02-Topics/AI/LLM/2026-04-06 llm中转站为何降智|2026-04-06 llm中转站为何降智]] — 讲了：API 格式转换导致降智的完整拆解 / 一、最核心的问题：Thinking（扩展思考） / 1. Claude 原生 API 的 Think
- [[02-Topics/AI/LLM/0-学习路径|0-学习路径]] — 讲了：🕰️ LLM 的历史发展大致可以分为这几个阶段： / 1.   早期探索：神经网络 & 词向量 (19 / 2.   从 RNN 到 Transformer
- [[02-Topics/AI/LLM/chatgpt-prompt|chatgpt-prompt]] — 讲了：✅ 推荐填入内容（通用型指令）、💡 可选增强版（如果你希望 ChatGPT 更贴近「思维导图
- [[02-Topics/AI/LLM/prompt gpt|prompt gpt]] — 讲了：（内容很短，待补充）

## 子主题

### web_search

- [[02-Topics/AI/LLM/web_search/2026-05-15 llm web_search 1 - 官方有哪些|2026-05-15 llm web_search 1 - 官方有哪些]] — 讲了：**OpenAI 主要自建搜索基础设施（与 Bing/必应深、**OpenAI:Bing 索引 + 自有爬虫 + 自研排序、**Anthropic Claude:Brave Searc
- [[02-Topics/AI/LLM/web_search/2026-05-15 llm web_search 2 - 自建怎么做|2026-05-15 llm web_search 2 - 自建怎么做]] — 讲了：**如果你是个人开发者或中小团队自建 Agent，最稳的方案、**一、先把 "web search" 这个需求拆成五个子问、**二、可选方案矩阵: 六类供应商各司其职**
- [[02-Topics/AI/LLM/web_search/2026-05-15 llm web_search 3 - 自建vs官方|2026-05-15 llm web_search 3 - 自建vs官方]] — 讲了：**自建未必输给官方——在垂直场景里几乎一定赢; 但在 " 、**第二层: 四家官方 web_search 的真实体验对比、**ChatGPT Search: 综合体验最强,但有 " 

- 入口：[[02-Topics/AI/LLM/接口格式发展/接口格式发展|接口格式发展]]（5 篇）
