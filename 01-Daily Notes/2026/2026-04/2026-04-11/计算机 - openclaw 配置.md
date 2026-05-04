你会迷糊，不是你理解差，而是 OpenClaw 官方文档里把 **几种完全不同的“维度“**  混在一起讲了：有的是**状态隔离**，有的是**Agent 定义**，有的是**消息路由**，有的是**会话切分**。它们不是一条 `profile -> agent -> channel -> account -> user -> session` 的单链，而是几条正交的轴交叉在一起。官方自己也存在“单 Agent 旧写法”和“多 Agent 新写法”并存的情况，所以看起来更乱。([OpenClaw](https://docs.openclaw.ai/cli "CLI Reference - OpenClaw"))

## 核心原理

OpenClaw 更接近下面这个模型：

**状态根 (profile)** → 读哪一套 `~/.openclaw-*`  
**Agent** → 这个“脑子”是谁、用什么 workspace / auth / tools  
**Channel / Account** → 消息从哪个平台、哪个登录账号进来  
**Binding** → 按 `(channel, accountId, peer...)` 把消息路由给哪个 agent  
**Session** → 在已经选定 agent 之后，这条消息该落进哪个会话

也就是说，`channel/account` 不是 `agent` 的子节点，`session` 也不是 `user` 的配置子节点；它们是**消息处理流水线中的不同阶段**。官方对 multi-agent 的定义就很明确：一个 `agentId` 是一个独立 brain，`binding` 负责把 `(channel, accountId, peer)` 路由到 `agentId`，然后 direct chat 再折叠到该 agent 下的某个 session key。([OpenClaw](https://docs.openclaw.ai/concepts/multi-agent "Multi-Agent Routing - OpenClaw"))

## 机制细节

### 1) `profile` 不是 `openclaw.json` 里的层级节点

CLI 里的 `--profile <name>` 是**状态目录隔离**：它把整套状态切到 `~/.openclaw-<name>`，相当于“换一套 OpenClaw 实例环境”。这不是你 `openclaw.json` 内部的配置继承层级。官方 CLI 文档写得很直接：`--profile <name>` 会隔离 state 到 `~/.openclaw-<name>`。([OpenClaw](https://docs.openclaw.ai/cli "CLI Reference - OpenClaw"))

更坑的是，“profile” 这个词在 OpenClaw 里还被重复使用：

- `tools.profile`：工具权限基线
- auth profile：模型认证档案
- browser profile：浏览器配置档案
- CLI `--profile`：状态目录隔离

所以你如果把这些 profile 当成同一种“父层级”，一定会乱。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))

### 2) `agent` / `agents` 两套写法并存，这是官方文档本身的历史包袱

官方示例页现在还在用单 Agent 简写：

```json5
{
  agent: { workspace: "~/.openclaw/workspace" },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

但另一份官方文档又明确说：**legacy `agent.*` 配置会被 `openclaw doctor` 迁移，今后更推荐 `agents.defaults + agents.list`**。这说明 `agent` 其实是旧时代/单 Agent 便利写法，而不是多 Agent 模型下的主结构。你觉得“openclaw.json 看起来又不是这么配”的根本原因之一，就是你看到的文档页可能一个还在展示单 Agent shortcut，另一个已经按多 Agent 正式模型写了。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-examples "Configuration Examples - OpenClaw"))

所以现在更稳的理解是：

- `agents.defaults`：所有 agent 的默认值
- `agents.list[]`：具体每个 agent 的覆盖项

并且 agent 是真正的一等实体：它有自己的 `workspace`、`agentDir`、模型、tools、sandbox、session store。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))

### 3) `channel` 和 `account` 负责“入口”，但**不直接决定 agent 配置归属**

`channels.<channel>` 是平台配置；`channels.<channel>.accounts.<id>` 是该平台下多个账号实例的配置。官方文档明确写了：base channel settings 会作用到所有 account，account 可以再覆盖；`default` account 在省略 `accountId` 时会被使用。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))

但重点在于：**account 到 agent 的映射不写在 `channels.*` 里面，而是写在顶层 `bindings[]`**。官方多 Agent 路由示例就是：

```json5
bindings: [
  { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
  { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
]
```

所以 `channel/account` 属于“消息来源描述”，`binding` 才是“来源到 agent 的路由规则”。这不是嵌套父子关系，而是**配置拆表**：一张表写平台账号，一张表写路由。这样做的原因是同一个 channel/account 还可能继续按 `peer`、`guildId`、`teamId` 细分路由，放进 `bindings[]` 更灵活。官方也给了 deterministic match order：先 `peer`，再 guild/team，再精确 `accountId`，再 `accountId:*`，最后 default agent。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))

### 4) 你说的 `user`，在 OpenClaw 里通常不是一级配置对象，而是 `peer`

这是最容易误解的地方。OpenClaw 的路由规则里，通用概念不是 “user”，而是 `match.peer`，也就是“对端标识”：可能是 direct 对话对象、某个群、某个频道。官方定义 binding 时写的是按 `(channel, accountId, peer)` 路由。([OpenClaw](https://docs.openclaw.ai/concepts/multi-agent "Multi-Agent Routing - OpenClaw"))

进一步说，DM 里“用户隔离”也不是通过一个 `user` 配置层来完成，而是通过 `session.dmScope` 决定 session key 怎么切：

- `main`
- `per-peer`
- `per-channel-peer`
- `per-account-channel-peer`

如果同一个人跨平台联系你，再用 `session.identityLinks` 把多个 provider-prefixed peer 绑成同一个身份。也就是说，所谓 “user” 在 OpenClaw 里更接近**路由/会话键的一部分**，而不是 `openclaw.json` 里的固定层级节点。([OpenClaw](https://docs.openclaw.ai/concepts/session "Session Management - OpenClaw"))

### 5) `session` 是路由之后产生的，不是前面的父层

官方 session 文档写得很清楚：OpenClaw 先把消息按来源组织进 sessions；DM 默认共享一个 session，group/room 按各自隔离，cron 每次新 session。session 状态最终是存到 `~/.openclaw/agents/<agentId>/sessions/...`，说明 session 是**属于某个 agent 的运行态**。([OpenClaw](https://docs.openclaw.ai/concepts/session "Session Management - OpenClaw"))

所以真正流程是：

**收到消息**  
→ 知道它来自哪个 `channel/account/peer`  
→ 用 `bindings[]` 选中 `agentId`  
→ 再根据 `session.dmScope` 等规则决定 session key  
→ 把这条消息送进该 agent 的那个 session

这就是为什么我前面说，它不是一棵配置树，而是一条处理管线。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))

## 具体示例

你可以把它记成下面这份“脑内分层图”：

```json5
{
  // 1. 所有 agent 的默认值
  agents: {
    defaults: {
      workspace: "~/.openclaw/workspace",
      model: { primary: "anthropic/claude-sonnet-4-6" }
    },

    // 2. 每个 agent 的个性化覆盖
    list: [
      { id: "main", default: true, workspace: "~/.openclaw/workspace-main" },
      { id: "work", workspace: "~/.openclaw/workspace-work" }
    ]
  },

  // 3. 平台 / 账号配置
  channels: {
    telegram: {
      accounts: {
        default: { botToken: "..." },
        alerts: { botToken: "..." }
      }
    }
  },

  // 4. 路由表：哪个来源走哪个 agent
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "work", match: { channel: "telegram", accountId: "alerts" } }
  ],

  // 5. 会话切分规则
  session: {
    dmScope: "per-account-channel-peer"
  }
}
```

这个例子里：

- `agents.*` 决定“脑子长什么样”
- `channels.*` 决定“门口有哪些入口和账号”
- `bindings[]` 决定“从哪个门进来该去哪个脑子”
- `session.*` 决定“进了这个脑子以后，聊天历史怎么分桶”

这和 `profile -> agent -> channel -> account -> user -> session` 完全不是一回事。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-reference "Configuration Reference - OpenClaw"))

## 结论

你现在最该采用的心智模型不是“层级树”，而是“**五个正交维度**”：

1. `--profile`：切哪套 OpenClaw 状态目录
2. `agents.defaults / agents.list`：定义 agent 本身
3. `channels.<channel>.accounts.<id>`：定义消息入口
4. `bindings[]`：把入口路由到 agent
5. `session.*`：把进入 agent 的消息切成哪个 session

再补一句最关键的：官方文档目前**确实有点自相矛盾**——示例页还在展示 `agent:` 单数写法，但多 Agent 文档已经明确推荐 `agents.defaults + agents.list`，并说明 legacy `agent.*` 会被迁移。你被搞晕，不是错觉，是官方信息面本来就有历史混杂。([OpenClaw](https://docs.openclaw.ai/gateway/configuration-examples "Configuration Examples - OpenClaw"))

把你的 `openclaw.json` 贴出来，我可以按这个模型直接帮你逐段标注：哪部分是 agent，哪部分是 channel/account，哪部分是 binding，哪部分是 session。