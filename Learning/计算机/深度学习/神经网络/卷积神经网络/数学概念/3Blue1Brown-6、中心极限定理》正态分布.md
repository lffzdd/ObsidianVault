>[【官方双语】到底为什么“中心极限”是正态分布啊\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1EP411p7bV/?spm_id_from=autoNext&vd_source=56499cc54ebd02db0ac739e485d74801)

# 两个正态函数的卷积
![[Pasted image 20240702110317.png]]
![[Pasted image 20240702110332.png]]
![[Pasted image 20240702110440.png]]
![[Pasted image 20240702110822.png]] 这个特性还可以唯一刻画出钟形曲线
![[Pasted image 20240702112210.png]]
![[Pasted image 20240702112242.png]]
![[Pasted image 20240702112359.png]]
比计算其他方向上的切片面积要简单得多
![[Pasted image 20240702112925.png]]
![[Pasted image 20240702112956.png]]
![[Pasted image 20240702113053.png]]
$$
\begin{aligned}\text{Area}=e^{-(s/{\sqrt{2})^{2}} }&\sqrt{\pi}\\&\boxed{\int_{-\infty}^\infty e^{-y^2}dy}\end{aligned}
$$
![[Pasted image 20240702113635.png]]
![[Pasted image 20240702113705.png]]
$$
e^{-x^{2}}\ast e^{-y^{2}}=\sqrt{\frac{\pi}{2}}\quad e^{-\frac{s^{2}}{2}}
$$
![[Pasted image 20240702133824.png]]

有两个分布 $X$ 和 $Y$，$P_{X}(x)=e^{-x^{2}}$，$P_{Y}(y)=e^{-y^{2}}$，考虑从空间角度去看：![[82891e85003a5aa03005a8c52c19a0d 1.jpg]]
$$
\begin{align}
P_{X+Y}(s)=P_{X}(x)\ast P_{Y}(y)&=\int^{+\infty}_{-\infty}e^{-(s-y)^2}e^{-y^{2}}dy\\
&=\frac{1}{\sqrt{2}}\int^{+\infty}_{-\infty}e^{-(\frac{s}{\sqrt{2}})^2}e^{-y^{2}}dy\\
&=\sqrt{\frac{\pi}{2}}\quad e^{-\frac{s^{2}}{2}}
\end{align}
$$
# 中心极限定理的证明
某一分布取很多随机变量相加总会趋近于万有的形状——高斯函数
要分两步证明：
1. 对于任意有限方差的初始分布，存在某个万有的形状，其为重复卷积的过程的极限。这通常要用到一个工具，叫做“***矩母函数***”（矩生成函数）
2. 第二步，证明两个高斯函数的卷积依然是高斯函数（如上文）

旋转对称性刻画了正态分布的分布函数必须为$e$ 的 $-x^2$ 次方的形式