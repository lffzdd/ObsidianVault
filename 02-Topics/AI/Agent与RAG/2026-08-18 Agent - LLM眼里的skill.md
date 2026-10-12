**对！完全正确！** 你的理解已经完全切中它的核心机制了。

我们用一个两轮对话的具体例子来看看这个“**即插即用、用完即弃**”的过程：

---

### 第 1 轮：大模型刚调用完 `load_skill("release-check")`

1. **真实的历史记录（存在数据库里）**：
   * `[消息1] User: "准备发版"`
   * `[消息2] Assistant: 调用 load_skill("release-check")`
   * `[消息3] Tool: "已激活 release-check"`

2. **发给大模型的网络请求（在尾部临时贴一张便签）**：
   * `[消息1] User: "准备发版"`
   * `[消息2] Assistant: 调用 load_skill("release-check")`
   * `[消息3] Tool: "已激活 release-check"`
   * `[临时便签A] User: <skill id="release-check">步骤1: pytest; 步骤2: git...</skill>` 👈 **临时贴上去的**

3. 大模型读完这张便签，决定执行 `pytest`。

---

### 第 2 轮：大模型执行完了 `pytest`，准备走下一步

1. **真实的历史记录更新为（注意：便签A根本没存进去）**：
   * `[消息1] User: "准备发版"`
   * `[消息2] Assistant: 调用 load_skill(...)`
   * `[消息3] Tool: "已激活..."`
   * `[消息4] Assistant: 调用 execute_command("pytest")`
   * `[消息5] Tool: "254 passed"`

2. **发给大模型的网络请求（重新生成便签B，再次贴在最新的第5条后面）**：
   * `[消息1] ...`
   * `[消息2] ...`
   * `[消息3] ...`
   * `[消息4] ...`
   * `[消息5] Tool: "254 passed"`
   * `[临时便签B] User: <skill id="release-check">步骤1: pytest; 步骤2: git...</skill>` 👈 **重新贴在最新消息后面**

---

### 第 3 轮：大模型全部执行完毕，调用了 `unload_skill`

1. **真实的历史记录**：
   * `[消息1 ~ 5] ...`
   * `[消息6] Assistant: 调用 unload_skill("release-check")`
   * `[消息7] Tool: "已卸载 release-check"`

2. **发给大模型的网络请求（因为已卸载，不再生成便签）**：
   * `[消息1 ~ 7] ...`
   * （尾部什么都不贴了，或者只贴极简的菜单清单）

---

### 这种机制的妙处：

1. **大模型每一轮看到的都是“最新进展 + 最新的状态提示”**；
2. **历史记录永远保持纯净**，不会在历史消息里到处堆满重复的 Skill 正文；
3. **只要一卸载，下一轮瞬间消失得无影无踪**！

---

Viewed prompt.py:1-65

你的理解非常到位！**`name` 和 `description` 确实就是专门写给大模型看的。**

我们来看看它们是**在什么时机、以什么形式**给 LLM 看的，以及为什么第一步要对它们做严格限制：

---

### 1. `description` 什么时候给 LLM 看？——【清单层（平时每一轮）】

在平时对话时，LLM **看不见 Skill 的具体步骤（正文）**，但每一轮的 Prompt 里都会注入一份极简的 **“技能目录（Catalog）”**：

```markdown
<system-reminder>
以下是当前可用的 skill。平时只看清单；需要完整步骤时调用 load_skill。
<skill-catalog>
- release-check: 本项目发布前必须执行的检查流程；当用户提到发布、上线、release 时使用
- db-migrate: 数据库结构变更与回滚标准操作流程；当涉及数据表修改时使用
</skill-catalog>
</system-reminder>
```

👉 **LLM 就是靠读 `description` 这句话来做“触发词判断”的**：
当用户说：“代码写好了，帮我准备上线吧。”
LLM 看到清单里的 `release-check: ...当用户提到发布、上线时使用`，就会立刻决定调用工具：
`load_skill("release-check")`。

---

### 2. `name` 和 `body` 什么时候给 LLM 看？——【正文层（激活后）】

一旦 LLM 触发了 `load_skill("release-check")`，从下一轮开始，Prompt 里才会把 `name` 和详细的正文注入给它：

```markdown
<system-reminder>
以下是本回合已激活 skill 的完整正文。
<skill id="release-check" name="发布前检查">
建议工具: execute_command, read_file

## 检查步骤
1. 运行 pytest 确保单测全过
2. 执行 git status 确认工作区无遗留
3. 检查 version 变量是否递增
</skill>
</system-reminder>
```

此时 LLM 拿到了具体的 SOP 步骤，开始一步步照着执行。

---

### 3. 那么，第一步（B1）为什么要对它们做那么严格的限制？

正因为 `description` 是**每一轮对话都要常驻塞给 LLM 的**，所以必须在文件解析阶段设防：

1. **防 Token 暴涨**：如果有人在 `description` 里洋洋洒洒写了 3000 字，它会在每一轮都消耗你的上下文预算。所以限制 `description <= 400 字符`，`name <= 80 字符`；
2. **防排版污染**：强制通过 `_single_line` 清理换行符，保证清单在 LLM 看来永远是一行一个 `- id: description`，格式整齐不混乱；
3. **缺字段即废弃**：如果开发者写了一个 `SKILL.md` 却忘了写 `name` 或 `description`，LLM 根本不知道这个技能是干嘛的、何时该用，所以解析时直接视为非法 Skill 并报警。

---

这样设计就实现了：**“平时只占一两行 Token（Progressive Disclosure），需要时才展开全文”**。

