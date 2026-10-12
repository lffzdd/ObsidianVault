---
type: 入口笔记
---
# R-CNN系列

## 当前理解

两阶段检测的演化：R-CNN 用 selective search 选候选框再分类，Fast R-CNN 共享特征并用 RoI Pooling，Faster R-CNN 用 RPN 和 anchor 生成候选框，FPN 处理多尺度，Mask R-CNN 用 RoI Align 加像素级分支。

## 笔记（按时间）

- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 Fast R-CNN 和 Faster R-CNN|2025-12-16 Fast R-CNN 和 Faster R-CNN]] — 讲了：第一阶段进化：Fast R-CNN (2015) / 1. 核心改进：共享卷积特征 (Shared C / 2. 核心数学工具：RoI Pooling 层
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 Fast R-CNN|2025-12-16 Fast R-CNN]] — 讲了：Fast R-CNN：把 R-CNN 的“慢”拆 / 一、先说一句总纲（非常重要） / 二、Fast R-CNN 的“一条完整检测流水线
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 Faster R-CNN|2025-12-16 Faster R-CNN]] — 讲了：Faster R-CNN：两次“框 → 向量 → / 一、先给一句“总纲式描述”（先立骨架） / 二、第一阶段：RPN 到底在“跑什么”？
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 HOG特征cell统计|2025-12-16 HOG特征cell统计]] — 讲了：Step 1：cell 内统计（不做任何归一化） / Step 2：block 内拼接（先拼） / Step 3：对整个 block 向量做归一化（
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 R-CNN|2025-12-16 R-CNN]] — 讲了：R-CNN 的完整计算流程 / 第一步：候选区域生成 (Region Propo / 第二步：特征提取 (Feature Extrac
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 selective search|2025-12-16 selective search]] — 讲了：怎么算 / 第一步：初始分割 (Initial Sub-Se / 第二步：计算相似度 (Similarity Ca
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 滑动检测 HOG 怎么算的|2025-12-16 滑动检测 HOG 怎么算的]] — 讲了：一、先把问题缩到“最小计算单元” / 二、第一步：从像素到“特征”（这是最关键的计算） / 1️⃣ 为什么不能直接用像素？
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 目标检测发展|2025-12-16 目标检测发展]] — 讲了：一、先看清：早期方法到底“卡”在哪一步？ / 二、R-CNN（2014）：第一次“结构断裂” / 1️⃣ R-CNN 解决的不是“快”，而是“算什
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-16 目标检测的起源|2025-12-16 目标检测的起源]] — 讲了：起源 / 一、为什么一开始根本没有“目标检测”这个问题？ / 1️⃣ 计算机视觉的最初目标：不是“找东西”，而
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 Fast 到 Fast R-CNN的改进|2025-12-17 Fast 到 Fast R-CNN的改进]] — 讲了：Fast 到 Fast R-CNN的改进
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 IoU-这么多个proposal怎么去对标那几个GT框|2025-12-17 IoU-这么多个proposal怎么去对标那几个GT框]] — 讲了：训练时到底发生了什么？（假设有 5 个 GT 框 / 1. 训练时你真正拥有的只有一样东西 / 2. 用什么建立对应关系？—— IoU（这是设计
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 Mask R-CNN 1 - RoI Align 与 Mask 分支数学|2025-12-17 Mask R-CNN 1 - RoI Align 与 Mask 分支数学]] — 讲了：1. 核心痛点：RoI Pooling 的“量化 / 为什么要改？ / 2. 核心数学创新：RoI Align
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 Mask R-CNN 2 - 完整流水线与设计动机|2025-12-17 Mask R-CNN 2 - 完整流水线与设计动机]] — 讲了：Mask R-CNN：在 Faster R-CN / 一、先给一句“总纲级描述”（这是你现在最缺的） / 二、Mask R-CNN 的完整检测流水线（一次
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 Mask 输出头的映射|2025-12-17 Mask 输出头的映射]] — 讲了：1. 为什么是  14 \times 14 \t / 2. 核心难点： 28 \times 28  怎 / 3. 为什么这样是可行的？（RoI Align
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 从Fast R-CNN开始，计算的维度问题|2025-12-17 从Fast R-CNN开始，计算的维度问题]] — 讲了：1. 核心概念：两个坐标系 / 2. 详细流程拆解 (以 Faster R-CN / 第一步：Anchor 的生成 (在原图坐标系)
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/2025-12-17 偏移量的量纲问题|2025-12-17 偏移量的量纲问题]] — 解决：t_x   (GT_x - Anchor_x) / Anchor_w？
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/1-R-CNN|1-R-CNN]] — 讲了：发展历程 / R-CNN 代码 / R-CNN 的主要步骤：
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/1.1-ROI|1.1-ROI]] — 讲了：选择性搜索的基本步骤 / 1.   图像分割（Superpixel Seg / 2.   区域合并（Region Merging
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/1.1.2-空间连通性|1.1.2-空间连通性]] — 讲了：计算空间连通性 / 空间连通性定义 / 计算空间连通性
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/1.1.3-纹理相似度和形状相似度|1.1.3-纹理相似度和形状相似度]] — 讲了：1. 纹理相似度（Texture Similar / 2. 形状相似度（Shape Similarit / 示例
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/1.1.3.2-L2范数归一化|1.1.3.2-L2范数归一化]] — 讲了：1. 计算向量的L2范数 / 2. 将向量除以其L2范数
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/1.2-双线性插值|1.2-双线性插值]] — 讲了：1.2-双线性插值
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/3.1-BBox Regreessor|3.1-BBox Regreessor]] — 讲了：工作原理 / 场景设定 / BBox Regressor 的工作流程
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/4-FPN|4-FPN]] — 讲了：代码示例
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/4-fast rcnn|4-fast rcnn]] — 讲了：代码结构概览 / 1.  data/  目录 / 2.  lib/  目录
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/4.1-Faster R-CNN和FPN的结合|4.1-Faster R-CNN和FPN的结合]] — 讲了：1. Faster R-CNN 简介 / 2. FPN 的优势 / 3. Faster R-CNN 与 FPN 的结
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/4.2-PRN|4.2-PRN]] — 讲了：PRN 的核心概念 / 1. 输入特征图 / 2. Anchor 机制
- [[02-Topics/AI/计算机视觉/目标检测/R-CNN系列/4.3-Anchor框匹配到BBox|4.3-Anchor框匹配到BBox]] — 解决：经过 nms 后，anchor 框会经过 rot 池化，然后再进行分开进行分类预测和边框回归，可是，一个图片有很多个 a？
