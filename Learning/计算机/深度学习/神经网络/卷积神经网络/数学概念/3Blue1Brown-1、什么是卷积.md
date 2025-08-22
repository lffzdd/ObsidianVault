>[【官方双语】那么……什么是卷积？\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1Vd4y1e7pj/?spm_id_from=333.337.search-card.all.click&vd_source=56499cc54ebd02db0ac739e485d74801)

![[Pasted image 20240614025122.png]]
$$
(a*b)=\sum_{\substack{i,j\\i+j=n}}a_{i}\cdot b_i
$$
## 快速卷积算法
从函数相乘的视角去看卷积
![[Pasted image 20240617184956.png]] 
而线性方程组可以解出函数的系数：
![[Pasted image 20240617185043.png]]
- ! 当需要对两个数组进行卷积时，可以把它们视为两个多项式的系数，对这些多项式的输出值尽可能多地采样，对这些样本逐点相乘，然后求解并恢复出系数，这就是为卷积找了个捷径 ![[Pasted image 20240617185352.png]]

但这样计算的复杂度一样很高，并没有简化计算，
我们可以**结合傅里叶变换**
![[Pasted image 20240617190249.png]]
样本点不再随机取诸如 $x=1,2,3\dots$，而是取复数，基本思路就是，找到一个**取幂时输出总在单位圆上循环**的数就行了，这样，在计算的不同项中会出现很多的相同项
![[Pasted image 20240617190546.png]]
这个输出集有一个特定的名字，叫做==*系数的离散傅里叶变化*== 
![[Pasted image 20240617190913.png]]
它还可以做逆变换，从 $\hat{h}\rightarrow h$
![[Pasted image 20240617192227.png]]
快速卷积算法使复杂度从 $O(N^2)\rightarrow  O(N\log N)$ 

*FFT*可以在各领域运用：![[Pasted image 20240617192818.png]]
```ad-note
title: 回顾小学乘法，其核心是卷积
![[Pasted image 20240617193132.png]]

---
简化超大位数相乘：![[Pasted image 20240617193343.png]]

```

