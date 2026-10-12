你看到的差异，根子在于**谁来把“像 JSON 的文本”解析成结构化的 `tool_calls`**：

- **DeepSeek 这条链路**：服务端原生支持“函数/工具调用”协议。模型按协定吐出“调用意图”文本片段，**DeepSeek 的网关先解析**，再把解析结果塞进响应里的 `tool_calls` 字段，并以 `finish_reason: "tool_calls"` 告诉你“该执行工具了”。LangChain 读到标准字段，就把它变成 `tool_calls=[...]`。
    
- **本地 vLLM + Qwen3 这条链路**：你用的是“OpenAI 兼容”接口，但**网关并没有替你做解析**（或该模型/路由本身就不支持原生 tool-calling 协议）。Qwen3 于是按训练习惯输出了**带 `<tool_call>...</tool_call>` 标签的纯文本**，`finish_reason` 是普通的 `"stop"`；LangChain 没拿到标准 `tool_calls` 字段，自然也就不会自动把它变成结构化调用。
    

换句话说：  
DeepSeek 是“**服务端解析**（server-side parsing）”，而你本地 Qwen3 是“**客户端自行解析**（client-side parsing）”。这就是两段打印为何不同的底层原因。

---

## 更细一点的“为什么”

1. **协议能力是否开启**  
    工具调用要在请求里带上“工具清单+schema”（OpenAI 风格是 `tools` / `tool_choice`，早期叫 `functions`）。支持的供应商会据此切换到“函数调用模式”，并在输出里**产生结构化 `tool_calls`**。
    
    - DeepSeek 路由支持 → 你看到了 `finish_reason: "tool_calls"`。
        
    - 你的 vLLM 路由多半**没启用或不支持**这个模式 → 只能返回“文本里的 `<tool_call>`”。
        
2. **模型模板与训练习惯**  
    一些模型（如 Qwen 系列）在未启用原生工具协议时，会**用自家标注格式**（XML/JSON 片段）表达“我要调哪个工具”。这在“纯文本模式”下是合理的，但需要你**自行解析**。
    
3. **LangChain 的封装策略**  
    LangChain 会优先相信**供应商标准字段**（`tool_calls`）。如果拿到的是**纯文本**，除非你挂了一个**输出解析器（OutputParser）**，否则它不会擅自猜测并拆出 `tool_calls`。
    

---

## 怎么办：三条可选的落地路径

### 路径 A：让“服务端解析”真正生效（首选）

- 确认你对 vLLM 的请求里确实传了工具清单（在 LangChain 里就是 `llm.bind_tools([...])`；底层应等价为 OpenAI 风格的 `tools=[...]`）。
    
- 确认你起的 vLLM OpenAI 兼容服务**支持并开启了 tool/function calling**。部分版本或路由只做“聊天兼容”，**不做函数调用解析**。
    
- 某些 Qwen 的推理模板可配置“OpenAI tool-calling 格式”，要确保用的就是这套模板/路由。
    

效果：响应里会出现标准 `tool_calls`，与你在 DeepSeek 的表现一致，LangChain 将自动走工具执行分支。

### 路径 B：保留“文本标签”，但在客户端**自己解析**

给当前的 Qwen3 链路挂一个极薄的解析器：

- 如果 `resp.tool_calls` 为空，就去 `resp.content` 里**提取 `<tool_call>...</tool_call>`**；
    
- 解析出 `{"name": "...","arguments": {...}}` 后，构造一个**伪 `tool_calls`** 交给你的编排层执行。
    

伪代码示意（思路）：

```python
from dataclasses import dataclass
import json, re

@dataclass
class ToolCall:  # 你的内部统一结构
    name: str
    args: dict
    id: str

def parse_tool_call_from_text(text: str) -> list[ToolCall]:
    m = re.search(r"<tool_call>\s*(\{.*?\})\s*</tool_call>", text, re.S)
    if not m:
        return []
    payload = json.loads(m.group(1))
    return [ToolCall(name=payload["name"],
                     args=payload.get("arguments", {}),
                     id="pseudo_"+str(hash(text)))]

# 使用：
msg = llm.invoke(messages)
calls = msg.tool_calls or parse_tool_call_from_text(msg.content or "")
```

优点：不依赖网关能力，立刻可用。  
缺点：健壮性要你自己兜底（多调用并行、异常格式、流式增量等）。

### 路径 C：提示工程 + few-shot，**禁止闲聊**、只产出调用

若短期无法改造网关，也不想写解析器，可以通过提示/示例约束模型：

- 要求“**先决策后回答**”：只有拿到工具结果才生成自然语言；首次响应只许输出 `<tool_call>{...}</tool_call>`；
    
- 给一两个**负例**（出现自然语言就判失败），提高一致性。  
    这能减少“我来帮您查询……”之类的自然语言前置，方便你用路径 B 解析。
    

---

## 你可以做的“规范化”小结

- **统一协议**：能用 OpenAI 风格 `tools/tool_choice` 的路由，就让所有模型都走这套；
    
- **统一解析**：写一个轻量“**工具调用标准化层**”，输入原始响应，输出统一的 `tool_calls=[...]`：
    
    1. 若供应商已解析 → 直接透传；
        
    2. 若是纯文本 `<tool_call>` → 走本地解析器；
        
    3. 若两者皆无 → 判定为“无工具调用”，进入普通回答或下轮思考。
        
- **统一编排**：后面的 LangChain/Graph 逻辑只面向**你的统一结构**编排，屏蔽供应商差异。
    

这样，不论 DeepSeek、Qwen3 还是别家，表现都能“看齐”：**要么标准 `tool_calls`，要么被你自动标准化成 `tool_calls`**。接下来你就可以把注意力放在“何时停、如何重试、怎样审计”这些更本质的工程问题上。