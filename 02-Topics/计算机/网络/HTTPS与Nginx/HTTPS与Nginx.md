---
type: 入口笔记
---
# HTTPS与Nginx

## 当前理解

HTTPS = HTTP + TLS，证书由 CA 签发（自动 / 手动申请）；Nginx 负责反向代理、证书挂载和多域名访问，配置文件按目录组织。

## 笔记（按时间）

- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 CA证书申请|2025-12 CA证书申请]] — 讲了：一、 CA (Certificate Autho / 1. 核心原理与角色 / 2. 历史背景：昂贵的“保护费”
- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 CA证书申请 - 手动|2025-12 CA证书申请 - 手动]] — 讲了：第一步：只拿证，不装修 ( certonly   / 1. 发起申请 / 2. 面对“挑战” (The Challenge
- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 CA证书申请 - 自动|2025-12 CA证书申请 - 自动]] — 讲了：⚠️ 动手前的必须准备 / 第一阶段：安装 Certbot (推荐使用 Sn / 1. 确保 Snap 环境是新的
- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 HTTPS|2025-12 HTTPS]] — 讲了：一、 核心痛点：HTTP 的“裸奔”时代 / 1. 原理层面的缺陷 / 二、 历史与发展：从 Netscape 到全网加
- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 nginx|2025-12 nginx]] — 讲了：1. 历史背景：C10k 问题与“旧时代的终结” / 2. 核心原理：自底向上的架构解析 / Level 1: 内核层的变革 (IO Mult
- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 nginx 的使用|2025-12 nginx 的使用]] — 讲了：使用 / 第一步：安装与验证 / 第二步：掌握“驾驶舱”指令
- [[02-Topics/计算机/网络/HTTPS与Nginx/2025-12 nginx 的配置文件夹说明|2025-12 nginx 的配置文件夹说明]] — 讲了：1. 底层原理：一切皆由  include  决 / 2. 为什么分文件夹？这是“管理哲学”的区别 / 派系 A：简单粗暴流 ( conf.d )
