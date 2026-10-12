---
type: 入口笔记
---
# NLP与Transformer

## 当前理解

NLP 表示学习的主线：从稀疏统计（BM25、词袋）到稠密嵌入（Word2Vec 的 CBOW/Skip-gram、GloVe），再到 RNN 引入顺序、Attention 引入全局关系，最后是 Transformer / BERT / SBERT。细节集中在位置编码、掩码、softmax 维度和 kv cache。

## 笔记（按时间）

- [[02-Topics/AI/NLP与Transformer/2025-11-11 分词算法|2025-11-11 分词算法]] — 讲了：一、在 Word2Vec 时代，分词是“外部手工 / 二、分词的“语义困境”：词的边界不是客观的 / 三、现代做法：从“词”走向“子词 (subwor
- [[02-Topics/AI/NLP与Transformer/2025-11-12 人脑|2025-11-12 人脑]] — 讲了：一、人脑和深度学习的相似之处 / 二、人脑的“不可解释性”其实更严重 / 三、解释性问题背后的哲学难题
- [[02-Topics/AI/NLP与Transformer/2025-12-31 BPE 分词算法|2025-12-31 BPE 分词算法]] — 讲了：1. 历史背景与底层原理 / 历史溯源：从压缩到分词 / NLP 领域的引入
- [[02-Topics/AI/NLP与Transformer/2026-01-27 分词器要不要加词边界|2026-01-27 分词器要不要加词边界]] — 讲了：Planner Response / Planner Response / 两种方法的对比
- [[02-Topics/AI/NLP与Transformer/4-代码示例|4-代码示例]] — 讲了：代码 / 1.   多头自注意力机制（Multi-Head / a. 自注意力机制

## 子主题

- 入口：[[02-Topics/AI/NLP与Transformer/Attention与Transformer/Attention与Transformer|Attention与Transformer]]（15 篇）

- 入口：[[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/RNN与早期语言模型|RNN与早期语言模型]]（6 篇）

- 入口：[[02-Topics/AI/NLP与Transformer/词嵌入与检索/词嵌入与检索|词嵌入与检索]]（11 篇）

- 入口：[[02-Topics/AI/NLP与Transformer/预训练模型演化/预训练模型演化|预训练模型演化]]（9 篇）
