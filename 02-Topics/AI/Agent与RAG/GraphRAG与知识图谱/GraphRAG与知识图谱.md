---
type: 入口笔记
---
# GraphRAG与知识图谱

## 当前理解

GraphRAG 先把文本抽成实体关系图（知识图谱 / 属性图 / RDF），再用 Leiden 算法按模块度 Q 分层聚类成社区，并为社区生成报告；查询时结合向量检索与图遍历做混合检索和多跳推理。

## 笔记（按时间）

- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-15 ?URI|2026-07-15 ?URI]] — 讲了：1. 关系图解：URI 是“父亲”，URL 和  / 2. 生活中的类比 / 3. RDF 为什么喜欢用“长得像 URL”的
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-15 PageRank|2026-07-15 PageRank]] — 讲了：1. 核心概念：民主投票与权重传递 / 2. 数学公式与“随机浏览者”模型 / 3. PageRank 是如何计算出来的？
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-15 RDF|2026-07-15 RDF]] — 讲了：you asked / googleaistudio response / 1. RDF (Resource Descrip
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-15 属性图|2026-07-15 属性图]] — 讲了：1. 什么是属性图数据模型？ / 2. 最直观的区别：如何表达“边上的属性”？ / 3. RDF 与 属性图 的全面对比
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-18 GraphRAG 的 Leiden 算法|2026-07-18 GraphRAG 的 Leiden 算法]] — 讲了：第一层（最底层）：原始数据到加权图建模 / 第二层：Leiden 算法的自底向上聚类 / 步骤 1：局部节点移动（Local Moveme
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-18 GraphRAG|2026-07-18 GraphRAG]] — 讲了：一、历史:三条支流,你只需要认第三条 / 二、数据模型:两种图,你都得认识 / 1. RDF 三元组(来自支流 A/B)
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-18 Leiden 模块度 Q 怎么算|2026-07-18 Leiden 模块度 Q 怎么算]] — 讲了：模块度  Q  怎么算 / 1. 设定网络结构（4 个人，3 条边） / 2. 模块度的计算公式
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-18 知识图谱|2026-07-18 知识图谱]] — 讲了：第 0 层：三元组——事实的最小单位 / 第 1 层：实体必须有 ID，而不是字符串 / 第 2 层：三元组自动粘成图
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-22 Agent工作流中的多跳推理实现|2026-07-22 Agent工作流中的多跳推理实现]] — 讲了：Agent工作流中的多跳推理实现
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-22 GraphRAG中的混合检索|2026-07-22 GraphRAG中的混合检索]] — 讲了：💡 当前做法 vs RRF 做法的区别 / 一、 微软官方 GraphRAG 的标准实现就是 / 二、 行业中两类“标准”的区别
- [[02-Topics/AI/Agent与RAG/GraphRAG与知识图谱/2026-07-22 工业界RAG系统的实现|2026-07-22 工业界RAG系统的实现]] — 讲了：工业界RAG系统的实现方式 / User: / Assistant:
