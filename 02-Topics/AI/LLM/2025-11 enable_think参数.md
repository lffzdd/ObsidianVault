非常好的问题——这正是理解「模板机制」和「模型本体」边界的关键点。  
结论先说：**`enable_thinking` 不会被模型看到。它不会出现在模型输入的 JSON 里。**  
这个参数只在客户端或 vLLM 的「模板渲染层」生效，也就是**在生成 prompt 文本之前**就被程序逻辑处理掉了。模型接收到的只是一段普通的字符串，没有任何“enable_thinking”的字样。

---

### 一、层次结构分解：从外到内

理解这一点要把「一次调用」拆成三层：

#### 1. **客户端调用层**

你发起请求时写：

```python
client.chat.completions.create(
    model="Qwen/Qwen3-14B-AWQ",
    messages=[{"role": "user", "content": "讲讲大模型"}],
    extra_body={"chat_template_kwargs": {"enable_thinking": False}}
)
```

这一步只是告诉服务器：“在渲染 prompt 时，请把 `enable_thinking` 设为 False”。  
此时服务器还没生成真正的 prompt 文本。

#### 2. **vLLM 模板渲染层**

vLLM 内部会调用 Qwen 的 **chat template**：

```python
tokenizer.apply_chat_template(
    messages,
    enable_thinking=False,  # 这里才用到
    add_generation_prompt=True
)
```

模板文件是一个 Jinja2 脚本，它会判断：

```jinja2
{% if enable_thinking %}
<|im_start|>assistant
<think>
...
</think>
{% endif %}
```

所以 `enable_thinking=False` 时，这段 `<think>` 包裹就**不会被写入 prompt**。

> 也就是说，这个参数只影响“要不要加 `<think>` 这几行文字”。

#### 3. **模型推理层**

模型看到的，是模板渲染完之后的纯文本，比如：

```
<|im_start|>system
你是一个有帮助的助手。<|im_end|>
<|im_start|>user
讲讲大模型。<|im_end|>
<|im_start|>assistant
```

它完全不知道“enable_thinking”这回事。  
此时输入已经变成一串 token 序列，模型只是按训练好的语言分布预测下一个 token。

---

### 二、所以说：

- `enable_thinking` 是**渲染层的参数**（在调用 `apply_chat_template` 时决定最终 prompt 长什么样）；
    
- 它不是传给模型的 JSON 字段；
    
- 模型从未见过 “enable_thinking” 这个词；
    
- 如果你没关掉它，模板默认是 `True`，就会多插入 `<think>` 标记，模型自然会按这个格式生成推理部分。
    

---

### 三、打个比喻

你可以把整个过程类比成「打印信件」：

- `enable_thinking=True/False` 是打印机的选项：决定是否加上“思考过程”页；
    
- 模型是信的收件人，它只读到信的内容；
    
- 打印机选项不会被写进信纸上。
    

---

### 四、验证方法

你可以在 vLLM 服务器上加一行调试：

```bash
INFO [logger.py] Rendered prompt: ...
```

当你切换 `enable_thinking=True/False` 时，会看到 `<think>` 段落是否出现。  
模型接收到的就是那段“Rendered prompt”，再无其他元参数。

---

### 五、总结成一句话

> `enable_thinking` 是 **模板渲染阶段的控制参数**，由客户端或服务端在构造 prompt 时决定是否插入 `<think>` 标签。  
> 模型只接收渲染后的纯文本，从未看到或理解过 “enable_thinking” 这个词。

---

如果你想更深入，我可以画出 Qwen3 的模板逻辑结构图（Jinja 渲染路径），让你看到 `<think>` 是怎么被插进去的、`assistant` 起始标记如何配合的、以及为什么 vLLM 能分离出 `reasoning_content`。是否要我接着讲这一层？