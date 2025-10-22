---
~
---
$$
\mathrm{P}(X\in A|Y=y)=\int_{A}f_{X|Y}(x|y)\mathrm{d}x.
$$

$$
f_{X|Y}(x|y)=\frac{f_{X,Y}(x,y)}{f_Y(y)}
$$

# 条件期望
记 $X$ 和 $Y$ 为联合连续随机变量, $A$ 是满足 $P(A)\gt0$ 的事件.
-  $X$ 在给定事件 $A$ 之下的条件期望由下式定义
$$\mathrm{E}[X|A]=\int_{-\infty}^{\infty}xf_{X|A}(x)\mathrm{d}x,$$
给定 $Y=y$ 之下的条件期望由下式定义
$$\mathrm{E}[X|Y=y]=\int_{-\infty}^{\infty}xf_{X|Y}(x|y)\mathrm{d}x.$$
- 期望规则仍然有效：
$$\mathrm{E}[g(X)|A]=\int_{-\infty}^{\infty}g(x)f_{X|A}(x)\mathrm{d}x,$$
$$\mathrm{E}[g(X)|Y=y]=\int_{-\infty}^{\infty}g(x)f_{X|Y}(x|y)\mathrm{d}x.$$
- 全期望定理：设 $A_1,A_2,\cdots,A_n$ 为互不相容的 $n$ 个事件，对每个 $i$ ,
$\overline{\mathrm{P}(A_i)>0,\text{并且这些事件形成样本空间的一个分割.则}}$
$$\operatorname{E}[X]=\sum_{i=1}^n\operatorname{P}(A_i)\operatorname{E}[X|A_i].$$
相似地，
$$\operatorname{E}[X]=\int_{-\infty}^{\infty}\operatorname{E}[X|Y=y]f_{Y}(y)\mathrm{d}y.$$
- 涉及几个随机变量的函数的情况，具有完全相似的结果.例如
$$\begin{aligned}\operatorname{E}[g(X,Y)|Y=y]&=\int g(x,y)f_{X|Y}(x|y)\mathrm{d}x,\\\operatorname{E}[g(X,Y)]&=\int\operatorname{E}[g(X,Y)|Y=y]f_{Y}(y)\mathrm{d}y.\end{aligned}$$
