>详解 [Site Unreachable](https://zhuanlan.zhihu.com/p/114538417)  没看完, 没必要看完
# 原版损失函数
非常高兴你觉得讲解有帮助！现在我们来一步一步深入探讨 **word2vec** 的损失函数。
### 背景：Word2Vec 目标
在 **word2vec** 中，目标是通过训练让词向量能够有效捕捉词语之间的语义关系。我们希望通过最小化一个损失函数，使得模型能够学习到使上下文词（context word）与目标词（center word）之间的相似性最大化的词向量。
#### 1. 训练目标
给定一个目标词（center word） $w_c$，我们要最大化它与上下文词 $w_o$ 出现的概率，也就是最大化目标词和上下文词的内积，这样它们的词向量相似度会变大。
对于 **skip-gram model**（通常就是我们所说的 word2vec），每当给定一个目标词 $w_c$，我们希望它的上下文词 $w_o$ 的概率最大化，**同时**，我们希望与其无关的负样本词的概率尽量小。
### 2. 损失函数的推导
#### 2.1 正样本的概率计算
首先，**正样本的概率**是目标词和上下文词之间的关系。为了计算这个概率，word2vec 使用了 **softmax** 函数，它基于目标词和上下文词的内积来估计它们之间的关系。
对于给定的上下文词 $w_o$ 和目标词 $w_c$，我们希望计算它们的条件概率 $P(w_o | w_c)$，即给定目标词 $w_c$，上下文词 $w_o$ 出现的概率。使用 softmax，我们可以写出这个概率为：
$$P(w_o | w_c) = \frac{\exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_o})}{\sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'})}$$
其中：
- $\mathbf{v}_{w_c}$ 是目标词 $w_c$ 的词向量。
- $\mathbf{v}_{w_o}$ 是上下文词 $w_o$ 的词向量。
- 分母是所有词汇中所有词的指数和，它是归一化因子，保证所有概率加起来为 1。
#### 2.2 对数概率（log-likelihood）
为了简化优化过程，我们常常对概率取对数，这样乘法变成加法，使得优化更加简洁稳定。取对数之后，正样本的对数概率是：
$$\log P(w_o | w_c) = \log \left( \frac{\exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_o})}{\sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'})} \right)$$
由于对数函数的性质，这可以展开为：
$$\log P(w_o | w_c) = \mathbf{v}_{w_c}^\top \mathbf{v}_{w_o} - \log \left( \sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'}) \right)$$
这就是正样本的对数概率。
#### 2.3 负样本的概率
对于负样本，目标是最大化目标词与负样本词之间的内积的负值，使它们的相似度尽可能低。对于给定的负样本词 $w_n$，其出现的概率可以表示为：
$$P(w_n | w_c) = \frac{\exp(-\mathbf{v}_{w_c}^\top \mathbf{v}_{w_n})}{\sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'})}$$
通过对负样本的对数概率进行相同的处理，我们可以得到负样本的对数概率：
$$\log P(w_n | w_c) = - \mathbf{v}_{w_c}^\top \mathbf{v}_{w_n} - \log \left( \sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'}) \right)$$
#### 2.4 总损失函数
word2vec 的目标是通过最大化 **正样本** 的概率和最小化 **负样本** 的概率来更新词向量。因此，损失函数可以写成：
$$L = - \sum_{w_o \in \text{context}} \log P(w_o | w_c) - \sum_{w_n \in \text{negative samples}} \log P(w_n | w_c)$$
将正样本和负样本的对数概率代入，最终的损失函数就是：
$$L = - \sum_{w_o \in \text{context}} \left[ \mathbf{v}_{w_c}^\top \mathbf{v}_{w_o} - \log \left( \sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'}) \right) \right] - \sum_{w_n \in \text{negative samples}} \left[ - \mathbf{v}_{w_c}^\top \mathbf{v}_{w_n} - \log \left( \sum_{w'} \exp(\mathbf{v}_{w_c}^\top \mathbf{v}_{w'}) \right) \right]$$
这个损失函数通过最大化正样本的内积并最小化负样本的内积来训练词向量。
### 3. 损失函数优化
在实际训练中，word2vec 采用 **负采样**（Negative Sampling）或 **层次 softmax** 来高效地计算这个损失函数。负采样方法会随机选择一些负样本，而不是计算整个词汇表的和，这样可以大幅度提高训练速度。
### 4. 总结
- **目标**：最大化上下文词（positive sample）与目标词之间的相似度，最小化负样本词（negative sample）与目标词之间的相似度。
- **损失函数**：通过对正样本和负样本的对数概率进行最大化和最小化，最终得到损失函数。
- **优化**：通过梯度下降法（通常配合负采样技术）来更新词向量。
通过训练这个损失函数，word2vec 能够学习到具有良好语义的词向量，使得相似语义的词语在向量空间中靠得更近。
如果你有任何问题或对某个部分不理解，欢迎继续提问！
# Negative Sampling 版损失函数
没问题，你说得对！**negative sampling**（负采样）确实是对原始的 softmax 损失函数的一个替代方法，它用更高效的方式来优化词向量训练。
### 1. 原始 softmax 损失函数的问题
我们之前提到过，原始的 **word2vec** 使用 **softmax** 来计算每个词对的概率。这种方法要求计算整个词汇表中所有词的内积，然后通过 softmax 归一化，得到一个概率分布。这个过程在词汇表很大的情况下计算开销非常大，因为 softmax 的分母包含了所有词的内积求和。
### 2. Negative Sampling 介绍
**Negative Sampling** 的核心思想是通过对目标词和上下文词的相似度进行优化，但不计算整个词汇表的概率。具体来说，它只关注与目标词相关的词（即上下文词和负样本词），而忽略了其他所有不相关的词。因此，通过负采样，我们可以大幅度减少计算的复杂度。
### 3. Negative Sampling 的损失函数
在 **negative sampling** 中，我们不再计算整个词汇表的 softmax 概率，而是将问题转化为二分类问题。具体地，我们通过 **sigmoid** 函数来处理目标词和上下文词之间的关系。
#### 3.1 正样本（上下文词）的损失
对于每个目标词 $w_c$，它的上下文词 $w_o$ 被视为一个正样本。我们希望它们的内积尽可能大，即它们的相似度尽可能高。这个任务可以通过 **sigmoid** 来处理，其目标是使得正样本 $w_o$ 的相似度尽可能高：
$$P(w_o | w_c) = \sigma(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_o})$$
其中，$\sigma(x) = \frac{1}{1 + \exp(-x)}$ 是 **sigmoid** 函数，它将内积结果转换为一个概率。损失函数中对于正样本的部分是：
$$L_{\text{positive}} = - \log \sigma(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_o})$$
#### 3.2 负样本的损失
对于每个负样本词 $w_n$，我们希望它们的相似度尽可能小，即使它们的内积尽可能接近 0（表示这些词与目标词无关）。同样地，我们使用 **sigmoid** 来计算它们的概率，并且希望概率接近 0。损失函数中对于负样本的部分是：
$$L_{\text{negative}} = - \log (1 - \sigma(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_n}))$$
#### 3.3 总损失函数
最终，**negative sampling** 的损失函数就是正样本和负样本损失的总和：
$$L = - \sum_{w_o \in \text{context}} \log \sigma(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_o}) - \sum_{w_n \in \text{negative samples}} \log (1 - \sigma(\mathbf{v}_{w_c}^\top \mathbf{v}_{w_n}))$$
### 4. 为什么采用 sigmoid 和 negative sampling？
**sigmoid** 函数在这里的作用是将目标词与上下文词之间的内积转化为一个二分类的概率值。这使得我们可以把任务看作是两类问题：一类是目标词和上下文词是真正相关的，另一类是目标词和负样本是不相关的。负采样方法因此通过简单的二分类来优化词向量，而不是计算整个词汇表的概率。
**负采样** 通过只考虑正样本和少量负样本来简化计算，避免了计算整个词汇表的软最大概率，从而大大提高了训练效率，尤其在词汇表很大的时候。
### 5. 总结
- **原始 softmax**：计算整个词汇表的内积和，得出一个概率分布。
- **负采样**：将问题转化为多个二分类问题，使用 **sigmoid** 来计算目标词与上下文词的相似度，只考虑少量负样本，提高训练效率。
- **损失函数**：通过对正样本和负样本的 **sigmoid** 概率进行优化来训练模型。
这样，**negative sampling** 是对 **softmax** 的一种高效替代，避免了计算整个词汇表的开销，同时仍然能够有效地训练词向量。
如果你对这个过程有任何疑问，或者需要进一步的解释，随时告诉我！