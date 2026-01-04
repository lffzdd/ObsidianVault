## 为什么要指定 device

### 问题：CPU 和 GPU 上的张量不能直接运算

```python
# 假设 x 在 GPU 上
x = torch.randn(32, 128, 512).cuda()  # GPU
print(x.device)  # cuda:0

# 如果不指定 device，默认在 CPU 上创建
pos_vec = torch.zeros(128, 512)  # CPU
print(pos_vec.device)  # cpu

# ❌ 报错！
x + pos_vec
# RuntimeError: Expected all tensors to be on the same device, 
# but found at least two devices, cuda:0 and cpu!
```

### 解决：确保在同一设备

```python
# ✅ 指定 device，和 x 在同一设备
pos_vec = torch.zeros(128, 512, device=x.device)
x + pos_vec  # ✅ 正常运行
```
---

## 什么时候需要？

|场景|需要 device 吗？|
|---|---|
|只用 CPU|不需要（但加上更好）|
|用 GPU 训练|**必须**|
|写通用代码|**必须**（推荐）|

---

## 你的代码

```python
def forward(self, x: Tensor) -> Tensor:

    embed_dim = x.shape[2]
    seq_len = x.shape[1]

    # ❌ 当前写法：默认在 CPU
    pos_vec = torch.zeros(seq_len, embed_dim)

    # ✅ 建议写法：和 x 在同一设备
    pos_vec = torch.zeros(seq_len, embed_dim, device=x.device)
```

---

## 总结

| 不加 `device`    | 加 `device=x.device` |
| -------------- | ------------------- |
| 默认在 CPU 创建     | 和输入 x 在同一设备         |
| 如果 x 在 GPU 会报错 | ✅ 永远不会报错            |
| 只能在 CPU 上用     | CPU/GPU 都能用         |

**建议：养成习惯，新建张量时总是加上** **`device=x.device`**🎯