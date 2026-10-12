---
type: 入口笔记
---
# AIGC与扩散模型

## 当前理解

扩散模型的核心是“前向加噪、逆向去噪”，在 VAE 的潜空间里一步步把噪声还原成图像。实际工作流（ComfyUI）里，ControlNet 管结构约束，IP-Adapter / Flux-Redux / Style Model 管风格和参考图，inpainting / 区域重绘只改 mask 区域，但非 mask 部分其实也参与计算。

## 笔记（按时间）

- [[02-Topics/AI/AIGC与扩散模型/2025-12 AI生图|2025-12 AI生图]] — 讲了：0. 核心目标：拟合数据分布 (Fitting  / 1. 早期探索：变分自编码器 (VAE) ——  / 1.1 原理
- [[02-Topics/AI/AIGC与扩散模型/2025-12 ControlNet|2025-12 ControlNet]] — 讲了：1. 核心痛点：Img2Img 的局限性 / 2. ControlNet 的底层原理：复制与零 / 第一步：克隆 U-Net 的编码器
- [[02-Topics/AI/AIGC与扩散模型/2025-12 ControlNet 和 IP-Adapter 看着很像|2025-12 ControlNet 和 IP-Adapter 看着很像]] — 讲了：1. 最核心的区别：注入位置不同 / ControlNet：注入到“卷积层” (Spa / IP-Adapter：注入到“注意力层” (At
- [[02-Topics/AI/AIGC与扩散模型/2025-12 Flux-Redux：Flux的IP-Adapter|2025-12 Flux-Redux：Flux的IP-Adapter]] — 讲了：1. 核心区别 / 2. Flux Redux 的工作原理 (数学视 / 3. 在 ComfyUI 中的实际表现
- [[02-Topics/AI/AIGC与扩散模型/2025-12 IP-Adapter|2025-12 IP-Adapter]] — 讲了：1. 核心痛点：为什么“图生图”还不够？ / 2. 底层原理：Decoupled Cross- / 传统流程 (Text Cross-Attenti
- [[02-Topics/AI/AIGC与扩散模型/2025-12 PN结中的扩散!|2025-12 PN结中的扩散!]] — 讲了：证据一：还没接触时，P 区和 N 区都是“电中性 / 证据二：这就是“热力学扩散” / 那个“坑”到底起了什么作用？
- [[02-Topics/AI/AIGC与扩散模型/2025-12 Style Model 发展史|2025-12 Style Model 发展史]] — 讲了：1. 核心问题：为什么要搞“风格模型”？ / 2. 第一代尝试：Textual Inversi / 3. 第二代尝试：Hypernetworks (
- [[02-Topics/AI/AIGC与扩散模型/2025-12 VAE 时代具体生图过程|2025-12 VAE 时代具体生图过程]] — 讲了：第一阶段：训练 (Training) —— 确实 / 第二阶段：生成 (Generation) ——  / 核心数学差异：为什么普通 Autoencoder
- [[02-Topics/AI/AIGC与扩散模型/2025-12 flux 生图原理|2025-12 flux 生图原理]] — 讲了：1. 数学原理：从“扩散”到“流匹配” (Rec / 1.1 核心思想：拉直轨迹 / 1.2 数学公式
- [[02-Topics/AI/AIGC与扩散模型/2025-12 inpainting 模型原理|2025-12 inpainting 模型原理]] — 讲了：1）为什么 Inpainting 扩散模型要“只 / 2）真实的 Inpainting 扩散原理（关键 / 原理核心 A：模型不会对未 mask 区域加噪
- [[02-Topics/AI/AIGC与扩散模型/2025-12 qwen-image-edit的nsfw|2025-12 qwen-image-edit的nsfw]] — 讲了：lora 下载 / 最推荐的 NSFW 专用 LoRA（强烈建议先试 / 下载渠道汇总（优先国内快）
- [[02-Topics/AI/AIGC与扩散模型/2025-12 referenceLatent节点|2025-12 referenceLatent节点]] — 讲了：一、你原本的认知是完全正确的（先确认地基） / 二、问题的根源：你把「latent」当成了一种东 / 三、历史动机：为什么会想“用 latent 当条
- [[02-Topics/AI/AIGC与扩散模型/2025-12 产品换背景工作流|2025-12 产品换背景工作流]] — 讲了：背景不变换产品 / 产品不变换背景
- [[02-Topics/AI/AIGC与扩散模型/2025-12 内补模型条件节点|2025-12 内补模型条件节点]] — 讲了：介绍 / 作用 / 原理
- [[02-Topics/AI/AIGC与扩散模型/2025-12 区域重绘模型|2025-12 区域重绘模型]] — 讲了：1.   为什么会出现 Inpainting？它 / 2.   历史演化：为什么扩散模型非常适合做 I / 2.1 早期方法：基于纹理的修补（PatchMa
- [[02-Topics/AI/AIGC与扩散模型/2025-12 和 controlnet的区别|2025-12 和 controlnet的区别]] — 讲了：一、逐句校正你的理解  非常重要 / ①「Reference 是被直接相加在去噪前和去 / ②「ControlNet 也是相加，不过是有权重
- [[02-Topics/AI/AIGC与扩散模型/2025-12 和ipAdapter的区别|2025-12 和ipAdapter的区别]] — 讲了：一、先把 UNet 的数学骨架立起来（否则一切都 / 二、两种“image conditioning” / ① image → encoder → 注入 d
- [[02-Topics/AI/AIGC与扩散模型/2025-12 扩散模型的具体计算过程|2025-12 扩散模型的具体计算过程]] — 讲了：1. 物理场所：U-Net 网络 / 2. 核心机制：交叉注意力 (Cross-Att / A. 角色分配
- [[02-Topics/AI/AIGC与扩散模型/2025-12 漂移和扩散的本质|2025-12 漂移和扩散的本质]] — 讲了：宇宙是个“懒人” （能量最低原理） / 第一部分：漂移电流的本质 —— 势能梯度的滑落 / 第二部分：相吸相斥的本质 —— 场能量密度的最小
- [[02-Topics/AI/AIGC与扩散模型/2025-12 非mask部分一样被计算了|2025-12 非mask部分一样被计算了]] — 讲了：✔   扩散模型在 inpainting 时并不 / UNet 仍然对整张图做推理，但只有掩码部分的结 / 1) 扩散模型的 UNet 没有“部分计算”机制
- [[02-Topics/AI/AIGC与扩散模型/2026-02 sd前向加噪|2026-02 sd前向加噪]] — 讲了：Stable Diffusion 前向加噪原理
- [[02-Topics/AI/AIGC与扩散模型/2026-02 具体计算|2026-02 具体计算]] — 讲了：1. 核心变量定义 / 2. 第一步：从  z_0  到  z_1 / 3. 第二步：从  z_1  到  z_2
- [[02-Topics/AI/AIGC与扩散模型/2026-02 理解|2026-02 理解]] — 讲了：1. 只有上帝视角看到的“分布” / 2. 为什么数学上要写成  \mathcal{N / 3.  z_t  既是图像，也是分布
- [[02-Topics/AI/AIGC与扩散模型/2026-02 逆向扩散|2026-02 逆向扩散]] — 讲了：一、 对齐：你的文件 vs. 扩散模型 / 二、 准备三个已知的高斯分布 / 三、 硬核计算：指数配方（完全复刻你的文件）
