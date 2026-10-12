---
type: 入口笔记
---
# AIGC与扩散模型

## 当前理解

扩散模型的核心是“前向加噪、逆向去噪”，在 VAE 的潜空间里一步步把噪声还原成图像。实际工作流（ComfyUI）里，ControlNet 管结构约束，IP-Adapter / Flux-Redux / Style Model 管风格和参考图，inpainting / 区域重绘只改 mask 区域，但非 mask 部分其实也参与计算。

## 笔记（按时间）

- [[02-Topics/AI/AIGC与扩散模型/生图进阶 - LoRA 之后学什么|生图进阶 - LoRA 之后学什么]] — 讲了：第一优先级：ControlNet —— 图像的“ / 第二优先级：IP-Adapter —— 图像的“ / 第三优先级：Inpainting & Outpa

## 子主题

- 入口：[[02-Topics/AI/AIGC与扩散模型/局部重绘与工作流/局部重绘与工作流|局部重绘与工作流]]（6 篇）

- 入口：[[02-Topics/AI/AIGC与扩散模型/扩散原理/扩散原理|扩散原理]]（8 篇）

- 入口：[[02-Topics/AI/AIGC与扩散模型/条件控制与参考图/条件控制与参考图|条件控制与参考图]]（8 篇）
