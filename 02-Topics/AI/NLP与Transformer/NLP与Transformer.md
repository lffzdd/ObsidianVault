---
type: 入口笔记
---
# NLP与Transformer

## 当前理解

NLP 表示学习的主线：从稀疏统计（BM25、词袋）到稠密嵌入（Word2Vec 的 CBOW/Skip-gram、GloVe），再到 RNN 引入顺序、Attention 引入全局关系，最后是 Transformer / BERT / SBERT。细节集中在位置编码、掩码、softmax 维度和 kv cache。

## 笔记（按时间）

- [[02-Topics/AI/NLP与Transformer/2025-11-05 kv cache|2025-11-05 kv cache]] — 讲了：一、先回忆 Transformer 的工作原理 / 二、问题：每生成一个字，重新算一遍太蠢了 / 三、解决方案：缓存它！
- [[02-Topics/AI/NLP与Transformer/2025-11-10 BM25的计算和编码|2025-11-10 BM25的计算和编码]] — 讲了：回答一 / 深入解析：BM25 如何“向量化” / 阶段一：索引（Indexing）- 构建文档向量
- [[02-Topics/AI/NLP与Transformer/2025-11-10 BM25详解|2025-11-10 BM25详解]] — 讲了：一、词频饱和 (Term Frequency S / 二、文档长度偏差 (Document Lengt / ✅ 更精确的修正版
- [[02-Topics/AI/NLP与Transformer/2025-11-10 CBOW计算详解|2025-11-10 CBOW计算详解]] — 讲了：一、模型的目标：预测中心词的概率 / 二、输入层：从词到向量 / 三、隐藏层：平均上下文向量
- [[02-Topics/AI/NLP与Transformer/2025-11-10 Skip-gram详解|2025-11-10 Skip-gram详解]] — 讲了：一、核心思想 / 二、网络结构（同 CBOW，但方向反了） / 三、训练目标
- [[02-Topics/AI/NLP与Transformer/2025-11-10 从稀疏到稠密的升维|2025-11-10 从稀疏到稠密的升维]] — 讲了：一、稀疏阶段：词表是“离散空间”的一维坐标 / 二、Word2Vec：让词“搬进”一个连续空间 / 三、为什么要“升维”
- [[02-Topics/AI/NLP与Transformer/2025-11-10 嵌入模型|2025-11-10 嵌入模型]] — 讲了：🏛️ 核心原理：从“文本”到“语义坐标” / 📜 发展脉络：从“计数”到“上下文理解” / ⚙️ RAG 对嵌入模型的关键考量
- [[02-Topics/AI/NLP与Transformer/2025-11-10 嵌入模型-从统计转向预测|2025-11-10 嵌入模型-从统计转向预测]] — 讲了：🏛️ 奠基之作：Word2Vec (2013,  / 1. 架构一：CBOW (Continuous  / 2. 架构二：Skip-gram
- [[02-Topics/AI/NLP与Transformer/2025-11-10 嵌入模型-稀疏阶段|2025-11-10 嵌入模型-稀疏阶段]] — 讲了：奠基石：词袋模型 (Bag-of-Words,  / 代表模型一：TF-IDF (词频-逆文档频率) / 1. TF (Term Frequency, 词
- [[02-Topics/AI/NLP与Transformer/2025-11-11 GloVe到RNN，引入语句前后顺序|2025-11-11 GloVe到RNN，引入语句前后顺序]] — 讲了：一、Word2Vec：无序的共现空间（语义但无语 / 二、RNN：顺序第一次进入语言表示 / 三、LSTM / GRU：让顺序记忆变得可靠
- [[02-Topics/AI/NLP与Transformer/2025-11-11 GloVe和Attention全局的不同|2025-11-11 GloVe和Attention全局的不同]] — 讲了：一、GloVe 的权重：靠“统计距离”衰减 / 二、Attention 的权重：靠“语义相似”动 / 三、为什么这比 GloVe 的“距离权重”更强大
- [[02-Topics/AI/NLP与Transformer/2025-11-11 GloVe模型详解|2025-11-11 GloVe模型详解]] — 讲了：GloVe模型详解
- [[02-Topics/AI/NLP与Transformer/2025-11-11 NLP演化史|2025-11-11 NLP演化史]] — 讲了：一、Word2Vec：语义空间的诞生，但语序被舍 / 二、语言模型 (Language Model)： / 三、Attention：让“相关性”取代“顺序”
- [[02-Topics/AI/NLP与Transformer/2025-11-11 Transformer的掩码机制|2025-11-11 Transformer的掩码机制]] — 解决：？“/场景完全不同。？
- [[02-Topics/AI/NLP与Transformer/2025-11-11 分词算法|2025-11-11 分词算法]] — 讲了：一、在 Word2Vec 时代，分词是“外部手工 / 二、分词的“语义困境”：词的边界不是客观的 / 三、现代做法：从“词”走向“子词 (subwor
- [[02-Topics/AI/NLP与Transformer/2025-11-11 阶段三 Transformer和BERT|2025-11-11 阶段三 Transformer和BERT]] — 讲了：历史转折点：从“链式”到“并行” / 🚀 革命的引擎：Transformer (201 / 🌟 阶段三的代表：BERT (2018, Goo
- [[02-Topics/AI/NLP与Transformer/2025-11-12 人脑|2025-11-12 人脑]] — 讲了：一、人脑和深度学习的相似之处 / 二、人脑的“不可解释性”其实更严重 / 三、解释性问题背后的哲学难题
- [[02-Topics/AI/NLP与Transformer/2025-11-12 阶段四 BERT到SBERT|2025-11-12 阶段四 BERT到SBERT]] — 讲了：🏛️ 奠基之作：Sentence-BERT (S / 🚀 革命的核心：SBERT 如何“修复”向量空间 / 总结：第四阶段的革命
- [[02-Topics/AI/NLP与Transformer/2025-12-18 Transformer的两种发展|2025-12-18 Transformer的两种发展]] — 讲了：1. 只有编码器 (Encoder-only)： / 2. 只有解码器 (Decoder-only)： / 3. 完整的编码器 - 解码器 (Encoder
- [[02-Topics/AI/NLP与Transformer/2025-12-18 自编码器|2025-12-18 自编码器]] — 讲了：历史 / 一、 历史背景：从“自动关联器”到深度学习的火种 / 1. 诞生初期（1980s）：自动关联器 (Au
- [[02-Topics/AI/NLP与Transformer/2025-12-18 自编码器解决了神经网络训练的问题|2025-12-18 自编码器解决了神经网络训练的问题]] — 讲了：1. 核心困境：消失的梯度与“随机”的开端 / 2. Hinton 的神来之笔：逐层预训练 (L / 第一步：自底向上的“无监督”预训练
- [[02-Topics/AI/NLP与Transformer/2025-12-18 自编码进入NLP|2025-12-18 自编码进入NLP]] — 讲了：一、 历史转折点：从递归到循环 (2011-20 / 1. 为什么是翻译？ / 2. 目的：寻找“定长表示”
- [[02-Topics/AI/NLP与Transformer/2025-12-31 BPE 分词算法|2025-12-31 BPE 分词算法]] — 讲了：1. 历史背景与底层原理 / 历史溯源：从压缩到分词 / NLP 领域的引入
- [[02-Topics/AI/NLP与Transformer/2026-01-03 位置编码|2026-01-03 位置编码]] — 讲了：Gemini / 1. 寻找数学规律：从二进制到几何级数 / 2. 设定边界：波长该多大？
- [[02-Topics/AI/NLP与Transformer/2026-01-04 Attention中softmax作用的维度|2026-01-04 Attention中softmax作用的维度]] — 讲了：为什么掩码不是 0 / 1. 原理推导：e 的幂次方 / 情况 A：如果你设置为 0
- [[02-Topics/AI/NLP与Transformer/2026-01-04 Attention的输出不需要加Q|2026-01-04 Attention的输出不需要加Q]] — 讲了：为什么不要  Q / User Input / Planner Response
- [[02-Topics/AI/NLP与Transformer/2026-01-05 Transformer不是U-Net架构|2026-01-05 Transformer不是U-Net架构]] — 讲了：Planner Response / Transformer 不是 U-Net 结构 / Transformer 整体结构
- [[02-Topics/AI/NLP与Transformer/2026-01-07 生成的token是重复的 - kv cache|2026-01-07 生成的token是重复的 - kv cache]] — 讲了：User Input / User Input / Planner Response
- [[02-Topics/AI/NLP与Transformer/2026-01-21 Transformer机制|2026-01-21 Transformer机制]] — 讲了：User Input / Planner Response / 一、训练阶段（Teacher Forcing）
- [[02-Topics/AI/NLP与Transformer/2026-01-21 Transformer的padding掩码|2026-01-21 Transformer的padding掩码]] — 讲了：User Input / Planner Response / 区分两者
- [[02-Topics/AI/NLP与Transformer/2026-01-27 Transformer 的训练问题|2026-01-27 Transformer 的训练问题]] — 讲了：核心问题：为什么 Transformer 这么难 / 第一层：Adam 在起步时的“致命缺陷” / 第二层：Warmup —— 强制的“缓步起跑”
- [[02-Topics/AI/NLP与Transformer/2026-01-27 位置编码采取对数形式|2026-01-27 位置编码采取对数形式]] — 讲了：历史背景 / 标准实现代码 / 使用示例
- [[02-Topics/AI/NLP与Transformer/2026-01-27 分词器要不要加词边界|2026-01-27 分词器要不要加词边界]] — 讲了：Planner Response / Planner Response / 两种方法的对比

### 早期专题笔记

- [[02-Topics/AI/NLP与Transformer/1-GPT|1-GPT]] — 讲了：过程 / 注意力模块 / 多层感知机
- [[02-Topics/AI/NLP与Transformer/2-Attention|2-Attention]] — 讲了：a fluffy blue creature r / W_{Q}  和  W_{K} / 注意力模式的计算
- [[02-Topics/AI/NLP与Transformer/3-MLP|3-MLP]] — 讲了：Transformer 的整体结构 / 编码器（Encoder）和解码器（Decoder / FFN
- [[02-Topics/AI/NLP与Transformer/4-代码示例|4-代码示例]] — 讲了：代码 / 1.   多头自注意力机制（Multi-Head / a. 自注意力机制
- [[02-Topics/AI/NLP与Transformer/5-重学 Transformer|5-重学 Transformer]] — 解决：为什么这里不直接用嵌入向量呢? 我猜可能是因为直接用嵌入向量的话结果就变成了嵌入向量的线性组合？
- [[02-Topics/AI/NLP与Transformer/6-MLP模块|6-MLP模块]] — 讲了：6-MLP模块
