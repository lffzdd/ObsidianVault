# 
### **Promise 的本质是"一个对未来某个值的占位符对象"，它把原本必须以参数形式传进去的回调，倒置成可以在拿到对象之后再挂上去的监听器，从而让异步结果重新变成可以被 return、被传递、被组合的头等值**

你之前选中的那段提到"第一次折叠"有点跳，是因为我直接给了结论没铺垫动机。Promise 在表面上看只是把 `f(arg, cb)` 改写成 `f(arg).then(cb)`，似乎只是语法挪了挪位置——但**这个挪位置背后的语义变化是根本性的**，它直接决定了为什么后来 async/await 能存在。我从动机、结构、状态机、关键设计、链式语义、错误传播、和事件循环的关系这几个层面，一层一层把它讲透。

---

### **一、动机：回调最致命的问题不是嵌套，是"异步操作没有返回值"**

我们先回到回调地狱真正的病根。前一轮我列了五重痛苦，但如果让我抽出**最核心的那一个**，是这条：

> **回调风格的异步函数没有返回值。**

同步世界里，函数的契约是"接收输入、返回输出"。这个契约让函数变成可以自由组合的零件——你可以把 `f()` 的返回值传给 `g()`、塞进数组、赋给变量、嵌进表达式。整个编程语言的所有控制流原语（`if`、`for`、`return`、`try`）都建立在"表达式有值"这个前提上。

回调风格打破了这个契约。`fs.readFile(path, cb)` 不返回任何东西——它的"结果"只能通过把 cb 传进去的方式被消费。这意味着：

- **结果不能被存储**：你没法写 `const data = fs.readFile(...)`。
- **结果不能被传递**：你没法把"读文件这个操作的结果"作为参数传给另一个函数，因为这个结果还不存在，能传的只有"读文件这个操作本身 + 它的回调"。
- **结果不能被组合**：你没法对"三个文件的内容"做 `Array.map`，因为这三个内容根本不在同一个时间点存在。

Promise 解决的就是这一件事——**它发明了一个对象，让"还没产生的值"提前变成可以被 return、被赋值、被传递的实体**。一旦异步操作能 return 东西，回调地狱的所有衍生问题（嵌套、错误传播、组合性）就有了统一的解法。

---

### **二、Promise 的结构：一个三态对象 + 一组待通知的监听器**

最朴素地把 Promise 当成一个普通 JavaScript 对象来看，它内部其实只有四个字段：

```js
class Promise {
  state       = 'pending';     // 'pending' | 'fulfilled' | 'rejected'
  value       = undefined;     // fulfilled 时存结果，rejected 时存原因
  onFulfilled = [];            // 等成功的回调队列
  onRejected  = [];            // 等失败的回调队列
}
```

它的生命周期是一个**只能单向走一次**的状态机：

```
                resolve(v)
   ┌──────────────────────────► fulfilled (value = v)
   │
pending
   │
   └──────────────────────────► rejected  (value = reason)
                reject(reason)
```

三个关键性质，每一个都是刻意设计的：

1. **状态只能从 pending 转移到 fulfilled 或 rejected 中的一个**，转移之后**永久冻结**。这就是 Promises/A+ 规范里说的 "settled once"。后续再调 resolve 或 reject 都会被静默忽略。
2. **value 一旦写入就不可变**。Promise 不是事件流，它只代表"一次性的、最终的"那个结果。
3. **监听器队列只在 settle 之前有意义**。settle 之后挂上来的回调会被立即（异步地）调度执行，settle 之前挂的会被排队，等 settle 时一起触发。

为什么这样设计？因为 Promise 模拟的是**函数返回值**的语义。一个函数只 return 一次，return 的值不会变，谁拿到这个值都看到同一个东西——Promise 就是把这三个性质平移到异步世界。如果你需要"多次发出值"的语义，那是另一种抽象（Observable、Stream、AsyncIterator），不是 Promise。

---

### **三、最小可用实现：把状态机写出来你就懂了**

光看字段定义不直观，我们手写一个最小版本，能跑通"创建-resolve-then 拿值"的全流程。先不管链式，先看核心：

```js
class MyPromise {
  constructor(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.onFulfilled = [];
    this.onRejected  = [];

    const resolve = (v) => {
      if (this.state !== 'pending') return;     // 单向、一次性
      this.state = 'fulfilled';
      this.value = v;
      this.onFulfilled.forEach(cb => queueMicrotask(() => cb(v)));
    };
    const reject = (e) => {
      if (this.state !== 'pending') return;
      this.state = 'rejected';
      this.value = e;
      this.onRejected.forEach(cb => queueMicrotask(() => cb(e)));
    };

    try { executor(resolve, reject); }
    catch (e) { reject(e); }
  }

  then(onFul, onRej) {
    if (this.state === 'fulfilled') {
      queueMicrotask(() => onFul(this.value));
    } else if (this.state === 'rejected') {
      queueMicrotask(() => onRej(this.value));
    } else {
      this.onFulfilled.push(onFul);
      this.onRejected.push(onRej);
    }
  }
}
```

来用一下：

```js
const p = new MyPromise((resolve) => {
  setTimeout(() => resolve(42), 1000);
});
p.then(v => console.log(v));   // 1 秒后打印 42
```

到这里你就抓住了 Promise 的骨架。我们停下来盘一下**它到底做了什么**：

- 调用 `new MyPromise(executor)` 时，executor 立刻同步执行——这一步至关重要，它是**启动异步操作的地方**（比如里面会调 `setTimeout`、`fs.readFile`、`fetch` 之类）。
- executor 收到两个由 Promise 内部生成、绑死了 this 的函数：`resolve` 和 `reject`。这两个函数是**写入 Promise 状态的唯一入口**。
- 异步操作完成时（setTimeout 触发时），它调用 resolve(42)。这一刻 Promise 从 pending 变成 fulfilled，并且把 42 存进 value。
- `then(cb)` 的语义是"当我 fulfilled 的时候请调你 cb，参数是我的 value"。如果调 then 的时候已经 fulfilled，就直接把 cb 排进 microtask 队列；否则就先存到 onFulfilled 数组里等 resolve 时统一触发。

注意我用了 `queueMicrotask` 来调度回调。**这一步是规范强制的**——Promise/A+ 规定 then 的回调**必须异步执行**，永远不能同步直接调 cb。原因是要保证一个不变量：**调用 then 时，无论 Promise 当时是不是已经 settled，cb 的执行时机都在当前同步代码之后**。这个不变量消除了一类极其难调的 bug——"有时候是同步执行有时候是异步执行"导致的竞争。

---

### **四、控制反转：从"把 cb 传进去"到"把对象拿出来"**

现在对比两种风格，注意箭头方向变了：

**回调风格**——cb 必须在异步操作开始前就传进去：

```js
//  调用方 ─────► f ──传进去──► cb
fs.readFile('a.txt', (err, data) => { ... });
```

**Promise 风格**——异步操作先返回一个对象，cb 后挂上去：

```js
//  调用方 ◄──返回 promise── f
//  调用方 ──then 挂载 cb──► promise
const p = readFile('a.txt');
p.then(data => { ... });
```

这个变化看起来微小，影响极其深远：

1. **异步操作和"消费结果"在时间和空间上解耦了**。`readFile('a.txt')` 这一行决定"启动什么操作"，`.then(...)` 那一行决定"拿到结果后干什么"——它们可以分开写、可以隔几十行、可以由不同的代码模块负责。
2. **同一个 Promise 可以被 then 多次**，每个 then 都拿到同一个 value。这给了我们一种自然的"广播"能力，回调风格做不到（你只能传一个 cb）。
3. **Promise 可以作为函数返回值在调用栈里冒泡**，完美兼容已有的"函数组合"思维：写工具函数 `retry(fn, times)`、`timeout(promise, ms)`、`map(arr, asyncFn)` 都变得自然。

我把这步叫做**第一次折叠**，是因为它把"传 cb 进去"这种基于参数的协作模式，**折叠回了"return 一个值"的经典函数语义**。原本被异步打碎的"函数有返回值"这个语言契约，被一个对象重新粘了起来。

---

### **五、关键魔法：then 必须返回一个新 Promise，而且自动展平**

到这一步，Promise 还只是把"挂回调"的方向反过来了，没解决嵌套。真正让它能取代回调地狱的关键设计，藏在 `then` 的返回值里：

> **`p.then(cb)` 必须返回一个新的 Promise `p2`，`p2` 的最终值由 `cb` 的返回值决定。**

这条规则把整个游戏带到了新的高度。我把上面的 then 完善一下：

```js
then(onFul, onRej) {
  return new MyPromise((resolve, reject) => {
    const handle = (cb, value, isFulfilled) => {
      queueMicrotask(() => {
        try {
          if (typeof cb !== 'function') {
            // 没传 handler，把值/错误透传给下一个 promise
            isFulfilled ? resolve(value) : reject(value);
            return;
          }
          const x = cb(value);                    // 跑用户的 then 回调
          if (x instanceof MyPromise) {
            x.then(resolve, reject);              // 关键:自动展平
          } else {
            resolve(x);                           // 普通值直接 resolve
          }
        } catch (e) {
          reject(e);                              // 抛异常 = reject
        }
      });
    };

    if (this.state === 'fulfilled')      handle(onFul, this.value, true);
    else if (this.state === 'rejected')  handle(onRej, this.value, false);
    else {
      this.onFulfilled.push(v => handle(onFul, v, true));
      this.onRejected .push(e => handle(onRej, e, false));
    }
  });
}
```

这段代码里有三处魔法值得逐个盯着看：

**魔法一：cb 的返回值变成新 Promise 的 value。**
意味着你在 then 里 `return 'hello'`，下一个 then 就能拿到 'hello'。这是把"函数返回值"的语义在 Promise 链上接力下去。

**魔法二：如果 cb 返回的本身就是一个 Promise，自动展平。**
这条规则叫 **assimilation（同化）**或 **flattening（展平）**。它的意思是：当你在 then 里返回 `fetch(url)`（一个 Promise），下一个 then 不会拿到"一个 Promise 对象"，而是会等这个 Promise 自己 settle，拿到它的最终值。所以——

```js
readFile('config.json')
  .then(cfg  => fetch(JSON.parse(cfg).url))  // 返回 Promise
  .then(resp => resp.json())                  // 返回 Promise
  .then(data => db.insert(data))              // 返回 Promise
  .then(()   => console.log('done'));
```

**金字塔被拍平成了一条线**。这就是"折叠"最直接的体现：原本必须靠嵌套表达的"先 A 完成、再 B、再 C"的串行依赖，现在用平铺的链表达，每一行就是一个步骤。

**魔法三：cb 里 throw 等价于 reject。**
try/catch 把同步抛出的异常转成下一个 Promise 的 rejected 状态。这就是为什么链尾一个 `.catch(handleErr)` 能兜住整条链上任何一处的错误——错误沿着链以"rejected 状态"的形式自动冒泡，跟同步代码里异常沿调用栈冒泡是同构的。

---

### **六、错误传播：把异常语义"接"回了异步世界**

上一轮你问回调地狱的时候我提过：回调脱离了调用栈，所以 try/catch 失效，错误必须靠 error-first 约定手动接力。Promise 用上面的设计**把这件事自动化了**。

具体来说，一条 Promise 链上有两条平行的传播通道：

```
            resolve──► onFul ──┐
pending ─►                     ├─► next promise
            reject ──► onRej ──┘
```

任何一环 reject 之后，如果它的 then 没传 onRej（或者 onRej 自己又 throw 了），这个 rejected 状态就**沿链向后透传**，直到遇到一个能处理它的 onRej（也就是 `.catch`，本质是 `then(undefined, cb)` 的别名）。

这意味着你可以写：

```js
fetchUser(id)
  .then(user => fetchOrders(user.id))
  .then(orders => render(orders))
  .catch(err => showError(err));   // 上面任何一步的错误都会到这里
```

**错误处理代码集中在一处**，而不是像 error-first callback 那样每层都要写一行 `if (err) return ...`。这一点是 Promise 给开发者体验带来的最大改善之一，也是后来 async/await 能直接复用 try/catch 的前提——因为 await 在底层就是把 reject 转成 throw。

---

### **七、组合子：Promise 作为头等值的真正威力**

异步操作变成"对象"以后，你可以像处理普通数据一样**对一组 Promise 做集合操作**。这就是 `Promise.all`、`Promise.race`、`Promise.allSettled`、`Promise.any` 存在的意义。它们的实现都很短，核心思路是"创建一个新 Promise，订阅输入数组里所有 Promise 的状态变化，按规则决定何时 resolve/reject 这个新 Promise"。

`Promise.all` 的最小实现：

```js
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let remaining = promises.length;
    if (remaining === 0) return resolve([]);
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        v => { results[i] = v; if (--remaining === 0) resolve(results); },
        e => reject(e)         // 任一失败立刻整体失败
      );
    });
  });
};
```

回想一下回调地狱那节我手写的那个 `done` 计数器加 `aborted` 标志位——`Promise.all` 把这一整套手工状态机封装成了一行调用。这就是把异步"对象化"之后的复利效应：**任何在同步世界里能对值做的高阶操作，都能平移到异步世界**。Promise 链可以 map、filter、reduce、可以放进数组、可以塞进 Map 当 value、可以作为缓存的条目（这就是著名的 "promise caching" 模式：缓存一个进行中的 Promise，多个请求方共享同一次实际请求）。

---

### **八、和事件循环的关系：microtask 队列保证了"看起来同步"**

到这里 Promise 的语义层面讲完了，还有最后一块拼图——**它具体是被谁、在什么时候推进的**。这一块直接关系到后面讲 async/await 时你能不能完全弄懂。

JavaScript 的事件循环里有两类回调队列：**macrotask**（也叫 task，包括 setTimeout、I/O 回调、UI 事件等）和 **microtask**（Promise 的 then 回调、queueMicrotask、MutationObserver 等）。规则是：

> **每执行完一个 macrotask，引擎会排干（drain）整个 microtask 队列后，才进入下一个 macrotask。**

所以前面我手写实现里反复用 `queueMicrotask(() => cb(v))`——这是规范要求。它带来一个非常重要的后果：**同一段同步代码触发的所有 then 回调，会在下一个 macrotask 之前连续执行完**。

举个例子：

```js
console.log('A');
Promise.resolve().then(() => console.log('B')).then(() => console.log('C'));
setTimeout(() => console.log('D'), 0);
console.log('E');
// 输出顺序: A E B C D
```

A 和 E 是当前 macrotask 的同步部分；当前 macrotask 结束后排 microtask，B 和 C 依次跑完；最后才轮到 setTimeout 的 macrotask，打 D。

这对 async/await 至关重要。当你写 `const x = await f();` 时，await 之后的代码（"continuation"）会被注册成 f 返回的 Promise 的 then 回调——而 then 回调走 microtask。这意味着 **await 醒来后的那段代码，几乎"紧贴"在 Promise resolve 那一刻执行，中间不会被任何 setTimeout、I/O、UI 事件插队**。这就是为什么 async/await 能给你"看起来同步"的体感——microtask 机制保证了控制流的连续性。

---

### **九、把 Promise 接回到主线：它在事件循环里到底扮演什么角色**

现在我们把它放回到上一轮那张大图里。整套异步栈从下到上是：

```
内核就绪通知   ──  epoll_wait 醒来
                    │
事件循环      ──  从就绪 fd 列表查到对应的 callback
                    │
Promise/Future ── callback 调 promise.resolve(data)
                    │
状态机回调    ──  promise 把 then 回调排进 microtask
                    │
async/await    ── then 回调里调 coro.send(data)，状态机往下走一格
```

**Promise 在这条链上的位置，是"内核事件"和"用户协程"之间的胶水层**。它向下接住运行时（libuv、asyncio、tokio reactor）从内核拿到的就绪通知，向上提供一个统一的、可链式、可组合、可 await 的抽象，让上层不用关心具体的事件源是 epoll、定时器、还是另一个协程的完成。

这也回答了你最初的疑问——"为什么前面跳到 Promise 有点突兀"。因为 Promise 不是一个独立的概念，它是**为了解决"异步操作没有返回值"这个根因而设计的中间层**，它的存在让事件循环的就绪通知能够以"值的形式"被消费，进而让编译器能在"等待这个值"的位置插入状态机切分点。没有 Promise/Future 这一层，async/await 在语义上就没有可以"await"的对象——你 await 一个 Promise，本质是 await 一个"未来会被事件循环写入结果的占位符"。

---

### **十、一句话归纳**

> **Promise 是把"未来才会有的值"对象化的抽象：它用一个三态状态机模拟了同步函数返回值的全部性质（可 return、可传递、可组合、错误能冒泡），用 then 返回新 Promise + 自动展平的设计把嵌套折叠成链，用 microtask 调度保证回调时机的确定性，从而让事件循环送来的就绪通知能以"值"的形式被上层代码消费——这一层一旦立住，async/await 才有东西可以"等"。**

理解 Promise 别只看 API，关键是抓住三件事：**它是一个对象**（所以能被 return、被传、被组合）、**它的 then 返回新 Promise 并自动展平**（所以能拍平嵌套）、**它的回调走 microtask**（所以 await 之后的代码紧贴 resolve 那一刻）。把这三件事在脑子里钉死，回头再看 Python 的 `asyncio.Future`、Rust 的 `Future` trait、C# 的 `Task<T>`，你会发现它们是同一个东西穿了不同语言的衣服——名字、惰性/急性、是否需要显式 poll、有没有 GC 这些都是工程取舍，**核心都是"把未来的值变成头等对象"这一个想法**。

# 

好,这次我们退一万步,从"那个年代的程序员遇到了什么具体的麻烦"讲起。等你感受到那个**痛**,Promise 的样子就会显得"非这样不可"。

---

## 一、痛在哪里?——回调地狱时代

JS 里大量操作是异步的:网络请求、定时器、读文件。它们的共同点是:**调用的那一刻,值还不在**。

那时候大家的写法就一种:把"想做的事"作为函数传进去,等好了你帮我调。

```js
getUser(1, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      render(comments);
    });
  });
});
```

这写法有几个具体的痛点,你体会一下:

**痛点 1:异步函数没法"返回"东西**

```js
function getUser(id) {
  setTimeout(() => { return { name: '小明' }; }, 1000);  // ❌ 这个 return 没用
}
const u = getUser(1);   // u 永远是 undefined
```

值是 1 秒后才有的,而 `getUser(1)` 这行**立刻就执行完了**。你想 `return` 也没东西可 return。所以只能反过来:你把"拿到值之后想干嘛"作为回调传给我。

**痛点 2:控制权交出去了**

`getUser(1, cb)` —— 这个 `cb` 什么时候被调,调几次,传什么参数,**全是 getUser 说了算**。它要是写得烂,调你两次,你的代码就崩。这叫"控制反转"。

**痛点 3:没法存、没法传、没法组合**

回调你没法存到变量里("某次异步操作的结果"这个东西不存在)。你也没法说"同时发 3 个请求,都好了再做某事"——只能层层嵌套。

---

## 二、关键的脑洞:**如果异步函数能返回一个"占位符"呢?**

某天有人想:既然 1 秒后才有值,那我先返回**一个代表"未来这个值"的对象**总行吧?

```js
const userPromise = getUser(1);    // 不是 user,是"未来的 user 的占位符"
```

这个占位符暂时是空壳,但它有个巨大的好处:**它是一个真实存在的对象**。你可以:

- 存到变量里 ✅
- 传给别的函数 ✅
- 在它身上挂方法 ✅

这就是 Promise 的根本动机。一句话:**让异步操作也能"返回值",哪怕返回的只是一个"将来会有值"的承诺。**

类比一下:你去餐厅点餐,菜还没做好。服务员给你一个**取餐器**(那个会震动的小方块)。取餐器不是菜,但它代表"你点的那份菜"。你可以拿着它去坐下、可以交给朋友、可以在上面"约定"什么——"它一响我就去拿菜"。

Promise 就是这个取餐器。

---

## 三、这个占位符身上要有什么?

你顺着这个比喻问自己:

**Q1: 取餐器要不要标识"现在是个啥状态"?** 要。三种:还没好(pending)、好了(fulfilled)、出问题了比如菜卖完了(rejected)。

→ `this.state`

**Q2: 好了之后,东西本身在哪儿存?** 得存起来,不然你来取我没东西给你。

→ `this.value`

**Q3: 谁有权把它从 pending 改成其他状态?** 做菜的那个厨房。也就是创建 Promise 时,执行那个异步操作的代码。所以构造函数里把改状态的"开关"(`resolve` / `reject`)交给它:

```js
new MyPromise((resolve, reject) => {
  // 这里是厨房,我做完菜就 resolve(菜)
  setTimeout(() => resolve('鱼香肉丝'), 1000);
});
```

**Q4: 我拿到取餐器,怎么"约定"它响了之后干嘛?**

得有个方法,让我能挂上我的"取餐计划":

```js
取餐器.then(菜 => 吃(菜));
```

→ 这就是 `.then`

---

## 四、`.then` 为什么写得那么纠结?

现在到了关键的地方。你挂这个"取餐计划"的时候,有两种可能:

**情况 A:菜已经好了,取餐器已经响过了。** 那简单,直接吃。

```js
if (this.state === 'fulfilled') onFul(this.value);
```

**情况 B:菜还没好,取餐器还在 pending。** 你**这一刻没东西可以传给 onFul**。但 onFul 又不能扔了——5 分钟后菜好了还得用它。

怎么办?**先记下来。** 拿个小本子(数组)把它写进去:

```js
this.onFulfilled.push(onFul);
```

——这就是那个数组的来历。它存在的**唯一原因**就是这个时间差:你想挂回调时,promise 还没 settle。

**Q5: 那写在小本子里的东西,谁来执行?**

当然是厨房做完菜、按下 `resolve` 的那一刻——它顺手把小本子翻一遍,挨个调:

```js
const resolve = (v) => {
  if (this.state !== 'pending') return;
  this.state = 'fulfilled';
  this.value = v;
  this.onFulfilled.forEach(cb => cb(v));   // ← 通知所有等着的人
};
```

整个故事就闭环了。

---

## 五、整理一下"它是怎么生出来的"

|遇到的问题|生出来的东西|
|---|---|
|异步操作没法 return 值|返回一个占位符对象 = Promise|
|占位符要表达"现在啥情况"|`state` 三种状态|
|占位符里要能装将来的值|`value`|
|谁能改状态?|构造函数把 `resolve`/`reject` 交给执行者|
|状态不能反复改|`if (state !== 'pending') return` 守卫|
|用户怎么注册"值来了之后干嘛"|`.then(onFul, onRej)`|
|注册时 promise 可能还没好|用数组把回调存起来|
|存起来的回调谁触发|`resolve`/`reject` 内部 `forEach` 调用|
|回调应该总是异步执行(规范)|`queueMicrotask` 包一层|
|executor 自己可能抛错|`try/catch` 包住|

每一行都是被一个**具体问题**逼出来的。这就是它"生出来"的过程。

---

## 六、回到你的感觉:为什么看完还是写不出?

我猜你的卡点是:**你是在"看代码"而不是在"看问题"。**

代码是答案,问题才是根。当你以后想自己实现一个东西时,试着这样问自己:

1. 我要解决什么用户场景?(异步值传递)
2. 这个场景下最简单的接口长什么样?(`p.then(cb)`)
3. 这个接口要工作,内部最少要存什么?(state、value、待执行回调)
4. 各种调用时序,会不会有"这一刻我没东西可用"的情况?(then 先于 resolve)
5. 那种情况下怎么把信息推迟到能用的时刻?(存起来,等触发)

写代码不是"想出每一行",而是"问出每一行"。

你试着合上我刚才写的代码,自己拿白纸,只写第一步——"Promise 是个占位符,它要有 state 和 value"——然后一步步问下去。卡住了告诉我你卡在哪个问题上,我们针对那个问题再聊。这样进步比看十遍解释都快。