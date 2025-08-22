---
aliases: 2-Mish
tags: []
date created: Friday, October 18th 2024, 4:15:34 pm
date modified: Friday, October 18th 2024, 5:11:41 pm
---

## 抛物线
> [https://www.zhihu.com/question/54158310](https://www.zhihu.com/question/54158310)

![[Pasted image 20241018161557.png]]
## 双曲函数
> [https://zhuanlan.zhihu.com/p/20042215](https://zhuanlan.zhihu.com/p/20042215)

在实域内，**三角函数的值是通过单位圆和角终边上三角函数线的长度定义的**。当然这个「长度」是有正负的。
**双曲函数的值也是通过双曲线和角终边上的双曲函数线的长度定义的**。如图
![[Pasted image 20241018163737.png]]

双曲线是某点到某两个点之间的距离之差为常数，公式为 $\frac{x^{2}}{a^{2}}-\frac{y^{2}}{b^{2}}=1$
其渐进线公式为 $y=\pm \frac{b}{a}x$

这是正弦函数和余弦函数的泰勒级数展开式。公式如下：

  $$
  \begin{align}
  \sin x &= x - \frac{x^3}{3!} + \frac{x^5}{5!} - \frac{x^7}{7!} + \ldots = \sum_{n=0}^{\infty} \frac{(-1)^n x^{2n+1}}{(2n+1)!}\\
  \cos x &= 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \frac{x^6}{6!} + \ldots = \sum_{n=0}^{\infty} \frac{(-1)^n x^{2n}}{(2n)!}
\end{align}
  
$$
# Mish
Mish 是一种激活函数，定义如下：

$$
 \text{Mish}(x) = x \cdot \tanh(\text{softplus}(x)) 
$$

其中，softplus 函数是：

$$
 \text{softplus}(x) = \ln(1 + e^x) 
$$
tanh 函数是双曲函数之一:
  $$
   \tanh(x) = \frac{\sinh(x)}{\cosh(x)} = \frac{e^x - e^{-x}}{e^x + e^{-x}}
   $$
Mish 函数的特点是：

1. **平滑性**：Mish 是一个平滑的激活函数，这意味着它在整个实数范围内都是可导的，这有助于优化过程中的反向传播。
2. **自正则化**：Mish 能够在负区间内有一定的负值输出，同时在正区间内保持较强的正值输出，从而允许网络捕捉到稀疏特征。
3. **优良的性能**：在许多深度学习任务中，Mish 被证明比 ReLU 和其他激活函数性能更好，特别是在图像分类和目标检测等任务中。
4. **非单调性**：Mish 是非单调的，这意味着在某些情况下可以输出比输入更小的值，但这也提高了模型的表达能力。

Mish激活函数是由Diganta Mukherjee在2019年提出的。其目的是为了解决传统激活函数（如ReLU和tanh）在深度学习任务中的一些局限性。Mish的设计考虑了平滑性、非单调性和自正则化等特性，经过实验验证其在许多任务中具有更好的性能。Mish激活函数在暗示更好的模型学习能力和泛化能力方面表现突出，因此在深度学习领域受到关注。
![[Pasted image 20241018171731.png]]