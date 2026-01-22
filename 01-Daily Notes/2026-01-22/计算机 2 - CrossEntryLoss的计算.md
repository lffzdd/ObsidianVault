好问题！让我解释一下 `CrossEntropyLoss` 的工作原理：

## CrossEntropyLoss 期望的输入形状

| 参数 | 期望形状 | 含义 |
|------|----------|------|
| **input** (预测) | `(N, C)` | N 个样本，每个样本对 C 个类别的概率分布 |
| **target** (标签) | `(N,)` | N 个样本的真实类别索引 |

## 你的情况

```
output:    [batch_size, seq_len, vocab_size]  → 比如 [32, 100, 8000]
tgt_label: [batch_size, seq_len]              → 比如 [32, 100]
```

本质上，**每个位置的预测都是一个独立的分类任务**（预测下一个词是词表中的哪个词）。

所以你有 `32 × 100 = 3200` 个独立的 " 样本 "，每个样本要从 8000 个词中预测正确的那个。

展平后：
```
output:    [3200, 8000]  → 符合 (N, C) 格式 ✓
tgt_label: [3200]        → 符合 (N,) 格式 ✓
```

## ⚠️ 你的代码有个 Bug

你只展平了 `output`，但 **没有展平 `tgt_label`**！

```python
output = output.reshape(-1, output.size(-1))  # [3200, 8000] ✓
loss = criterion(output, tgt_label)           # tgt_label 还是 [32, 100] ✗
```

应该改成：

```python
output = output.reshape(-1, output.size(-1))      # [batch*seq, vocab]
tgt_label = tgt_label.reshape(-1)                 # [batch*seq]
loss = criterion(output, tgt_label)
```

或者更简洁的写法：
```python
loss = criterion(output.view(-1, output.size(-1)), tgt_label.view(-1))
```