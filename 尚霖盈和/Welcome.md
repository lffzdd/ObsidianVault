### 如何在多个 Dify 应用中指定调用正确的那个

你提到有多个应用，想知道 cURL 请求如何确保调用你想要的那个——这点官方文档确实没强调，但核心机制是通过 **API 密钥（API Key）** 来区分的。Dify 的 API 端点（如 `/chat-messages` 或 `/completion-messages`）是通用的（共享的），但每个应用都有自己独特的 API 密钥。使用哪个密钥，请求就会路由到对应的应用，不会混淆。 这就像每个应用有自己的“门钥匙”，密钥绑定了应用 ID，确保隔离。

#### 为什么这样设计？
- Dify 的 API 是基于应用的：当你为一个应用生成密钥时，它内部关联了该应用的 ID、工作流配置和变量。 通用端点简化了调用，但密钥提供安全性与路由。
- 如果你有多个应用（如 App A 是聊天机器人，App B 是总结工具），每个在 “API Access” 页面生成的密钥都是独立的。请求时带上 App A 的密钥，就只会执行 App A 的逻辑。

#### 如何操作：一步步确认和使用
1. **为每个应用生成专属密钥**：
   - 登录 Dify 控制台，进入 “Apps” > 选择特定应用（比如你想要的那个） > 左侧导航 “API Access”。
   - 点击 “Create Credential” 或类似按钮，生成新密钥（格式如 `app-xxxxxxxxxxxxxxxx`）。
   - 每个应用可以生成多个密钥（用于不同用户或环境），但所有这些密钥都绑定到这个应用，不会跨应用。 如果密钥泄露，可以在这里删除或再生。

2. **在 cURL 请求中使用密钥**：
   - 端点不变（如 `https://api.dify.ai/v1/chat-messages`），但在头部用 `Authorization: Bearer YOUR_SPECIFIC_APP_KEY`。
   - 示例（假设你有两个应用，密钥分别是 KEY_FOR_APP_A 和 KEY_FOR_APP_B）：
     - 调用 App A：
       ```
       curl --location --request POST 'https://api.dify.ai/v1/chat-messages' \
       --header 'Authorization: Bearer KEY_FOR_APP_A' \
       --header 'Content-Type: application/json' \
       --data-raw '{
           "inputs": {},
           "query": "Hello",
           "response_mode": "streaming",
           "conversation_id": "",
           "user": "user-123"
       }'
       ```
     - 调用 App B：只需换成 `Bearer KEY_FOR_APP_B`，其他不变。Dify 后端会根据密钥自动路由到正确的应用。

3. **验证是否正确**：
   - 在 “API Access” 页面的 “Try It” 或 Swagger 界面，直接用该应用的密钥测试请求，看响应是否匹配应用的逻辑（比如输出是否是该应用的 Prompt 生成的）。
   - 如果自部署，检查日志（在服务器控制台或 Docker logs）确认请求的 app_id。
   - 注意：密钥是唯一的，如果用错密钥，会返回 401 Unauthorized 或无效应用错误。

#### 常见问题和注意事项
- **如果密钥不对**：请求会失败，不会调用其他应用——这是安全设计。
- **多版本或环境**：如果你的多个应用是同一个的变体，Dify 支持多版本路由，但默认是基于密钥。 对于高级场景（如生产/测试环境），可以复制应用生成新密钥。
- **集成到其他工具**：在代码（如 Python requests）或工具（如 Postman）中，同样用特定密钥。不要共享密钥，避免滥用。
- **限额和监控**：每个应用的调用限额独立，付费版可升级。

这样就清楚了：密钥就是“应用选择器”。如果你在控制台查看，每个应用的 API 页面都会显示它的专属文档和密钥。 如果还是有问题，建议去 Dify GitHub Issues 搜 “multiple apps API” 或官方社区问。