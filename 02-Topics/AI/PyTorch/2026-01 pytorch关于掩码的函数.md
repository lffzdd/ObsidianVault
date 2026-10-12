### User Input

mask要右上三角包括中间斜线,要怎么做才好,我对pytorch的很多方法不了解,自己写的话会很用循环很笨地去写

### Planner Response

PyTorch 有现成的函数！不需要用循环。

## 使用 `torch.triu` 创建上三角 mask

### 方法 1：使用 `torch.triu`（上三角）

```python
import torch

seq_len = 5

# 创建全 1 矩阵
ones = torch.ones(seq_len, seq_len)

# triu = "triangle upper"，获取上三角部分
# diagonal=1 表示从对角线上方 1 格开始（不包括对角线）
# diagonal=0 表示从对角线开始（包括对角线）
mask = torch.triu(ones, diagonal=1)

print(mask)
# tensor([[0., 1., 1., 1., 1.],   ← 位置 0 只能看自己
#         [0., 0., 1., 1., 1.],   ← 位置 1 能看 0, 1
#         [0., 0., 0., 1., 1.],   ← 位置 2 能看 0, 1, 2
#         [0., 0., 0., 0., 1.],   ← 位置 3 能看 0, 1, 2, 3
#         [0., 0., 0., 0., 0.]])  ← 位置 4 能看所有
#
# 1 = 需要屏蔽的位置（未来位置）
# 0 = 可以看的位置（当前和过去）
```

### 方法 2：使用 `torch.tril`（下三角，更直观）

```python
# tril = "triangle lower"，获取下三角部分
mask = torch.tril(torch.ones(seq_len, seq_len))

print(mask)
# tensor([[1., 0., 0., 0., 0.],
#         [1., 1., 0., 0., 0.],
#         [1., 1., 1., 0., 0.],
#         [1., 1., 1., 1., 0.],
#         [1., 1., 1., 1., 1.]])
#
# 1 = 可以看的位置
# 0 = 需要屏蔽的位置
```

## 如何应用 mask？—— 用 `-inf` 屏蔽

```python
# 把需要屏蔽的位置设为 -inf
# softmax(-inf) = 0，完美屏蔽！

# 方法：用 masked_fill
mask = torch.triu(torch.ones(seq_len, seq_len), diagonal=1).bool()
#                                                           ↑ 转成布尔值

scores = scores.masked_fill(mask, float('-inf'))
# mask 为 True 的位置，填入 -inf
```

## 完整代码

```python
def forward(self, x: Tensor):
    batch_size, seq_len, embed_dim = x.shape

    Q = self.w_q(x)
    K = self.w_k(x)
    V = self.w_v(x)

    scores = Q @ K.transpose(-2, -1) / self.scale  # (batch, seq_len, seq_len)

    # 创建 causal mask（右上三角为 True，包括对角线上方）
    mask = torch.triu(torch.ones(seq_len, seq_len, device=x.device), diagonal=1).bool()
    #                                              ↑ 确保 mask 在同一设备上（CPU/GPU）
    
    # 应用 mask：把右上角设为 -inf
    scores = scores.masked_fill(mask, float('-inf'))

    # softmax 后，-inf 位置变成 0
    attention_weights = scores.softmax(dim=-1)  # (batch, seq_len, seq_len)

    # 加权求和
    output = attention_weights @ V  # (batch, seq_len, k_dim)

    return output
```

## 图解

```
scores（应用 mask 前）:
┌─────────────────────────┐
│ 0.5   0.3   0.8   0.2   │
│ 0.1   0.6   0.4   0.9   │
│ 0.7   0.2   0.5   0.3   │
│ 0.4   0.8   0.1   0.6   │
└─────────────────────────┘

mask（diagonal=1，右上角为 True）:
┌─────────────────────────┐
│  F     T     T     T    │   F = False (可以看)
│  F     F     T     T    │   T = True  (屏蔽)
│  F     F     F     T    │
│  F     F     F     F    │
└─────────────────────────┘

scores（应用 mask 后）:
┌─────────────────────────┐
│ 0.5  -inf  -inf  -inf   │
│ 0.1   0.6  -inf  -inf   │
│ 0.7   0.2   0.5  -inf   │
│ 0.4   0.8   0.1   0.6   │
└─────────────────────────┘

softmax 后：
┌─────────────────────────┐
│ 1.0   0.0   0.0   0.0   │  ← 只看自己
│ 0.38  0.62  0.0   0.0   │  ← 只看 0,1
│ 0.38  0.23  0.39  0.0   │  ← 只看 0,1,2
│ 0.21  0.31  0.15  0.33  │  ← 看所有
└─────────────────────────┘
```

## 常用函数总结

| 函数 | 作用 |
|------|------|
| `torch.triu(x, diagonal=k)` | 上三角，k=0 包含对角线，k=1 不包含 |
| `torch.tril(x, diagonal=k)` | 下三角，k=0 包含对角线 |
| `tensor.masked_fill(mask, value)` | 把 mask 为 True 的位置填充为 value |
| `tensor.bool()` | 转换为布尔张量 |