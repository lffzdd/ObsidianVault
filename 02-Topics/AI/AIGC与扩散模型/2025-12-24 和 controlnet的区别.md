你这段总结**方向对了，但在两个关键点上还需要“掰正一下物理直觉”**。  
如果按你现在的说法继续用下去，后面一遇到 SDXL / 多 ControlNet / CFG，就会开始自相矛盾。

我会做三件事：

1️⃣ **逐句判定你说的哪部分是对的，哪部分需要修正**  
2️⃣ **用“去噪动力系统”的视角，把三者放在同一数学框架下**  
3️⃣ **给你一个“不会再混淆”的最终心智模型**

---

## 一、逐句校正你的理解 $非常重要$

### ①「Reference 是被直接相加在去噪前和去噪过程中的 latent 上」

❌ **这句话不准确，但很容易让人这么以为**

**修正版本是：**

> **Reference 不是直接加在 $x_t$ 扩散 latent$上**  
> **而是加在 UNet 内部的中间特征 $h_l$ 上**

数学上不是：

$$  
x_t' = x_t + r  
$$

而是：

$$  
h_l' = h_l + \alpha_l \cdot r_l  
$$

这两者差别极大：

- 改 $x_t$：你在改 SDE 的初值 / 状态
- 改 $h_l$：你在改**去噪函数本身**

👉 Reference 属于**后者**

---

### ②「ControlNet 也是相加，不过是有权重的相加」

✔ **方向对，但表达还不够精确**

更准确地说：

$$  
h_l' = h_l + \lambda_l \cdot c_l  
$$

其中：

- $c_l$：ControlNet 分支算出来的 feature
- $\lambda_l$：可调强度 $weight$

**但关键区别是**：

- ControlNet 的 $c_l$ 是：
    - **由一个完整的、结构对齐的 UNet 分支算出来的**
- Reference 的 $r_l$ 是：
    - **直接由 latent 轻量投影而来**

所以：

> 两者都是“加”，  
> 但**算 $c_l$ 的那条路复杂度天差地别**

---

### ③「IP-Adapter 是把图像转成提示词一样的 condition，然后和文本提示词相加再 attention」

✔ **这是目前为止你说得最准确的一句**

但我还是给你一个更“工程正确”的版本：

> **IP-Adapter 不是“相加文本提示词”，而是：  
> 把 image embedding 当作额外 token，  
> 在 cross-attention 中与文本 token 共同参与 softmax 权重竞争**

也就是说：

$$  
c = $$c_{\text{text}}; c_{\text{img}}$$  
$$

而不是：

$$  
c = c_{\text{text}} + c_{\text{img}}  
$$

这个细节决定了它的行为特性。

---

## 二、把三者放进同一个数学框架 $这是你真正要的$

我们统一用这个去噪形式：

$$  
\epsilon_\theta$x_t, t, \mathcal{C}$  
$$

区别只在 **$\mathcal{C}$ 如何进入 $\theta$**。

---

## 1️⃣ Reference / ControlNet：改的是 **特征流**

统一形式可以写成：

$$  
h_l = f_l(h_{l-1}) + \Phi_l(\mathcal{C})  
$$

- $\Phi_l$：条件投影函数
- $\mathcal{C}$：
    - Reference：latent
    - ControlNet：edge / depth / pose

📌 **本质**：

> 改 UNet 的中间状态 → 改去噪向量场

---

## 2️⃣ IP-Adapter / prompt：改的是 **注意力分布**

统一形式：

$$  
\text{Attn}(Q, K, V), \quad K,V \leftarrow \mathcal{C}  
$$

📌 **本质**：

> 不改状态本身，只改“关注哪里”

---

## 三、你现在可以用一句话区分它们 $非常重要$

你可以这样记：

> **Reference / ControlNet 是在“推 UNet 的手”**  
> **IP-Adapter / prompt 是在“指 UNet 的眼睛”**

- 推手：改变生成轨迹
- 指眼：改变决策偏好

---

## 四、为什么 Reference 看起来“介于 img2img 和条件之间”

现在我们可以彻底回答你最开始那个“不应该啊”的直觉：

- 它确实用了 latent
- 但：
    - 没有重置 $x_t$
    - 没有改噪声 schedule
- 只是：
    - 在每一步去噪时**持续施加偏置**

所以你可以非常准确地说：

> **ReferenceLatent 是一种“持续条件场”，而不是初始化**

---

## 五、最终心智模型 $你以后不会再乱$

你以后看到任何新插件，只需要问三个问题：

1. **它有没有改 $x_t$？**  
    → 有：img2img / SDEdit
    
2. **它有没有改中间 feature $h_l$？**  
    → 有：ControlNet / Reference / Adapter
    
3. **它只改 attention 权重吗？**  
    → 是：prompt / IP-Adapter / TI

所有扩散条件方法，都跑不出这三类。

---

如果你愿意，下一步我可以继续带你把这套模型**推进到连续时间 SDE / ODE 的视角**，那时你会发现：

- CFG
- ControlNet
- Reference
- IP-Adapter

其实全是在**同一个向量场上加不同形态的“力”**。

那一层理解，会让你彻底脱离“节点视角”。