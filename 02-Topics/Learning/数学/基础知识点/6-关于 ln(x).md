# 导数
考虑对数的导数
$$
\frac{\ln(x+dx)-\ln x}{dx}
$$
考虑 
$$
\frac{e^{dx}-1}{dx}=1 ,\qquad e^{dx}=dx+1
$$
即 $x$ 每走一步 $dx$，$e^x$ 就走一步 $dx+1$
则 $\ln(x+dx)-\ln x$ 每走一步 $d(\ln(x+dx)-\ln x)$，$e^{\ln(x+dx)-\ln x}$ 就走一步 $d(\ln(x+dx)-\ln x)+1$
，则
$$
\begin{align}
&\frac{e^{\ln(x+dx)-\ln x}}{dx}\\
=&\frac{\frac{x+dx}{x}}{x}\\
=&\frac{1+\frac{dx}{x}}{dx}
\end{align}
$$
则 
$$e^{\ln(x+dx)-\ln x}=d(\ln(x+dx)-\ln x)+1=\frac{dx}{x}+1$$
$$
d(\ln(x+dx)-\ln x)=\frac{x}{dx}
$$
则
$$
\frac{\ln(x+dx)-\ln x}{dx}=\frac{\frac{dx}{x}}{dx}=\frac{1}{x}
$$
则 $x$ 走一步 $dx$，$\ln(x)$ 走一步 $d(\ln(x))=\frac{1}{x}\cdot dx$
从 $x=1$ 开始，此时 $\ln(x)=0$, $x$ 每向右走一步 $dx$，$\ln(x)$ 就向上走一步 $\frac{1}{x}\cdot dx$,
则$$
\ln x=\int^{x}_{1}\frac{dt}{t}\qquad(x>0)
$$
几何上也即曲线 $\frac{1}{t}$ 与 $t=1,t=x$ 围成的面积，只是要注意到当 $0<x<1$ 时，这面积要取为相应的负值。
# $\ln a+\ln b=\ln{ab}$
按前述定义也就是要证明
$$
\int_1^a\frac{\mathrm{d}x}{x}+\int_1^b\frac{\mathrm{d}x}{x}=\int_1^{ab}\frac{\mathrm{d}x}{x},
$$



 
 基于定积分（直观上也即面积）的[线性可加性](https://www.zhihu.com/search?q=线性可加性&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType":"answer","sourceId":2761634811})，这等价于

$$\color{red}{\int_1^a\frac{{\rm d}x}{x}}=\color{black}\int_1^{ab}\frac{{\rm d}x}{x}-\int_1^b\frac{{\rm d}x}{x}=\color{blue}{\int_b^{ab}\frac{{\rm d}x}{x}}.\\
$$

 也就是说，要证明 $\frac{1}{x}$ 在 $[1,a]$ 上的积分等于在 $[b,ab]$ 上的积分，也即下图图示中红色区域面积与蓝色区域面积相等。

下面我们就通过虽不太严谨但却很有说服力的直观来解释这个事情。

首先，请注意，对 $\frac{1}{x}$ 在 $[1,a]$ 中的每个点来说，将其变换进 $[b,ab]$ 内，横坐标扩大了 $b$ 倍，但是纵坐标又补偿性地缩小到原来的 $\frac{1}{b}$ 。基于这个简单的事实，我们来考虑定积分所涉及的**[黎曼和](https://www.zhihu.com/search?q=黎曼和&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType":"answer","sourceId":2761634811})**。

![](https://pic1.zhimg.com/v2-a41632ededce08f518cfb8aeb6be4900_r.jpg?source=2c26e567)

如图所示，将红色区域所对应的区间 $[1,a]$ 作 $n$ 等分，也对蓝色区域所对应的区间$[b,ab]$ 作 $n$ 等分，按这分划作出一列矩形。显然，蓝区域中第 $i$ 个矩形，其宽是红区域中第 $i$ 个矩形的 $b$ 倍，但高却变成了它的 $\frac{1}{b},$ 于是蓝色区域中每个矩形的面积始终等于红色区域中相同位置的那个矩形面积，于是这些面积之和也相等着，进而我们有理由相信，即使取了极限，这相等关系也不会有任何改变。

* * *
