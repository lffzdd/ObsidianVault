### 常用的张量创建方法

|方法|说明|示例|
|---|---|---|
|```<br>torch.zeros(shape)<br>```|全零张量|```<br>torch.zeros(10, 512)<br>```|
|```<br>torch.ones(shape)<br>```|全一张量|```<br>torch.ones(10, 512)<br>```|
|```<br>torch.empty(shape)<br>```|未初始化|```<br>torch.empty(10, 512)<br>```|
|```<br>torch.arange(n)<br>```|序列 0 到 n-1|```<br>torch.arange(100)<br>```<br><br> → [0,1,2,...,99]|
|```<br>torch.randn(shape)<br>```|标准正态分布|```<br>torch.randn(10, 512)<br>```|

```python
def forward(self, x: Tensor) -> Tensor:
    batch_size, seq_len, embed_dim = x.shape
    
    # 位置索引 [seq_len, 1]
    pos = torch.arange(seq_len, device=x.device).unsqueeze(1)
    
    # 维度索引 [embed_dim // 2]
    div_term = 10000 ** (torch.arange(0, embed_dim, 2, device=x.device) / embed_dim)
    
    # 创建位置编码 [seq_len, embed_dim]
    pos_encoding = torch.zeros(seq_len, embed_dim, device=x.device)
    pos_encoding[:, 0::2] = torch.sin(pos / div_term)  # 偶数位置
    pos_encoding[:, 1::2] = torch.cos(pos / div_term)  # 奇数位置
    
    return x + pos_encoding
```