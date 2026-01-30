[[01-Daily Notes/2026-01-30/attachments/3d8945d254363cb27b408e08cff8d0ab_MD5.png|Open: Pasted image 20260130150729.png]]
![[01-Daily Notes/2026-01-30/attachments/3d8945d254363cb27b408e08cff8d0ab_MD5.png]]

[[01-Daily Notes/2026-01-30/attachments/f0dc45ea2721f75f18e2f3a20e3acbb3_MD5.png|Open: Pasted image 20260130150956.png]]
![[01-Daily Notes/2026-01-30/attachments/f0dc45ea2721f75f18e2f3a20e3acbb3_MD5.png]]

这份学习文档基于 StatQuest 的视频内容整理而成，继续保持 Josh Starmer 充满活力且条理清晰的风格，带你解锁 Bootstrapping 的进阶技能：**如何计算 P 值 (p-values)**。

---

# 📚 StatQuest 学习笔记：用 Bootstrapping 计算 P 值

**视频来源**: [Using Bootstrapping to Calculate p-values!!!](https://www.youtube.com/watch?v=N4ZQQqyIf6k)

你好！欢迎来到 StatQuest。在上一集中，我们学习了如何用 Bootstrapping（自助法）来计算置信区间。今天，我们要更进一步，看看如何用这个强大的工具来计算统计检验中的核心指标——**P 值 (p-value)**。

_注意：本期内容假设你已经熟悉了“Bootstrapping 的基本概念”以及“假设检验”的基础知识。_

---

### 💊 回顾：新药实验的困境

**时间戳**: [[00:30](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=30)]

记得那个新药实验吗？

- 我们给 8 位病人服用了新药。
- 得到的**平均改善值 (Mean) 是 0.5**。

虽然 0.5 看起来像是改善了，但我们依然怀疑：这会不会只是随机运气？

在上一集中，我们用 Bootstrapping 生成了 **95% 置信区间**。结果发现区间里包含 **0**（即无效的可能性），所以我们无法肯定药物有效。

今天，我们要换一种方式来回答这个问题：**直接计算 P 值**。

---

### 🧪 第一步：建立零假设 (Null Hypothesis)

**时间戳**: [[02:06](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=126)]

要算 P 值，首先得有一个靶子，也就是 **零假设**。

- **零假设**：这种药**完全无效**。
- **这意味着**：如果零假设是真的，那么在一个完美的世界里，我们测量到的平均值应该是 **0**。

**但现实很骨感**：我们的实际数据平均值是 **0.5**。

怎么办？为了模拟“零假设成立”的世界，我们需要对数据做一点“手脚”。

### 🔄 第二步：平移数据 (Shifting the Data)

**时间戳**: [[02:29](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=149)]

既然零假设说平均值应该是 0，那我们就强制让它变 0！

- 我们把原始那 8 个数据，统统 **向左平移 0.5**。
- **结果**：这组新数据的平均值变成了 **0**。

现在，这组“平移后的数据”就完美代表了 **零假设成立时（药物无效时）** 的情况。

---

### 🎲 第三步：Bootstrapping 模拟

**时间戳**: [[02:46](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=166)]

现在，我们用这组 **“平移后的数据”**（平均值为 0）来进行 Bootstrapping。

1. **重抽样**：从这组平移数据中，随机有放回地抽取 8 个数。
2. **算均值**：算出这个新样本的平均值。
3. **重复**：重复几千次，画出这些平均值的分布直方图。

**关键点**：这个直方图展示了 **“如果药物真的无效，我们可能会随机看到的各种平均值”**。

---

### 📊 第四步：计算 P 值

**时间戳**: [[03:37](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=217)]

现在我们有了“药物无效时的分布图”，我们要问：

> “在这个分布里，出现像我们实际观察到的那样（0.5），或者更极端（离 0 更远）的结果，概率有多大？”

我们回到原始数据，记得我们实际观测到的平均值是 **0.5**。

所谓的“更极端”，就是指离 0 的距离比 0.5 还要远的值（即 >= 0.5 或 <= -0.5）。

我们直接数直方图里的比例：

1. **右边**：Bootstrapping 产生的均值里，有 **47%** 大于等于 0.5。
2. **左边**：Bootstrapping 产生的均值里，有 **16%** 小于等于 -0.5。

**P 值计算**：

$$P\text{-value} = 0.47 (\text{右边}) + 0.16 (\text{左边}) = 0.63$$

**结论**：

因为 **0.63 > 0.05**（通常的显著性水平），我们 **无法拒绝零假设**。

也就是说，我们不能证明这药有效。

**Double Bam!** 💥

---

### 🌟 进阶：不只是平均值！

**时间戳**: [[05:46](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=346)]

Bootstrapping 最酷的地方在于它的 **通用性**。

刚才我们用的是 **平均值 (Mean)**，但如果你的数据里有**离群值 (Outliers)**，平均值可能就不准了。

这时，你可以完全照搬上面的流程，但把统计量换成 **中位数 (Median)**！

**流程演示（用中位数）**：

1. 计算原始数据的中位数（比如是 1.8）。
2. **平移数据**：让数据的中位数变成 0。
3. **Bootstrap**：从平移数据中重抽样，算出成千上万个“Bootstrap 中位数”，画出直方图。
4. **算 P 值**：看直方图里有多少比例的中位数比 1.8 更极端（或比 -1.8 更极端）。
    
    - 在视频的例子中，算出来 P 值是 0.2。
    - 结论依然是：无法拒绝零假设（药还是没法证明有效）。

**Triple Bam!** 💥💥💥

---

### 📝 总结

**时间戳**: [[07:09](http://www.youtube.com/watch?v=N4ZQQqyIf6k&t=429)]

1. **核心思路**：通过平移数据构建一个符合零假设（比如均值为 0）的数据集。
2. **模拟分布**：对这个“零假设数据集”进行 Bootstrapping，模拟出“如果零假设为真，结果会如何波动”。
3. **位置对比**：看我们实际观测到的数据，在这个模拟分布中处于什么位置，从而算出 P 值。
4. **极致灵活**：这套方法对平均值、中位数、标准差等任何统计量都适用！

下次面对复杂的数据分布不知道怎么算 P 值时，别忘了 Bootstrapping 这个好帮手！

# 解释说明

我先用一句话把这集的主线说清楚：他在做的是“**在假设药物没效果的世界里，用自助法重采样，看看像 0.5 这么大的均值有多罕见**”，罕见程度就是 p‑value。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

下面我按视频的步骤，用尽量口语一点的方式讲一遍，你可以对照着视频重看一遍，就会顺很多。

---

## 1. 他为什么要先把数据「平移到 0」？

设定：

- 原始 8 个病人的“好转程度”数据，算出来的样本均值是 0.5。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
- 原假设：药物没效果 → 在“完美世界”里，真正的均值应该是 0。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

矛盾点：

- 真正均值（在原假设下）= 0
- 我们观测到的样本均值 = 0.5

他做的事是：

1. 把每个观测值都整体向左挪 0.5 个单位。
2. 这样一来，这 8 个「平移后的数」的均值就正好是 0。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

直觉理解：

- 这 8 个“平移后的数”，就是“在原假设为真（均值=0）的世界里，我们可能看到的一种数据样子”。
- 换句话说：**它是一个“假如药没用”的数据版本**。

他这么做，是为了构造一个“原假设为真”的数据集，好在这个前提下玩 bootstrap。

---

## 2. 在“原假设为真”的世界里做 bootstrap

有了这 8 个“均值为 0 的数据”之后，他开始做熟悉的四步 bootstrap：[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

1. 从这 8 个数中「有放回」地抽 8 个，组成一组 bootstrap 数据（可以重复）。
2. 算这组的均值。
3. 把这个均值丢进直方图里。
4. 重复几千次，得到一条“bootstrap 均值分布”的直方图。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

这条直方图什么意思？

- 所有 bootstrap 样本都是从“均值恰好为 0 的那 8 个数”里抽出来的。
- 所以它刻画的是：**如果药真的没效果（均值=0），那在重复实验中，我们会看到的“样本均值”会怎么分布**。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

你可以把这句话牢牢记住：

> 这条直方图 ≈ “原假设为真时，样本均值的可能性分布”。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

---

## 3. 他是怎么从这条直方图里读出概率的？

接下来他开始数条形：

- 36% 的 bootstrap 均值落在 −0.5-0.5−0.5 到 0.50.50.5 之间。  
    → 代表“在药没用时，得到均值处在 −0.5 和 0.5 之间的概率是 0.36”。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
    
- 16% 的 bootstrap 均值 ≤ −0.5。  
    → 代表“均值比 −0.5 还小的概率是 0.16”。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
    
- 48% 的 bootstrap 均值 ≥ 0.5。  
    → 代表“均值比 0.5 还大的概率是 0.48”。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

这些数字只是“在原假设为真时，样本均值落在不同区域的频率”。

---

## 4. p‑value 那一步到底在干什么？

现在回到现实：

- 我们真实观测到的样本均值是 0.5（还没平移之前）。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
- 在原假设下，“均值=0”的世界给了我们一条分布（那条 bootstrap 直方图）。

问题变成：

> 如果药真的没用（均值=0），我们看到“均值至少像 0.5 这么远离 0”的概率有多大？

这里“这么远离”是双侧的：

- 向右：≥ 0.5
- 向左：≤ −0.5

所以 p‑value =

- P(均值 ≥ 0.5) + P(均值 ≤ −0.5)
- = 0.47 + 0.16 ≈ 0.63（视频里具体是 0.47 和 0.16，转录中写 0.48/0.16，核心是大约 0.63）。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

解释：

- 如果药真的一点用都没有，那看到像 0.5 这么极端甚至更极端的均值的概率有 63%。
- 63% 远大于 5%，所以“这种结果一点都不罕见”；
- 于是：**没证据说药有效 → 不能拒绝“药没用”的原假设**。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

---

## 5. 换成中位数时，套路完全一样

后面他又做了一遍类似的流程，只不过：

- 把“统计量”从 mean 换成 median。
- 把数据平移到“中位数 = 0”。
- 再 bootstrap，得到“在药没用时，中位数会怎么分布”的直方图。
- 然后算“观察到的中位数 1.8 或更极端的概率”，作为 p‑value。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

本质没变：

> 先在“原假设为真”的世界里，用 bootstrap 得到统计量的分布，再看真实观测值在这条分布里有多极端。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

---

## 6. 你可能卡住的几种典型点（逐个拆）

你可以看看是不是卡在下面某一条，如果有我可以单独展开一段小例子讲：

1. 「为什么要先把数据平移到均值（或中位数）为 0？」  
    → 因为原假设说“药没用 → 真均值=0”，我们得先造出一个“真的就是 0 的世界”。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
    
2. 「bootstrap 时为什么从这 8 个平移后的数里抽？不是该从总体抽吗？」  
    → 真正总体拿不到，只能用手头这 8 个数去近似那个“在 H0 下的总体”，bootstrap 就是用样本来模拟“重复抽样”。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
    
3. 「p‑value 为什么要两边都加起来？」  
    → 这是双侧检验：只要“比 0 更远”的都算“比我们现在看到的更极端”，不管是正方向还是负方向。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

你可以告诉我：

- 大概是上面哪一块你感觉最别扭，
- 或者你卡住的是视频中的哪一句/哪一帧（比如“平移这一步”、“36% 那里”、“为什么要加 0.47 + 0.16”之类），

我就用那一个点，配一个你可以在纸上手算的小玩具例子，帮你彻底打通。

# 卡住的点是为什么偏偏选 0.5 和 1.6，是可以随机选的吗

先说结论：0.5 和 1.6 不是随便选的，它们是这一次真实实验的数据算出来的**样本均值**和**样本中位数**，是“观测到的真实结果”，所以必须用这两个数。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

## 1. 0.5 是怎么来的？

- 他一开始有 8 个病人的“好转程度”数据。
- 用这 8 个数算出来的**样本均值 = 0.5**，这是这次实验真实算出来的结果，不是他挑的。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
- 之后所有操作都在问：
    
    > 如果药其实没用（真均值 = 0），那我们还会有多大概率看到“均值至少像 0.5 这么大”？[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
    
- 所以阈值必须用“你这次实验真正观察到的那个均值 = 0.5”，不能换成 0.3、0.8 之类的别的数。

如果你换一个数（比如 0.3 或 1），你算出来的就不再是“这次观察结果有多极端”的 p‑value，而是“某个假想的结果有多极端”的概率了，那就跟这次实验不对应了。

## 2. 1.6 又是怎么来的？

- 后面那一部分，他把“统计量”从 mean 换成了 **median（中位数）**。
- 同样地，他先把 8 个病人的好转数据取中位数，得到这次实验的**样本中位数 = 1.6**（视频里给出的数）。[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​
- 然后问的问题就变成：
    
    > 如果药没用、真中位数是 0，那我们看到“中位数至少像 1.6 这么大”的概率有多大？[[youtube](https://www.youtube.com/watch?v=N4ZQQqyIf6k&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=30)]​

同样道理，这个 1.6 也是由数据决定的，是本次实验真实的样本中位数，不是他想象出来的。

## 3. 可以“随便选一个数”来算吗？

只能说：

- 你**可以**选择别的阈值来问别的问题（比如“在药没用时，看到均值 ≥ 1 的概率是多少？”）。
- 但如果你是在做**这次实验的假设检验**，那阈值就必须是“这次实验真实观测到的统计量值”：
    - 均值检验 → 用这次样本算出来的均值 0.5
    - 中位数检验 → 用这次样本算出来的中位数 1.6

因为 p‑value 的定义本质就是：

> 在原假设为真时，**观察到“至少像这次实际结果这么极端”的结果**的概率。  
> “这次实际结果”就是你那 0.5 或 1.6。

所以：

- 0.5、1.6 都是“数据给你的”；
- 不是“偏偏选”的，更不是“随便选”的；
- 换成别的数，就不再是在评价“这次实验的结果有多极端”。

如果你愿意，我可以给你造一个极小玩具数据（比如 4 个数），带你从头算一遍：

- 先算出“样本均值/中位数”；
- 再 bootstrap 出“在 H0 下的分布”；
- 再围绕这个观测值算 p‑value。
