```python
    def forward(self, x: Tensor):
        batch_size, seq_len, embed_dim = x.shape

        Q = self.w_q(x)
        # 等价于 x @ self.w_q.weight.T + self.w_q.bias , 得到 [batch_size, seq_len, k_dim]
        K: Tensor = self.w_k(x)
        V = self.w_v(x)
  
        scores = Q @ K.transpose(-2, -1) / self.scale
        scores = scores.softmax(dim=-1)
```

$Q,K$ 都是 $[batch\_{size},seq\_{len},k\_{dim}]$,
$Q$ 从上到下 $seq\_len$ 行是每一行都是一个 Q 向量,K 转置后是从左到右每一列都是一个 K 向量,
$Q\cdot K^T$ 就是下面这个矩阵:

$$
\begin{bmatrix}
Q_1 \cdot K_1 & Q_1 \cdot K_2 & \cdots & Q_1 \cdot K_{\text{seq\_len}} \\
Q_2 \cdot K_1 & Q_2 \cdot K_2 & \cdots & Q_2 \cdot K_{\text{seq\_len}} \\
\vdots        & \vdots        & \ddots & \vdots                      \\
Q_{\text{seq\_len}} \cdot K_1 & Q_{\text{seq\_len}} \cdot K_2 & \cdots & Q_{\text{seq\_len}} \cdot K_{\text{seq\_len}}
\end{bmatrix}
$$
要针对每个 $Q$ 自身进行更新,针对自己的每一项进行 $softmax$,所以是对每一行进行,那么就是针对最后一个维度,也就是 `softmax(-1)`