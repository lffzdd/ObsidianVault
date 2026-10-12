[[attachments/17d3eff0978019c83d998a42e97d33f4_MD5.png|Open: Pasted image 20260112121943.png]]
![[attachments/17d3eff0978019c83d998a42e97d33f4_MD5.png]]
p-hacking 这期视频，实际上是在延续前面几集 p-value / 假设检验的主线：它不是在教你一个“新统计方法”，而是在讲“同样的统计方法，一旦被滥用，会怎样把你一步步骗进假阳性（false positive）的陷阱，以及如何有系统地避免这种自欺欺人”。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

下面给你一份可以当「学习文档 / 读书笔记」的版本，结构上尽量贴着视频推进节奏来讲，读起来就像 Josh 在你面前慢慢讲故事，但又比视频更利于回顾。你后面要做自己的 slides、讲解稿或者课程笔记，可以直接在这个基础上删改补充。

---

## 1. 从“找到一颗好药”开始的陷阱

想象现在有一种病毒，大家从感染到痊愈，时间大致服从一个「正常分布」：绝大多数人恢复时间在 5～15 天之间，极少数人特别快或特别慢。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

你是研究者，想发明一颗可以「缩短恢复时间」的新药，于是从药 A 一路做到药 Z：

- 对每一种药，你都找 3 个人吃药；
- 再找 3 个人不吃药，当作对照组；
- 测量他们从感染到康复的时间，比较两组平均恢复时间。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

前面很多药（A、B、C……）看起来效果都一般，两组人恢复时间差不多。你一边换药一边做统计检验，一直做到药 Z，终于画出一个“看起来差很多”的图，算出来 p-value = 0.02，小于阈值 0.05，于是你宣布：

> “不吃药 vs 吃药 Z，有显著差异！药 Z 有效！”[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

听起来很像标准操作，但这其实就是最典型的一种 p-hacking：

- 单看“药 Z 这一组”，你只看到“我们做了一个实验，p=0.02”；
- 真相是：你前面已经对无数候选药做了一大堆检验，只是没人看见那些“不显著”的结果。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

如果所有药其实都“没用”，也就是所有人都来自同一个恢复时间分布，那么你一次次随机抽样、一次次做检验，

- 理论上，哪怕所有药都没有任何真实效果，约 5% 的检验仍然会给你一个“p < 0.05 的假阳性”；
- 当你反复试很多次，只看那几个“刚好显著”的，再端出来写论文，这就是在「搜寻幸运的随机波动」，而不是在发现真实规律。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

---

## 2. 什么叫 p-hacking？背后其实是「多重检验问题」

视频里给了一个非常朴素的定义：

> p-hacking = 滥用 / 误用分析方法，最后把纯粹的随机波动当成“显著结果”，从而被假阳性骗了。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

故事换一个更抽象的视角就变成了：

- 真正的总体：所有人的恢复时间服从某个固定的分布；
- 在这个分布下，你不断随机抽出两小组人，各 3 个、再各 3 个……；
- 每一次，你都做一次假设检验，问“这两组是不是来自不同分布”。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

如果所有人都来自同一个真实分布，那么：

- 在显著性阈值设为 0.05 的前提下，大约 5% 的检验会给出“错误的显著结论”（假阳性）；
- 你做 100 次检验，大约会遇到 5 次“p<0.05”；
- 你做 10,000 次检验，大约会遇到 500 次“p<0.05”。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

所以 p-hacking 的本质不是“你算错了公式”，而是：

- 你反复地、以各种方式「试探」数据；
- 每试一次，就给自己一次“抽卡抽到稀有显著结果”的机会；
- 然后只挑那些“抽到 SSR 卡”的结果拿出去展示，让别人误以为那是「一次认真、预先规划好的实验」的结论。[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC4359000/)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

统计学里，这一堆行为在更中性、技术的语言里叫「多重检验问题（multiple testing problem）」，而 p-hacking 就是在这个问题上踩坑的现实表现：

- 做很多检验，却像是只做了一次那样来解释 p-value；
- 于是原本 “5% 假阳性率” 的含义被完全误读成“5% 的概率错”，而实际上你已经试了 N 次。[datacamp](https://www.datacamp.com/tutorial/p-hacking)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

---

## 3. 如何「正经地」面对多重检验：别只看那一个漂亮的 p

视频接着给了一个重要转折：

> 既然「多重检验」不可避免，就不要装作“只做了一次检验”。你需要把所有检验都放到同一套校正框架里。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

其中一个主角就是 **False Discovery Rate（FDR，错误发现率）**。[datacamp](https://www.datacamp.com/tutorial/p-hacking)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

故事可以这么理解：

- 你把所有做过的检验的 p-value 都列出来，比如对药 A 到药 Z 的所有比较；
- FDR 方法（例如 Benjamini–Hochberg）会基于这些 p-value 的排序，给每个检验一个“调整后”的门槛；
- 结果往往是：“原来 p=0.02 的那几个结果，在多重检验校正后，变成了不再显著”。[datacamp](https://www.datacamp.com/tutorial/p-hacking)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

这背后的态度非常关键：

- 不要只把“看起来最有希望的那几个”拿出来算 p-value；
- 应该“所有计划中的检验都算 p-value”，再用 FDR 或其他方法统一校正；
- 这样才能把“试得越多、越容易撞上假阳性”这个结构性风险考虑进去。[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC4359000/)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

换句话说，当你看到一篇文章只报“我们试了一个东西，p=0.03，所以它有效”，没有提“到底试了多少东西、做了多少检验”，你就要警惕：

- 那些没被提到的检验，可能就是被“丢进抽屉”的不显著结果；
- 某种意义上，这是「结果筛选版的 p-hacking」。[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC4359000/)​

---

## 4. 更隐蔽的一种 p-hacking：边看结果边“加样本”

第二种形式的 p-hacking 更像是“过程上的动手脚”，而且非常贴近日常工作流：

- 假设你做了一个对照实验，两组样本，各 3 个人；
- 算出来 p-value = 0.06，刚好“差一点点就显著”；
- 于是你心想：再多招一点人，应该就能把 p 拉到 0.05 以下吧？[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

于是你每组又加了一个受试者，重新算 p-value，结果这次就是 0.02 了，你高高兴兴写上：

> “我们有 4+4 个样本，p=0.02，显著。”[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

问题出在哪？

- 从“统计教科书”视角看，你似乎只是“提高了样本量”；
- 但真实流程是：你先看了一个 p=0.06 的中途结果，再决定要不要继续加样本。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

视频强调的一点是：

- 标准的 p-value 理论假设，你在 **设计实验时就已经决定好了样本量**；
- 也就是你只计算一次 p-value，然后把这一次的结果拿来做决策；
- 一旦你“边看结果边加样本”，实质上又变成了「多次试探同一个研究问题」，结构上又回到“多重检验”的坑。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

对这种行为的更严谨说法是「optional stopping」或者「sequential testing without correction」，如果你不做相应的序贯检验修正，就等于在悄悄推高假阳性率。datacamp+1​

---

## 5. 如何避免这种“边走边改”的 p-hacking：事先算好需要多少人

视频在后半段给了一个非常工程化、你在实际研究里可以直接用的原则：

> 在实验开始之前，用 **Power Analysis（检验力分析）** 来决定样本量，而不是中途看着 p-value 决定停不停。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

检验力分析回答的核心问题是：

- 如果真实差异是某个规模（例如药物确实能平均减少 2 天恢复时间）；
- 在给定显著性水平（比如 0.05）下，要招多少受试者，才能有足够高的概率（比如 80% power）把这个差异检出来？[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

一旦你事先用检验力分析确定了样本量，实验设计就相当于写死了：

- 你预先声明：“我们每组会招 N 个人”；
- 实验结束、数据收完，算一次 p-value，做一次决策；
- 不再根据“中途略微不显著”来动态加人。[datacamp](https://www.datacamp.com/tutorial/p-hacking)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

这既有助于控制假阳性率，也有助于你作为研究者在心理上保持纪律：

- 不被那种“再多一点点数据就能过线”的诱惑牵着走；
- 把「有没有足够的数据」这个问题，提前变成工程化计算，而不是事后找借口调整。[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

---

## 6. 视频要你真正带走的几个原则

最后，视频在结尾做了一个很清晰的“做人指南式”的总结，可以抽象成几条你以后做实验、看论文、做 A/B test 都可以用的 checklist：[datacamp](https://www.datacamp.com/tutorial/p-hacking)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

- 如果你要比较很多东西（很多药、很多变量、很多模型）
    - 不要只给那些“看起来有差异”的算 p-value；
    - 把所有计划中的检验都算出来，然后用 FDR 等方法做多重校正。
- 如果你在做实验过程中看到一个 p=0.06，就想“多加几个人看看能不能变成 0.04”
    - 那就是站在 p-hacking 的悬崖边上了；
    - 用检验力分析在一开始就确定样本量，而不是根据中途结果改动。
- 总之，p-hacking 不是一个“统计技术问题”，而是一个「研究流程和习惯」的问题
    - 关键不在于你用的是哪种检验，而在于：你有没有老老实实地把所有的检验、所有的设计决策，都摊在阳光下；
    - 以及，你有没有事先想清楚：“在没有结果之前，我就已经把游戏规则写死了吗？”[pmc.ncbi.nlm.nih](https://pmc.ncbi.nlm.nih.gov/articles/PMC4359000/)​[youtube](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)​

---

如果你希望把这份学习文档再往「可操作」方向推进，比如：

- 加上你自己的例子（比如在线实验、广告点击 A/B test、模型调参）；
- 写成一套「团队做实验时的 anti p-hacking checklist」；  
    可以说一下你的具体场景（科研 / 产品实验 / 模型评估），可以再帮你改成更贴近你日常的版本。

1. [https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14](https://www.youtube.com/watch?v=HDCOUXE3HMM&list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9&index=14)
2. [https://pmc.ncbi.nlm.nih.gov/articles/PMC4359000/](https://pmc.ncbi.nlm.nih.gov/articles/PMC4359000/)
3. [https://www.datacamp.com/tutorial/p-hacking](https://www.datacamp.com/tutorial/p-hacking)
4. [https://blog.csdn.net/Little_Carter/article/details/147492763](https://blog.csdn.net/Little_Carter/article/details/147492763)
5. [https://cloud.tencent.com/developer/article/2257432](https://cloud.tencent.com/developer/article/2257432)
6. [https://www.toutiao.com/i6817660654011286028/](https://www.toutiao.com/i6817660654011286028/)
7. [https://www.waytoagi.com/question/70373](https://www.waytoagi.com/question/70373)
8. [https://www.taokeshow.com/69760.html](https://www.taokeshow.com/69760.html)
9. [https://blog.csdn.net/weixin_29840475/article/details/156029995](https://blog.csdn.net/weixin_29840475/article/details/156029995)
10. [https://m.36kr.com/p/1971330155007109](https://m.36kr.com/p/1971330155007109)
11. [https://www.youtube.com/watch?v=HDCOUXE3HMM](https://www.youtube.com/watch?v=HDCOUXE3HMM)