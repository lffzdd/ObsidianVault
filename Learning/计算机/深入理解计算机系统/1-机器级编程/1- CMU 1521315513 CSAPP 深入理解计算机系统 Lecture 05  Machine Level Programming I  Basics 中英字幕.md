---
aliases: 1- CMU 1521315513 CSAPP 深入理解计算机系统 Lecture 05  Machine Level Programming I  Basics 中英字幕
title: "CMU 15213/15513 CSAPP 深入理解计算机系统 Lecture 05  Machine Level Programming I  Basics 中英字幕"
source: "https://www.youtube.com/watch?v=ViP6V-U4y8M&list=PL22J-I2Pi-Gf0s1CGDVtt4vuvlyjLxfem&index=5&ab_channel=RobbieZhou"
author:
  - '[[Robbie Zhou]]'
published: 2020-09-26
created: 2025-01-17
description:
tags: ["clippings"]
date created: Friday, January 17th 2025, 1:59:01 pm
date modified: Friday, January 17th 2025, 3:45:27 pm
---

> [CMU 15213/15513 CSAPP 深入理解计算机系统 Lecture 05  Machine Level Programming I  Basics 中英字幕 - YouTube](https://www.youtube.com/watch?v=ViP6V-U4y8M&list=PL22J-I2Pi-Gf0s1CGDVtt4vuvlyjLxfem&index=5&ab_channel=RobbieZhou)

##
前二十五分钟描述了一些历史
## 代码编译过程
![[Pasted image 20250117140234.png]]
## `mov`
### Src 和 Dest
![[Pasted image 20250117152818.png]]
出于硬件设计原因, 不能从 memory 移动到 memory (看看后面的存储器结构就知道了)
![[Pasted image 20250117153035.png]]
当你将寄存器的名称放在括号中时, 就是将寄存器中的值作为地址
### 函数参数寄存器
![[Pasted image 20250117153449.png]]
事实证明、使用（64）x86-64、函数参数总是出现在某些特定的寄存器中
`rdi`: 第一个
`rsi`: 第二个
![[Pasted image 20250117155840.png]]

### 寻址指令
![[Pasted image 20250117154203.png]]
`Rb`: R 是寄存器, b 是 base, 称为基址, 可以是任何 16 为整数寄存器
`Ri`: i 是 index, 索引, 可以是除 rsp 的所有寄存器
`S`: scale 尺度
之所以这么写, 是因为这是实现数组引用的一种自然方式
如果它是一个 int,必须将索引 l 缩放四倍，如果它是 long，我必须将其缩放八倍
![[Pasted image 20250117154557.png]]
有以上这些精简形式
### `lea`
一个让你困惑的指令
![[Pasted image 20250117155020.png]]
**它的目标是利用 C 的 & 符号操作来计算地址**, 当也是一种非常方便的算数运算方式, C 编译器喜欢使用
特别是，看起来它的格式看起来像一个有源和目的地的mov指令, 但目的必须是寄存器, 源就是上面寻址格式
它实际上写入寄存器的是地址值
## 其它指令
![[Pasted image 20250117155425.png]]
注意算数移位和逻辑移位, 对应有符号和无符号
![[Pasted image 20250117155624.png]] 
![[Pasted image 20250117155722.png]]
计算被转换为了 lea 和移位,编译器不断的尝试去找到更聪明的汇编实现方式