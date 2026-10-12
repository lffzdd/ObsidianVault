---
type: 入口笔记
---
# LLM

## 当前理解

LLM 本质是对模板化 token 序列做续写：role/content、im_start 这类标记是训练出来的格式，不是所有模型都一样；enable_think 等参数只是改模板。中转站“降智”多半来自模板、参数或模型被替换。

## 笔记（按时间）

- [[02-Topics/AI/LLM/2025-11-05 role，content字段是训练好的|2025-11-05 role，content字段是训练好的]] — 讲了：一、 role  是训练时的固定结构，不可随意改 / 二、API 层也强制限制 / 三、为什么要固定这几个？
- [[02-Topics/AI/LLM/2025-11-05 模型的输入是Json，输出不一定|2025-11-05 模型的输入是Json，输出不一定]] — 讲了：一、接口层：JSON 包裹，文本在里面 / 二、LangChain 等上层框架怎么处理这个输 / 三、模型“输出 JSON”的两种含义
- [[02-Topics/AI/LLM/2025-11-06 enable_think参数|2025-11-06 enable_think参数]] — 讲了：一、层次结构分解：从外到内 / 1.   客户端调用层 / 2.   vLLM 模板渲染层
- [[02-Topics/AI/LLM/2025-11-06 llm模板|2025-11-06 llm模板]] — 讲了：一、模板是什么？ / 二、模板存在的意义 / 1.   统一格式
- [[02-Topics/AI/LLM/2025-11-06 不是所有llm都采用im_xxx|2025-11-06 不是所有llm都采用im_xxx]] — 讲了：不是所有llm都采用im_xxx
- [[02-Topics/AI/LLM/2026-01-27 知识蒸馏|2026-01-27 知识蒸馏]] — 讲了：1. 核心矛盾：越强越慢 vs. 越快越弱 / 2. 历史背景：Hinton 的神来之笔 (20 / 3. 原理详解：温度 (Temperature)
- [[02-Topics/AI/LLM/2026-04-06 llm中转站为何降智|2026-04-06 llm中转站为何降智]] — 讲了：API 格式转换导致降智的完整拆解 / 一、最核心的问题：Thinking（扩展思考） / 1. Claude 原生 API 的 Think

### 早期专题笔记

- [[02-Topics/AI/LLM/0-学习路径|0-学习路径]] — 讲了：🕰️ LLM 的历史发展大致可以分为这几个阶段： / 1.   早期探索：神经网络 & 词向量 (19 / 2.   从 RNN 到 Transformer
- [[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/1-n-gram模型|1-n-gram模型]] — 讲了：什么是 n-gram 模型？ / 为什么要用 n-gram？ / n-gram 怎么建模？
- [[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/2-word2vec|2-word2vec]] — 讲了：原版损失函数 / 背景：Word2Vec 目标 / 1. 训练目标
- [[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/3-RNN-循环神经网络|3-RNN-循环神经网络]] — 讲了：3-RNN-循环神经网络
- [[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/4-LSTM|4-LSTM]] — 讲了：4-LSTM
- [[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/5-GRU|5-GRU]] — 讲了：5-GRU
- [[02-Topics/AI/NLP与Transformer/RNN与早期语言模型/6-RNN-递归神经网络|6-RNN-递归神经网络]] — 讲了：6-RNN-递归神经网络
- [[02-Topics/AI/AIGC与扩散模型/生图进阶 - LoRA 之后学什么|图片]] — 讲了：第一优先级：ControlNet —— 图像的“ / 第二优先级：IP-Adapter —— 图像的“ / 第三优先级：Inpainting & Outpa
