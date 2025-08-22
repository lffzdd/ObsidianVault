---
aliases: 3Blue1Brown-4、中心极限定理
tags: []
date created: Invalid date
date modified: 2024, 六月 17日, 05:03:56,  星期一下午
---

> [But what is the Central Limit Theorem? - YouTube](https://www.youtube.com/watch?v=zeJD6dqJ5lo&ab_channel=3Blue1Brown)

投掷骰子，计算点数之和即出现概率，骰子越多，分布越趋于正态分布
![[Pasted image 20240617004941.png]]
均值会成倍递增
方差：
![[Pasted image 20240617005056.png]]
$$\sigma_{X+Y}^2=\sigma_X^2+\sigma_Y^2$$
- ? 为什么 ->3 b 1 b 概率论

考虑 $e^{-|x|}$, 连续但不光滑的偶函数
再考虑 $e^{-x^2}$, 光滑的偶函数，再加上一个常数，$e^{-cx^2}$, $c>1$ 时，函数被挤压，$c<1$ 时，函数被拉伸 ![[Pasted image 20240617011338.png]]
再考虑曲线下的面积，对 $e^{-ax^2}$, 面积为 $\sqrt{\frac{\pi}{a}}$, 于是 $\frac{1}{\sqrt{\pi}}e^{-x^2}$ 的面积为 1，再考虑 $\frac1{\sqrt{\pi}}e^{-\frac12(x/\sigma)^2}$，其效果是将图像拉伸 $\sigma\sqrt{2}$ 倍，于是有 $\frac1{\sigma\sqrt{2}}\frac1{\sqrt{\pi}}e^{-\frac12(x/\sigma)^2}$，其面积为 1 ![[Pasted image 20240617013345.png]]
$\sigma=1$ 时，称之为标准正态分布 ![[Pasted image 20240617013827.png]]
大的要来了：![[Pasted image 20240617014142.png]]
![[Pasted image 20240617014521.png]]
$\frac{(X_1+\cdots+X_{10})-10\cdot\mu}{\sigma\cdot\sqrt{10}}$ 本质上是说**这个总和与平均值相差多少个标准差**？

```ad-important
title:**思考**
即使骰子扔出各个数的概率不定（例如1的概率为$\frac{1}{2}$，2和6的概率为$\frac{1}{4}$，其它一概为0），一个骰子扔出的点数仍然有一个均值$\mu$和标准差$\sigma$，N个这样的骰子扔出的点数的均值就为$N\mu$，标准差为$\sigma\sqrt{N}$。
```

$x$ 替换为 $\frac{(X_1+\cdots+X_{N})-N\cdot\mu}{\sigma\cdot\sqrt{N}}$，即 $\frac1{\sqrt{2\pi}}e^{-\frac12x^2}$ ，均值为 0，标准差为 1，而**此时的 $x$ 表示扔出点数与均值的距离有 $x$ 个标准差**。
约有 68.3% 的概率落在平均值的 1 个标准差范围内，由上可推出 $$\int^{1\cdot\sigma=1}_{-1}\frac1{\sqrt{2\pi}}e^{-\frac12x^2}dx\approx0.683$$
再想象一下扔骰子，一个骰子很可能在 3 到 5 之间，而两个骰子均值增大，扔出的点数也更加分散，因此标准差增大

中心极限定理：![[Pasted image 20240617115809.png]]
3. $0<Var(X_i)<\infty$ And finite $E[X]$ for that matter.![[Pasted image 20240617120155.png]]
# 公式详解

> [【官方双语】为什么正态分布里会有一个π？（不止是积分技巧）\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1wu411W7uU/?spm_id_from=333.337.search-card.all.click&vd_source=56499cc54ebd02db0ac739e485d74801)

来自泊松的经典证明： ![[Pasted image 20240617120609.png]]
- ? 为什么是 $e^{-x^2}$![[Pasted image 20240617121136.png]]
## Step 1
 ![[Pasted image 20240617122548.png]]
## Step 2
![[Pasted image 20240617161619.png]]
- Property 1
	- The probability (density) depends ==only on the distance== from the origin
- Property 2
	- The $x$ and $y$ coordinates are ==independent== from each other.

由 1，可得函数 $f(r)$
由 2，可得函数 $f_2(x,y)=g(x)h(y)=g(x)g(y)$
可得：
$$
f_2(r,0)=f(r)=g(r)g(0)
$$
表明了 $f(r)$ 等于 $g(r)$ 乘上一个常数，$f$ 和 $g$ 算是同一类东西，只是差了一个常数倍
![[Pasted image 20240617162943.png]]
$$
f(r)=f(x)f(y)
$$

我们不是在求解一个未知数 $x$,而是已知这个等式对所有的 $x$ 和 $y$ 都成立,我们要找的是满足条件的未知函数 $f(x)$
函数方程 (Functional Equation)：
$$
f(r)=f(\sqrt{x^2+y^2})=f(x)f(y)
$$
$$f(\sqrt{(8.01)^2+(1.00)^2})=f(8.01)f(1.00)$$
在我们脑海中，已经知道有一个函数满足这个性质：$f(r)=e^{-r^2}$ 
重点是，你要假装自己不知道这个函数，然后去推导满足这个性质的函数都有哪些

---
首先，引入函数 $h(x)=f(\sqrt{x})$
有 $h(x^2)=f(x)$
引入辅助函数的原因是,这会使函数 $f$ 的关键性质变得更好看
$$
\begin{align}
f(r)=f(\sqrt{x^2+y^2})=f(x)f(y)\\
h(x^2+y^2)=h(x^2)h(y^2)
\end{align}
$$
这里要注意，对 $f$ 来说，有 $\sqrt{x^2+y^2}=\sqrt{x^2+y^2}$，但对 $h$ 来说，并没有 $x^2+y^2=\sqrt{x^4+y^4}$，因为 $h$ 不是 $f$，$f$ 的性质不能直接套用到 $h$ 中
对 $h(x^2+y^2)=h(x^2)h(y^2)$，输入加法，输出乘法
![[Pasted image 20240617165030.png]]
我们直接给 $h(1)$ 取名为 $b$，于是 $h(n)=b^n$
对于任何有理数 $\frac{p}{q}$，有
$$
\begin{align}
h(\frac{p}{q})=[h(\frac{1}{q})]^p\\
[h(\frac{1}{q})]^q=h(1)\\
h(\frac{1}{q})=[h(1)]^\frac{1}{q}\\
h(\frac{p}{q})=[h(\frac{1}{q})]^p=h(1)^\frac{p}{q}=b^\frac{p}{q}
\end{align}
$$
因此，$h(n)=b^n$ 对有理数成立
```ad-lem
title: <mark class="hltr-亮橙">思考</mark>
collapse: closed
若$g(r)=f(x)f(y)=f(r)f(0)$,设$f(0)=c$，可推出
$$
\begin{align}
ch(x^2+y^2)&=h(x^2)h(y^2)\\
ch(a+b)&=h(a)h(b)\\
[h(1)]^n=f(0)^{n-1}h(n)&=c^{n-1}h(n)
\end{align}
$$
待续

```

在 $h$ 是连续函数的情况下，我们可以得出 , 其中 $x$ 和 $b$ 都是正实数，我们定义的函数 $h$ 本来就只接受正数

比起把指数函数写为某个底数 $b$ 的 $x$ 次方，数学家们更爱写 $e$ 的“常数 $c$ 乘以 $x$"次方，即 $h(x)=e^{cx}$，一直选用 $e$ 作为底数，再用参数 $c$ 来确定你讨论的是哪个具体的指数函数，这样万一要做微积分的计算也会变得更方便
于是
$$
f(x)=h(x^2)=e^{cx^2}
$$
关联 Step 1，1 中描述了 $e^{-x^2}$ 的图像，若 $c$ 为正数，$e^{cx^2}$ 的函数就会在所有方向上都膨胀到无穷大 ![[Pasted image 20240617172326.png]]
因此 $c$ 必须是一个负值，并且这个特定的常数决定了分布的分散度
![[Pasted image 20240617172643.png]]
如果我们认为这两个性质（径向对称性，各坐标独立分布）定义了高斯分布的话，那么 $\pi$ 的出现也就不那么意外了

## Step 3
>[【官方双语】到底为什么“中心极限”是正态分布啊\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1EP411p7bV/?spm_id_from=autoNext&vd_source=56499cc54ebd02db0ac739e485d74801)
