---
type: 入口笔记
---
# Python

## 当前理解

Python 的关键机制：导包按 sys.path 查找（包、子包、循环导入）、with/contextmanager 基于 yield 暂停、for 基于迭代器协议、int 是无限精度对象（因此较慢）、错误处理类型的选择，以及 pydantic、dataclass、TypedDict 等类型工具。

## 笔记（按时间）

- [[02-Topics/计算机/编程语言/Python/2025-11 REPL|2025-11 REPL]] — 讲了：什么是REPL / 一、REPL 是什么？ / 二、为什么会有 REPL？
- [[02-Topics/计算机/编程语言/Python/2025-11 pydantic|2025-11 pydantic]] — 讲了：一、起点：Python 的“信任型”数据哲学 / 二、第一阶段：类型注解只是“提示” / 三、Pydantic 登场：把“类型提示”变成“
- [[02-Topics/计算机/编程语言/Python/2025-11 python 中的 logging|2025-11 python 中的 logging]] — 讲了：一、Python 为什么要做 logging？ / 1. 历史背景：print 已经不够用 / 二、logging 的总体架构
- [[02-Topics/计算机/编程语言/Python/2025-11 python 导包方法|2025-11 python 导包方法]] — 讲了：解决方法 / 一、最原始的做法：手动把目录塞进 sys.pat / 二、再进化一点：通过环境变量 PYTHONPAT
- [[02-Topics/计算机/编程语言/Python/2025-11 python 魔术方法|2025-11 python 魔术方法]] — 讲了：一、Python 的类为什么会有这些“奇怪的双下 / 二、为什么 Python 要设计这样一套机制？ / 三、这些方法为什么都用双下划线？
- [[02-Topics/计算机/编程语言/Python/2025-11 包和路径|2025-11 包和路径]] — 讲了：🧩   核心结论（可以先记住） / 🧠   为什么用  -m  才能找到真正的包结构 / 🚫   情况 A：你直接运行脚本
- [[02-Topics/计算机/编程语言/Python/2025-11 底层导包机制|2025-11 底层导包机制]] — 讲了：一、module 在 Python 里到底是什么 / 二、 __name__  和  __packag / 三、相对导入到底是怎么用  __package_
- [[02-Topics/计算机/编程语言/Python/2025-11-05 python导包机制|2025-11-05 python导包机制]] — 讲了：一、核心规律：Python 只从  sys.pa / 二、举例讲透路径行为 / 场景 1：从 project 目录运行
- [[02-Topics/计算机/编程语言/Python/2025-11-17 python导包机制|2025-11-17 python导包机制]] — 讲了：演化历史 / 🧩 一、为什么 Python 的导包机制让人困惑 / 🧭 二、Python 导包的底层逻辑：它不是“看
- [[02-Topics/计算机/编程语言/Python/2025-12 contextmanager|2025-12 contextmanager]] — 讲了：1. 历史背景与痛点 (PEP 343) / 痛点：太麻烦了 / 2. 核心原理：从  yield  到  __e
- [[02-Topics/计算机/编程语言/Python/2025-12 contextmanager利用的是yield的暂停而不是迭代|2025-12 contextmanager利用的是yield的暂停而不是迭代]] — 讲了：1.  yield  的两副面孔 / 面孔 A：迭代器（Iterator）—— 你的理 / 面孔 B：暂停键（Suspension）——  
- [[02-Topics/计算机/编程语言/Python/2025-12 python for|2025-12 python for]] — 讲了：一、答案最简版： / 二、为什么  for  需要这两个方法？（设计动 / （1） __iter__()
- [[02-Topics/计算机/编程语言/Python/2025-12 python with|2025-12 python with]] — 讲了：一、先说问题本质：Python 打开文件，只是第 / 二、 file   open(...)  到底干 / 1.  open()  是一个工厂函数（不是类构
- [[02-Topics/计算机/编程语言/Python/2025-12 yield|2025-12 yield]] — 讲了：1. 核心定义： yield  是什么？ / 2. 历史背景：为什么我们需要  yield ？ / 3. 底层原理：生成器的工作流程
- [[02-Topics/计算机/编程语言/Python/2025-12 为什么一定要 try 包裹起来|2025-12 为什么一定要 try 包裹起来]] — 讲了：1. 这种写法：只在“太平盛世”下有效 / 2. 灾难现场：当  with  块里有炸弹 / 3. 底层原理：异常是如何“杀死”生成器的？
- [[02-Topics/计算机/编程语言/Python/2026-01 python 对象属性的三种定义方式|2026-01 python 对象属性的三种定义方式]] — 讲了：三种属性定义方式的区别 / 1️⃣   类属性（在   前定义） / 2️⃣   实例属性（在   中定义）
- [[02-Topics/计算机/编程语言/Python/2026-01 python中的int|2026-01 python中的int]] — 讲了：torch 中的 int 比普通的小？ / Planner Response / PyTorch 整数类型 vs Python 整
- [[02-Topics/计算机/编程语言/Python/2026-01 python怎么计算和存储无限精度|2026-01 python怎么计算和存储无限精度]] — 讲了：User Input / Planner Response / 核心思想：和小学竖式计算一样！
- [[02-Topics/计算机/编程语言/Python/2026-01 python计算较慢的底层逻辑|2026-01 python计算较慢的底层逻辑]] — 讲了：为什么 python 的运算较慢 / Planner Response / 1. 解释执行 vs 编译执行
- [[02-Topics/计算机/编程语言/Python/2026-01 代码学习思路|2026-01 代码学习思路]] — 讲了：Chat Conversation / User Input / Planner Response
- [[02-Topics/计算机/编程语言/Python/2026-01 前向引用|2026-01 前向引用]] — 讲了：Planner Response / 问题：前向引用（Forward Referenc / 代码执行顺序
- [[02-Topics/计算机/编程语言/Python/2026-02 引用|2026-02 引用]] — 讲了：1. 类型匹配的“强制约束” / 2. 底层“两次转换”的真相 / 3. 如果允许  return ptr;  会发
- [[02-Topics/计算机/编程语言/Python/2026-04 Python - Enum,TypedDict,dataclass|2026-04 Python - Enum,TypedDict,dataclass]] — 讲了：1. Enum（枚举）：告别“魔法字符串” / 2. TypedDict（类型化字典）：给外部传 / 3. dataclass（数据类）：现代面向对象
- [[02-Topics/计算机/编程语言/Python/2026-05 Python 错误处理类型 1 - 有什么类型|2026-05 Python 错误处理类型 1 - 有什么类型]] — 讲了：一、 为什么对 Enum 查找，基本只有这两类  / 二、 Python 编程中，其他最常用的 6 大 / 💡 总结一个小技巧：
- [[02-Topics/计算机/编程语言/Python/2026-05 Python 错误处理类型 2 - 怎么判断要捕获的类型|2026-05 Python 错误处理类型 2 - 怎么判断要捕获的类型]] — 讲了：1. 七分靠预判：我们为什么能知道会报什么错？ / 2. 三分靠实验：不知道会报什么错时，我们怎么办 / 招数一：故意使坏，查看报错（最常用）
- [[02-Topics/计算机/编程语言/Python/2026-05 Python 错误处理类型 3 - 怎么用|2026-05 Python 错误处理类型 3 - 怎么用]] — 讲了：一、 为什么在“局部业务逻辑”中，不应该用它？ / 1. 它会“指鹿为马”，掩盖真正的 Bug / 正确做法：
- [[02-Topics/计算机/编程语言/Python/2026-05 代码 - 闭包|2026-05 代码 - 闭包]] — 讲了：第一步：变量存在哪？ / 第二步：函数里返回函数 / 第三步：这就是闭包
- [[02-Topics/计算机/编程语言/Python/2026-08 子包的具体样子|2026-08 子包的具体样子]] — 讲了：一、打开真实的子包看一看，它里面到底有什么？ / 子包的目录结构： / 子包的  package.json ：
- [[02-Topics/计算机/编程语言/Python/2026-09 Python 的循环导入|2026-09 Python 的循环导入]] — 讲了：示例 / 一、 错误原因与底层机制 / 触发过程示意：
