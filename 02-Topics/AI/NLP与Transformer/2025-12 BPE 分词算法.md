Byte Pair Encoding (BPE) 是一种简单而强大的**子词（Subword）分词算法**，它在现代自然语言处理（特别是大语言模型，如 GPT 系列、LLaMA）中占据核心地位。

为了满足你关于**历史背景、底层原理、数学公式及完整流程**的要求，我们将采用自底向上的方式进行拆解。

---

### 1. 历史背景与底层原理

#### 历史溯源：从压缩到分词

BPE 最初并非为了 NLP 发明，而是由 **Philip Gage 在 1994 年** 提出的一种**通用数据压缩算法**。其核心思想非常直观：**用一个未在数据中出现的字节，替换数据中出现频率最高的一对连续字节**。

#### NLP 领域的引入

在 2016 年之前，NLP 模型主要面临两个极端：

- **Word-level（词级）：** 词表太大，且无法处理 **OOV（Out-Of-Vocabulary，未登录词）**。例如，模型见过 "look"，但没见过 "looking"，它就不知道如何处理。
- **Character-level（字符级）：** 粒度太细，序列过长，且难以捕捉语义。

2016 年，Rico Sennrich 等人在论文 《Neural Machine Translation of Rare Words with Subword Units》 中将 BPE 引入 NLP。

核心理念： 单词是由更有意义的子结构（词根、前缀、后缀）组成的。BPE 寻找的是字符与单词之间的**“甜蜜点”（Sweet Spot），即子词（Subword）**。

---

### 2. BPE 算法的数学定义与初始化

#### 符号定义

- **语料库 (Corpus, $C$)**：待训练的文本数据。
- **词表 (Vocabulary, $V$)**：模型能识别的所有 Token 的集合。
- **目标词表大小 ($K$)**：这是一个超参数，决定了最终词表有多大。

#### 预处理 (Pre-tokenization)

在运行 BPE 之前，需要先对句子进行简单的预分词（通常按空格或标点切分），并给每个单词添加一个特殊的**终止符**（通常标记为 `</w>` 或 `_`）。

- _目的：_ 区分单词内部的字符序列和单词结尾的字符序列（例如 `est` 在 `lowest` 中和 `est` 作为单词是不同的）。

假设我们的语料库 $C$ 极其简单，只有以下单词及其频率：

- "hug": 10 次
- "pug": 5 次
- "pun": 12 次
- "bun": 4 次

初始状态：

我们将单词拆解为字符序列。

$$V_0 = \{h, u, g, p, n, b, </w>\}$$

当前语料库状态（以元组形式表示单词构成及频率）：

$$C_0 = \{ (h, u, g, </w>): 10, \quad (p, u, g, </w>): 5, \quad (p, u, n, </w>): 12, \quad (b, u, n, </w>): 4 \}$$

---

### 3. BPE 训练流程（Training）

BPE 的训练过程是一个**迭代合并（Iterative Merging）**的过程。

#### 步骤 1：统计频次 (Counting)

计算语料库中所有**相邻字符对（Symbol Pairs）**的共现频率。

数学表达：对于当前语料 $C_t$ 中的每一对相邻符号 $(x, y)$，计算其总频率：

$$Freq(x, y) = \sum_{w \in C_t} count(w) \times \mathbb{I}((x, y) \in w)$$

其中 $\mathbb{I}$ 是指示函数，表示该对在单词 $w$ 中出现的次数。

在我们的例子中：

- `u` + `g` : 出现在 "hug" (10) + "pug" (5) $\rightarrow$ **15**
- `u` + `n` : 出现在 "pun" (12) + "bun" (4) $\rightarrow$ **16**
- `h` + `u` : 10
- `p` + `u` : 5 + 12 = 17
- `g` + `</w>` : 10 + 5 = 15
- ...以此类推

#### 步骤 2：选择最佳合并 (Selection)

选择频率最高的符号对 $(x_{best}, y_{best})$。

$$(x_{best}, y_{best}) = \operatorname*{arg\,max}_{(x,y)} Freq(x, y)$$

在例子中，最高频对是 `p` + `u`（17 次）或者 `u` + `n`（16 次）。假设我们按照某种排序规则先选了 `p` + `u`。

#### 步骤 3：执行合并与更新 (Merge & Update)

1. 更新词表： 将新符号 $z = x_{best}y_{best}$ 加入词表。
    
    $$V_{t+1} = V_t \cup \{ pu \}$$
    
2. **更新语料：** 将语料中所有的 $x_{best}, y_{best}$ 替换为 $z$。

更新后的语料 $C_1$：

- "hug": `h`, `u`, `g`, `</w>` (10)
- "pug": `pu`, `g`, `</w>` (5) $\leftarrow$ **变化**
- "pun": `pu`, `n`, `</w>` (12) $\leftarrow$ **变化**
- "bun": `b`, `u`, `n`, `</w>` (4)

#### 步骤 4：迭代 (Iteration)

重复步骤 1-3，直到词表大小达到预设值 $K$。

---

### 4. 具体的迭代演示

为了让你彻底理解，我们继续刚才的例子，直到完成几次关键合并。

**当前状态 $C_1$:**

- `h, u, g, </w>` (10)
- `pu, g, </w>` (5)
- `pu, n, </w>` (12)
- `b, u, n, </w>` (4)

下一轮统计 (Round 2):

我们要统计新的对：

- `u` + `n` (仅在 "bun" 中): 4
- `pu` + `n` (在 "pun" 中): 12
- `pu` + `g`: 5
- `u` + `g`: 10
- `n` + `</w>`: 12 (from pun) + 4 (from bun) = **16**
- `g` + `</w>`: 10 + 5 = 15

假设最高频是 n + </w> (16 次)。

Merge: 创建新 token n</w>。

Update $C_2$:

- ...
- "pun": `pu`, `n</w>` (12)
- "bun": `b`, `u`, `n</w>` (4)

再下一轮 (Round 3):

现在的最高频可能是 pu + n</w> (12 次)。

Merge: 创建 pun</w>。

Update $C_3$:

- "pun": `pun</w>` (12) -> 此时 "pun" 变成了一个完整的 Token。

---

### 5. 推理阶段 (Inference/Tokenization)

训练完成后，我们得到了一张有序的**合并规则表 (Merge Rules)**，例如：

1. `p` + `u` $\rightarrow$ `pu`
2. `n` + `</w>` $\rightarrow$ `n</w>`
3. `pu` + `n</w>` $\rightarrow$ `pun</w>`

当给模型一个新单词（例如 "thugs"）时：

1. **拆分：** `t`, `h`, `u`, `g`, `s`, `</w>`
2. **应用规则：** 按照优先级（训练时的合并顺序）遍历规则。
    
    - 如果规则里有 `u`+`g` $\rightarrow$ `ug`，则合并。
    - 如果规则里有 `ug`+`s` $\rightarrow$ `ugs`，则合并。
        
3. **停止：** 当没有规则适用时，剩下的符号序列即为 Token 序列。

如果遇到完全陌生的字符（例如 emoji 😂），通常会回退到 `<UNK>` (Unknown) 或者字节级处理（Byte-level BPE，如 GPT-2/3 采用的方式，将 UTF-8 字节视为基础字符，从而彻底消除 `<UNK>`）。

---

### 6. 代码实现 (Python)

为了确保可用性，我为你编写了一个最小化但完整的 BPE 实现代码。

Python

```
import re
from collections import Counter, defaultdict

def get_stats(vocab):
    """
    统计相邻字符对的频率
    vocab: dict, key是空格分隔的字符串(如 'h u g </w>'), value是频率
    """
    pairs = defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols) - 1):
            pairs[symbols[i], symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    """
    将词表中指定的字符对合并为一个新的符号
    pair: 要合并的字符对 ('p', 'u')
    v_in: 旧词表
    """
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    
    for word in v_in:
        # 将匹配到的 'p u' 替换为 'pu'
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

# 1. 数据准备
vocab = {
    'h u g </w>': 10,
    'p u g </w>': 5,
    'p u n </w>': 12,
    'b u n </w>': 4
}

# 2. 训练循环
num_merges = 5 # 假设我们想做5次合并
print(f"Initial Vocabulary: {vocab}")

for i in range(num_merges):
    pairs = get_stats(vocab)
    if not pairs:
        break
    
    # 获取频率最高的对
    best = max(pairs, key=pairs.get)
    print(f"\nStep {i+1}: Best pair {best} with freq {pairs[best]}")
    
    # 合并
    vocab = merge_vocab(best, vocab)
    print(f"Updated Vocabulary: {vocab}")

# 最终你会看到 'pu', 'un' 等子词的形成过程
```

### 7. 总结

BPE 的核心价值在于它权衡了**词表大小**和**序列长度**：

1. **常见词**（如 "the", "hug"）最终会被合并成一个完整的 Token，保证了编码效率。
2. **罕见词**（如 "transcendentalism"）会被切分为多个常见的子词（"trans", "cendent", "al", "ism"），保证了语义的泛化能力。
3. **数学本质**：通过贪心算法最大化语料的压缩率（或最小化编码长度）。

---

接下来我也能为你做的是：

是否需要我进一步解释 GPT-2/3 使用的 Byte-level BPE 具体是如何处理 Unicode 字符（如中文、Emoji）的？这在处理多语言模型时非常关键。