好，这一步我们就**真的下到“数学与结构层”**，把你提的两点**硬拆开**：

1️⃣ **image → down block 注入 vs image → cross-attention 注入，在数学上到底差在哪**  
2️⃣ **Reference 这一套为什么“没有论文却能成立”，它在扩散发展史里的真实位置**

我会尽量用**最少但必要的公式**，重点放在：

> **这些公式“为什么长这样”**

---

# 一、先把 UNet 的数学骨架立起来（否则一切都没法精确）

扩散模型在第 (t) 步的核心计算是：

[  
\epsilon_\theta(x_t, t, c)  
]

其中：

- (x_t \in \mathbb{R}^{H \times W \times C})：当前 noisy latent
- (t)：时间步
- (c)：conditioning（文本、图像、参考等）

UNet 本质是一个**带条件的函数族**，而关键不在于有没有条件，而在于：

> **(c) 以什么数学形式进入这个函数**

---

# 二、两种“image conditioning”的数学分叉点

你说的这两种，**不是工程差异，而是函数形态差异**。

我们分别看。

---

## ① image → encoder → 注入 down block（Reference / ControlNet 系）

### 1️⃣ 数学上发生了什么？

在 UNet 的第 (l) 个 down block，我们原本有：

[  
h_l = f_l(h_{l-1})  
]

现在加一个 image-derived 条件 (r_l)：

[  
h_l = f_l(h_{l-1}) + g_l(r)  
]

或者更一般：

[  
h_l = f_l(h_{l-1}, r)  
]

其中：

- (r = E_{\text{img}}(\text{image or latent}))
- (g_l) 通常是：
    - conv
    - 或线性投影 + broadcast

👉 **关键点**：  
(r) 是一个 **空间张量（spatial feature map）**

[  
r \in \mathbb{R}^{H_l \times W_l \times C_l}  
]

---

### 2️⃣ 这意味着什么（非常重要）

这在数学上等价于：

> **直接改变 UNet 的中间表示流形**

也就是说：

- 不是“提醒模型”
- 而是：
    - **把解空间的局部坐标系改了**

从优化角度看：

[  
\epsilon_\theta(\cdot) \quad \text{的等高线被扭曲了}  
]

所以这种方法：

- 影响早
- 影响强
- 不可逆
- 非常适合 **结构 / 构图 / 姿态**

这就是为什么：

- ControlNet
- ReferenceLatent

都偏向 down / middle block。

---

### 3️⃣ ReferenceLatent 的特殊点

ReferenceLatent 的 (r) 不是来自 RGB，而是：

[  
r = E_{\text{latent}}(z_{\text{ref}})  
]

而 latent 本身满足：

[  
z \approx \text{VAE.encode}(x)  
]

所以它已经在 **扩散工作空间附近**，这会导致：

- 更强的结构耦合
- 更少的像素噪声干扰

---

## ② image → encoder → cross-attention（IP-Adapter 系）

现在看另一条路线。

---

### 1️⃣ 先回忆 cross-attention 的标准公式

在 UNet 某一层，我们有：

[  
\text{Attn}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d}}\right)V  
]

其中：

- (Q = W_Q h)（来自 UNet feature）
- (K, V = W_K c, W_V c)（来自 conditioning）

**文本 prompt 就是 (c)**。

---

### 2️⃣ IP-Adapter 做了什么？

它把 image encoder 得到的 embedding：

[  
e_{\text{img}} = E_{\text{img}}(x)  
]

当作**额外 token**，拼进 conditioning：

[  
c = [c_{\text{text}} ; e_{\text{img}}]  
]

于是 cross-attention 变成：

[  
\text{softmax}\left(  
\frac{Q [K_{\text{text}};K_{\text{img}}]^\top}{\sqrt{d}}  
\right)  
[V_{\text{text}};V_{\text{img}}]  
]

---

### 3️⃣ 这在数学上意味着什么？

👉 image 条件 **只参与“注意力加权”**  
👉 它**不能直接改变 feature map 本身**

它的影响是：

- 改变 **UNet 在“看什么”**
- 而不是 **UNet 的内部状态是什么**

这就是为什么：

- 它很适合 identity
- 很适合风格
- 却不擅长强制构图

---

## ③ 一个一句话的数学总结

你可以用这句话判断任何方法：

> **它是在改 (h) 本身，还是在改 (QK^\top) 的权重？**

- 改 (h) → down block / ControlNet / Reference
- 改 attention → IP-Adapter / prompt / textual inversion

---

# 三、为什么 Reference “没有论文”？——这是一个历史问题，不是技术问题

你这个观察非常敏锐，而且答案**有点反直觉**。

---

## 1️⃣ Reference 的思想并不新

ReferenceLatent 并不是一个“新理论”，它本质上是：

> **ControlNet 思想的弱化 + 泛化版本**

把 ControlNet 抽象一下：

[  
h_l = f_l(h_{l-1}) + \lambda g_l(E_{\text{cond}}(x))  
]

Reference 只是把：

- 显式结构（edge / depth）
- 换成：
- 隐式结构（latent / image feature）

👉 **这是工程组合，而不是理论突破**

---

## 2️⃣ 学术界为什么不写？

学术论文需要至少一个：

- 新目标函数
- 新架构
- 新理论解释
- 新 benchmark

而 Reference：

- 不改 loss
- 不改 UNet 主体
- 不引入新任务

它更像是：

> “我们发现 latent 也可以当 condition，用起来挺好”

这在学术界很难单独成文。

---

## 3️⃣ 它真正的“祖先”是谁？

如果你要追溯血统，它更接近这些论文：

- **Classifier-Free Guidance**（条件注入哲学）
- **ControlNet**（条件作用在 feature map）
- **T2I-Adapter**（轻量条件分支）
- **SDEdit**（从 latent 空间操控生成轨迹）

但它本身是 **ComfyUI / 社区工程的产物**，不是 paper-driven。

---

## 四、你现在应该形成的“底层认知模型”

如果你真的吃透了，现在你脑子里应该是这样的：

- 扩散不是“模型 + prompt”
- 而是：  
    [  
    \text{生成轨迹} = \text{噪声动力系统} + \text{条件约束}  
    ]

不同方法的差别只是：

- **约束加在动力系统的哪里**

ReferenceLatent 的位置是：

> **直接在动力系统状态空间里施加偏置**

这就是为什么它强、野、但不优雅。

---

如果你愿意，下一步我们可以继续往下：

- 为什么 **CFG 本质上是一个向量场差分**
- 为什么 **Reference + CFG 容易“打架”**
- 为什么 **ControlNet 一定要复制 UNet 权重**

这些其实都是同一条“扩散动力系统”逻辑的自然结果。