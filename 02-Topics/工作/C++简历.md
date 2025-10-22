---
aliases: C++简历
date created: 星期二, 七月 1日 2025, 11:18:22 上午
date modified: 星期二, 七月 1日 2025, 2:18:39 下午
---
>  刘非凡 | 男 / 24 岁
### **教育背景**
- 中南林业科技大学 | 硕士在读，电子信息（2024.09 - 2027.06）
- 中南林业科技大学 | 本科，信息与计算科学（2020.09 - 2024.06）
### **职业技能**
- 熟练掌握 C++ / Python，掌握面向对象与常见数据结构（链表、树、排序等）
- 掌握 Linux 系统编程，精通 socket / epoll / 多线程与状态机模型
- 熟悉 CMake 构建、GDB 调试、Git 协作，了解 MySQL/MongoDB、TCP/IP 协议栈
- 具备 x86-64 汇编与系统安全机制分析能力（Canary、NX）
- 熟悉 PyTorch 框架，掌握图像分类 / 目标检测等 AI 模型训练与调优
# 项目经历
##### **高性能代理服务器 | C++ / C / epoll / OpenSSL**
> 技术栈：Linux、 epoll、 socket、 OpenSSL、 HTTP、 TCP/IP
- 开发支持 HTTP/HTTPS 的高并发代理服务器，基于 epoll 实现事件驱动模型；
- 状态机精准管理连接生命周期，支持非阻塞通信、SSL 加密、CONNECT 隧道；
- 模块包括 socket 封装、HTTP 解析、状态切换与事件分发；
- 支持 curl/wget 测试，构建跨平台自动化编译（CMake）；
- 项目地址：[github.com/lffzdd/ProxyWebServer](https://github.com/lffzdd/ProxyWebServer)
##### **二进制漏洞利用与逆向分析 | C / 汇编 / GDB**
> 技术栈: x86-64 、汇编、 GDB
- 手动分析栈帧，构造 payload 实现函数劫持与控制流重定向；
- 绕过 Canary、NX 等机制，完成自动化调试与复现；
- 编写测试脚本，深入理解 ELF 格式与 GCC 编译流程
##### **道路病害检测系统 | YOLO + PyTorch**
> 技术栈：YOLO、PyTorch、OpenCV、TensorBoard、pandas、matplotlib
- 基于 RDD 2022 构建端到端目标检测系统，覆盖坑洞、裂缝等多类病害；
- 使用 Albumentations 增强图像多样性，结合 TensorBoard 可视化训练过程；
- 模型调参后 mAP\@0.5 达 92.2%，大幅优于 baseline
### **实习经历**
##### **IoT 教学 | 2025.02 - 2025.05**
- 负责传感器与通信协议（MQTT/HTTP）教学，组织学生完成嵌入式项目；
- 指导使用 Python 编写设备通信程序，模拟多端通信场景，搭建完整 IoT 应用链路