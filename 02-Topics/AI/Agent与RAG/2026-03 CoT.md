

### 第二步：历史脉络与发展阶段 (The Evolution of CoT)

在高质量的英文研究文献中，CoT 的发展有一条非常清晰的脉络：

1. **里程碑诞生 (2022年1月)**：Google Brain 的 Jason Wei 等人发表了著名论文 _Chain-of-Thought Prompting Elicits Reasoning in Large Language Models_。他们首次证明，只要在 Prompt 里给模型演示几个带有“思考过程”的示例（**Few-Shot CoT**），模型在数学和常识推理上的准确率就会获得惊人的飞跃（在 GSM8K 数学基准测试上甚至超越了专门微调过的模型）。
    
2. **“魔法咒语”的发现 (2022年5月)**：东京大学的 Kojima 等人发现了一个更暴力的做法——完全不需要给示例，只需要在 Prompt 结尾加上一句 **"Let's think step by step"**。这被称为 **Zero-Shot CoT**。它证明了这些大模型在海量的预训练数据中，其实已经学到了逻辑推导的模式，这句咒语只是激活了它们。
    
3. **自洽性优化 (2022年底至今)**：研究者进一步提出了 **Self-Consistency CoT**。既然模型有随机性，那就让它针对同一个问题“思考”多条不同的路径，最后采取多数投票（Majority Vote）决定最终答案，这大大增强了推理的鲁棒性。
    

