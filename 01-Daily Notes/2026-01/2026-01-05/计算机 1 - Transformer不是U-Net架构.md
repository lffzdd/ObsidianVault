整体的结构是 Unet 结构吧？编码器往下，解码器往上？具体是怎么样的？

### Planner Response

## Transformer 不是 U-Net 结构

**U-Net** 是图像分割用的，有 " 下采样→上采样 + 跳跃连接 "。

**Transformer** 是序列处理用的，结构不同。让我解释 Transformer 的真正结构：

---

## Transformer 整体结构

```
输入序列                                    输出序列
   ↓                                           ↑
┌─────────────────┐                  ┌─────────────────┐
│                 │                  │                 │
│    Encoder      │ ───────────────→ │    Decoder      │
│   （编码器）    │   传递 K, V      │   （解码器）    │
│                 │                  │                 │
└─────────────────┘                  └─────────────────┘
   ↑                                           ↑
 输入嵌入                                   输出嵌入
+位置编码                                  +位置编码
```

---

## 详细结构

### Encoder（编码器）- 左边

```
输入: "I love you"
        ↓
   Embedding + 位置编码
        ↓
┌─────────────────────────┐
│   Encoder Layer × N     │  ← 堆叠 N 层（通常 6 层）
│  ┌───────────────────┐  │
│  │  Self-Attention   │  │  ← 每个词看所有词
│  │       ↓           │  │
│  │  Add & Norm       │  │  ← 残差连接 + 归一化
│  │       ↓           │  │
│  │  Feed Forward     │  │  ← 前馈网络
│  │       ↓           │  │
│  │  Add & Norm       │  │
│  └───────────────────┘  │
└─────────────────────────┘
        ↓
   编码器输出 (传给解码器)
```

### Decoder（解码器）- 右边

```
输出: "<BOS> 我 爱 你"（已生成的部分）
        ↓
   Embedding + 位置编码
        ↓
┌─────────────────────────────────┐
│   Decoder Layer × N             │  ← 堆叠 N 层
│  ┌───────────────────────────┐  │
│  │  Masked Self-Attention    │  │  ← 只能看前面的词！
│  │           ↓               │  │
│  │  Add & Norm               │  │
│  │           ↓               │  │
│  │  Cross-Attention          │  │  ← 看编码器的输出
│  │  (Q来自解码器，K/V来自编码器) │  │
│  │           ↓               │  │
│  │  Add & Norm               │  │
│  │           ↓               │  │
│  │  Feed Forward             │  │
│  │           ↓               │  │
│  │  Add & Norm               │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
        ↓
   Linear + Softmax
        ↓
   预测下一个词
```

---

## 对比 U-Net vs Transformer

| 特点 | U-Net | Transformer |
|------|-------|-------------|
| **用途** | 图像分割 | 序列处理（NLP、翻译等） |
| **结构** | 下采样 → 上采样 | 编码器 → 解码器 |
| **维度变化** | 空间缩小再放大 | 维度不变 |
| **跳跃连接** | ✅ 有（连接对应层） | ❌ 无（只有残差连接） |
| **核心操作** | 卷积 | Attention |

---

## Transformer 中的三种 Attention

| 类型 | 位置 | Q, K, V 来源 | 作用 |
|------|------|--------------|------|
| **Self-Attention** | 编码器 | 全来自输入 | 输入词之间互相看 |
| **Masked Self-Attention** | 解码器 | 全来自输出 | 只能看前面的词 |
| **Cross-Attention** | 解码器 | Q 来自解码器，K/V 来自编码器 | 输出词看输入词 |

---

## 模块关系图

```
Transformer
├── Encoder (编码器)
│   ├── Embedding
│   ├── Positional Encoding
│   └── Encoder Layer × N
│       ├── Multi-Head Self-Attention  ← 你正在实现的
│       ├── Add & Norm
│       ├── Feed Forward (MLP)
│       └── Add & Norm
│
└── Decoder (解码器)
    ├── Embedding
    ├── Positional Encoding
    └── Decoder Layer × N
        ├── Masked Multi-Head Self-Attention
        ├── Add & Norm
        ├── Multi-Head Cross-Attention
        ├── Add & Norm
        ├── Feed Forward (MLP)
        └── Add & Norm
```

---

## 你目前实现的模块

| 模块 | 状态 |
|------|------|
| ✅ Tokenizer | 已完成 |
| ✅ Embedding | 已完成 |
| ✅ Positional Encoding | 已完成 |
| 🔄 Attention | 正在实现 |
| ⏳ Feed Forward | 待实现 |
| ⏳ Encoder Layer | 待实现 |
| ⏳ Decoder Layer | 待实现 |
