---
aliases: 2-Attention
tags:
date created: Saturday, August 3rd 2024, 4:18:50 pm
date modified: Sunday, August 4th 2024, 11:26:17 pm
---

```ad-col2
![[Pasted image 20240803162212.png]]

![[Pasted image 20240803162254.png]]
```

![[Pasted image 20240803162402.png]]
空间中的方向应该是无穷无尽的
Transformer 的目标是逐步调整这些嵌入，使它们不单单编码单个词，还能融入更丰富的上下文含义

- American shrew mole 美国鼩鼱鼹鼠
- One mole of carbon dioxide 一摩尔二氧化碳
- Take a biopsy of the mole 对痣进行活检
![[Pasted image 20240803162928.png]]
嵌入空间中有多个方向，编码了「mole』一词的不同含义
训练得好的注意力模块能计算出，需要给初始的泛型嵌入加入什么向量，才能把它移动到上下文对应的具体方向上

```ad-col2
![[Pasted image 20240803163006.png]]

![[Pasted image 20240803163226.png]]
```

塔前面加上 `埃菲尔` 就应该使向量移动到新的位置
![[Pasted image 20240803163306.png]]
模型的预测完全基于最后一个向量
就像推理故事，该序列中的最后一个向量即 `是` 的嵌入向量，必须经过所有注意力模块的更新，以包含远超单个词的信息量。也就是要设法编码整个 上下文窗口中，与预测下一个词相关的所有信息

```ad-col2
![[Pasted image 20240803163619.png]]

![[Pasted image 20240803163811.png]]
```

---
# a fluffy blue creature roamed the verdant forest
举个例子：a fluffy blue creature roamed the verdant forest.
我们暂时只关心形容词调整对应名词的含义，这种类型的更新
![[Pasted image 20240803164045.png]]
我们先来了解 **`单头注意力机制`**，之后再介绍 注意力模块中 多个注意力头如何并行运算
![[Pasted image 20240803164520.png]]
在深度学习中，模型的实际行为往往是很难解释的，因为它只是通过调整大量参数来最小化某人代价函数。只是我们举个具体的例子，有助于理解
# $W_{Q}$ 和 $W_{K}$
![[Pasted image 20240803164724.png]]
![[Pasted image 20240803164800.png]]
![[Pasted image 20240803164825.png]]
![[Pasted image 20240803164942.png]]
给每个 token 算出一个查询向量
![[Pasted image 20240803165004.png]]
$W_Q$ 是参数矩阵
假设这个查询矩阵，将嵌入空间中的名词，映射到较小的查询空间中的某个方向 ![[Pasted image 20240803165510.png]]
用向量来编码 『寻找前置形容词』 的概念
键矩阵：
![[Pasted image 20240803165912.png]]
![[Pasted image 20240803165946.png]]
![[Pasted image 20240803170049.png]] 当键与查询的方向相对齐时就能认为它们相匹配 ![[Pasted image 20240803170205.png]]

为了衡量每人键与每人查询的匹配程度，我们要计算所有可能的键——查询对之间的点积
![[Pasted image 20240803170614.png]]
『毛茸茸的」 和 『蓝色』的嵌入 注意到了 『生物」 的嵌入
![[Pasted image 20240803170711.png]]
分数的含义：
![[Pasted image 20240803170859.png]]
而这些分数的用法就是对每一列进行加权求和
![[Pasted image 20240803171056.png]]
![[Pasted image 20240803171136.png]]

这要求 $W_{Q}\cdot K_{Q}$ 得来的分数不能介于 0 到 1 之间，而且每列和为 1-->softmax：
![[Pasted image 20240803171824.png]]

```ad-col2
![[Pasted image 20240803171926.png]]

![[Pasted image 20240803171934.png]]
```

# 注意力模式的计算

![[Pasted image 20240803172038.png]]
为了数值稳定性，要除以 $\sqrt{d_{k}}$ (d 表示查询空间维度)
![[Pasted image 20240803172406.png]]
在训练过程中，对给定示例文本跑模型时，模型会根据正确预测出下一词的概率高低，来进行奖惩并稍微调整各个权重。而效率更高的一种方法是，让它同时预测，每个初始 token 子序列之后，所有可能的下一个 token ![[Pasted image 20240803173056.png]]
![[Pasted image 20240803173121.png]]
就注意力模式而言，这意味着不能让后词影响前词，不然就会泄露接下来的 『答案」，矩阵左下角的值需要被变为 0
![[Pasted image 20240803173301.png]]
但如果直接变为 0，每列总和就不是 1 了，不再是归一化的了。所以常见方法是，在应用 softmax 之前，先把它们设为负无穷 ![[Pasted image 20240803173541.png]]
![[Pasted image 20240803173809.png]]
也有的注意力机制不应用掩码

注意力模式，其大小等于上下文长度的平方。这就是为何上下文长度，会成为大语言模型的巨大瓶颈，而扩大上下文长度绝非易事。可想而知，出于对更大上下文窗口的渴求，近年来，注意力机制出现了一些变体 ![[Pasted image 20240803174203.png]]
# 值矩阵
回到主线，算出该模式，就能让模型推断出，每个词与其它哪些词有关，然后要做的，就是去更新嵌入向量，把各个词的信息，传递给与之相关的其它词 ![[Pasted image 20240803183631.png]]
## 值矩阵 $W_{V}$
![[Pasted image 20240803221157.png]]
![[Pasted image 20240803221247.png]]

```ad-col2
![[Pasted image 20240803221656.png]]

![[Pasted image 20240803221714.png]]
```

 ![[Pasted image 20240803221911.png]]

## 值向量
通过 $W_{V}$ 得到值向量后，将值向量加入到原始的嵌入向量上，得 $\vec{E_{4}}+\Delta \vec{E_{4}}$

```ad-col2
![[Pasted image 20240804132623.png]]

![[Pasted image 20240804132634.png]]
```

$$
\begin{array}{ccccccccc}\vec{\mathbf{E}}_1&\vec{\mathbf{E}}_2&\vec{\mathbf{E}}_3&\vec{\mathbf{E}}_4&\vec{\mathbf{E}}_5\\
+&+&+&+&+
\\
\Delta\vec{\mathbf{E}}_{1}&\Delta\vec{\mathbf{E}}_{2}&\Delta\vec{\mathbf{E}}_{3}&\Delta\vec{\mathbf{E}}_{4}&\Delta\vec{\mathbf{E}}_{5}& \\
 \downarrow&\downarrow&\downarrow&\downarrow&\downarrow\\
\vec{\mathbf{E}}_1^{\prime}&\vec{\mathbf{E}}_2^{\prime}&\vec{\mathbf{E}}_3^{\prime}&\vec{\mathbf{E}}_4^{\prime}&\vec{\mathbf{E}_{5}} 
\end{array}$$ 
放眼来看，整个过程就是『单头注意力」机制。  ：![[Pasted image 20240804133622.png]] 
## 参数数量
![[Pasted image 20240804133739.png]]
$E_{1}\times W_K$ 得到 $V_1$，总共有 12288 个 $V$，可以视作每个 token 各自的语义向量
$$
E_{1}+\Delta E_{1}=E_{1}+(w_{11}V_1+w_{12}V_2+\cdots+w_{112288}V_{112288})
$$
- ! 其中有许多个 $w$ 为 0，$w_{i}$ 越大，$E$ 和该 $token_{i}$ 的关系越大，被该 $token$ 影响得也就越大，即加上该 $token$ 的语义向量乘以 $w_i$
设计上，值矩阵的参数量，可以比键矩阵和查询矩阵多几个数量级
![[Pasted image 20240805091835.png]]
但实际上，值矩阵的参数量等于键矩阵和查询矩阵的参数量之和==，对于并行运行多个注意力头来说，这一点尤为重要==
**具体做法是，将值矩阵分解为两个小矩阵相乘**
![[Pasted image 20240805092113.png]] ![[Pasted image 20240805092224.png]]
 ![[Pasted image 20240805092335.png]] 
### $Value\downarrow$ 和 $Value\uparrow$
先左乘第一个矩阵，将 12288 维向量降维到 128 维
![[Pasted image 20240805092654.png]]
左乘第二个矩阵，将 128 维向量升维到 12288 维。
![[Pasted image 20240805092815.png]]
参数数量
![[Pasted image 20240805100017.png]]
目前为止我们讨论的都是自注意力头，与其它模型中的变体 『交叉注意力』 头有所不同
![[Pasted image 20240805100242.png]]
![[Pasted image 20240805100257.png]]
![[Pasted image 20240805100603.png]]
![[Pasted image 20240805100634.png]]
例如文本翻译模型中，键可能来自一种语言，而查询则来自另一种语言，这个注意力模式，就可以描述，一种语言中的哪些词对应另一种语言中的哪些词。在这种情况下，通常不会用到掩码，因为不存在后面 token 影响前面 token的问题 ![[Pasted image 20240805100932.png]]
剩下要讲的，就是解释如何多次重复这一过程 
我们的例子里，关注的是形容词更新名词含义，但实际上，上下文影响词义的方式多种多样
```ad-col2
![[Pasted image 20240805102148.png]]

![[Pasted image 20240805102212.png]]
```

```ad-col2
![[Pasted image 20240805102244.png]]

![[Pasted image 20240805102250.png]]
```
+以捕捉不同的注意力模式
而值矩阵的参数，也会因嵌入的更新值不同而改变。
$Q$ 和 $K$ 对应某种问题，$V$ 对应这种问题下 token 向量的衡量方式
# 多头注意力
Transformer 内完整的注意力模块由多头注意力组成 
![[Pasted image 20240805103123.png]]
![[Pasted image 20240805103152.png]]
![[Pasted image 20240805103206.png]]
 ![[Pasted image 20240805103236.png]]+产生96种不同的注意力模式

![[Pasted image 20240805103322.png]]
![[Pasted image 20240805103415.png]]

![[Pasted image 20240805105050.png]]
我的推测没错
![[Pasted image 20240805105129.png]]
![[Pasted image 20240805105154.png]]
**参数数量**：
![[Pasted image 20240805105259.png]]

实际上，这些 『值矩阵 +
![[Pasted image 20240805105442.png]]
+与整个多头注意力模块相关联
至于 $Value\downarrow$，如图：
![[Pasted image 20240805111227.png]]
除了 Attention 模块+
![[Pasted image 20240805111427.png]]
其次，它还会多次重复这两种操作。这意味着，某个词吸收了一些上下文信息后，含义细致的嵌入向量还有更多的机会，被周围含义细致的词语所影响，越是接近网络的深处，每个嵌入就会从其他嵌入中吸收越多的含义，使得嵌入本身也越来越细致丰富
理想的话，对于输入的内容，这样能提炼出更高级，更抽象的概念，而不仅仅是修饰和语法结构，也许还包括，情感语气诗意，以及话题涉及的科学原理，等等 ![[Pasted image 20240805111756.png]] 
**参数数量**：![[Pasted image 20240805111830.png]]
![[Pasted image 20240805111855.png]]
注意力机制成功的主要原因，并不在于它能实现什么特定的行为+ ![[Pasted image 20240805111951.png]]
+这样就能用GPU在短时间内运行大量计算
在过去的一二十年里，深度学习的一个重要经验就是，仅靠**扩大模型规模**，就能为模型性能带来质的飞跃，而可并行架构的巨大优势，就在于能轻易扩大规模

- & 本视频，我直接介绍了当今的注意力机制，但如果你对相关发展历史感兴趣，以及好奇如何自己从头发明一遍+ ![[Pasted image 20240805112244.png]] +![[Pasted image 20240805112315.png]]
