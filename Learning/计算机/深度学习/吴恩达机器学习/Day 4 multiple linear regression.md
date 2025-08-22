---
aliases: Day 4
tags: []
date modified: 2024, 五月 21日, 09:47:09,  星期二晚上
date created: Invalid date
---

- Batch gradient descent：每个步骤都使用了所有的数据
- multiple linear regression：多元线性回归，not multivariate regression
 - ! 以下的 $x$ 都未注明上标 $i$.


$$
\begin{align}
h_\vec{\theta}(x)&=\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n\\
J(\theta)&=\frac{1}{2m}\sum_{i=1}^{m}\left(h_{\theta}(x^{(i)})-y^{(i)}\right)^{2}\\
&=\frac{1}{2m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)^{2}
\end{align}
$$

则梯度为:

$$
\begin{align}
\vec{grad}&=(\frac{\partial J}{\partial\theta_0},\frac{\partial J}{\partial\theta_1},\frac{\partial J}{\partial\theta_1},\dots,\frac{\partial J}{\partial\theta_n})\\
\frac{\partial J}{\partial\theta_1}&=\frac{1}{m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)\cdot x_1\\
&\vdots\\
\frac{\partial J}{\partial\theta_n}&=\frac{1}{m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)\cdot x_n\\
\frac{\partial J}{\partial\theta_0}&=\frac{1}{m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)
\end{align}
$$

依照梯度分量和学习率进行更新:

$$
\begin{align}
\theta_1=&\theta_1-\alpha\frac{\partial J}{\partial\theta_1}\\
=&\theta_1-\alpha\frac{1}{m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)\cdot x_1\\
&\vdots\\
\theta_2=&\theta_2-\alpha\frac{1}{m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)\cdot x_2\\
\theta_0=&\theta_0-\alpha\frac{1}{m}\sum_{i=1}^{m}\left(\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n-y^{(i)}\right)
\end{align}
$$

重复到最后得新的拟合函数:

$$
h_\vec{\theta}(x)=\theta_0+\theta_{1}x_1+\theta_2x_2+\dots+\theta_nx_n
$$

整理：
$$
J=\frac{1}{2m}[(\vec{w}\cdot\vec{X_1}+b-y_1)^2+(\vec{w}\cdot\vec{X_2}+b-y_2)^2+\dots+(\vec{w}\cdot\vec{X_m}+b-y_m)^2]
$$
$$
\begin{align}
\frac{\partial J}{\partial b}=&\frac{1}{m}[(\vec{w}\cdot\vec{X_1}+b-y_1){}\cdot 1+(\vec{w}\cdot\vec{X_2}+b-y_2){}\cdot 1+\dots+(\vec{w}\cdot\vec{X_m}+b-y_m){}\cdot 1]\\
\frac{\partial J}{\partial w_1}=&\frac{1}{m}[(\vec{w}\cdot\vec{X_1}+b-y_1){}\cdot x_{11}+(\vec{w}\cdot\vec{X_2}+b-y_2){}\cdot x_{21}+\dots+(\vec{w}\cdot\vec{X_m}+b-y_m){}\cdot x_{m1}]\\
\frac{\partial J}{\partial w_2}=&\frac{1}{m}[(\vec{w}\cdot\vec{X_1}+b-y_1){}\cdot x_{12}+(\vec{w}\cdot\vec{X_2}+b-y_2){}\cdot x_{22}+\dots+(\vec{w}\cdot\vec{X_m}+b-y_m){}\cdot x_{m2}]\\
&\vdots\\
\frac{\partial J}{\partial w_n}=&\frac{1}{m}[(\vec{w}\cdot\vec{X_1}+b-y_1){}\cdot x_{1n}+(\vec{w}\cdot\vec{X_2}+b-y_2){}\cdot x_{2n}+\dots+(\vec{w}\cdot\vec{X_m}+b-y_m){}\cdot x_{mn}]
\end{align}
$$

## 梯度下降算法的向量化
>[梯度下降算法的向量化](https://zhuanlan.zhihu.com/p/77253076?from_voters_page=true)

$$\bigtriangledown J(\theta)=\frac{1}{m}\begin{bmatrix}\sum_{i=1}^m(y^{(i)}-X^{(i)}\theta)\cdot(-X_0^{(i)})\\\sum_{i=1}^m(y^{(i)}-X^{(i)}\theta)\cdot(-X_1^{(i)})\\\sum_{i=1}^m(y^{(i)}-X^{(i)}\theta)\cdot(-X_2^{(i)})\\\cdots\\\sum_{i=1}^m(y^{(i)}-X^{(i)}\theta)\cdot(-X_n^{(i)})\end{bmatrix}=\frac{1}{m}\cdot X^T\cdot(X\theta-y)$$

其中
$$\left.X=\left[\begin{array}{ccccc}{1}&{X_{1}^{(1)}}&{X_{2}^{(1)}}&{\ldots}&{X_{n}^{(1)}}\\{1}&{X_{1}^{(2)}}&{X_{2}^{(2)}}&{\ldots}&{X_{n}^{(2)}}\\{\ldots}&&&&{\ldots}\\{1}&{X_{1}^{(m)}}&{X_{2}^{(m)}}&{\ldots}&{X_{n}^{(m)}}\end{array}\right.\right]\quad\theta=\left[\begin{array}{c}{\theta_{0}}\\{\theta_{1}}\\{\theta_{2}}\\{\ldots}\\{\theta_{n}}\end{array}\right]$$