---
type: 入口笔记
---
# PyTorch

## 当前理解

PyTorch 的核心是张量（存储 + 形状/步长视图）和自动求导：view/transpose 只改视图，广播和向量化替代循环，device 管位置；训练流程固定为“数据加载 → 前向 → loss → backward → 优化器更新”，再加保存/加载与 DataLoader 调优。

## 笔记（按时间）

- [[02-Topics/AI/PyTorch/2026-01-04 Tensor 存储方式|2026-01-04 Tensor 存储方式]] — 讲了：为什么需要  .T （转置）？ / 维度分析 / 矩阵乘法的规则
- [[02-Topics/AI/PyTorch/2026-01-04 nn.Parameter和zeros的区别|2026-01-04 nn.Parameter和zeros的区别]] — 讲了：核心是是否被训练 / 好问题！让我解释  nn.Linear  和普通 / 核心区别：是否需要被训练学习
- [[02-Topics/AI/PyTorch/2026-01-04 pytorch中的张量创建和device|2026-01-04 pytorch中的张量创建和device]] — 讲了：常用的张量创建方法
- [[02-Topics/AI/PyTorch/2026-01-04 pytorch关于掩码的函数|2026-01-04 pytorch关于掩码的函数]] — 讲了：User Input / Planner Response / 使用  torch.triu  创建上三角 ma
- [[02-Topics/AI/PyTorch/2026-01-04 transpose方法|2026-01-04 transpose方法]] — 讲了：目标：Q·K^T 点积 / 当前形状 / 想要得到
- [[02-Topics/AI/PyTorch/2026-01-04 关于device-三种状态|2026-01-04 关于device-三种状态]] — 讲了：三种状态 / User Input / Planner Response
- [[02-Topics/AI/PyTorch/2026-01-04 向量化运算|2026-01-04 向量化运算]] — 讲了：逐行解释向量化位置编码 / 第 1 步：获取形状 / 第 2 步：创建位置索引
- [[02-Topics/AI/PyTorch/2026-01-04 广播机制|2026-01-04 广播机制]] — 讲了：PyTorch 广播机制（Broadcastin / 什么是广播？ / 最简单的例子
- [[02-Topics/AI/PyTorch/2026-01-04 指定device|2026-01-04 指定device]] — 讲了：为什么要指定 device / 问题：CPU 和 GPU 上的张量不能直接运算 / 解决：确保在同一设备
- [[02-Topics/AI/PyTorch/2026-01-05 PyTorch 张量形状操作速查表|2026-01-05 PyTorch 张量形状操作速查表]] — 讲了：PyTorch 张量形状操作速查表
- [[02-Topics/AI/PyTorch/2026-01-05 view的原理|2026-01-05 view的原理]] — 讲了：view  的原理：按内存顺序重新解释 / 核心概念：张量在内存中是一维的！ / 例子： view  怎么拆分的
- [[02-Topics/AI/PyTorch/2026-01-07 pytorch 的张量方法|2026-01-07 pytorch 的张量方法]] — 讲了：User Input / Planner Response / 1.  .all()  方法
- [[02-Topics/AI/PyTorch/2026-01-08 PyTorch更新流程|2026-01-08 PyTorch更新流程]] — 讲了：一、先给一张“对象关系脑图”（文字版） / 二、 nn.Module ：它只关心一件事 —— / 1️⃣  nn.Module  本质上是什么？
- [[02-Topics/AI/PyTorch/2026-01-08 pytorch数据处理|2026-01-08 pytorch数据处理]] — 讲了：1. 历史背景与设计哲学 / 2. Dataset：数据的仓库 / 代码示例：自定义一个简单的数字数据集
- [[02-Topics/AI/PyTorch/2026-01-08 zip(batch)|2026-01-08 zip(batch)]] — 讲了：一、问题背景：这行代码想解决什么？ / 二、核心思想：把“横着放的东西”竖过来 / 三、 zip( batch)  在“干什么事”（
- [[02-Topics/AI/PyTorch/2026-01-22 CrossEntryLoss的计算|2026-01-22 CrossEntryLoss的计算]] — 讲了：CrossEntropyLoss 期望的输入形状 / 你的情况 / ⚠️ 你的代码有个 Bug
- [[02-Topics/AI/PyTorch/2026-01-22 pytorch 训练流程|2026-01-22 pytorch 训练流程]] — 讲了：一、整体流程概览（标准 Pipeline） / 二、最标准的训练代码结构（通用模板） / 三、每一步的“关键点解释”（面试 / 实战常考）
- [[02-Topics/AI/PyTorch/2026-01-22 pytorch中的底层张量布局和读取|2026-01-22 pytorch中的底层张量布局和读取]] — 讲了：0. 先约定：PyTorch 张量   (sto / 1) 从一个“标准连续”的 x 开始 / 1.1 storage 长什么样？
- [[02-Topics/AI/PyTorch/2026-01-22 tqdm|2026-01-22 tqdm]] — 讲了：一、安装 / 二、最基础用法（包一层循环就行） / 1️⃣ 普通 for 循环
- [[02-Topics/AI/PyTorch/2026-01-23 num_workers和pin_memory|2026-01-23 num_workers和pin_memory]] — 讲了：num_workers：多进程数据加载 / 工作原理 / 关键点
- [[02-Topics/AI/PyTorch/2026-01-23 pytorch保存和加载模型|2026-01-23 pytorch保存和加载模型]] — 讲了：一、最推荐、最标准的方法（99% 项目用这个） / ✅ 只保存参数： state_dict （强烈推 / 🔹 保存模型
- [[02-Topics/AI/PyTorch/2026-02-01 pytorch的配对索引|2026-02-01 pytorch的配对索引]] — 讲了：Chat Conversation / User Input / Planner Response
- [[02-Topics/AI/PyTorch/2026-02-01 简易pytorch实现|2026-02-01 简易pytorch实现]] — 讲了：第一阶段：构建核心单元  Tensor / 1.1 基础结构：不仅存数值，还要存关系 / 第二阶段：实现运算与动态图构建
- [[02-Topics/AI/PyTorch/2026-02-01 重新梳理pytorch计算流程|2026-02-01 重新梳理pytorch计算流程]] — 讲了：第一部分：底层原理与角色分工 / 1. Loss（损失函数）：裁判员 / 2. Autograd（自动微分）：探测器
- [[02-Topics/AI/PyTorch/2026-02-07 torch linspace|2026-02-07 torch linspace]] — 讲了：函数签名 / 参数说明 / 基本用法
- [[02-Topics/AI/PyTorch/2026-02-07 torch 累积类函数|2026-02-07 torch 累积类函数]] — 讲了：函数签名 / 参数说明 / 基本用法

### 早期专题笔记

- [[02-Topics/AI/PyTorch/1-matplotlib|1-matplotlib]] — 讲了：结构 / 用法 / 子图
- [[02-Topics/AI/PyTorch/2-加载和处理输入图像|2-加载和处理输入图像]] — 讲了：1.   TensorFlow和Keras / 2.   OpenCV / 3.   PIL（Python Imaging 
- [[02-Topics/AI/PyTorch/3-keras|3-keras]] — 讲了：1. 卷积步长 / 详细解释 / 步幅（Strides）
- [[02-Topics/AI/PyTorch/4-add和concatenate的区别|4-add和concatenate的区别]] — 讲了：原理 / keras.layers.add / 示例
- [[02-Topics/AI/PyTorch/5-model 流程|5-model 流程]] — 讲了：1. 导入必要的库 / 2. 准备数据 / 3. 创建模型
- [[02-Topics/AI/PyTorch/6-图像数据增强|6-图像数据增强]] — 讲了：6-图像数据增强
- [[02-Topics/AI/PyTorch/7-图模式和eager模式|7-图模式和eager模式]] — 讲了：1. 图模式（Graph Mode） / 特点： / 例子：
- [[02-Topics/AI/PyTorch/8-pytorch|8-pytorch]] — 讲了：8-pytorch
- [[02-Topics/AI/PyTorch/9-einsum|9-einsum]] — 讲了：爱因斯坦求和约定 / 基础概念 / 具体例子
