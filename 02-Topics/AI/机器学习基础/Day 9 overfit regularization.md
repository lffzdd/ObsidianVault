---
aliases: Day 9 overfit
tags: []
date created: Invalid date
date modified: 2024, 六月 13日, 05:30:10,  星期四下午
---
- underfit
- generalization
- overfit
---
- ~ **To reduce overfit**
1. Collect more data
2. Select features
	 - Feature selection
3. Reduce size of parameters
	 - "Regularization"
# 正则化
## 范数惩罚
> [白话解释正则化原理-CSDN博客](https://blog.csdn.net/LEEANG121/article/details/102692653)
### $L2$ 正则化
先看看 $L2$ 正则化：
$$J(w)=\frac{1}{2m}\left(\sum^{m}_{i=1}(wx-y)^2+\lambda\sum^{n}_{i=1}w^2\right)$$
则 $$\frac{\partial J}{\partial w}=\frac{1}{m}\dots+\frac{\lambda}{m}w$$
梯度下降时就有
$$w_{{j}} = w_{{j}} (1 - \alpha\frac{\lambda}{{m}} ) - \alpha\frac{1}{{m}} \sum_{{i}=1}^{{n}} ({h}w({x}^{({i})}) - {y}^{({i})}){x}_{{j}}^{({i})}$$
相当于把 $w_j$ 缩小了，$\lambda$ 越大，缩小越明显
若是 $L1$ 正则化，有
$$
\frac{\partial J}{\partial w}=\frac{1}{m}\dots+\frac{\lambda}{2m}
$$
- ! 实际上，这个 $+$ 号可能为减号
$$
w_{{j}} = w_{{j}} - \alpha\frac{\lambda}{{2m}} - \alpha\frac{1}{{m}} \sum_{{i}=1}^{{n}} ({h}w({x}^{({i})}) - {y}^{({i})}){x}_{{j}}^{({i})}
$$
### $L1$ 正则化
$L1$ 正则化项的导数可以表示为：
$$
\begin{cases}1 & \text{if } w_j > 0 \\ -1 & \text{if } w_j < 0 \\ 0 & \text{if } w_j = 0 \\ \end{cases}  $$
#### 梯度更新公式 
将正则化项的导数加入到梯度更新公式中，我们得到： 
$$ w_j := w_j - \alpha \left( \frac{\partial J}{\partial w_j} + \lambda \text{sign}(w_j) \right)  $$
其中 $\text{sign}(w_j)$ 表示 $w_j$ 的符号函数，其值为1（若 \( w_j > 0 \)），-1（若 \( w_j < 0 \)），或者0（若 \( w_j = 0 \)）。
#### $L1$正则化导致稀疏的原因 
$L1$ 正则化导致稀疏性的主要原因在于其梯度更新中的 $\lambda \text{sign}(w_j)$ 项。这个项对  $w_j$  施加了一个固定的力度，推动权重向零靠近。
当 $w_j$  足够小时，梯度更新中的正则化项可能会超过原始损失函数的梯度项，从而使 $w_j$  变为零。具体来说： 
1. **大权重的减小**：对于较大的权重 $w_j$ ，$\lambda \text{sign}(w_j)$ 会逐渐减小它们的绝对值。 
2. **小权重变为零**：对于接近零的权重 $w_j$ ，\(\lambda \text{sign}(w_j)\) 会有足够的力量将它们推向零并保持在零。 
#### 数学解释假设
$\frac{\partial J}{\partial w_j}$ 的值相对较小，更新公式中的 $\lambda \text{sign}(w_j)$ 就可能占据主导地位： $w_j := w_j - \alpha \lambda \text{sign}(w_j)$ 在这种情况下，$w_j$  将快速趋向零。一旦 $w_j$  变为零，更新公式会维持它为零，除非 $\frac{\partial J}{\partial w_j}$足够大，能够克服正则化项的影响。 
#### 总结 
$L1$正则化通过在损失函数中加入权重绝对值之和的惩罚项，直接对每个权重施加了一个推动力，迫使它们趋向零。这种推动力使得许多不重要的权重最终变为零，从而实现参数稀疏化。这种特性在特征选择和模型简化方面非常有用，可以提高模型的可解释性和泛化能力。
## 正则化逻辑回归
>[!info] [[Day 8 Logistics#成本函数（代价函数）]]

$$J(\vec{\mathbf{w}},b)=-\frac{1}{m}\sum_{i=1}^{m}\left[y^{(i)}\mathrm{log}\left(f_{\vec{\mathbf{w}},b}(\vec{\mathbf{x}}^{(i)})\right)+\left(1-y^{(i)}\right)\mathrm{log}\left(1-f_{\vec{\mathbf{w}},b}(\vec{\mathbf{x}}^{(i)})\right)\right]+\frac{\lambda}{2m}\sum_{j=1}^{n}w_{j}^{2}$$
$$
w_{{j}} = w_{{j}} (1 - \alpha\frac{\lambda}{{m}} ) - \alpha\frac{1}{{m}} \sum_{{i}=1}^{{n}} (\sigma({x}^{({i})}) - {y}^{({i})}){x}_{{j}}^{({i})}
$$