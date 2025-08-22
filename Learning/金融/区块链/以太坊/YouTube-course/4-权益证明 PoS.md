---
aliases: 4-权益证明 PoS
tags: []
date created: Invalid date
date modified: Wednesday, April 16th 2025, 12:40:17 pm
URL: https://www.youtube.com/watch?v=TcYdEAWch_4&list=PL6gx4Cwl9DGBrtymuJUiv9Lq5CAYpN8Gl&index=6&ab_channel=thenewboston
---

不同于工作量证明
节点可以将其资产抵押或锁定在网络中, 这种抵押或锁定的资产被称为权益证明, 这种节点就成为了能够生成区块的节点
权益证明的节点会根据其抵押的资产数量和时间来决定其生成区块的概率

权益证明不仅节省了能源, 还允许一些技术或架构上的创新
# 分片链 (Shard)
从技术上来讲,分片 (sharding) 是水平拆分数据库以分配负载的过程
![[Pasted image 20250416011544.png]]
假设你有一台计算机, 它只有 3 GB 的内存, 和 3 GB 的硬盘, 你在计算机上安装和运行一个数据库, 很快内存和硬盘就会被填满, 你要么购买更大的内存和硬盘, 要么将数据拆分放在多台计算机上 (其实也就是分布式存储), 这就是分片

以太坊将分片这一概念引入到区块链中, 以太坊的分片是将整个网络分成多个小网络, 每个小网络都有自己的区块链, 这样就可以在每个小网络上并行处理交易, 从而提高交易的吞吐量 $\downarrow$
![[Pasted image 20250416011937.png]]
被拆分出的链称为分片链 (shard chain), 以太坊的计划是有 64 条分片链, 每条分片链可以处理 1000 笔交易, 这样就可以达到 64000 笔交易每秒的吞吐量
既然被拆分了, 节点就只需要维护自己所在的分片链, 这样就可以减少每个节点的存储和计算压力, 降低了性能要求, 也提高了去中心化程度
# 组织结构
## 分片验证者 (Shard Validator)
一旦一个处于验证者池 ( Validator Pool) 的节点质押 (staking) 足够的 eth ,那么它就可以被分配到一个分片链上, 这个节点就可以在这个分片链上进行交易验证, 也就是成为这个分片链的验证者
验证者池只是一个合格节点的列表
只要你 staking 了 32 个 eth, 你就可以成为验证者池中的一个节点, 你就可以被分配到一个分片链上
![[Pasted image 20250416013017.png]]

## Beacon Chain, proposer 和 attester
系统中还有另一层链, 它叫做 Beacon Chain, 它的作用是协调所有的分片链, 也就是协调所有的验证者
Beacon Chain 会将分片链的数据发送给其它分片链, 也就是将分片链的数据进行汇总, 使得整个链保持同步 ![[Pasted image 20250416013237.png]]
某些验证节点将由 Beacon Chain 通过算法随机选择出来, 它们可以提议新区块, 这些验证者被称为提议者 (proposer), 其它验证者负责验证 proposer 提出的区块, 它们被称为证明者 (attester)
![[Pasted image 20250416013817.png]]
这些验证者如果下线或验证失误, 就会被罚掉一部分质押, 也就是被 slashed
如果它们故意串通, 可能会损失全部的质押

所以吗每当你提交一个新的交易, proposer 会将这个交易打包进一个区块
![[Pasted image 20250416014135.png]]

attester 会证明这个区块的合法性, 实际上证明记录在 Beacon Chain 上, 这也是 Beacon Chain 负载的主要来源 ![[Pasted image 20250416014714.png]]

## Committee, epoch 和 slot
Committee 是一个分片链的验证者集合, 它们负责验证这个分片链上的交易, 也就是 attester, 一个 committee 由 128 个验证者组成, 包含 attester 和 proposer
Committee propose 和 attest 一个区块有时间限制, 称为 slot, 一个 slot 是 12 秒
一个 epoch 是 32 个 slot, 也就是 6 分钟
![[Pasted image 20250416115509.png]]
一个 epoch 结束后, committee 会被重新分配到不同的分片链上, 这个过程叫做 shuffle
## Crosslinks
一旦一个分片区块通过验证, 它就会被添加到 Beacon Chain 上, 这个过程叫做 crosslink, BeaconChain 通过这个最新的分片区块来保持分片链的同步 $\downarrow$ ![[Pasted image 20250416120026.png]]