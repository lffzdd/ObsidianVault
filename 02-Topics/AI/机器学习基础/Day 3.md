
-  $$
\frac{\partial}{\partial w}J(w,b)=\frac{\partial}{\partial w}\frac{1}{2m}\sum(wx^i+b-y^i)^2=\frac{1}{m}\sum(f(x^i)-y^i)x^i
$$
- $$
\frac{\partial}{\partial b}J(w,b)=\frac{\partial}{\partial b}\frac{1}{2m}\sum(wx^i+b-y^i)^2=\frac{1}{m}\sum(f(x^i)-y^i)
$$
事实证明，当使用线性回归的平方误差成本函数时，成本函数不会也永远不会有多个局部最小值。它是一个凸函数(convex function)，只有一个极小值

