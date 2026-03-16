IPC,Inter-Process Communication（进程间通信）

[[01-Daily Notes/2026-03-13/attachments/657ecbfe65cb0f6926314f93563827d9_MD5.mp4|Open: PixPin_2026-03-13_21-31-10.mp4]]
![[01-Daily Notes/2026-03-13/attachments/657ecbfe65cb0f6926314f93563827d9_MD5.mp4]]

[[01-Daily Notes/2026-03-13/attachments/94af31a475fe3c1ca38c57b09e1017f4_MD5.png|Open: Pasted image 20260313213703.png]]
![[01-Daily Notes/2026-03-13/attachments/94af31a475fe3c1ca38c57b09e1017f4_MD5.png]]

PCB

[[01-Daily Notes/2026-03-13/attachments/a353ef57273e4e8cb509e234733544a2_MD5.png|Open: Pasted image 20260313214902.png]]
![[01-Daily Notes/2026-03-13/attachments/a353ef57273e4e8cb509e234733544a2_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/ca7690cca75b409ee20d632b42f9a16c_MD5.png|Open: Pasted image 20260313220358.png]]
![[01-Daily Notes/2026-03-13/attachments/ca7690cca75b409ee20d632b42f9a16c_MD5.png]]

并行计算
复杂的任务可以被划分为更小的任务，并可同时执行

[[计算机 1.3 - 并行计算]]
[[计算机 1.4 - 模块化]]

# 两种方式

有两种方式：
1. **共享内存（Shared Memory）**：多个进程访问同一块内存区域来交换数据。
2. **消息传递（Message Passing）**：进程通过发送和接收

[[01-Daily Notes/2026-03-13/attachments/b6203887a9e9d262b32254d23c283070_MD5.png|Open: Pasted image 20260313224747.png]]
![[01-Daily Notes/2026-03-13/attachments/b6203887a9e9d262b32254d23c283070_MD5.png]]

## 共享内存

进程的内存空间是相互隔离的，不能直接访问对方的内存。

[[01-Daily Notes/2026-03-13/attachments/4671b80b344439d8a6f74247625cbded_MD5.png|Open: Pasted image 20260313225020.png]]
![[01-Daily Notes/2026-03-13/attachments/4671b80b344439d8a6f74247625cbded_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/bc0afa59bb058a7121768b4d19b44d26_MD5.png|Open: Pasted image 20260313225118.png]]
![[01-Daily Notes/2026-03-13/attachments/bc0afa59bb058a7121768b4d19b44d26_MD5.png]]

需要使用系统调用来申请共享内存或者发送消息

共享内存区域被创建以后,这块区域由进程去管理
[[../../copilot/copilot-conversations/共享内存被创建以后,是不是由进程去管理,这块内存是真的挨@20260313_225529|共享内存被创建以后,是不是由进程去管理,这块内存是真的挨@20260313_225529]]

[[01-Daily Notes/2026-03-13/attachments/cbe2ff4576ac54ef45b691d61ccbdaaf_MD5.mp4|Open: PixPin_2026-03-13_22-59-45.mp4]]
![[01-Daily Notes/2026-03-13/attachments/cbe2ff4576ac54ef45b691d61ccbdaaf_MD5.mp4]]

[[01-Daily Notes/2026-03-13/attachments/168984d3c4f561cb6d4356a0634c2b8b_MD5.png|Open: Pasted image 20260316151816.png]]
![[01-Daily Notes/2026-03-13/attachments/168984d3c4f561cb6d4356a0634c2b8b_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/6be34ce637f41f2aef8ad21d6f9ad94b_MD5.png|Open: Pasted image 20260316151947.png]]
![[01-Daily Notes/2026-03-13/attachments/6be34ce637f41f2aef8ad21d6f9ad94b_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/b728a8760cefe32ec03bfe4dc7a16fe6_MD5.png|Open: Pasted image 20260316154803.png]]
![[01-Daily Notes/2026-03-13/attachments/b728a8760cefe32ec03bfe4dc7a16fe6_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/5b9bde792de5c7a9a51d91e366337eaf_MD5.png|Open: Pasted image 20260316154836.png]]
![[01-Daily Notes/2026-03-13/attachments/5b9bde792de5c7a9a51d91e366337eaf_MD5.png]]

## 消息传递

操作系统提供的一种方式，让一个进程可以向另一个进程发送消息（Message），并且等待对方回复。在不共享相同地址空间的情况下同步操作，地址空间可以保持隔离

消息不用写进内存，直接通过操作系统内核进行传递

[[01-Daily Notes/2026-03-13/attachments/95a799aea80c8b4a574737bc5b34669b_MD5.png|Open: Pasted image 20260316155213.png]]
![[01-Daily Notes/2026-03-13/attachments/95a799aea80c8b4a574737bc5b34669b_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/a2889a39f5e68e27bb4c8c42d0d5300e_MD5.png|Open: Pasted image 20260316155413.png]]
![[01-Daily Notes/2026-03-13/attachments/a2889a39f5e68e27bb4c8c42d0d5300e_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/a2889a39f5e68e27bb4c8c42d0d5300e_MD5.png|Open: Pasted image 20260316155406.png]]
![[01-Daily Notes/2026-03-13/attachments/a2889a39f5e68e27bb4c8c42d0d5300e_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/da1506dfeeb2b51f59c7e9202f90b1aa_MD5.png|Open: Pasted image 20260316155832.png]]
![[01-Daily Notes/2026-03-13/attachments/da1506dfeeb2b51f59c7e9202f90b1aa_MD5.png]]

全双工

[[01-Daily Notes/2026-03-13/attachments/1b286afee1684479e7d1b8ef8b0bacde_MD5.png|Open: Pasted image 20260316160006.png]]
![[01-Daily Notes/2026-03-13/attachments/1b286afee1684479e7d1b8ef8b0bacde_MD5.png]]

mailbox 位于内核空间，用户进程无法直接访问

[[01-Daily Notes/2026-03-13/attachments/ff02aa3633ed6a64e12467a2c0a24329_MD5.png|Open: Pasted image 20260316160042.png]]
![[01-Daily Notes/2026-03-13/attachments/ff02aa3633ed6a64e12467a2c0a24329_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/44e8aa0f13adb7abc4645667fbb90360_MD5.png|Open: Pasted image 20260316160204.png]]
![[01-Daily Notes/2026-03-13/attachments/44e8aa0f13adb7abc4645667fbb90360_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/8614bd7cc68d8ef1b266fc1e62a05791_MD5.png|Open: Pasted image 20260316160226.png]]
![[01-Daily Notes/2026-03-13/attachments/8614bd7cc68d8ef1b266fc1e62a05791_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/42d5948b9eb9cea3850a20cc9d7b6efa_MD5.png|Open: Pasted image 20260316160706.png]]
![[01-Daily Notes/2026-03-13/attachments/42d5948b9eb9cea3850a20cc9d7b6efa_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/63a8327ae58ced717d8c86c8cd8ee54d_MD5.png|Open: Pasted image 20260316160839.png]]
![[01-Daily Notes/2026-03-13/attachments/63a8327ae58ced717d8c86c8cd8ee54d_MD5.png]]

### 监听端口

[[01-Daily Notes/2026-03-13/attachments/15120010f151d758ac3b8a5a2356dbcd_MD5.png|Open: Pasted image 20260316160949.png]]
![[01-Daily Notes/2026-03-13/attachments/15120010f151d758ac3b8a5a2356dbcd_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/94cf93d3c01b5e1ffcad8c33bf47894f_MD5.png|Open: Pasted image 20260316161003.png]]
![[01-Daily Notes/2026-03-13/attachments/94cf93d3c01b5e1ffcad8c33bf47894f_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/381f2cd09db910aced3743c2ec22b5dd_MD5.png|Open: Pasted image 20260316161059.png]]
![[01-Daily Notes/2026-03-13/attachments/381f2cd09db910aced3743c2ec22b5dd_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/f3633714b3626d816ed5bdbaee2cfa7c_MD5.png|Open: Pasted image 20260316161233.png]]
![[01-Daily Notes/2026-03-13/attachments/f3633714b3626d816ed5bdbaee2cfa7c_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/5730d5c0746ef9d2ca2cfe82aa1dd3c4_MD5.png|Open: Pasted image 20260316161351.png]]
![[01-Daily Notes/2026-03-13/attachments/5730d5c0746ef9d2ca2cfe82aa1dd3c4_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/8a390d22b0803a67f9c8ab4fc3746f06_MD5.png|Open: Pasted image 20260316161430.png]]
![[01-Daily Notes/2026-03-13/attachments/8a390d22b0803a67f9c8ab4fc3746f06_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/e28228753b7eb7775453d97369466726_MD5.png|Open: Pasted image 20260316161514.png]]
![[01-Daily Notes/2026-03-13/attachments/e28228753b7eb7775453d97369466726_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/04a55390c7ac96098bdb085583aed03a_MD5.png|Open: Pasted image 20260316161528.png]]
![[01-Daily Notes/2026-03-13/attachments/04a55390c7ac96098bdb085583aed03a_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/ae5b5ed3fbddb581ca52a35910a72188_MD5.png|Open: Pasted image 20260316161614.png]]
![[01-Daily Notes/2026-03-13/attachments/ae5b5ed3fbddb581ca52a35910a72188_MD5.png]]

### 缺点

[[01-Daily Notes/2026-03-13/attachments/3a322dc0758ad0ea970ed132fc632a15_MD5.png|Open: Pasted image 20260316161809.png]]
![[01-Daily Notes/2026-03-13/attachments/3a322dc0758ad0ea970ed132fc632a15_MD5.png]]

[[01-Daily Notes/2026-03-13/attachments/d016d4f4419c0eafe84ea91fe7e7925b_MD5.png|Open: Pasted image 20260316162102.png]]
![[01-Daily Notes/2026-03-13/attachments/d016d4f4419c0eafe84ea91fe7e7925b_MD5.png]]

消息传递每次都要经过内核，效率较低

而共享内存不需要经过内核，效率较高，但需要进程自己去管理内存的访问和同步，比较麻烦

不过 99% 的情况下,消息传递就够了,共享内存的效率优势并不明显,而且更麻烦

[[01-Daily Notes/2026-03-13/attachments/c1246cb853f53b72a3a0fdc6e712d7ba_MD5.png|Open: Pasted image 20260316162204.png]]
![[01-Daily Notes/2026-03-13/attachments/c1246cb853f53b72a3a0fdc6e712d7ba_MD5.png]]