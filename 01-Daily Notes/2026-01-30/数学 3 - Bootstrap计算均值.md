[[01-Daily Notes/2026-01-30/attachments/3d8945d254363cb27b408e08cff8d0ab_MD5.png|Open: Pasted image 20260130150729.png]]
![[01-Daily Notes/2026-01-30/attachments/3d8945d254363cb27b408e08cff8d0ab_MD5.png]]

[[01-Daily Notes/2026-01-30/attachments/f0dc45ea2721f75f18e2f3a20e3acbb3_MD5.png|Open: Pasted image 20260130150956.png]]
![[01-Daily Notes/2026-01-30/attachments/f0dc45ea2721f75f18e2f3a20e3acbb3_MD5.png]]

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
