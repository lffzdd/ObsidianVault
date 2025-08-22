---
aliases: 3Blue1Brown-5、卷积（continuous）
tags: []
date created: Invalid date
date modified: 2024, 七月 1日, 06:39:44,  星期一晚上
---

> [【官方双语】卷积的两种可视化|概率论中的X+Y既美妙又复杂\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1Yk4y1K7Az/?spm_id_from=333.337.search-card.all.click&vd_source=56499cc54ebd02db0ac739e485d74801)
考虑两个正态分布 $X$ 和 $Y$ 即两者之和 $X+Y$：
![[Pasted image 20240630210347.png]]

 - ? 如何求两个随机变量的和 ![[Pasted image 20240630210705.png]]
这相当于一个特殊的操作来结合两个随机变量的分布，这操作有一个专门的名字：叫做卷积 
$$
\begin{align}
P_{X+Y}(s)&=\sum_{x=1}^{6}P_{X}(x) P_Y(s-x)\\\\
P_{X+Y}(s)&=\int_{-\infty}^{\infty}P_{X}(x) P_Y(s-x)dx
\end{align}
$$
![[Pasted image 20240630211230.png]]

# 三维下的思考
![[Pasted image 20240701111748.png]]
![[Pasted image 20240701111844.png]]
沿着对角线相加
卷积：
![[Pasted image 20240701112041.png]] ![[Pasted image 20240701112225.png]]
# 卷积的含义

```ad-lem
卷积可以结合两个不同的函数，并得到一个新的函数
```

![[Pasted image 20240701120650.png]]
![[Pasted image 20240701121013.png]]
$$
P_{X+Y}=[P_X*P_Y](s)=\sum_{x=1}^6P_X(x)\cdot P_Y(s-x)
$$
考虑连续分布：
![[Pasted image 20240701121621.png]]
![[Pasted image 20240701122024.png]]
![[Pasted image 20240701123819.png]]

```ad-col2
![[Pasted image 20240701134820.png]]

![[Pasted image 20240701134917.png]]
```

$$
[f*g](s)=\int_{-\infty}^{\infty}f(x)\cdot g(s-x)dx
$$
对左边图像来说，$x$ 是输入值，$s$ 只是参数，对右边来说：![[Pasted image 20240701135318.png]]
# 举例
若有两个均匀分布：![[Pasted image 20240701135525.png]]
$$f_Z(z)=\int_{-1/2}^{1/2}f_X(x)f_Y(z-x) dx$$
当 $x$ 和 $z-x$ 都处于 $[-\frac{1}{2},\frac{1}{2}]$ 时 $f_X(x)f_Y(z-x)$ 才不为 $0$，考虑 $x$ 固定在 $[-\frac{1}{2},\frac{1}{2}]$，$(z-x) \in[-\frac{1}{2},\frac{1}{2}]\rightarrow z-\frac{1}{2}\le x \le z+\frac{1}{2}$, 对 $x$ 有两种积分域：
1. $[-\frac{1}{2},z+\frac{1}{2}]$ ，此时 $z-\frac{1}{2}\le-\frac{1}{2}(左边界),z+\frac{1}{2}\le \frac{1}{2}(右边界)$，得 $-1\le z \le0$
2. 和 $[z-\frac{1}{2},\frac{1}{2}]$, 此时 $z-\frac{1}{2}\ge-\frac{1}{2}(左边界),z+\frac{1}{2}\ge \frac{1}{2}(右边界)$，得 $0\le z \le1$
对第一种积分域：
$$
\begin{align}
f_Z(z)&=\int_{-1/2}^{z+1/2}f_X(x)f_Y(z-x) dx\\
&=\int_{-1/2}^{z+1/2}1dx\\
&=z+1
\end{align}
$$
由积分边界得定义域 $-1\le z \le0$
对第二种积分域：
$$
\begin{align}
f_Z(z)&=\int_{z-1/2}^{1/2}f_X(x)f_Y(z-x) dx\\
&=\int_{z-1/2}^{1/2}1dx\\
&=-z+1
\end{align}
$$
由积分边界得定义域 $0\le z \le1$
综上
$$
f_Z(z)=\begin{cases}z+1,&\text{如果}-1\leq z\leq0\\1-z,&\text{如果}0<z\leq1\\0,&\text{其他情况}\end{cases}
$$
>[!hint] **从空间的角度去看**
>![[82891e85003a5aa03005a8c52c19a0d.jpg]]


![[Pasted image 20240701145658.png]]
# 三个变量之和
![[Pasted image 20240701145825.png]]
![[Pasted image 20240701145839.png]]
![[Pasted image 20240701150408.png]]
礼帽函数的好处在于，乘以它的效果是滤掉上方图像的值 ![[Pasted image 20240701150551.png]] ![[Pasted image 20240701150654.png]]

```ad-note
title:滑动平均
collapse:closed
滑动平均（Moving Average）是一种统计方法，用于分析数据序列，通过计算固定大小的窗口内数据点的平均值，从而平滑数据，减少波动和噪声，使数据趋势更加明显。
```
当加更多个变量
![[Pasted image 20240701151155.png]]
![[Pasted image 20240701151258.png]]
若不放缩，函数将不断被推平：![[Pasted image 20240701151446.png]]
# 任意情况
![[Pasted image 20240701151606.png]]
中心极限定理，说的就是：![[Pasted image 20240701151642.png]]
![[Pasted image 20240701151657.png]] ![[Pasted image 20240701151717.png]]
钟形曲线就好比是 ![[Pasted image 20240701151916.png]]
# 为什么是标准正态分布
![[Pasted image 20240701151935.png]] ![[Pasted image 20240701152100.png]]
![[Pasted image 20240701152123.png]]
![[Pasted image 20240701152211.png]]
![[Pasted image 20240701152229.png]] ![[Pasted image 20240701152243.png]]
## 转三维
![[Pasted image 20240701152302.png]]
![[Pasted image 20240701152352.png]]
如果从一个专门的角度观察：（对于任一点 $(x,y)$，从正对 $x$ 和 $y$ 的方向看都是原平面图像，正对 $x$ 是 $f(x)\cdot g(y_0)$，正对 $y$ 是 $g(y)\cdot x(0)$）![[Pasted image 20240701152435.png]]
如果你仅仅想观察 $x$ 与 $y$ 之和固定的那些结果，![[Pasted image 20240701152606.png]]
![[Pasted image 20240701152628.png]] ![[Pasted image 20240701152652.png]]
![[Pasted image 20240701152723.png]]
![[Pasted image 20240701152747.png]]
![[Pasted image 20240701152900.png]]
![[Pasted image 20240701152913.png]]
![[Pasted image 20240701152952.png]]
![[Pasted image 20240701153003.png]]
 ![[Pasted image 20240701153305.png]]
 ![[Pasted image 20240701153428.png]]
## 为什么是 $\sqrt{2}$ 倍
![[Pasted image 20240701154931.png]]
![[Pasted image 20240701155025.png]]
![[Pasted image 20240701155043.png]]
![[Pasted image 20240701155121.png]]
## 两个正态分布相加
![[Pasted image 20240701155547.png]]
![[Pasted image 20240701155601.png]]
![[Pasted image 20240701155633.png]]
![[Pasted image 20240701170111.png]]
