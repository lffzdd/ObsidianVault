你这个直觉非常对 —— 这是 agent 设计里很关键的一点:**模型对文件系统是"盲"的**,它不知道有哪些文件、目录长什么样。所以光给读写远远不够,你得给它"看"的能力。

正确的做法是把工具配成一组,形成一个闭环:**先发现 → 再读 → 推理 → 再写**。

## 三件套

```python
import os
import traceback

WORK_DIR = "./workspace"   # agent 只能在这个目录里活动

def _safe_path(path: str) -> str:
    """把相对路径锁在 WORK_DIR 里,防止模型读 /etc/passwd 之类"""
    full = os.path.abspath(os.path.join(WORK_DIR, path))
    if not full.startswith(os.path.abspath(WORK_DIR)):
        raise ValueError("路径越界")
    return full

def list_dir(path: str = ".") -> str:
    """列出目录内容 —— 这就是你说的'缺的那块'"""
    try:
        full = _safe_path(path)
        items = []
        for name in sorted(os.listdir(full)):
            p = os.path.join(full, name)
            mark = "/" if os.path.isdir(p) else ""
            size = os.path.getsize(p) if os.path.isfile(p) else ""
            items.append(f"{name}{mark}\t{size}")
        return "\n".join(items) or "(空目录)"
    except Exception:
        return f"Error:\n{traceback.format_exc()}"

def read_file(path: str) -> str:
    try:
        full = _safe_path(path)
        with open(full, "r", encoding="utf-8") as f:
            content = f.read()
        # 大文件别整个塞进 context,会爆 token 还可能溢出
        if len(content) > 5000:
            return content[:5000] + f"\n\n...(已截断,共 {len(content)} 字符)"
        return content
    except Exception:
        return f"Error:\n{traceback.format_exc()}"

def write_file(path: str, content: str) -> str:
    try:
        full = _safe_path(path)
        os.makedirs(os.path.dirname(full), exist_ok=True)
        with open(full, "w", encoding="utf-8") as f:
            f.write(content)
        return f"已写入 {path} ({len(content)} 字符)"
    except Exception:
        return f"Error:\n{traceback.format_exc()}"
```

`list_dir` 就是你感觉缺的那一环。有了它,模型的推理链条才能跑起来:

```
Thought: 我需要修改配置,但不知道配置文件叫什么
Action: list_dir
Action Input: .
Observation: config.json	340
             main.py	1200
             data/	

Thought: 是 config.json,先读一下
Action: read_file
Action Input: config.json
Observation: {"debug": false, ...}

Thought: 把 debug 改成 true
Action: write_file
Action Input: {"path": "config.json", "content": "{\"debug\": true, ...}"}
```

## 还有个常被忽略的点:给模型一个"起点"

即使有了 `list_dir`,如果模型完全不知道当前在哪、目录大概什么样,它第一步会很迷茫,可能瞎试。**两种常见解法:**

一种是在 system prompt 里直接注入初始目录树,让它一上来就有概念:

```python
system_prompt = f"""你是一个能操作文件的 agent。
当前工作目录结构:
{list_dir(".")}

你可以用这些工具: list_dir, read_file, write_file...
"""
```

另一种是让它自己第一步先 `list_dir`。注入初始信息更省一轮交互,但目录大时会占 token —— 看你的场景权衡。

## 几个实践坑

参数从单个变成多个了(`write_file` 要 path 和 content),所以 `Action Input` 通常改成 **JSON 格式**,解析时 `json.loads` 一下,比纯文本好拆。

读文件一定要**截断**。模型很容易让你读一个几万行的 log,整个塞回 context 既烧钱又可能直接超长度。上面用了简单的字符数截断,讲究点可以按行、或者加个 `offset/limit` 参数让模型分页读。

`_safe_path` 那个越界检查别省。模型偶尔会"创造性"地写出 `../../etc/...` 这种路径。

要不要我把这套文件工具 + 之前的代码执行工具,一起接进一个能跑的 ReAct 主循环给你看?这样你能看清"工具注册 → 解析 Action → 调用 → 回填 Observation"整条链路。