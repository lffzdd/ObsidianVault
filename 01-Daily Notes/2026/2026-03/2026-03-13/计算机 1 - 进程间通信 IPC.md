IPC,Inter-Process Communication（进程间通信）

[[attachments/657ecbfe65cb0f6926314f93563827d9_MD5.mp4|Open: PixPin_2026-03-13_21-31-10.mp4]]
![[attachments/657ecbfe65cb0f6926314f93563827d9_MD5.mp4]]

[[attachments/94af31a475fe3c1ca38c57b09e1017f4_MD5.png|Open: Pasted image 20260313213703.png]]
![[attachments/94af31a475fe3c1ca38c57b09e1017f4_MD5.png]]

PCB

[[attachments/a353ef57273e4e8cb509e234733544a2_MD5.png|Open: Pasted image 20260313214902.png]]
![[attachments/a353ef57273e4e8cb509e234733544a2_MD5.png]]

[[attachments/ca7690cca75b409ee20d632b42f9a16c_MD5.png|Open: Pasted image 20260313220358.png]]
![[attachments/ca7690cca75b409ee20d632b42f9a16c_MD5.png]]

并行计算
复杂的任务可以被划分为更小的任务，并可同时执行

[[计算机 1.3 - 并行计算]]
[[计算机 1.4 - 模块化]]

# 两种方式

有两种方式：
1. **共享内存（Shared Memory）**：多个进程访问同一块内存区域来交换数据。
2. **消息传递（Message Passing）**：进程通过发送和接收

[[attachments/b6203887a9e9d262b32254d23c283070_MD5.png|Open: Pasted image 20260313224747.png]]
![[attachments/b6203887a9e9d262b32254d23c283070_MD5.png]]

进程的内存空间是相互隔离的，不能直接访问对方的内存。

[[attachments/4671b80b344439d8a6f74247625cbded_MD5.png|Open: Pasted image 20260313225020.png]]
![[attachments/4671b80b344439d8a6f74247625cbded_MD5.png]]

[[attachments/bc0afa59bb058a7121768b4d19b44d26_MD5.png|Open: Pasted image 20260313225118.png]]
![[attachments/bc0afa59bb058a7121768b4d19b44d26_MD5.png]]

需要使用系统调用来申请共享内存或者发送消息

共享内存区域被创建以后,这块区域由进程去管理
[[../../../../copilot/copilot-conversations/共享内存被创建以后,是不是由进程去管理,这块内存是真的挨@20260313_225529|共享内存被创建以后,是不是由进程去管理,这块内存是真的挨@20260313_225529]]

[[attachments/cbe2ff4576ac54ef45b691d61ccbdaaf_MD5.mp4|Open: PixPin_2026-03-13_22-59-45.mp4]]
![[attachments/cbe2ff4576ac54ef45b691d61ccbdaaf_MD5.mp4]]

