---
aliases: Day 7 SIGMOD
tags:
date created: 2024, 六月 6日, 01:38:02,  星期四下午
date modified: 2024, 六月 6日, 01:41:22,  星期四下午
---
### 自我推导
<iframe src="https://www.geogebra.org/graphing/jqw8m4py?embed" width="800" height="600" allowfullscreen style="border: 1px solid #e4e4e4;border-radius: 4px;" frameborder="0"></iframe>

 $\frac{1-e^{-x}}{1+e^{-x}}$ 有以下性质：
1. 奇函数
2. 单调递增
3. 有范围性，在 $(-1,1)$ 之间

1. 先把范围大小放缩到 1，即 $(-\frac{1}{2},\frac{1}{2})$
	- $\frac{1-e^{-x}}{2(1+e^{-x})}$
2. 再加 $\frac{1}{2}$
	- $\frac{1-e^{-x}}{2(1+e^{-x})}+\frac{1}{2}=\frac{1}{1+e^{-x}}$
即 sigmoid 函数