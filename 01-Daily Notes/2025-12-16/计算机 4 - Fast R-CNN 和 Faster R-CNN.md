R-CNN 发布后（2014 年），虽然效果震撼，但它的**速度瓶颈**非常明显。正如我们刚才拆解的，它有三个主要问题：

1. **训练/推理太慢**：2000 个候选框，每个都要单独过一遍 CNN。
2. **训练步骤繁琐**：特征提取、SVM 分类、边框回归是分开训练的，无法联合优化。
3. **存储巨大**：需要把特征向量存到硬盘上供 SVM 训练。

为了解决这些问题，R-CNN 家族经历了两次重大的进化：**Fast R-CNN** 和 **Faster R-CNN**。

这是一条非常清晰的**“端到端”（End-to-End）**进化之路。

---

### 第一阶段进化：Fast R-CNN (2015)

**核心解决的问题：** 也就是那个“2000 次前向传播”的笨办法。

Ross Girshick 借鉴了 SPP-Net（Spatial Pyramid Pooling）的思想，提出了 Fast R-CNN。

#### 1. 核心改进：共享卷积特征 (Shared Convolution)

R-CNN 是“先切图，再卷积”。Fast R-CNN 改为**“先卷积，再切特征”**。

- **流程**：
    
    1. 把**整张图片**输入 CNN，得到一个特征图（Feature Map）。
    2. 把 Selective Search 生成的候选框坐标，直接映射到这个特征图上。
    3. 只计算一次卷积，效率提升了极多。

#### 2. 核心数学工具：RoI Pooling 层

由于 CNN 的全连接层需要固定尺寸的输入（比如 $7 \times 7$），但候选框在特征图上的大小各不相同。R-CNN 用的是暴力缩放（Warping），Fast R-CNN 引入了 **RoI Pooling**。

- 映射公式：
    
    假设 CNN 的降采样率（Stride）为 $S$（例如 VGG16 中 $S=16$）。
    
    原图上的候选框坐标 $(x, y, w, h)$ 映射到特征图上的坐标为：
    
    $$x' = \lfloor x / S \rfloor, \quad y' = \lfloor y / S \rfloor, \quad w' = \lfloor w / S \rfloor, \quad h' = \lfloor h / S \rfloor$$
    
- 网格化 (Gridding)：
    
    将映射后的区域划分为 $H \times W$ 个子网格（例如 $7 \times 7$）。
    
    $$h_{bin} = h' / H, \quad w_{bin} = w' / W$$
    
- 最大池化 (Max Pooling)：
    
    对每个子网格内的数值取最大值。无论原框多大，最后都变成了 $7 \times 7$ 的特征块。

#### 3. 损失函数：多任务损失 (Multi-task Loss)

Fast R-CNN 抛弃了 SVM，直接用 Softmax 进行分类，并且将“分类”和“回归”放在同一个网络里一起训练。

$$L(p, u, t^u, v) = L_{cls}(p, u) + \lambda [u \ge 1] L_{loc}(t^u, v)$$

- $L_{cls}$：分类损失（Log Loss）。$p$ 是预测概率，$u$ 是真实类别标签。
    
    $$L_{cls}(p, u) = -\log p_u$$
    
- $L_{loc}$：回归损失。$t^u$ 是对应类别 $u$ 的预测偏移量，$v$ 是真实偏移量。
    
    Fast R-CNN 使用了 Smooth L1 Loss，比 R-CNN 的 L2 Loss 对异常值（Outliers）更不敏感：
    
    $$L_{loc}(t, v) = \sum_{i \in \{x,y,w,h\}} \text{smooth}_{L1}(t_i - v_i)$$
    
    $$\text{smooth}_{L1}(x) = \begin{cases} 0.5x^2 & \text{if } |x| < 1 \\ |x| - 0.5 & \text{otherwise} \end{cases}$$

结果：Fast R-CNN 训练速度比 R-CNN 快了 9 倍，推理速度快了 213 倍。

遗留问题：候选区域生成（Selective Search）依然跑在 CPU 上，处理一张图要 2 秒，成为了整个系统的最大瓶颈。

---

### 第二阶段进化：Faster R-CNN (2015)

**核心解决的问题：** 干掉 Selective Search，让神经网络自己生成候选框。

何恺明团队提出了 Faster R-CNN，引入了 **RPN (Region Proposal Network)**，实现了真正的**端到端**实时检测。

#### 1. 核心创新：RPN (Region Proposal Network)

RPN 是一个轻量级的全卷积网络，它“寄生”在主干网络的特征图上。它的作用是**在特征图的每一个点上，都猜一猜有没有物体**。

#### 2. 核心数学概念：锚点 (Anchors)

这是 Faster R-CNN 的灵魂。既然不知道物体多大、长宽比如何，我们就设计一组预设的框，叫 **Anchors**。

- **设定**：在特征图的每一个像素点上，生成 $k$ 个锚点框。
    - 3 种尺度 (Scales): $\{128^2, 256^2, 512^2\}$
    - 3 种比例 (Ratios): $\{1:1, 1:2, 2:1\}$
    - 总共 $k=9$ 个锚点。
    - 如果特征图大小是 $W \times H$，那么全图就有 $W \times H \times 9$ 个锚点（约 2 万个）。

#### 3. RPN 的计算过程

RPN 在特征图上滑动一个 $3 \times 3$ 的窗口，然后通过两个 $1 \times 1$ 卷积层输出两个分支：

1. **分类分支 (cls layer)**：
    
    - 输出维度：$2k$ (每个 anchor 是前景还是背景的概率)。
    - 注意：这里只区分“是不是物体”，不区分“是猫还是狗”。
        
2. **回归分支 (reg layer)**：
    
    - 输出维度：$4k$ (每个 anchor 的平移缩放量 $d_x, d_y, d_w, d_h$)。
    - 这里的数学公式与 R-CNN 完全一致，只是对象变成了 Anchor。

#### 4. RPN 的损失函数

RPN 也是通过多任务损失训练的：

$$L(\{p_i\}, \{t_i\}) = \frac{1}{N_{cls}} \sum_i L_{cls}(p_i, p_i^*) + \lambda \frac{1}{N_{reg}} \sum_i p_i^* L_{reg}(t_i, t_i^*)$$

- $p_i^*$：标签。如果 Anchor 与 Ground Truth 的 IoU > 0.7，则为 1（正样本）；IoU < 0.3 为 0（负样本）。
- $p_i^* L_{reg}$：这一项表示**只有正样本才计算回归损失**（背景不需要修框）。

#### 5. 整体流程

1. **主干 CNN**：提取特征图。
2. **RPN**：生成候选框（Proposals），并进行粗略的二分类（前景/背景）和位置修正。
3. **RoI Pooling**：利用 RPN 生成的框，在特征图上扣取特征。
4. **最终分类与回归**：类似 Fast R-CNN，进行具体的类别判定（猫/狗/车）和精细位置修正。

---

### 总结：三代 R-CNN 的数学演进

|**模型**|**候选框生成 (Proposals)**|**特征提取 (Feature)**|**分类器 (Classifier)**|**边框回归 (Regression)**|**速度 (FPS)**|**核心数学差异**|
|---|---|---|---|---|---|---|
|**R-CNN**|Selective Search (CPU)|每个框单独过 CNN (重复计算)|SVM (Hinge Loss)|Log-space regression|0.02 (最慢)|分阶段训练，硬 Warp|
|**Fast R-CNN**|Selective Search (CPU)|**整图过 CNN (共享计算)** + RoI Pooling|Softmax (Cross Entropy)|Smooth L1 Loss|0.5|引入 RoI Pooling，多任务 Loss|
|**Faster R-CNN**|**RPN (GPU, CNN)**|整图过 CNN (共享计算)|Softmax|Smooth L1 Loss|**7~17**|**引入 Anchors**，全网可导|

### 这一脉络的后续...

Faster R-CNN 基本上确立了**双阶段（Two-Stage）**目标检测的霸主地位。

在此之后，发展分为了两个流派：

1. **追求精度**：**Mask R-CNN**（在 Faster R-CNN 基础上加了一个 Mask 分支做实例分割，并将 RoI Pooling 升级为数学上更精确的 **RoI Align**）。
2. **追求速度**：**YOLO / SSD** 系列（单阶段 One-Stage 检测器，去掉了生成候选框的步骤，直接一步到位回归坐标）。

**如果您想了解“RoI Align 具体是怎么通过双线性插值解决精度丢失问题的”（Mask R-CNN 的核心），或者想了解 YOLO 是如何做到更快的，请告诉我。**