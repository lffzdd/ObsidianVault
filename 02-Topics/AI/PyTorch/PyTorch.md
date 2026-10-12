---
type: 入口笔记
---
# PyTorch

## 当前理解

PyTorch 的核心是张量（存储 + 形状/步长视图）和自动求导：view/transpose 只改视图，广播和向量化替代循环，device 管位置；训练流程固定为“数据加载 → 前向 → loss → backward → 优化器更新”，再加保存/加载与 DataLoader 调优。

## 笔记（按时间）

- [[02-Topics/AI/PyTorch/2026-02-07 torch linspace|2026-02-07 torch linspace]] — 讲了：函数签名 / 参数说明 / 基本用法
- [[02-Topics/AI/PyTorch/2026-02-07 torch 累积类函数|2026-02-07 torch 累积类函数]] — 讲了：函数签名 / 参数说明 / 基本用法
- [[02-Topics/AI/PyTorch/1-matplotlib|1-matplotlib]] — 讲了：结构 / 用法 / 子图
- [[02-Topics/AI/PyTorch/8-pytorch|8-pytorch]] — 讲了：8-pytorch

## 子主题

- 入口：[[02-Topics/AI/PyTorch/Keras与TensorFlow/Keras与TensorFlow|Keras与TensorFlow]]（6 篇）

- 入口：[[02-Topics/AI/PyTorch/张量操作/张量操作|张量操作]]（14 篇）

- 入口：[[02-Topics/AI/PyTorch/训练流程与数据/训练流程与数据|训练流程与数据]]（11 篇）
