### **glob 是 Unix 世界里"用通配符来描述一组文件路径"的古老机制——比如 `*.js`、`src/**/*.ts` 这种写法——而 fast-glob、globby 这些是 Node.js 生态里把这套机制工程化、性能化、跨平台化之后的现代库,几乎所有前端构建工具(webpack、Vite、ESLint、Prettier、tsc...)在"找文件"这一步,底层都靠它们。**

你已经懂了 grep 是"过滤文本流",那 glob 就是它的孪生兄弟——**过滤文件路径**。grep 处理的是"一行一行的文本",glob 处理的是"一个一个的文件名"。两者背后的世界观、历史脉络、甚至命名美学都一脉相承。下面我从最底层讲起,把 glob 这件事彻底讲清楚。

---

### **第 1 层:glob 这个词到底是什么意思**

很多人用了多年 `*.txt` 都不知道这种写法的正式名字叫 glob,更不知道这个词从哪来。

**glob 是 "global" 的缩写**,源自 1969 年 Unix 早期的一个独立小程序,名字就叫 `/etc/glob`。它的工作是:**把命令行里的通配符(比如 `*.c`)展开成一个真实的文件名列表**。

那时候 shell(命令解释器)本身还很弱,处理不了通配符。当你输入:

```
ls *.c
```

shell 自己看不懂 `*.c`,于是它会先调用 `/etc/glob` 这个外部程序,让它把 `*.c` 展开成 `main.c foo.c bar.c`,再把展开结果交给 `ls`。这个"全局展开文件名"的动作,就被叫做 **globbing**,而那个程序本身被叫做 **glob**。

后来(大约 1970 年代中期),这个功能被直接吸收进 shell 内部,`/etc/glob` 这个独立程序消失了,但**"glob"这个名字、这套语法、这个概念**留了下来,并扩散到所有类 Unix 系统、几乎所有编程语言的标准库里。

所以现在当我们说"glob 模式"(glob pattern),指的就是 `*.js`、`src/**/*.ts`、`!**/*.test.js` 这一类**用通配符描述路径**的写法。

---

### **第 2 层:glob 语法到底有哪些通配符**

glob 的语法刻意设计得比正则表达式**简单得多**——因为它的目标用户是普通用户,而不是程序员。一共就那么几个核心符号:

| 符号 | 含义 | 例子 |
|------|------|------|
| `*` | 匹配除 `/` 外的任意字符(0 个或多个) | `*.js` 匹配 `a.js`、`foo.js`,但不匹配 `dir/a.js` |
| `?` | 匹配除 `/` 外的任意单个字符 | `?.txt` 匹配 `a.txt`,不匹配 `ab.txt` |
| `[abc]` | 匹配方括号里任意一个字符 | `[ab].js` 匹配 `a.js`、`b.js` |
| `[a-z]` | 字符范围 | `[0-9].log` 匹配 `1.log`\~`9.log` |
| `[!abc]` 或 `[^abc]` | 排除 | `[!a].js` 匹配除 `a.js` 外的单字符 .js |
| `**` | **跨目录递归匹配**(扩展语法) | `src/**/*.ts` 匹配 src 下任意深度的 .ts |
| `{a,b,c}` | **任选一个**(花括号扩展) | `*.{js,ts,jsx}` 匹配三种后缀 |
| `!pattern` | 排除模式(库特性,非原生 shell) | `!**/*.test.js` |

注意一个关键点:**`*` 不跨目录,`**` 才跨目录**。这是 glob 最容易踩坑的地方:

- `*.js` 只匹配**当前目录**下的 .js 文件
- `**/*.js` 才匹配**所有子目录**下的 .js 文件

跟正则表达式对比一下,你会立刻看出差异:

| 需求 | glob | 正则 |
|------|------|------|
| 任意字符 | `*` | `.*` |
| 单个字符 | `?` | `.` |
| 字符集 | `[abc]` | `[abc]` |
| 任选项 | `{a,b}` | `(a\|b)` |

glob 不支持量词、不支持分组捕获、不支持环视。**它故意残废**——因为文件名不需要正则那么强,简单才好用。

---

### **第 3 层:glob 在 shell 里到底是怎么工作的**

这一层是大多数人没意识到的:**当你在终端输入 `ls *.js` 时,真正"看到" `*.js` 的不是 ls,是 shell**。

执行流程是这样的:

1. 你敲下 `ls *.js`
2. **shell 先做 globbing**:它扫描当前目录,找到所有匹配 `*.js` 的文件,比如 `a.js`、`b.js`
3. shell 把命令重写成 `ls a.js b.js`,再执行
4. ls 收到的参数已经是展开后的文件名列表,它根本不知道你写的是 `*.js`

这件事有几个非常重要的后果:

**(1) 如果当前目录没有匹配的文件,行为很微妙**
默认情况下 shell 会**原封不动**把 `*.js` 传给 ls,于是 ls 会报错"找不到 *.js 这个文件"。这是著名的 bash 默认行为,可以通过 `shopt -s nullglob` 改成"展开成空"。

**(2) 想让命令自己处理通配符,要加引号**
比如:

```bash
find . -name *.js     # 错!shell 会先展开
find . -name '*.js'   # 对!引号阻止 shell 展开,find 自己处理
```

这是新手最容易困惑的地方之一。

**(3) glob 在 shell 里不递归(默认)**
原始 POSIX glob **没有 `**` 语法**。bash 4.0(2009 年)才加入 `globstar` 选项支持 `**`,zsh 更早就有。这就是为什么你会看到老脚本写 `find . -name "*.js"` 而不是 `ls **/*.js`——历史包袱。

---

### **第 4 层:为什么编程语言纷纷"重新实现"了 glob**

shell 的 globbing 只能在 shell 里用。但程序员在写 Python、Node.js、Go 程序时,经常需要同样的功能:**给我一个模式,返回所有匹配的文件路径**。

于是几乎所有语言都在标准库里实现了一份 glob:

- **Python**: `glob.glob('**/*.py', recursive=True)`
- **Go**: `filepath.Glob("*.go")`(标准库版本不支持 `**`,需要第三方库)
- **Ruby**: `Dir.glob('**/*.rb')`
- **PHP**: `glob('*.txt')`
- **Java**: `Files.newDirectoryStream(path, "*.java")`
- **C**: POSIX `glob(3)` 函数

这些实现的语法**大体相同,细节各有差异**——比如 `**` 是否支持、`{a,b}` 是否支持、是否区分大小写、是否匹配隐藏文件(以 `.` 开头的)等等。这种"方言分裂"在 Node.js 生态里被推到了极致,我们下一层就讲它。

---

### **第 5 层:Node.js 生态里 glob 工具的演化史**

这是你真正关心的部分。前端工程在过去十年爆炸式增长,而前端工具链对"高效地找一堆文件"有近乎苛刻的需求——webpack 启动时要扫上万个文件、ESLint 要 lint 整个项目、Vite 要 dev server 实时响应。glob 库就是在这个压力下迭代了一代又一代。

**第一代:`node-glob`(2010 年,Isaac Schlueter,npm 之父)**

[npm/node-glob](https://github.com/isaacs/node-glob) 是 Node.js 上最早的 glob 库,作者就是 npm 的创始人。它把 bash 的 glob 语法移植到 Node,加上 `**`、`{a,b}` 等扩展。**它是事实上的标准**,被早期几乎所有工具(包括 npm 本身、mocha、grunt、gulp)依赖。

但它有几个老问题:
- **基于回调**(那个年代还没流行 Promise/async)
- **单线程串行扫描**,大型项目慢
- 内部用了大量正则,启动开销大

**第二代:`minimatch`(同作者,glob 的"匹配核心")**

Isaac 把 glob 拆成两件事:
1. **遍历文件系统**(由 `glob` 包负责)
2. **判断一个路径字符串是否匹配某个 glob 模式**(由 [`minimatch`](https://github.com/isaacs/minimatch) 负责)

minimatch 单独成为了一个底层库,它把 glob 模式编译成正则表达式,然后用正则做匹配。npm 自己的 package.json 里 `"files"` 字段、`.npmignore` 都用 minimatch 解析。**直到今天,minimatch 仍然是 npm 生态里最被广泛依赖的包之一**(每周下载量数十亿次)。

**第三代:`micromatch`(2016 年,Jon Schlinkert)**

[micromatch](https://github.com/micromatch/micromatch) 是对 minimatch 的重写,目标是**更快、更准、特性更全**。它的优势:
- 比 minimatch 快好几倍(避免了一些昂贵的正则操作)
- 支持更多扩展语法(extglob:`@(a|b)`、`+(a|b)` 等)
- 模块化:把 brace expansion、extglob、nanomatch 拆成独立小包

micromatch 后来成为新一代 glob 库的"匹配引擎"底层。

**第四代:`fast-glob`(2017 年,Denis Malinochkin)——你问的主角**

[fast-glob](https://github.com/mrmlnc/fast-glob) 是目前 Node.js 生态里**性能最强、用得最广**的 glob 库。它解决了 node-glob 的所有痛点:

1. **基于 Promise / Stream / 同步**三种 API,可以 `await fg('**/*.js')`
2. **底层用 micromatch 做模式匹配**,继承了所有性能红利
3. **优化文件系统遍历策略**:批量读目录、合理利用 `fs.readdir` 的 `withFileTypes` 选项一次拿到文件类型(避免额外的 stat 调用)
4. **早期剪枝**:能从模式里推断出"哪些目录根本不用进"。比如 `src/**/*.ts`,它绝不会去爬 `node_modules/`、`dist/`(只要这些目录不在 src 下)
5. **支持负向模式做 ignore**:`fg(['**/*.js', '!**/*.test.js'])`
6. **跨平台**:Windows 下的反斜杠 / 正斜杠问题、大小写问题都处理得当

它的速度有多猛?在一个有上万文件的项目里,fast-glob 比 node-glob 快 **3\~10 倍**,比 Python 的 `glob.glob` 快得多得多。

**第五代周边:`globby`(便利封装)**

[globby](https://github.com/sindresorhus/globby) 是 [Sindre Sorhus](https://github.com/sindresorhus)(Node.js 生态最高产的开发者之一)在 fast-glob 之上的便利封装。它提供:
- 自动读取 `.gitignore`(对,跟 ripgrep 一样的思路)
- 更友好的多模式数组 API
- 内置常用的"忽略 node_modules"等默认值

很多工具(比如 [Prettier](https://prettier.io)、ESLint、Husky)用的就是 globby。

**最新一代:`tinyglobby`(2024 年)**

[tinyglobby](https://github.com/SuperchupuDev/tinyglobby) 是最新趋势:**fast-glob + globby 在很多场景下其实带了过多依赖**。tinyglobby 用极小的依赖、更快的速度,提供 fast-glob 兼容 API。Vite、Vitest 这些前沿工具已经在迁移到它。这是 JS 生态典型的"轻量化轮回"——几年一次的"造一个更小更快的轮子"运动。

---

### **第 6 层:fast-glob 凭什么这么快——拆开看**

把 fast-glob 拆开,可以看到它的优化思路其实和 ripgrep 异曲同工:

**(1) 模式预分析(Pattern Analysis)**

fast-glob 收到 `src/**/*.ts` 后,不会傻乎乎地从根目录开始爬。它会分析模式:
- 提取出**静态前缀** `src/` —— 这部分是确定的,直接 `cd` 过去
- `**` 之后才是动态部分

于是它从 `src/` 开始爬,而不是从 `.` 开始。这一步就能节省 95% 的工作量。

**(2) 一次系统调用拿全信息**

老式 glob 库的循环大致是:
```
for each entry in readdir(dir):
    stat(entry)  // 系统调用!判断是文件还是目录
    if is_dir: recurse
    if is_file and matches: collect
```

每个文件都 `stat` 一次,系统调用开销巨大。fast-glob 用 [`fs.readdir(dir, {withFileTypes: true})`](https://nodejs.org/api/fs.html#fsreaddirpath-options-callback),**一次调用**就拿到所有 entry 的类型(文件/目录/符号链接),省掉每个文件的额外 stat。在大型项目里这是数量级的差异。

**(3) 并发 IO**

Node.js 的 `fs` 异步 API 天然非阻塞。fast-glob 把多个目录的 readdir 并发发出去,让操作系统/磁盘队列去调度,而不是串行等待。

**(4) 动态正则 vs 字符串快速路径**

如果你的模式是纯字面量(没有任何通配符),fast-glob 会走"快速路径",直接用 `===` 比较,不动用正则。

**(5) 负向模式提前剪枝**

`fg(['**/*.js', '!node_modules/**'])` 时,fast-glob 在递归到 `node_modules` 之前就判断"这整个目录都被排除",直接不进去——而不是进去之后再一个个过滤。

把这五点叠加,你就理解了为什么 fast-glob 能在万级文件项目里毫秒级完成扫描。

---

### **第 7 层:你每天都在用它,只是不知道**

跟 ripgrep 一样,fast-glob 已经是 Node.js 生态的**隐形基础设施**:

- **Vite**:dev server 启动时扫描 entry、HMR 时找受影响的文件
- **webpack**:`CopyWebpackPlugin`、`entry: './src/**/*.js'` 等场景
- **ESLint**:`eslint 'src/**/*.{js,ts}'` 这种命令行参数的展开
- **Prettier**:`prettier --write '**/*.{js,ts,css,md}'`
- **Jest / Vitest**:测试文件发现(`**/*.test.ts`)
- **TypeScript**:`tsconfig.json` 里的 `"include": ["src/**/*"]`
- **npm**:`package.json` 的 `"files"` 字段(用 minimatch)
- **Tailwind CSS**:`content: ['./src/**/*.{html,js}']` 扫描类名
- **Nx / Turborepo / Rush** 等 monorepo 工具

打开你随便一个前端项目的 `node_modules`,搜一下 `fast-glob` 或 `micromatch`,你会发现几乎每个稍微有点规模的工具都直接或间接依赖它。

---

### **第 8 层:实战示例——把它具象化**

```javascript
import fg from 'fast-glob';

// 最基础:找当前目录所有 .js
const files = await fg('*.js');

// 递归找 src 下所有 .ts 和 .tsx
const tsFiles = await fg('src/**/*.{ts,tsx}');

// 多模式 + 排除
const sources = await fg([
  'src/**/*.ts',
  '!**/*.test.ts',
  '!**/node_modules/**'
]);

// 流式处理(大项目内存友好)
const stream = fg.stream('**/*.js');
for await (const path of stream) {
  console.log(path);
}

// 同步版本
const syncFiles = fg.sync('**/*.md');

// 拿到完整 Entry 信息(stats、dirent)
const entries = await fg('**/*.js', {
  stats: true,
  onlyFiles: true,
  ignore: ['**/node_modules/**']
});
```

每个选项都对应前面讲过的某个设计点,你应该都能"看懂为什么"。

---

### **第 9 层:glob 与正则、grep、ripgrep 的关系图**

把这些东西放在一张图里,你会看到一个完整的版图:

```
                  描述"形状"的语言
       ┌─────────────────────────────────┐
       │                                  │
  glob 模式                          正则表达式
  (匹配路径,简单)                  (匹配文本,强大)
       │                                  │
       ▼                                  ▼
 ┌──────────┐                      ┌──────────┐
 │ glob 工具 │                      │  grep 类  │
 │          │                      │          │
 │ shell *  │                      │   grep    │
 │ node-glob│                      │   ack     │
 │ fast-glob│                      │   ag      │
 │ globby   │                      │ ripgrep   │
 │ tinyglob │                      │           │
 └──────────┘                      └──────────┘
       │                                  │
       ▼                                  ▼
   "找文件"                          "找内容"
```

ripgrep 之所以那么实用,正是因为它**同时把两边都做对了**:它内部也实现了一套完整的 gitignore-aware 文件遍历(类似 fast-glob 的工作),再在每个文件里跑高性能正则匹配(grep 的工作)。所以 ripgrep 可以理解为 **"glob + grep" 的合体最终形态**。

而 fast-glob 这类纯 glob 工具,定位是**只做找文件这一半**——因为它要给上游的成千上万个工具(Vite、ESLint、Jest...)当地基,这些工具拿到文件列表后,各自有自己的事要做(打包、检查、测试),不需要 glob 库管内容匹配。

---

### **第 10 层:它的局限——什么时候 glob 不够用**

为了认知完整,讲一下边界:

1. **glob 不能描述"内容"条件**。"找所有 import 了 lodash 的 .js 文件"——glob 做不到,得 glob 找文件 + grep/AST 找内容。
2. **glob 模式跨平台路径分隔符是个老大难**。Windows 的 `\` 和 Unix 的 `/` 让早期库踩了无数坑,fast-glob 现在统一用 `/`,但代码里写路径时仍要小心。
3. **超大项目的实时监听不是 glob 的活**。监听文件变化需要 [chokidar](https://github.com/paulmillr/chokidar)、`fs.watch`,glob 只负责"这一刻有哪些文件"。
4. **不同库的 glob 方言略有差异**。比如 `**` 在 minimatch、micromatch、Python glob、bash globstar 下的行为细节都有微小区别,跨工具迁移要测试。

---

### **一句话回到顶层**

把这十层叠起来,glob 这件事就是:

> **1969 年 Unix 为了让用户不用敲完整文件名而发明的"路径通配符语言",经过 50 年演化,变成了今天前端工程里"让构建工具知道该处理哪些文件"的事实标准接口。fast-glob 是这条历史长河在 Node.js 时代的性能巅峰实现。**

下次你在 Vite 配置里写 `'src/**/*.{ts,tsx}'`、在 ESLint 命令里写 `'**/*.js'`、在 .gitignore 里写 `dist/**`,你写的都是 1969 年那个叫 `/etc/glob` 的小程序留下的遗产。一行通配符背后,是半个世纪的工程沉淀。