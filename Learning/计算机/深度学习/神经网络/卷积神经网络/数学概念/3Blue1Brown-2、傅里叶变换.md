>[【官方双语】形象展示傅里叶变换\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1pW411J7s8/?spm_id_from=333.337.search-card.all.click&vd_source=56499cc54ebd02db0ac739e485d74801)

由欧拉公式 $e^{i\theta}=\cos{\theta}+i\sin{\theta}$, 在复平面中，转一圈后的落点可以用 $e^{2\pi i}$ 表示，一秒转多少圈可以用 $e^{2\pi it}$ 表示，再加上频率 $f$，得
$$
e^{2\pi{ift}}
$$
通常认为，在傅里叶变换的语境下，旋转方向为顺时针，因此
$$
e^{-2\pi{ift}}
$$
想象一下，$e^{-2\pi{ift}}$ 的图像，是一个圆，再乘以波形图像函数 $g(t)$，会得到这样一个图像：![[Pasted image 20240615010700.png]] ^432294
```ad-question
title:为什么$e$的虚数次方长这样？
collapse: closed

[【官方双语】欧拉公式与初等群论\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1fx41187tZ/?spm_id_from=333.337.search-card.all.click&vd_source=56499cc54ebd02db0ac739e485d74801)
```

再考虑质心，$\frac{1}{N}\sum_{k=1}^{N}g(t_{k})e^{-2\pi ift_{k}}$：![[Pasted image 20240615011110.png]]
取极限：
$$\frac{1}{t_{2}-t_{1}}\int_{t_{1}}^{t_{2}}g(t)e^{-2\pi ift}dt$$
而傅里叶变换是其中的积分部分：
$$\int_{t_{1}}^{t_{2}}g(t)e^{-2\pi ift}dt$$
[[3Blue1Brown-3、欧拉公式与初等群论#个人小结]]
其含义不再是质心，而是把它倍增。如果说原图像持续了3秒，那就把质心乘上3，持续 6 秒就乘 6。物理效果上，如果某个频率持续了很长时间，这个频率的傅里叶变换的模长就被放得很大，**信号存在越久**，此频率的傅里叶变换的值就越大
  ![[Pasted image 20240615012901.png]]