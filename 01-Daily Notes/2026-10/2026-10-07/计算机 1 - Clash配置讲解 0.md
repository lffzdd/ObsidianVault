Clash 的配置文件==采用 YAML 语法，默认主配置文件名为== `config.yaml`。一个完整的 Clash 配置文件通常由五个核心部分组成：常规基础设置、DNS 配置、代理节点（Proxies）、策略组（Proxy Groups）以及分流规则（Rules）。 [1, 2, 3, 4] 

以下是一个标准且包含中文注释的 Clash 配置文件模板，您可以根据自己的节点信息直接修改使用：

## 🛠️ Clash 核心配置文件模板

```yaml
#============================================================
# 1. 常规基础设置
#============================================================
mixed-port: 7890             # 混合端口（同时支持 HTTP 和 SOCKS5 协议）
allow-lan: true              # 允许局域网设备连接
mode: rule                   # 运行模式：rule（规则分流）, global（全局代理）, direct（直接连接）
log-level: info              # 日志级别：info, warning, error, debug
ipv6: false                  # 是否开启 IPv6 支持
external-controller: '127.0.0.1:9090' # 外部控制 API 端口（供面板连接）
secret: ''                   # 外部控制 API 的密码

#============================================================
# 2. DNS 高级配置（防止 DNS 泄漏）
#============================================================
dns:
  enable: true
  ipv6: false
  listen: 0.0.0.0:53
  enhanced-mode: redir-host # 或 fake-ip
  fake-ip-range: 198.18.0.1/16
  default-nameserver:
    - 119.29.29.29
    - 223.5.5.5
  nameserver:
    - https://alidns.com
    - https://doh.pub
  fallback:
    - https://cloudflare-dns.com
    - https://dns.google

#============================================================
# 3. 代理节点（填写您的实际节点信息）
#============================================================
proxies:
  - name: "香港 Shadowsocks 节点"
    type: ss
    server: ://example.com
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "your_password"

  - name: "美国 Shadowsocks 节点"
    type: ss
    server: ://example.com
    port: 443
    cipher: chacha20-ietf-poly1305
    password: "your_password"

  - name: "日本 VMess 节点"
    type: vmess
    server: ://example.com
    port: 443
    uuid: "your_uuid"
    alterId: 0
    cipher: auto
    tls: true

#============================================================
# 4. 策略组（决定流量如何分配）
#============================================================
proxy-groups:
  # 代理自动选择：自动测试延迟并切换到最快的节点
  - name: "🚀 节点选择"
    type: select
    proxies:
      - "⚡ 自动选择"
      - "香港 Shadowsocks 节点"
      - "美国 Shadowsocks 节点"
      - "日本 VMess 节点"
      - "DIRECT"

  # 自动测速组
  - name: "⚡ 自动选择"
    type: url-test
    url: http://gstatic.com
    interval: 300
    tolerance: 50
    proxies:
      - "香港 Shadowsocks 节点"
      - "美国 Shadowsocks 节点"
      - "日本 VMess 节点"

  # 媒体流媒体专用策略
  - name: "🎬 国际媒体"
    type: select
    proxies:
      - "🚀 节点选择"
      - "香港 Shadowsocks 节点"
      - "日本 VMess 节点"
      - "DIRECT"

#============================================================
# 5. 分流规则（从上到下按顺序匹配）
#============================================================
rules:
  # 局域网和本地流量直接连接
  - DOMAIN-SUFFIX,local,DIRECT
  - IP-CIDR,127.0.0.0/8,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT

  # 常见国内流媒体或网站直连
  - DOMAIN-SUFFIX,bilibili.com,DIRECT
  - DOMAIN-SUFFIX,baidu.com,DIRECT

  # 广告拦截
  - DOMAIN-KEYWORD,adserver,REJECT

  # 国外流媒体走媒体策略
  - DOMAIN-SUFFIX,netflix.com,🎬 国际媒体
  - DOMAIN-SUFFIX,youtube.com,🎬 国际媒体

  # 境外常用网站走节点选择
  - DOMAIN-SUFFIX,google.com,🚀 节点选择
  - DOMAIN-SUFFIX,github.com,🚀 节点选择
  - DOMAIN-KEYWORD,twitter,🚀 节点选择

  # 最终规则：GeoIP 中国区直连，其余走代理
  - GEOIP,CN,DIRECT
  - MATCH,🚀 节点选择
```


Clash 配置文件中的 `mode` 参数用于决定流量如何被路由。它有三个可选值：`Rule`（规则模式）、`Global`（全局模式）和 `Direct`（直连模式）。在 YAML 文件中通常不区分大小写。

### 🎯 三种模式详解

| 模式 | 名称 | 作用 |
| :--- | :--- | :--- |
| **Rule** | 规则模式 | **最常用**。Clash 会根据配置文件中定义的 `rules` 字段逐条匹配，决定流量走代理、直连还是拒绝。 |
| **Global** | 全局模式 | 所有流量都会**强制通过代理**转发，忽略所有 `rules` 规则。 |
| **Direct** | 直连模式 | 所有流量都**不经过代理**，直接连接目标服务器，相当于关闭了代理功能。 |

### 📝 如何配置

在你的 `config.yaml` 文件中，通常位于文件顶部：

```yaml
# 可选值: rule, global, direct
mode: rule
```

### 💡 补充说明

- **默认值**：如果配置文件中没有明确指定 `mode`，Clash 默认会使用 **`rule`** 模式。
- **枚举映射**：在 Clash 的源码实现中，这三种模式对应的字符串映射是 `"Global"`、`"Rule"` 和 `"Direct"`。
- **其他含义**：极少数情况下，`mode` 也可能出现在其他上下文中（如 `dns.enhanced-mode`），但那属于 DNS 的解析模式，与这里的流量路由模式不同。