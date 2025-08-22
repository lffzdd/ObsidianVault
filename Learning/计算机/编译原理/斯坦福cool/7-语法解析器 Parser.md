---
aliases: 7-语法解析器 Parser
date created: 星期四, 四月 24日 2025, 12:45:42 下午
date modified: 星期四, 四月 24日 2025, 12:52:44 下午
---

自动机无法识别字符串的长度, 无法识别嵌套
![[Pasted image 20250424125251.png]]
# 上下文无关文法 (Context-Free Grammar)
![[Pasted image 20250425172157.png]]
一个 CFG 由一个四元组 (T, N, S, P) 定义:
- T: terminal symbols, 终结符号
- N: non-terminal symbols, 非终结符号
- S: start symbol, 开始符号, $S\in N$
- P: production rules, 产生式:$X\to Y_{1},Y_{2},\cdots,Y_{n}$, 
	- $X\in N$, $Y_{i}\in T\cup N\cup \epsilon$
	- 产生式的左侧是一个非终结符号
	- 产生式的右侧是一个终结符号和非终结符号的任意组合
	- 产生式的右侧可以是空串 $\epsilon$
==产生式可以理解为替换规则, 例如 $s\to (s)$ 表示只要在字符串中找到一个 $s$ 就可以替换成 $(s)$==

对一个字符串, 不断地使用产生式进行替换, 直到字符串全都为终结符号为止

![[Pasted image 20250425182921.png]]
![[Pasted image 20250425183450.png]]
# 分析树
![[Pasted image 20250426105020.png]]
![[Pasted image 20250426105105.png]]
![[Pasted image 20250426105752.png]]

- 叶子节点: 终结符
- 非叶子节点: 非终结符
- 中序遍历会得到原始输入

语法树由推导产生, 上述例子中采用的是最左推导 (leftmost derivation), 每一次替换都是替换最左边的非终结符

# Ambiguity: 语法歧义, 语法树不唯一
最左|最右推导: ![[Pasted image 20250426111207.png]]
解决歧义的方法:
- 重新定义文法 ![[Pasted image 20250426112407.png]]
	- 图中将产生式分为了两类: ![[Pasted image 20250426112910.png]]
		- 1. $E\to E'+E|E'$ : 负责加法
		- 2. $E'\to id*E'|id|(E)*E'|(E)$: 负责乘法