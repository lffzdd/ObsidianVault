`torch.cumprod` 是 PyTorch 中的**累积乘积**函数，属于"累积类"函数家族。

## 函数签名

```python
torch.cumprod(input, dim, *, dtype=None, out=None)
```

## 参数说明

| 参数 | 说明 |
|------|------|
| `input` | 输入张量 |
| `dim` | 沿哪个维度进行累积乘积 |
| `dtype` | 输出的数据类型（可选） |

## 基本用法

```python
import torch

# 一维张量
x = torch.tensor([1, 2, 3, 4])
print(torch.cumprod(x, dim=0))
# tensor([ 1,  2,  6, 24])
# 解释: [1, 1*2, 1*2*3, 1*2*3*4]

# 二维张量
y = torch.tensor([[1, 2, 3],
                  [4, 5, 6]])

# 沿 dim=0（按列累积）
print(torch.cumprod(y, dim=0))
# tensor([[ 1,  2,  3],
#         [ 4, 10, 18]])
# 第一行不变，第二行 = 第一行 * 第二行

# 沿 dim=1（按行累积）
print(torch.cumprod(y, dim=1))
# tensor([[  1,   2,   6],
#         [  4,  20, 120]])
# 每行内部累积: [1, 1*2, 1*2*3] 和 [4, 4*5, 4*5*6]
```

## 累积类函数家族（cum* 系列）

| 函数 | 作用 | 示例 |
|------|------|------|
| `torch.cumsum` | 累积**求和** | `[1,2,3]` → `[1, 3, 6]` |
| `torch.cumprod` | 累积**乘积** | `[1,2,3]` → `[1, 2, 6]` |
| `torch.cummax` | 累积**最大值** | `[3,1,4]` → `[3, 3, 4]` |
| `torch.cummin` | 累积**最小值** | `[3,1,4]` → `[3, 1, 1]` |

### 对比示例

```python
x = torch.tensor([2, 3, 4])

torch.cumsum(x, dim=0)   # tensor([2, 5, 9])    累加: 2, 2+3, 2+3+4
torch.cumprod(x, dim=0)  # tensor([ 2,  6, 24]) 累乘: 2, 2*3, 2*3*4
```

## 与"归约类"函数的区别

这是另一个容易���淆的点：

| 类型 | 特点 | 函数示例 |
|------|------|----------|
| **累积类 (cum*)** | 保留中间结果，输出形状**不变** | `cumsum`, `cumprod` |
| **归约类 (reduce)** | 只返回最终结果，输出形状**缩小** | `sum`, `prod`, `mean`, `max` |

```python
x = torch.tensor([1, 2, 3, 4])

# 归约：返回单个值
torch.prod(x)      # tensor(24)  → 直接得到最终乘积

# 累积：返回每一步的结果
torch.cumprod(x, dim=0)  # tensor([1, 2, 6, 24]) → 保留所有中间值
```

## 常见应用场景

1. **计算阶乘序列**
   ```python
   n = torch.arange(1, 6)  # tensor([1, 2, 3, 4, 5])
   factorials = torch.cumprod(n, dim=0)
   # tensor([1, 2, 6, 24, 120])  → 1!, 2!, 3!, 4!, 5!
   ```

2. **概率累积（如存活概率）**
   ```python
   # 每一步的存活概率
   survival_probs = torch.tensor([0.9, 0.8, 0.95, 0.85])
   # 累积存活概率
   cumulative_survival = torch.cumprod(survival_probs, dim=0)
   # tensor([0.9000, 0.7200, 0.6840, 0.5814])
   ```

3. **强化学习中的折扣因子**
   ```python
   gamma = 0.99
   steps = 5
   discounts = torch.cumprod(torch.full((steps,), gamma), dim=0)
   # tensor([0.99, 0.9801, 0.9703, 0.9606, 0.9510])
   ```

## PyTorch 函数分类速记

帮你整理一下常见函数的分类：

| 分类 | 特点 | 常见函数 |
|------|------|----------|
| **生成类** | 创建新张量 | `linspace`, `arange`, `zeros`, `ones`, `rand` |
| **累积类** | 保留中间结果 | `cumsum`, `cumprod`, `cummax`, `cummin` |
| **归约类** | 汇总为标量/降维 | `sum`, `prod`, `mean`, `max`, `min`, `std` |
| **变形类** | 改变形状 | `reshape`, `view`, `squeeze`, `unsqueeze` |
| **索引类** | 选取元素 | `index_select`, `gather`, `where`, `masked_select` |

希望这样能帮你把 PyTorch 的函数分门别类记清楚！还有什么想了解的吗？