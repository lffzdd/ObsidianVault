# Python并发编程

**Created:** 6/9/2026, 8:10:50 PM
**Updated:** 6/9/2026, 8:13:24 PM
**Exported:** 6/9/2026, 11:11:39 PM
**Model:** claude-fable-5
**Link:** [https://claude.ai/chat/67ec03af-12b0-462b-a168-2111a7b91a1b](https://claude.ai/chat/67ec03af-12b0-462b-a168-2111a7b91a1b)

---

## User
**2026-06-10T03:10:50.980Z**

跟我说说Python的并发

## Claude
**2026-06-10T03:13:24.214Z**

### Thinking
````
 用户用中文提问Python并发话题，他们偏好从底层开始、避免黑盒、先理解原理再学方法、定义术语、展示依赖关系、提供可运行的例子和权衡分析。我需要用中文回答，这是个大话题，应该结构清晰但不过度格式化，在教育内容中适度使用标题，同时深入解释。

我的计划是：先讲并发存在的原因（CPU与I/O速度差异、多任务处理），然后定义关键概念（并发vs并行、进程、线程、GIL），接着分层讲解从操作系统进程/线程到Python线程、GIL的存在原因（CPython的引用计数内存管理）、多进程、再到异步IO（事件循环、非阻塞I/O、epoll/select系统调用、协程和生成器的底层实现）。用散文形式展示各种方案的权衡，提供可运行的实验代码来演示GIL对CPU密集型任务的影响、I/O密集型任务的加速、多进程和异步IO的用法，还要提到Python 3.13的无GIL实验特性。

我需要先补充进程、线程、系统调用和阻塞I/O的基础概念，然后再深入讲解GIL的具体机制。

GIL本质上是CPython解释器中的一个互斥锁，确保同一时刻只有一个线程执行Python字节码。这个设计的原因在于CPython使用引用计数进行内存管理——如果多个线程同时修改对象的引用计数而不加锁，就会导致竞态条件和内存损坏。虽然1999年有过尝试使用细粒度的每对象锁（Greg Stein的"自由线程"补丁），但这会让单线程性能下降约2倍，所以最终采用了一个全局锁。

GIL的实际影响是：CPU密集型的多线程程序不会获得性能提升，因为线程无法真正并行执行；但I/O密集型程序可以受益，因为在阻塞系统调用和C扩展中GIL会被释放。线程切换间隔由sys.getswitchinterval()控制，默认5毫秒。

相比之下，multiprocessing创建独立的进程和解释器，每个都有自己的GIL，但代价是进程创建开销和通过pickling进行进程间通信。asyncio则提供了另一种并发模型。

最近的发展包括PEP 703的自由线程CPython（3.13实验性，3.14支持）和PEP 734的子解释器（3.14的concurrent.interpreters模块）。

现在我要开始做一些实验，首先测试CPU密集型的情况。 I have Python 3.14 coverage through my knowledge cutoff, so no need to search. For this tutorial, I'll structure it with headers and code examples for clarity while keeping the prose flowing and thorough, written in Chinese, with pointers for deeper exploration.
````

# Python 并发:从问题本身讲到 CPython 的实现

这是个层次很多的话题。我按你的习惯,从"为什么需要并发"开始,一路下到 GIL 的实现动机和事件循环底下的系统调用。先搭依赖图,你确认一下哪些已经知道,哪些需要补:

```
硬件:CPU 核心、内存
  └─ 操作系统:进程、线程、调度、系统调用、阻塞 I/O
       └─ CPython 解释器:字节码、引用计数、GIL
            ├─ threading(包装 OS 线程)
            ├─ multiprocessing(包装 OS 进程)
            └─ asyncio(单线程协作式调度,底下是 epoll/kqueue)
```

下面每一层我都会给一段刚好够用的铺垫,如果哪段你已经熟,跳过即可。

---

## 第 0 层:要解决的问题是什么

并发要解决的根本问题是:**程序经常在"等",等的时候 CPU 闲着**。

两种典型的"慢":

1. **I/O 密集(I/O-bound)**:程序大部分时间在等外部世界——等网络包到达、等磁盘读完、等数据库返回。一次内存访问约 100 纳秒,一次同城网络往返约 1 毫秒,差了 4 个数量级。顺序执行 100 个网络请求,99% 的墙钟时间 CPU 在发呆。
2. **CPU 密集(CPU-bound)**:程序在真算东西(压缩、加密、数值计算)。单核已经跑满,想更快只能用更多核。

这两种慢,解法完全不同——这是理解 Python 并发的第一个关键。很多困惑都来自拿错了工具:用多线程加速 CPU 密集任务(在 CPython 里基本无效,后面解释为什么),或者用多进程处理大量小 I/O(开销划不来)。

再定义两个常被混用的词:

- **并发(concurrency)**:多个任务在重叠的时间段内推进,但不一定同一瞬间都在跑。一个厨师轮流照看三口锅,是并发。
- **并行(parallelism)**:多个任务在同一物理瞬间真的同时执行,需要多个核心。三个厨师各看一口锅,是并行。

并发是程序结构问题,并行是硬件执行问题。asyncio 是纯并发无并行;multiprocessing 是真并行;threading 在 CPython 里——这就是故事开始的地方。

---

## 第 1 层:操作系统给的原语——进程和线程

(如果你熟悉 OS 进程/线程模型,跳到第 2 层。)

**进程(process)**:操作系统分配资源的基本单位,核心是一套独立的虚拟地址空间——进程 A 里地址 `0x7f...` 指向的物理内存,和进程 B 里同一个地址指向的物理内存毫无关系。这种隔离是好处(一个进程崩溃不连累别人)也是代价(两个进程想交换数据,必须通过内核安排的通道:管道、socket、共享内存段等,统称 IPC,进程间通信)。

**线程(thread)**:进程内部的执行流。同一进程的多个线程**共享同一个地址空间**——线程 A 写一个全局变量,线程 B 立刻能读到,因为那就是同一块物理内存。每个线程有自己独立的:调用栈、寄存器状态(包括程序计数器)。在 Linux 上,线程是用 `clone()` 系统调用创建的,带上 `CLONE_VM` 等标志表示"和父任务共享地址空间"——内核视角里线程和进程都是 task,区别只是共享了什么。

共享内存带来线程的核心难题:**数据竞争(data race)**。两个线程同时执行 `x += 1`,这在机器层面是"读 x → 加 1 → 写回"三步,可能交错成两次读到同一个旧值,最终只加了 1。防御手段是**锁(mutex)**:一种保证同一时刻只有一个线程能进入某段代码的同步原语。记住这个问题,GIL 的存在理由就是它。

还有一个词后面反复出现:**阻塞(blocking)**。线程发起一个 `read()` 系统调用读 socket,而数据还没到,内核会把这个线程置为睡眠状态、从 CPU 上拿下来,直到数据到达再唤醒它。"阻塞"期间线程什么都干不了,但也不耗 CPU。

---

## 第 2 层:threading 与 GIL——CPython 最著名的设计决策

Python 的 `threading` 模块创建的是**货真价实的 OS 线程**(Linux 上是 pthread),由内核调度,可以被分到不同核心上。但是:

**GIL(Global Interpreter Lock,全局解释器锁)**:CPython 解释器内部的一把互斥锁,规定**任一时刻只有一个线程可以执行 Python 字节码**。所以哪怕你有 16 个核、开 16 个线程,执行 Python 代码的永远只有一个。

### 为什么要有 GIL(不是偷懒,是权衡)

CPython 的内存管理基于**引用计数(reference counting)**:每个 Python 对象在 C 层是一个 `PyObject` 结构体,头部有个 `ob_refcnt` 字段记录"有多少地方引用着我",归零即刻释放。你可以亲眼看:

```python
import sys
a = [1, 2, 3]
sys.getrefcount(a)   # 通常是 2:变量 a 一次 + 作为参数传入 getrefcount 临时加的一次
b = a
sys.getrefcount(a)   # 3
```

问题:`ob_refcnt += 1` 和上面的 `x += 1` 一样不是原子操作。如果两个线程同时增减同一个对象的引用计数而不加锁,计数会错——要么提前释放(悬垂指针,解释器崩溃或内存损坏),要么永不释放(泄漏)。

两条路:
1. **细粒度锁**:每个对象自带锁,每次改引用计数都加锁解锁。1999 年有人真做过(Greg Stein 的 free-threading 补丁),结果单线程程序慢了近一倍——因为 Python 里**一切都是对象**,引用计数操作密得可怕,锁开销摊不掉。
2. **一把大锁**:整个解释器一把 GIL,持锁期间随便改任何引用计数都安全。单线程零额外开销,C 扩展也好写(默认不用考虑线程安全)。

CPython 选了 2。代价就是字节码执行无法并行。这不是"Python 语言"的规定,而是 CPython 这一实现的选择——Jython(JVM 上的 Python)就没有 GIL,因为 JVM 有自己的并发 GC。

### GIL 什么时候松手

如果 GIL 从不释放,多线程就完全没意义了。两种释放时机:

1. **时间片轮转**:运行中的线程每隔一个"切换间隔"(默认 5 毫秒,`sys.getswitchinterval()` 可查可改)会被要求释放 GIL,给其他线程抢的机会。这保证了并发(交替推进),但不是并行。
2. **阻塞 I/O 前主动释放**:CPython 在发起会阻塞的系统调用之前(socket 读写、文件读写、`time.sleep`……)释放 GIL,调用返回后再重新获取。C 扩展里这对应 `Py_BEGIN_ALLOW_THREADS / Py_END_ALLOW_THREADS` 宏。**这就是为什么多线程对 I/O 密集任务有效**:一个线程在内核里等数据时,GIL 在别的线程手里,大家等待的时间重叠了。很多数值库(NumPy 的大矩阵运算)在 C 代码里也会释放 GIL,所以"线程对 CPU 密集无效"在调 C 库时有例外。

### 实验 1:亲手测出 GIL 的存在

```python
# gil_demo.py
import time, threading, sys

N = 50_000_000

def count(n):
    while n > 0:
        n -= 1

# 单线程跑两份
t0 = time.perf_counter()
count(N); count(N)
print(f"sequential: {time.perf_counter() - t0:.2f}s")

# 双线程各跑一份
t0 = time.perf_counter()
ts = [threading.Thread(target=count, args=(N,)) for _ in range(2)]
for t in ts: t.start()
for t in ts: t.join()
print(f"2 threads:  {time.perf_counter() - t0:.2f}s")
```

预测后再跑:如果线程真能并行,双线程应该约快一倍。实际你会看到双线程**不快,通常还略慢**——慢出来的部分就是 GIL 在两个线程间倒手的开销。

改动观察:把 `count` 换成 `time.sleep(2)`(模拟 I/O),双线程会几乎正好 2 秒而不是 4 秒——因为 sleep 期间 GIL 被释放了。再试 `sys.setswitchinterval(0.0001)` 之后重跑 CPU 版,观察切换更频繁时变得更慢。

想看更底层:跑的时候开 `top -H`(Linux,按线程显示),你会看到两个线程各占约 50% CPU 而不是各 100%——它们在轮流持有 GIL。

要再深一层:GIL 在 CPython 源码 `Python/ceval_gil.c` 里,本质是条件变量 + 一个 `gil_drop_request` 标志,eval 循环定期检查它。

---

## 第 3 层:multiprocessing——绕开 GIL 的钝器

思路简单粗暴:GIL 是**每个解释器进程**一把,那就开多个进程,每个进程有自己的解释器、自己的 GIL,真并行。

```python
# mp_demo.py —— 接实验 1,把线程换成进程
import time
from multiprocessing import Process

N = 50_000_000

def count(n):
    while n > 0:
        n -= 1

if __name__ == "__main__":          # 为什么必须有这行?见下文
    t0 = time.perf_counter()
    ps = [Process(target=count, args=(N,)) for _ in range(2)]
    for p in ps: p.start()
    for p in ps: p.join()
    print(f"2 processes: {time.perf_counter() - t0:.2f}s")
```

在双核以上的机器上,这次你会看到接近 2 倍加速。`top` 里能看到两个 python 进程各吃 100% CPU。

但代价都藏在第 1 层讲的进程隔离里:

1. **创建贵**。Linux 上默认 `fork()`(复制父进程的页表,写时复制),macOS/Windows 上是 spawn(起一个全新解释器、重新 import 你的模块——这就是为什么必须有 `if __name__ == "__main__"` 守卫,否则子进程 import 主模块时又会创建子进程,无限递归)。注意:Python 3.14 起 Linux 的默认方式也改成了 forkserver,不再是 fork,因为 fork 一个多线程进程有死锁风险——这是个值得单独深挖的坑。
2. **传数据贵**。进程不共享内存,`args=(N,)` 和返回值都要经过**序列化**:Python 用 `pickle` 把对象转成字节流,通过管道(底层是 `pipe()` 系统调用)送过去再反序列化。传一个大 NumPy 数组给子进程,等于完整拷贝一次。逃生通道是 `multiprocessing.shared_memory`(真共享内存段)——但那又把数据竞争问题请回来了。
3. **不能 pickle 的对象传不过去**(打开的文件、socket、lambda……)。

改动观察:给 `count` 传一个 100MB 的 bytes 对象作为额外参数(函数内不用它),测启动时间的变化——你能直接量出序列化的开销。

---

## 第 4 层:asyncio——换一个完全不同的问题模型

### 它解决的问题:线程模型在海量连接下撑不住

设想一个聊天服务器要同时挂 1 万个连接(经典的 "C10K 问题"),每连接绝大部分时间无事发生。线程方案是每连接一个线程,1 万个线程意味着:每线程一个调用栈(默认预留几 MB 虚拟内存),以及更致命的——内核在 1 万个线程间上下文切换,每次切换要保存/恢复寄存器、可能污染 CPU 缓存,微秒级的开销乘上巨大的切换频率,CPU 大量时间花在"换人"而不是干活上。而且这些线程 99% 的时间在阻塞等数据,纯属浪费。

asyncio 的思路:**一个线程照看全部 1 万个连接**。可行的前提是两个 OS 机制:

1. **非阻塞 I/O**:把 socket 设为非阻塞模式后,`read()` 在没数据时不再让线程睡眠,而是立刻返回错误码 `EWOULDBLOCK`("现在没有,别等了")。
2. **I/O 多路复用(I/O multiplexing)**:一种系统调用,让你把一大批文件描述符交给内核,问"这里面哪些就绪了?",然后只在一个调用上阻塞。Linux 上是 `epoll`,macOS/BSD 上是 `kqueue`,古老通用的是 `select`。Python 的 `selectors` 模块封装了它们。

于是单线程的循环长这样(这就是**事件循环 event loop** 的本质,不是黑魔法):

```
循环:
    就绪列表 = epoll_wait(全部被监视的 fd)   # 唯一阻塞点
    对每个就绪的 fd:
        恢复执行等在这个 fd 上的那段代码,跑到它下一次需要等 I/O 为止
    顺便检查到期的定时器
```

### 协程:可以"跑到一半暂停"的函数

上面"跑到它下一次需要等待为止"要求一种能力:函数执行到中途,把自己挂起、保存现场、交还控制权,之后再从断点恢复。普通函数做不到(一旦 return,栈帧就销毁了),Python 里具备这种能力的东西是**协程(coroutine)**,语法上由 `async def` 定义,机制上脱胎于生成器——你可以先用生成器摸到它的本质:

```python
def gen():
    print("step 1")
    x = yield "pause A"      # 执行到此暂停,把控制权和值交还给调用方
    print(f"step 2, got {x}")
    yield "pause B"

g = gen()
print(next(g))        # 跑到第一个 yield 停下
print(g.send("hi"))   # 从断点恢复,"hi" 成为 yield 表达式的值
```

生成器的局部变量和暂停位置保存在堆上的一个帧对象里(普通函数帧随调用栈销毁,生成器帧不随),所以才能恢复。`async def` 协程是同一机制的专用版本;`await` 就是暂停点,意思是"我要等这个东西,事件循环你先去忙别的,它好了叫我"。

关键词:这叫**协作式(cooperative)调度**——切换点由代码自己用 `await` 标明。对比线程的**抢占式(preemptive)**——内核随时可以打断你。协作式的好处:两个 `await` 之间的代码绝不会被打断,大量数据竞争问题直接消失,基本不需要锁;坏处:**一个协程不让路,全员卡死**。在协程里写 `time.sleep(5)`(阻塞整个线程)或一段长计算,整个事件循环停摆 5 秒,1 万个连接全部无响应。这是 asyncio 的头号天坑。

### 实验 2:看到协作与卡死

```python
# aio_demo.py
import asyncio, time

async def worker(name, delay):
    print(f"{name} start  {time.strftime('%X')}")
    await asyncio.sleep(delay)        # 挂起自己,注册一个定时器,让出控制权
    print(f"{name} done   {time.strftime('%X')}")

async def main():
    t0 = time.perf_counter()
    await asyncio.gather(worker("A", 2), worker("B", 2), worker("C", 2))
    print(f"total: {time.perf_counter() - t0:.2f}s")

asyncio.run(main())
```

三个各等 2 秒的任务,总耗时约 2 秒——等待重叠了,而且全程一个线程(可以打印 `threading.current_thread()` 验证)。

改动观察:把 `await asyncio.sleep(delay)` 换成 `time.sleep(delay)`,总耗时变 6 秒——你亲眼看到一个阻塞调用如何把协作式调度打回顺序执行。

再深一层的观察:用 `strace -e epoll_wait,epoll_ctl python aio_demo.py`(Linux)跑一遍,能直接看到事件循环发出的 epoll 系统调用——黑盒到此拆穿。想读实现,看 CPython 源码 `Lib/asyncio/selector_events.py` 和 `Lib/selectors.py`。

asyncio 的真实代价不在性能而在**传染性**:`await` 只能出现在 `async def` 里,所以一个函数变 async,它的调用方也得变,一路传染到顶;同步生态的库(经典 `requests`)在协程里用就是阻塞炸弹,得换 async 原生库(`aiohttp`/`httpx`)。生态被劈成了两个世界。

---

## 选型:同一张问题地图

| 任务性质 | 工具 | 原因(对应上面的层) |
|---|---|---|
| CPU 密集 | `multiprocessing` | 唯一绕开 GIL 拿到多核的标准库手段 |
| I/O 密集,几十个并发 | `threading` | 阻塞时释放 GIL,等待重叠;同步代码不用改写 |
| I/O 密集,成百上千并发 | `asyncio` | 单线程扛海量连接,没有线程栈和上下文切换开销 |
| CPU 密集但在调 C 库(NumPy 等) | 先试 `threading` | C 代码内部常释放 GIL,实测为准 |

实践提示:多数时候不直接用 `Thread`/`Process`,而是用 `concurrent.futures` 的 `ThreadPoolExecutor`/`ProcessPoolExecutor`——同一套接口,池化复用线程/进程,免去反复创建的开销。理解了底层再用它,它就不是黑盒,只是省事。

---

## 正在变动的部分:GIL 的黄昏(注意时效)

这块是近两年的活跃区,值得知道但别当成稳定地基:

1. **自由线程构建(free-threaded build,PEP 703)**:从 3.13 开始,CPython 提供一个编译选项去掉 GIL 的版本,3.14 起官方从"实验性"升级为"受支持"。它把引用计数改成了**有偏引用计数**(biased reference counting:创建对象的线程用快速无锁路径,其他线程走原子操作)等一组技术来换掉那把大锁,代价是单线程性能仍有个位数百分比的损耗,且 C 扩展生态需要逐个适配。普通 `python3` 默认安装仍是带 GIL 的;自由线程版通常叫 `python3.14t`。装一个,把实验 1 在两个版本上各跑一遍——双线程从"不快反慢"变成"接近两倍",是这十几年 CPython 最大的语义变化,亲手测一次值得。
2. **子解释器(PEP 734,3.14 的 `concurrent.interpreters`)**:同一进程里跑多个隔离的解释器,各持各的 GIL——介于线程和进程之间的第三条路。

细节(具体版本的性能数字、哪些扩展已适配)我不敢保证最新,验证方式:`python3.14t -c "import sys; print(sys._is_gil_enabled())"`,以及查 py-free-threading.github.io 的兼容性追踪。

---

## 建议的消化顺序

先把实验 1(GIL)→ 实验 1 的 sleep 变体 → mp_demo → 实验 2 及其 `time.sleep` 变体按顺序跑完,每个都先写下预测再看结果。然后挑一个方向往下钻,按依赖关系我推荐的顺序:① 用生成器手写一个 30 行的迷你事件循环(彻底拆掉 asyncio 的黑盒,网上搜 David Beazley 的 "Build Your Own Async" 讲座,是这条路的最佳材料);② 读 `Lib/selectors.py` 源码 + `man epoll`;③ 线程同步原语(`Lock`、`Condition`)和一个真实数据竞争的复现——自由线程时代它会变成必修课。

哪一层你想先展开?GIL 的 C 实现、事件循环手写、还是进程的 fork/spawn 细节,深度都还能再下两层。

