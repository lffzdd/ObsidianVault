这个问题问得**非常关键** 👍。  
模型训练完如果不会正确保存 / 加载，前面全白干。

我先给你一句结论，然后系统讲清楚 **3 种标准方式 + 最佳实践 + 常见坑**。

---

## 一、最推荐、最标准的方法（99% 项目用这个）

### ✅ 只保存参数：`state_dict`（强烈推荐）

### 🔹 保存模型

```python
torch.save(model.state_dict(), "model.pth")
```

---

### 🔹 加载模型

```python
model = MyModel()                      # 先建模型结构
model.load_state_dict(torch.load("model.pth"))
model.eval()                           # 切换到评估模式（重要）
```

这就是 PyTorch **官方推荐方式**。

---

## 二、为什么要用 `state_dict` 而不是直接存模型？

你也可以这样存：

```python
torch.save(model, "model.pth")
model = torch.load("model.pth")
```

但 ❌ **强烈不推荐**，原因：

- 依赖当时的类定义路径
- 换文件名 / 换环境 / 换版本容易直接加载失败
- 不利于部署和长期保存

👉 工业界 & 论文几乎全部用：

> ✅ `model.state_dict()`

---

## 三、训练中断恢复（进阶必会：checkpoint 保存法）

真实训练经常要：

- 训练几天
- 中途断电 / 关机
- 继续训练

这时要保存的不只是模型，还有：

- optimizer 状态
- 当前 epoch
- 学习率调度器

### 🔹 保存 checkpoint（标准工业模板）

```python
torch.save({
    "epoch": epoch,
    "model_state_dict": model.state_dict(),
    "optimizer_state_dict": optimizer.state_dict(),
    "loss": loss,
}, "checkpoint.pth")
```

---

### 🔹 加载 checkpoint 继续训练

```python
checkpoint = torch.load("checkpoint.pth")

model.load_state_dict(checkpoint["model_state_dict"])
optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
start_epoch = checkpoint["epoch"] + 1
loss = checkpoint["loss"]

model.train()   # 继续训练
```

这套你以后做**长训练 / 实验管理必用**。

---

## 四、GPU / CPU 加载问题（新手最常踩的坑）

### 🔥 情况 1：GPU 训练 → CPU 加载

如果你在 GPU 上存的模型，现在在 CPU 上加载：

```python
model.load_state_dict(
    torch.load("model.pth", map_location="cpu")
)
```

---

### 🔥 情况 2：CPU 训练 → GPU 加载

```python
device = torch.device("cuda")

model.load_state_dict(torch.load("model.pth"))
model.to(device)
```

---

## 五、推理 / 测试时的标准流程（非常规范）

当你加载好模型准备用来预测时，**标准三步**：

```python
model.load_state_dict(torch.load("model.pth"))
model.eval()                 # 关闭 dropout / 固定 BN
model.to(device)

with torch.no_grad():        # 关闭梯度（重要）
    output = model(x)
```

---

## 六、保存“最优模型”（训练中最常见用法）

通常我们不是每个 epoch 都存，而是：

👉 **验证集效果最好的时候才存**

```python
best_acc = 0.0

for epoch in range(num_epochs):
    ...
    if val_acc > best_acc:
        best_acc = val_acc
        torch.save(model.state_dict(), "best_model.pth")
```

这是比赛 / 论文 / 工程的标准套路。

---

## 七、完整最小示例（强烈建议你收藏）

### 🔹 训练 + 保存

```python
model = MyModel()
optimizer = torch.optim.Adam(model.parameters())

# 训练完
torch.save(model.state_dict(), "model.pth")
```

---

### 🔹 加载 + 测试

```python
model = MyModel()
model.load_state_dict(torch.load("model.pth"))
model.eval()

with torch.no_grad():
    y = model(x)
```

---

## 八、面试 / 考试高频问法（给你准备标准答案）

如果别人问你：

> PyTorch 中如何保存和加载模型？

你可以答（专业版）：

> 通常使用 `state_dict` 方式保存模型参数：  
> `torch.save(model.state_dict(), path)`，  
> 加载时先构建模型结构，再调用 `load_state_dict`。  
> 这种方式更稳定，不依赖类路径，适合长期保存和部署。  
> 若需要恢复训练，还应同时保存 optimizer 和 epoch 组成 checkpoint。

这是**非常标准的满分答案**。

---

## 九、结合你现在阶段，说一句非常重要的建议

你现在正处在：

- 学 PyTorch 基础
- 写训练循环
- 理解 forward / autograd

👉 现在就把下面这三件事当“基本功”练熟：

1. `state_dict` 保存 / 加载
2. checkpoint 恢复训练
3. `model.eval()` + `torch.no_grad()` 推理规范

这三点会直接决定你以后：

- 能不能跑长实验
- 能不能复现实验
- 能不能部署模型

---

## 十、我可以继续帮你进阶一层（很有价值）

如果你愿意，我可以下一步给你讲：

- 🔹 `state_dict` 里面到底存了什么（参数名结构）
- 🔹 不同模型结构加载失败怎么处理（`strict=False`）
- 🔹 多卡 / DDP 保存模型的正确方式
- 🔹 如何保存 TorchScript / 用于部署

或者你可以告诉我：  
👉 你现在是**分类 / 回归 / CNN / NLP** 哪种任务？  
我可以按你当前项目，给你一份**完全实战级的保存 + 恢复模板**。