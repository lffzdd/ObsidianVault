# 构建正则表达式的步骤

**Created:** 6/3/2026, 1:05:31 AM
**Updated:** 6/3/2026, 2:21:21 AM
**Exported:** 6/3/2026, 2:38:41 AM
**Model:** claude-opus-4-8
**Link:** [https://claude.ai/chat/8abe44b0-3837-439a-9fc4-3015e2b16d94](https://claude.ai/chat/8abe44b0-3837-439a-9fc4-3015e2b16d94)

---

## User
**2026-06-03T08:05:32.476Z**

这样看来，还需要一段，就是读入regex字符串的时候把它build成regex，我发现好像还挺复杂的，不知道要怎么做，你一步步教我，不必一次性回答完，把我当成初学者

### Attachment: 1780473870543_level5.py _(3.5 KB, text/x-python-script)_
````python
StateId = int
Char = str | None
EPS = None


class Frag:
    def __init__(
        self,
        start_id: StateId,
        accept_id: StateId,
    ) -> None:
        self.start_id = start_id
        self.accept_id = accept_id


class Builder:
    def __init__(self) -> None:
        self.transition: dict[StateId, dict[Char, set[StateId]]] = {}
        self._next_state_id = 0

    def new_state(self) -> StateId:
        state_id = self._next_state_id
        self._next_state_id += 1
        self.transition[state_id] = {}
        return state_id

    def add_edge(self, frm_id: StateId, char: Char, to_id: StateId):
        self.transition[frm_id].setdefault(char, set()).add(to_id)

    def lit(self, char: str) -> Frag:
        start_id = self.new_state()
        accept_id = self.new_state()
        self.add_edge(start_id, char, accept_id)
        return Frag(start_id, accept_id)

    def concat(self, left_frag: Frag, right_frag: Frag) -> Frag:
        self.add_edge(left_frag.accept_id, EPS, right_frag.start_id)
        return Frag(left_frag.start_id, right_frag.accept_id)

    def union(self, left_frag: Frag, right_frag: Frag) -> Frag:
        start_id = self.new_state()
        accept_id = self.new_state()
        self.add_edge(start_id, EPS, left_frag.start_id)
        self.add_edge(start_id, EPS, right_frag.start_id)
        self.add_edge(left_frag.accept_id, EPS, accept_id)
        self.add_edge(right_frag.accept_id, EPS, accept_id)
        return Frag(start_id, accept_id)

    def star(self, frag: Frag) -> Frag:
        start_id = self.new_state()
        accept_id = self.new_state()
        self.add_edge(start_id, EPS, frag.start_id)
        self.add_edge(frag.accept_id, EPS, frag.start_id)
        self.add_edge(frag.accept_id, EPS, accept_id)
        self.add_edge(start_id, EPS, accept_id)

        return Frag(start_id, accept_id)

    def compile(self, frag: Frag) -> "NFA":
        # 拷贝转移表，编译出的 NFA 不再受后续 builder 操作影响
        transition = {
            state: {char: set(targets) for char, targets in edges.items()}
            for state, edges in self.transition.items()
        }
        # transition=copy.deepcopy(self,transition) 无脑且正确
        return NFA(transition, frag.start_id, frag.accept_id)


class NFA:
    def __init__(
        self,
        transition: dict[StateId, dict[Char, set[StateId]]],
        start_id: StateId,
        accept_id: StateId,
    ) -> None:
        self.transition = transition
        self.start_id = start_id
        self.accept_id = accept_id

    def _eps_closure(self, states: set[StateId]) -> set[StateId]:
        closure = set(states)
        stack = list(states)
        while stack:
            state = stack.pop()
            for to_id in self.transition[state].get(EPS, ()):
                if to_id not in closure:
                    closure.add(to_id)
                    stack.append(to_id)
        return closure

    def run(self, text: str) -> bool:
        current_states = self._eps_closure({self.start_id})
        for char in text:
            next_states = set()
            for state in current_states:
                next_states.update(self.transition[state].get(char, ()))

            current_states = self._eps_closure(next_states)
        return self.accept_id in current_states


builder = Builder()
frag_a = builder.lit("a")
frag_ab = builder.concat(builder.lit("a"), builder.lit("b"))
frag_a_or_ab = builder.union(frag_a, frag_ab)
regex = builder.star(frag_a_or_ab)

nfa = builder.compile(regex)
print(nfa.run("ababab"))

````

## Claude
**2026-06-03T08:06:34.215Z**

### Thinking
````
 The user has built an NFA regex engine with the core Builder/Frag/NFA machinery working, but they're missing the parser that converts a regex string into the appropriate build operations like lit, concat, union, and star. They're asking me to teach this step-by-step as a beginner would learn it, and they've explicitly requested I don't try to answer everything at once.

The key insight is that regex parsing follows a standard grammar with operator precedence: alternation (|) binds loosest, then concatenation, then the Kleene star (*) binds tightest. I should start by explaining this conceptual foundation and the grammar structure before diving into implementation, keeping things incremental so they can absorb each piece. I'm thinking about how to structure this explanation for a beginner who wants step-by-step guidance. The core issue is that a regex string is flat, but it has nested hierarchical structure — like `(a|ab)*` where the star applies to a union, which itself contains concatenations. Parsing recovers this tree from the flat input. The key stumbling block for beginners is operator precedence: in `ab*`, the star binds only to `b`, not the whole sequence, and this precedence rule needs to be encoded in the grammar. I should start by framing the problem clearly, explain why precedence matters, introduce the grammar approach, then stop at a natural checkpoint rather than overwhelming them with the full solution at once. I need to be thoughtful about pacing here—they want me to build the mental model first without overwhelming them. I'll structure this as: establish what parsing really means (recovering structure from a flat string), walk through the precedence rules with concrete examples to show why they matter, then present the grammar rules that encode those precedences. I'll end by outlining the overall plan and showing where the code will live, but hold off on implementing the full parser methods until they're ready. The layered grammar that encodes precedence, mapped to their existing functions. I'll write a small recursive-descent parser, one layer at a time, starting with the scaffolding: a Parser class that tracks position in the string with helper methods like peek and eat. This gives us the mechanical foundation everything builds on before moving to the recursive methods.

Now I'm mapping out the grammar to match their functions—union for the `|` operator, concatenation for adjacent factors, star for the `*` operator, and literals for individual characters. I'll explain this clearly in Chinese with the grammar structure and then show the parser scaffolding inline, keeping the formatting minimal and conversational.

Writing the parser scaffolding... Setting up the Parser class...

I'm thinking through how to structure this explanation—I should give them the framing, precedence rules, and grammar first, then show the scaffolding code for the Parser class with peek and eat methods. After that, I'll explicitly tell them we're moving to parse_atom next as the foundation layer. This keeps the first message focused and digestible without overwhelming them, and I'll wrap it up in Chinese since that's what they're working in.
````

你已经把"积木"都搭好了（`lit`、`concat`、`union`、`star`），现在缺的这一步，本质上是要把一个**扁平的字符串**变回一棵**有层次的结构树**，然后照着这棵树去调用你的那些函数。这个过程叫**解析（parsing）**。

我们先不写代码，先把脑子里的图景建立起来，这是最关键的一步。

## 为什么这件事"看起来复杂"

字符串是平的，一个字符接一个字符；但正则的含义是有嵌套结构的。比如 `(a|ab)*` 真正的意思是：

```
star
 └─ union
     ├─ a
     └─ concat(a, b)
```

你要做的，就是从 `( a | a b ) *` 这串字符，恢复出上面这棵树。难点几乎全部来自一个东西：**优先级**。

看两个例子，先别往下看答案，自己想一下：

- `ab*` —— 这个 `*` 是作用在 `b` 上，还是作用在 `ab` 上？
- `a|bc` —— 这个是 `(a) | (bc)`，还是 `(a|b) c`？

答案是：`*` 只管 `b`（即 `a` 接上 `b*`）；第二个是 `a | (bc)`。规律是这三个运算的**结合紧密程度（优先级）**不同：

```
*          最紧（先结合）
连接(concat) 其次
|          最松（最后结合）
```

## 怎么把"优先级"变成代码

这里有个非常经典、非常优雅的套路：**把优先级写成分层的语法规则**，越松的运算放在越外层。每一层对应一个函数，函数之间互相调用，从外到内一层层"剥"下去：

```
expr   = term ('|' term)*      ← 最外层，处理 |，对应 builder.union
term   = factor+               ← 处理连接，对应 builder.concat
factor = atom '*'*             ← 处理 *，对应 builder.star
atom   = 字符  |  '(' expr ')' ← 最里层，对应 builder.lit 或 递归回 expr
```

注意最后一行：遇到括号时，又跳回最外层的 `expr`——这就是为什么括号能强行改变优先级。这种"函数照着语法层层调用"的写法叫**递归下降解析**，是手写解析器里最好懂的一种。

我们的计划就是：从下往上，把这四个函数一个一个填出来，每填一个就跑一下验证。

## 第一步：先搭一个"游标"骨架

在写那四个函数之前，需要一个最基础的东西：一个能记住"我现在读到字符串第几个位置"的小工具。所有解析函数都靠它来读字符。

```python
class Parser:
    def __init__(self, pattern: str, builder: Builder) -> None:
        self.pattern = pattern
        self.pos = 0          # 当前读到哪个位置
        self.builder = builder

    def peek(self) -> str | None:
        # 偷看当前字符，但不移动游标；到末尾了就返回 None
        if self.pos < len(self.pattern):
            return self.pattern[self.pos]
        return None

    def eat(self) -> str:
        # 吃掉当前字符，游标往后挪一位，并把这个字符返回
        char = self.pattern[self.pos]
        self.pos += 1
        return char
```

`peek` 和 `eat` 的区别值得记一下：`peek` 是"探头看一眼"用来做决定（比如"接下来是不是 `|`？"），`eat` 是"真的把它读走"。整个解析过程就是不停地 peek 一下、决定要干嘛、然后 eat 掉若干字符。

先把这段抄进去、读懂它在干嘛。理解了之后告诉我，我们就从最里层的 `atom`（最简单的那个）开始往上填。

## User
**2026-06-03T08:20:14.712Z**

下一步

## Claude
**2026-06-03T08:20:47.347Z**

### Thinking
````
 Now I'm moving into the bottom-up construction of the parser, starting with the `atom` rule since it's the simplest layer. I need to implement parsing for either a single character or a parenthesized expression, which will form the foundation for the more complex rules above it.

There's a mutual recursion issue here—`atom` needs to call `expr` for handling parentheses, but `expr` doesn't exist yet in the bottom-up approach. The cleanest way forward is to write `atom`'s logic and test it on simple characters first, then let the parenthesis branch call `expr` when we get there. Python won't complain until that code path actually executes, so we can build incrementally and test the full parser once all four functions are in place.

Writing the atom function... Writing test code...
````

好。我们从最里层 `atom` 开始往上爬。`atom` 负责处理两种"最小单元"：一个普通字符，或者一对括号包起来的东西。

把这个方法加到 `Parser` 类里：

```python
def atom(self) -> Frag:
    char = self.peek()
    if char == "(":
        self.eat()           # 吃掉 '('
        frag = self.expr()   # 括号里面又是一个完整正则，跳回最外层
        self.eat()           # 吃掉 ')'
        return frag
    else:
        self.eat()           # 吃掉这个普通字符
        return self.builder.lit(char)
```

两个分支分别对应语法里的 `atom = char | '(' expr ')'`：

普通字符的分支很直接——读走它，交给你已经写好的 `builder.lit` 变成一个片段。

括号分支是整个解析器里最妙的地方：看到 `(` 就先吃掉它，然后调用 `self.expr()`——也就是回到**最外层**重新解析括号里的内容，解析完再吃掉 `)`。这一跳就是括号能"重置优先级"的原因。

这里有个你可能会困惑的点要先说清楚：`atom` 里调用了 `self.expr()`，但 `expr` 我们还没写。**这在 Python 里没问题**，因为 `self.expr` 是在真正执行到那一行时才去查找的，不是定义 `atom` 时就要求存在。所以只要我们现在测试时**先不输入括号**，就不会碰到那行，也就不会报错。

来跑一下，验证普通字符这条路通了：

```python
builder = Builder()
parser = Parser("a", builder)
frag = parser.atom()

nfa = builder.compile(frag)
print(nfa.run("a"))   # True
print(nfa.run("b"))   # False
print(nfa.run("aa"))  # False
```

如果这三行输出是 `True / False / False`，说明 `atom` 的字符分支工作正常。

顺便提一句：我们现在**完全不做错误检查**（比如括号没配对、字符串提前结束），也不处理转义。这是故意的——先让主干跑起来，健壮性以后再补。

读懂、跑通了告诉我，下一步我们写 `factor`，也就是处理 `*` 的那一层。

## User
**2026-06-03T08:35:47.568Z**

下一步

## Claude
**2026-06-03T08:36:05.993Z**

### Thinking
````
 Now I'm implementing the `factor` function, which handles the `*` operator by parsing an atom and then repeatedly wrapping it with the star operation for each `*` encountered in the input.

接下来是 `term`（连接）。
````

继续往上一层：`factor`，处理 `*`。对应语法 `factor = atom '*'*`，意思是"一个 atom，后面跟着零个或多个 `*`"。

```python
def factor(self) -> Frag:
    frag = self.atom()
    while self.peek() == "*":
        self.eat()                    # 吃掉 '*'
        frag = self.builder.star(frag)
    return frag
```

逻辑很短：先解析出一个 `atom`，然后**不断往后看**——只要下一个字符是 `*`，就吃掉它，并把当前片段用 `star` 包一层。

为什么用 `while` 循环而不是一个 `if`？因为 `a**` 是合法的，意思是对 `a` 连续套两层 star。用循环就能自然地处理任意多个 `*`：第一次循环把 `a` 包成 `a*`，第二次再把 `a*` 包成 `(a*)*`。

这里也藏着优先级的实现：`factor` 是调用 `atom` 来拿"被 `*` 作用的对象"的，而 `atom` 只会是单个字符或一对括号。所以 `*` 永远只能粘住它紧挨着的那个最小单元——这正是 `*` 优先级最高的体现。等下一步写 `term` 时你会看到，`ab*` 里的 `b*` 会先在 `factor` 这层抱成一团，然后才轮到连接。

跑一下验证：

```python
builder = Builder()
parser = Parser("a*", builder)
frag = parser.factor()

nfa = builder.compile(frag)
print(nfa.run(""))      # True   零个 a
print(nfa.run("a"))     # True
print(nfa.run("aaaa"))  # True
print(nfa.run("b"))     # False
```

输出应该是 `True / True / True / False`。注意第一个空字符串也是 `True`，因为 `a*` 允许零个 `a`——这说明 star 的语义是对的。

通了告诉我，下一步写 `term`，也就是把多个 factor 拼起来的"连接"层。

## User
**2026-06-03T08:42:46.325Z**

下一步

## Claude
**2026-06-03T08:43:06.750Z**

### Thinking
````
 Now I'm implementing the `term` layer, which chains together one or more factors using concatenation. The key is recognizing when to stop parsing—I need to keep reading factors as long as the next character isn't a pipe, closing paren, or end of input, since those signal the end of a term.

I'm folding the factors left-to-right with `concat`, so "abc" becomes `concat(concat(a,b),c)`. Then I'm testing with simple cases like "ab" and "ab*" to verify the precedence works correctly.

现在用"ab*"验证星号只绑定到`b`而不是整个"ab"，这样就确认了运算符优先级。接下来要实现`expr`层来处理并集和括号，这样整个解析器就完整了。
````

上一层：`term`，把连续的若干个 `factor` 拼接起来。对应语法 `term = factor+`，意思是"一个或多个 factor 连在一起"。

```python
def term(self) -> Frag:
    frag = self.factor()
    while self.peek() is not None and self.peek() not in ("|", ")"):
        right = self.factor()
        frag = self.builder.concat(frag, right)
    return frag
```

这一层最值得琢磨的是那个**循环停止条件**。连接是"贪心"的——只要后面还有东西，就继续往右接。那什么时候算"没东西可接了"？三种情况：

字符串读完了（`peek()` 是 `None`）；遇到 `|`，它属于上一层的 union，不归我们管；遇到 `)`，说明括号要收尾了，那个 `(` `)` 是当初调用 `atom` 的人的事。碰到这三种里的任何一种，`term` 就收手，把后续交给外层处理。

拼接的方式：因为你的 `concat` 一次只吃两个片段，所以我们**从左往右折叠**——先拿到第一个 factor，再把后面每一个依次 `concat` 上去。`abc` 就变成 `concat(concat(a, b), c)`。

先测纯连接：

```python
builder = Builder()
parser = Parser("ab", builder)
frag = parser.term()
nfa = builder.compile(frag)
print(nfa.run("ab"))   # True
print(nfa.run("a"))    # False
print(nfa.run("abb"))  # False
```

再测一个能验证优先级的 `ab*`：

```python
builder = Builder()
parser = Parser("ab*", builder)
frag = parser.term()
nfa = builder.compile(frag)
print(nfa.run("a"))      # True    b 出现零次
print(nfa.run("ab"))     # True
print(nfa.run("abbbb"))  # True
print(nfa.run("aab"))    # False
```

第二组里 `a` 单独就能匹配、`abbbb` 也行——这证明 `*` 确实只粘住了 `b`，而不是整个 `ab`。优先级分层起作用了。

通了告诉我。下一步是最后一块 `expr`（处理 `|`），写完之后整个解析器就完整了，括号也会跟着活过来。

## User
**2026-06-03T09:20:52.795Z**

下一步

## Claude
**2026-06-03T09:21:21.073Z**

### Thinking
````
 Now I'm implementing the top-level expression parser that handles the union operator, which chains multiple terms together with the `|` operator and combines them using the builder's union method. This completes the recursive descent parser since the atom rule can now properly resolve calls back to expr, making parenthesized expressions work correctly.

I'm wrapping everything in a convenience function that instantiates the builder and parser, then returns the compiled NFA. Testing it against patterns like `(a|ab)*` to verify the whole grammar works end-to-end. 我现在要展示完整的表达式解析函数，解释循环如何闭合，给出包装器和测试用例。这样就完成了核心部分。最后我会简要回顾整个结构，提一下故意留出的部分（错误处理、转义、`+`、`?`、字符类）作为后续可能的扩展——但要保持简洁，不要让用户感到压倒。
````

最外层 `expr`，处理 `|`。对应语法 `expr = term ('|' term)*`，意思是"一个 term，后面可以跟着若干个 `| term`"。这也是整个解析器的入口。

```python
def expr(self) -> Frag:
    frag = self.term()
    while self.peek() == "|":
        self.eat()                       # 吃掉 '|'
        right = self.term()
        frag = self.builder.union(frag, right)
    return frag
```

结构和 `term` 几乎一样：先解析一个 term，然后只要看到 `|` 就吃掉它、再解析一个 term，用 `union` 把两边并起来，同样是从左往右折叠。

到这里发生了一件大事：还记得 `atom` 里那行调用 `self.expr()` 吗？现在 `expr` 终于存在了，那行能跑通了，**括号也就跟着活过来了**。整个语法形成了一个闭环：

```
expr → term → factor → atom → （遇到括号）→ expr
```

最松的 `|` 在最外面，最紧的 `*` 在里面，括号能把你一脚踹回最外层——优先级就是靠这个调用顺序天然实现的。

再封装一个入口函数，省得每次手动 `new` 一堆东西：

```python
def compile_regex(pattern: str) -> NFA:
    builder = Builder()
    parser = Parser(pattern, builder)
    frag = parser.expr()
    return builder.compile(frag)
```

现在跑你文件里那个原始例子：

```python
nfa = compile_regex("(a|ab)*")
print(nfa.run("ababab"))  # True
print(nfa.run("aab"))     # True
print(nfa.run("ba"))      # False
print(nfa.run(""))        # True
```

再验证一下 union 的优先级，`a|bc` 应该被理解成 `a | (bc)` 而不是 `(a|b)c`：

```python
nfa = compile_regex("a|bc")
print(nfa.run("a"))    # True
print(nfa.run("bc"))   # True
print(nfa.run("ab"))   # False
print(nfa.run("b"))    # False
```

如果这些都对上了，恭喜——你现在有一个能从字符串编译的完整正则引擎了。

回头看整件事其实就三步：用 `peek`/`eat` 当游标，照着分层语法写四个互相调用的函数，每个函数把活儿交给你早就写好的 `lit`/`concat`/`union`/`star`。难的从来不是代码，是想清楚"优先级 = 调用层次"这件事。

如果之后想继续加 level，几个自然的方向（现在故意没做的）：错误检查（括号不配对、`)` 缺失时给出报错而不是崩溃）、转义字符 `\*` 让 `*` 当普通字符用、以及补上 `+`（一个或多个）和 `?`（零个或一个）这两个运算符——它们和 `*` 同一层，加进 `factor` 就行。需要的话随时叫我。

