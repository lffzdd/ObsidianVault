---
type: 入口笔记
---
# Clash配置

## 当前理解

Clash 配置由 dns/nameserver、proxies（节点）、proxy group（选择策略）和 rules（分流规则）组成，决定每条连接走哪个出口。

## 笔记（按时间）

- [[02-Topics/计算机/网络/代理/Clash配置/2026-10-07 6 rules|2026-10-07 6 rules]] — 讲了：第一步： rules  的基本长相与格式 / 1.  DOMAIN-SUFFIX （域名后缀匹 / 2.  DOMAIN-KEYWORD （域名关键
- [[02-Topics/计算机/网络/代理/Clash配置/2026-10-07 7 claude|2026-10-07 7 claude]] — 讲了：先把两个“DNS 解析”分开 / fake-ip 模式下到底发生了什么 / 你的质疑成立的地方
- [[02-Topics/计算机/网络/代理/Clash配置/2026-10-07 Clash配置讲解 0|2026-10-07 Clash配置讲解 0]] — 讲了：🛠️ Clash 核心配置文件模板 / 🎯 三种模式详解 / 📝 如何配置
- [[02-Topics/计算机/网络/代理/Clash配置/2026-10-07 Clash配置讲解 1 - dns|2026-10-07 Clash配置讲解 1 - dns]] — 讲了：enable / 第一步：最基础的开关  enable / enhanced-mode
- [[02-Topics/计算机/网络/代理/Clash配置/2026-10-07 proxies|2026-10-07 proxies]] — 讲了：第一步： proxies  的基本长相 / 1.  name （节点名称） / 2.  type （协议类型/核心内核）
- [[02-Topics/计算机/网络/代理/Clash配置/2026-10-07 proxy group|2026-10-07 proxy group]] — 讲了：第一步：策略组的基本长相 / 1.  name: 🚀 节点选择 / 2.  proxies:
