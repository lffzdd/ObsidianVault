---
type: 入口笔记
---
# Attention与Transformer

## 当前理解

Attention 用 Q、K、V 计算全局相关性，softmax 作用在 key 维度；Transformer 叠加多头注意力、MLP、残差和位置编码，用掩码处理 padding 和因果关系。

## 笔记（按时间）

- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2025-11-11 Transformer的掩码机制|2025-11-11 Transformer的掩码机制]] — 解决：？“/场景完全不同。？
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2025-11-11 阶段三 Transformer和BERT|2025-11-11 阶段三 Transformer和BERT]] — 讲了：历史转折点：从“链式”到“并行” / 🚀 革命的引擎：Transformer (201 / 🌟 阶段三的代表：BERT (2018, Goo
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-03 位置编码|2026-01-03 位置编码]] — 讲了：Gemini / 1. 寻找数学规律：从二进制到几何级数 / 2. 设定边界：波长该多大？
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-04 Attention中softmax作用的维度|2026-01-04 Attention中softmax作用的维度]] — 讲了：为什么掩码不是 0 / 1. 原理推导：e 的幂次方 / 情况 A：如果你设置为 0
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-04 Attention的输出不需要加Q|2026-01-04 Attention的输出不需要加Q]] — 讲了：为什么不要  Q / User Input / Planner Response
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-05 Transformer不是U-Net架构|2026-01-05 Transformer不是U-Net架构]] — 讲了：Planner Response / Transformer 不是 U-Net 结构 / Transformer 整体结构
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-21 Transformer机制|2026-01-21 Transformer机制]] — 讲了：User Input / Planner Response / 一、训练阶段（Teacher Forcing）
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-21 Transformer的padding掩码|2026-01-21 Transformer的padding掩码]] — 讲了：User Input / Planner Response / 区分两者
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-27 Transformer 的训练问题|2026-01-27 Transformer 的训练问题]] — 讲了：核心问题：为什么 Transformer 这么难 / 第一层：Adam 在起步时的“致命缺陷” / 第二层：Warmup —— 强制的“缓步起跑”
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2026-01-27 位置编码采取对数形式|2026-01-27 位置编码采取对数形式]] — 讲了：历史背景 / 标准实现代码 / 使用示例
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/2-Attention|2-Attention]] — 讲了：a fluffy blue creature r / W_{Q}  和  W_{K} / 注意力模式的计算
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/3-MLP|3-MLP]] — 讲了：Transformer 的整体结构 / 编码器（Encoder）和解码器（Decoder / FFN
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/5-重学 Transformer|5-重学 Transformer]] — 解决：为什么这里不直接用嵌入向量呢? 我猜可能是因为直接用嵌入向量的话结果就变成了嵌入向量的线性组合？
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/6-MLP模块|6-MLP模块]] — 讲了：6-MLP模块
- [[02-Topics/AI/NLP与Transformer/Attention与Transformer/多头自注意力 - 改进想法|多头自注意力 - 改进想法]] — 讲了：原本的多头自注意力，各个头中值向量的长度为词嵌入长度/ 头数，最后再拼接成长度为嵌入长度的向量，这样…
