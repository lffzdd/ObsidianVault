### 常用的张量创建方法

|方法|说明|示例|
|---|---|---|
|```<br>torch.zeros(shape)<br>```|全零张量|```<br>torch.zeros(10, 512)<br>```|
|```<br>torch.ones(shape)<br>```|全一张量|```<br>torch.ones(10, 512)<br>```|
|```<br>torch.empty(shape)<br>```|未初始化|```<br>torch.empty(10, 512)<br>```|
|```<br>torch.arange(n)<br>```|序列 0 到 n-1|```<br>torch.arange(100)<br>```<br><br> → [0,1,2,...,99]|
|```<br>torch.randn(shape)<br>```|标准正态分布|```<br>torch.randn(10, 512)<br>```|

