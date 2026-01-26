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

```python
"""scores的形状:

        | Q_1·K_1 | Q_1·K_2 | ... | Q_1·K_n |
        | Q_2·K_1 | Q_2·K_2 | ... | Q_2·K_n |
        | ...   | ...   | ... | ...   |
        | Q_n·K_1 | Q_n·K_2 | ... | Q_n·K_n |

        对每一行进行了softmax,得到
        
        | s_11 | s_12 | ... | s_1n |
        | s_21 | s_22 | ... | s_2n |
        | ...  | ...  | ... | ...  |
        | s_n1 | s_n2 | ... | s_nn |

        V的形状是:
        | V_1 |
        | V_2 |
        | ... |
        | V_n |

        然后 scores @ V 就是
        
        | s_11·V_1 | s_12·V_2 | ... | s_1n·V_n |
        | s_21·V_1 | s_22·V_2 | ... | s_2n·V_n |
        | ...     | ...     | ... | ...     |
        | s_n1·V_1 | s_n2·V_2 | ... | s_nn·V_n |
"""
```

不能让后面的单词影响前面的单词,也不能让自己影响自己,某些地方要置空,上面没有考虑到这一点,
且 $softmax$ 是针对已有分数进行的归一化,这里出现的 " 已有分数 " 就不应该有那些改为 0 的分数了,所以置零操作要在进入 `softmax` 之前,考虑到 $softmax$ 是计算 $e^{score}$,改为 0 的分数应该是 $e^{score}=0$,也就是分数不能是 $0$,而是 $-\infty$,才有$e^{-\infty}\approx 0$

# 为什么掩码不是 0

这是一个在实际工程落地时非常关键的细节，直接影响模型是否能够正确“忽略”掉不需要关注的部分（如 Padding 或 Decoder 中的未来词）。

结论先行：**必须设置为 `-1e9`（或者 `-inf`），绝对不能设置为 `0`**。

这是由 **Softmax 函数的数学性质** 决定的。我们要从 Softmax 的“指数机制”讲起。

---

### 1. 原理推导：e 的幂次方

Mask（掩码）操作通常发生在 Attention Score 计算之后，Softmax 归一化之前。

公式回顾：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}} + \text{Mask}\right)V$$

Softmax 的定义是：

$$\text{Softmax}(x_i) = \frac{e^{x_i}}{\sum_{j} e^{x_j}}$$

我们来看两种设置带来的不同结果：

#### 情况 A：如果你设置为 0

假设 Mask 的位置被填补为 0。

根据指数运算规则：

$$e^0 = 1$$

这意味着，在计算概率分布时，这些本来应该被“屏蔽”掉的位置（比如句子的填充部分 padding），会获得一个非零的权重（值为 1 的分子）。

- **后果**：模型会把注意力分配给无意义的 padding token，导致有效信息的权重被稀释，模型学到错误的关联。

#### 情况 B：如果你设置为 -1e9 (负无穷)

假设 Mask 的位置被填补为 $-10^9$。

根据指数运算规则：

$$e^{-10^9} \approx \frac{1}{e^{10^9}} \approx 0$$

这意味着，在 Softmax 的分子中，这一项变成了 0。

- **后果**：该位置的概率权重为 0。在后续 $V$（Value）的加权求和中，这个位置对应的信息会被完全忽略。这正是我们要的效果。

---

### 2. 为什么是 -1e9 而不是真正的 -inf？

在数学上，理想的值确实是负无穷（$-\infty$）。但在计算机工程实践（特别是使用 GPU 训练）中，我们通常使用 `-1e9` 或 `-1e4` 这样的“大负数”来代替，主要有两个历史和工程原因：

1. FP16（半精度浮点数）的数值稳定性：
    
    在深度学习发展的中期（特别是 Transformer 刚出来那几年，以及 NVIDIA Tensor Core 普及初期），混合精度训练（Mixed Precision Training）非常流行。
    
    - `float16` 的表示范围很小，最小负数大约在 $-65504$。
    - 如果代码中直接使用 `-inf`，在某些旧版本的 CUDA 核函数或特定的优化器实现中，可能会因为下溢出（Underflow）导致计算出现 `NaN`（Not a Number），进而炸毁整个训练过程。
    - 使用 `-1e9`（对于 float32）或 `-1e4`（对于 float16）是一个**安全区**。它足够小，能让 Softmax 结果为 0；又没有小到触碰浮点数的边界。
        
2. 现代框架的优化：
    
    值得注意的是，在现代的 PyTorch 中，直接使用 float('-inf') 已经非常安全且标准了。PyTorch 的 softmax 实现对特殊值做了很好的处理。
    
    但在阅读许多经典论文的代码（如 BERT 的 HuggingFace 早期实现）时，你会经常看到 `-10000.0` 或 `-1e9`，这是一种为了兼容性和稳定性的“防御性编程”习惯。

---

### 3. 代码实战对比

为了让你更直观地看到这两者的区别，我们可以看一段 PyTorch 的微型演示：

Python

```
import torch
import torch.nn.functional as F

# 假设 attention scores (未经过 softmax)
scores = torch.tensor([1.0, 2.0, 3.0, 4.0])

# 我们要 mask 掉最后一个元素 (假设它是 padding)
# 错误做法：设置为 0
mask_zero = torch.tensor([1.0, 2.0, 3.0, 0.0])
prob_zero = F.softmax(mask_zero, dim=0)
print(f"Mask with 0:   {prob_zero}")
# 结果类似: [0.08, 0.23, 0.64, 0.03] -> 最后一个元素竟然还有 3% 的注意力！

# 正确做法：设置为 -1e9
mask_inf = torch.tensor([1.0, 2.0, 3.0, -1e9])
prob_inf = F.softmax(mask_inf, dim=0)
print(f"Mask with -1e9:{prob_inf}")
# 结果类似: [0.09, 0.24, 0.66, 0.00] -> 最后一个元素也就是我们想屏蔽的位置，完全为 0。
```

### 4. 总结

- **设置为 0**：会导致 $e^0=1$，无效区域依然分走注意力，**逻辑错误**。
- **设置为 -1e9**：会导致 $e^{-1e9} \approx 0$，无效区域注意力归零，**逻辑正确**。

这也体现了深度学习中“**加法逻辑**”与“**乘法/指数逻辑**”的区别：在累加操作（如 Bias）中，无影响是 0；但在 Softmax 的指数操作前，无影响必须是负无穷。

### 下一步建议

既然您正在研究 Transformer 的底层代码，下一步可以让我为您演示 **PyTorch 中 `MultiheadAttention` 模块是如何处理 `key_padding_mask` 和 `attn_mask` 的**。它们在内部实现上略有不同（一个是布尔值 Mask，一个是加法 Mask），这往往是初学者最容易混淆的地方。

**Would you like to see how to correctly implement the mask generation code for a batch of sentences with different lengths?**
