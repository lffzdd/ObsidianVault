---
aliases:
  - Day 8 Logistics
tags: 
date modified: 2024-06-06T16:06:00
date created: 2024, 六月 8日, 02:23:19,  星期六凌晨
source: 吴恩达
---

# 逻辑回归

> [一文彻底理解逻辑回归：从公式推导到代码实现 - Marvin-wen - 博客园](https://www.cnblogs.com/marvin-wen/p/15966151.html)

逻辑函数有一个很好的特性，其导函数可以转化成本身的一个表达式，推导过程如下：
$$\begin{aligned}
\sigma^{\prime}(z)& =\left(\frac{1}{1+e^{-z}}\right)^{\prime} \\
&=\frac{e^{-z}}{\left(1+e^{-z}\right)^2} \\
&=\frac{1+e^{-z}-1}{\left(1+e^{-z}\right)^2} \\
&=\frac{1}{(1+e^{-z})}\Bigg(1-\frac{1}{(1+e^{-z})}\Bigg) \\
&=\sigma(z)(1-\sigma(z))
\end{aligned}$$
在二分类模型中，事件发生与不发生的概率之比 $\frac p{1-p}$ 称为事件的几率 (odds)。令$\sigma(z)=p$,解得：
$$z=ln(\frac{p}{1-p})$$
也就是说，**线性回归的结果（即 $𝑧$ ）等于对数几率**。-
- ! 这里 $\ln$ 就是 $\ln$ 的意思

# 最大似然估计
>[详解最大似然估计（MLE）、最大后验概率估计（MAP），以及贝叶斯公式的理解-CSDN博客](https://bln.csdn.net/u011508640/article/details/72815981)
>[【学习笔记】二分类问题中的“最大似然”与“交叉熵损失”概念的理解 - jrltx - 博客园](https://www.cnblogs.com/asdfsag/p/15737631.html)

```ad-note
title: ==**贝叶斯定律**==
collapse: open

>[一文搞懂贝叶斯定理（原理篇） - 廖雪峰的官方网站](https://www.liaoxuefeng.com/article/1565255725482019)
>
>[一文读懂贝叶斯原理（Bayes‘ theorem）\_贝叶斯定理 原始参考文献-CSDN博客](https://bln.csdn.net/gsjthxy/article/details/110825514)

$$P(A|B) P(B)=P(A\cap B)=P(B|A) P(A).$$

```

# 成本函数（代价函数）
使用极大似然估计法来表示代价函数。
对于一个二分类模型，似然函数为：
$$
L(w)=\prod_{i=1}^{m} P\left(y^{(i)} \mid x^{(i)} ; w\right)=\prod_{i=1}^{m}\left(\sigma\left(x^{(i)}\right)\right)^{y^{(i)}}\left(1-\sigma\left(x^{(i)}\right)\right)^{1-y^{(i)}}
$$
等式两边同时取对数：
$$
l(w)=\ln L(w)=\sum_{i=1}^{m}\left(y^{(i)} \ln \left(\sigma\left(x^{(i)}\right)\right)+\left(1-y^{(i)}\right) \ln \left(1-\sigma\left(x^{(i)}\right)\right)\right)
$$
成本函数为：
$$
\begin{align}
J(w)=-\frac{1}{m}l(w)&=-\frac{1}{m}\sum^{m}_{i=1}\left[y\ln{\frac{1}{1+e^{-z}}}+(1-y)\ln{\frac{e^{-z}}{1+e^{-z}}}\right]\\
&=\frac{1}{m}\sum^{m}_{i=1}\left[y\ln{(1+e^{-z})+(1-y)\ln{(1+e^z)}}\right]
\end{align}
$$
对 $J(w)$ 求编导，得：
$$
\begin{aligned}
\frac{\partial}{\partial w_{j}} J(w)&=\frac{\partial}{\partial w_{j}}\left(-\frac{1}{m} \sum_{i=1}^{m}\left(-y^{(i)} \ln \left(1+e^{-w^{T} x^{(i)}}\right)-\left(1-y^{(i)}\right) \ln \left(1+e^{w^{T} x^{(i)}}\right)\right)\right) \\
&=-\frac{1}{m} \sum_{i=1}^{m}\left(-y^{(i)} \frac{-x_{j}^{(i)} e^{-w^{T} x^{(i)}}}{1+e^{-w^{T} x^{(i)}}}-\left(1-y^{(i)}\right) \frac{x_{j}^{(i)} e^{w^{T} x^{(i)}}}{1+e^{w^{T} x^{(i)}}}\right) \\
&=-\frac{1}{m} \sum_{i=1}^{m}\left(y^{(i)}-\sigma\left(x^{(i)}\right)\right) x_{j}^{(i)} \\
&=\frac{1}{m} \sum_{i=1}^{m}\left(\sigma\left(x^{(i)}\right)-y^{(i)}\right) x_{j}^{(i)}
\end{aligned}
$$
所以：
$$
w_
{j}:=w_{j}-\frac{\partial}{\partial w_{j}} J(w)=w_{j}-\alpha \frac{1}{m} \sum_{i=1}^{m}\left(\sigma\left(x^{(i)}\right)-y^{(i)}\right) x_{j}^{(i)}
$$
