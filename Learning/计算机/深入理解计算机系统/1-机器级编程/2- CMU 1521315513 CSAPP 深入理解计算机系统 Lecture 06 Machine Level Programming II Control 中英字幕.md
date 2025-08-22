---
title: "CMU 15213/15513 CSAPP 深入理解计算机系统 Lecture 06 Machine Level Programming II Control 中英字幕"
source: "https://www.youtube.com/watch?v=LqSN8OOdLQw&list=PL22J-I2Pi-Gf0s1CGDVtt4vuvlyjLxfem&index=7&ab_channel=RobbieZhou"
author:
  - "[[Robbie Zhou]]"
published: 2020-09-26
created: 2025-01-18
description:
tags:
  - "clippings"
---
加法和减法指令相当于在 C 语言中只使用 `+=`, `-=`
## 比较指令
`cmp` 指令比较两个寄存器  
`test` 指令虽然有两个操作数, 但一般用来测试一个寄存器, 例如 `test %rax, %rax`
有一个奇怪的规定, 对寄存器进行 4 字节的操作会自动将高 4 字节设为 0, 但是非 4 字节的操作不会影响别的位
![[Pasted image 20250118193150.png]]
如图, `setg` 根据符号位将目标寄存器设为 0 或 1, `movzbl` 将 `al` 的零拓展到 4 字节, 然会 cpu 会自动将更高 4 位页设为 0
## 条件控制
`jmp` 指令: 分为无条件跳转和有条件跳转: ![[Pasted image 20250118193703.png]]
### 条件跳转和条件移动
条件移动是执行两种情况并比较, 其实从代码层面上看不出为什么更高效, 要结合硬件, 条件移动的代码是顺序进行的, 在流水线中可以更好地并行执行, 并且减少了条件预测误差
下面是使用跳转表
### `switch`
  ![[Pasted image 20250118200730.png]]
   ![[Pasted image 20250118200933.png]]
   ![[Pasted image 20250118201434.png]]
   如上 `ja`, 之所以不使用 `jg`, 是因为 `ja` 涵盖了 ` rdi ` 大于 6 或小于 0 的情况
   `*.L4` 是 JumpTable, 每隔 8 个字节存储一个地址, JumpTarget 是多个函数, 每个函数 
   ```ad-col2
   ![[Pasted image 20250118202201.png]]
   
   ![[Pasted image 20250118202657.png]]
```
  不管要判断的值是多少, 比如 switch 分支判断中出现了负数, 它会加上偏置使得条件从 case 0 开始
   另一个如果值的范围很大，并且相对稀疏,用到的值都会被转变成if-else代码