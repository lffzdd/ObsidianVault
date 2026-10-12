---
type: 入口笔记
---
# LangChain

## 当前理解

LangChain 只是把 LLM 的输入输出包装成消息对象（AIMessage、ToolMessage）：模型“看到”的始终是拼好的模板文本，工具调用是一问一答的循环，解析器只处理模型生成的内容；不同后端（DeepSeek、vLLM）的工具调用格式并不统一。

## 笔记（按时间）

- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 AIMessage，ToolMessage|2025-11 AIMessage，ToolMessage]] — 讲了：一、为什么需要  AIMessage  和  T / 二、 AIMessage ：模型的发言（带“意图 / 三、 ToolMessage ：工具的回应（给模
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 Agent的思维如何停止|2025-11 Agent的思维如何停止]] — 讲了：一、Agent 的基本循环逻辑（底层思想） / 二、那 ChatGPT / Gemini 页面是 / ✅ ChatGPT（OpenAI 网页版）
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 langchain llm看到的是什么|2025-11 langchain llm看到的是什么]] — 讲了：一、模型真正看到的是什么？ / 二、这些角色的意义（模型侧的理解） / 三、如果模型不支持角色结构会怎样？
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 langchain 解析器只处理llm的生成内容|2025-11 langchain 解析器只处理llm的生成内容]] — 讲了：一、区分两层 JSON / 1.   API 层 JSON / 2.   模型输出 JSON
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 langchain教程|2025-11 langchain教程]] — 讲了：课程地图（先给你个整体框架，不一次讲完） / 第 1 课：准备环境 & 第一个“可复用链” / 1. 安装
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 langchain解析器要求用户知道生成内容的结构|2025-11 langchain解析器要求用户知道生成内容的结构]] — 讲了：一、三种常见“知道程度”的场景 / 1. ✅ 我知道输出一定是纯文本 / 2. ⚙️ 我大概知道结构（比如 JSON 或列
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 为什么deepseek和vllm返回的工具调用不一样|2025-11 为什么deepseek和vllm返回的工具调用不一样]] — 讲了：更细一点的“为什么” / 怎么办：三条可选的落地路径 / 路径 A：让“服务端解析”真正生效（首选）
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 为什么langchain自己的agent调用工具是一问一答|2025-11 为什么langchain自己的agent调用工具是一问一答]] — 讲了：提示词模板 / 核心差异 / 2.py（你的手写 Agent）
- [[02-Topics/AI/Agent与RAG/LangChain/2025-11 包含tool时llm的输入输出|2025-11 包含tool时llm的输入输出]] — 讲了：一、LangChain 只是「语法糖」 / 二、模型返回的原始输出（function cal / 一、外层JSON是服务器包装的，不是模型输出的
