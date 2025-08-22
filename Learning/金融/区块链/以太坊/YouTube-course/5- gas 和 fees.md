---
aliases:
  - 5- gas 和 fees
tags: 
date created: Wednesday, April 16th 2025, 12:03:59 pm
date modified: Wednesday, April 16th 2025, 1:03:11 pm
URL: https://www.youtube.com/watch?v=dmguh81VBQ8&list=PL6gx4Cwl9DGBrtymuJUiv9Lq5CAYpN8Gl&index=7&pp=iAQB
---

节点需要 gas 来接受你的 smart contract $\downarrow$ ![[Pasted image 20250416120534.png]]
执行 smart contract 需要节点执行代码进行工作, 工作量越大, 要的 gas 越多 $\downarrow$ ![[Pasted image 20250416120804.png]]

## EVM 指令集
智能合约由 operation 组成

| Stack  栈 | Name  名称 | Gas  气值 | Initial Stack  初始栈 | Resulting Stack  结果栈 | Mem / Storage | Notes                                  |
| -------- | -------- | ------- | ------------------ | -------------------- | ------------- | -------------------------------------- |
| 00       | STOP     | 0       |                    |                      |               | halt execution                         |
| 01       | ADD      | 3       | `a, b`             | `a + b`              |               | (u)int256 addition modulo 2**256       |
| 02       | MUL      | 5       | `a, b`             | `a * b`              |               | (u)int256 multiplication modulo 2**256 |
| 03       | SUB      | 3       | `a, b`             | `a - b`              |               | (u)int256 addition modulo 2**256       |
| 04       | DIV      | 5       | `a, b`             | `a // b`             |               | uint256 division                       |
| 05       | SDIV     | 5       | `a, b`             | `a // b`             |               | int256 division                        |
| 06       | MOD      | 5       | `a, b`             | `a % b`              |               | uint256 modulus                        |
| 07       | SMOD     | 5       | `a, b`             | `a % b`              |               | int256 modulus                         |
| 08       | ADDMOD   | 8       | `a, b, N`          | `(a + b) % N`        |               | (u)int256 addition modulo N            |
| …      |          |         |                    |                      |               |                                        |
EVM 是以太坊的智能合约虚拟机, 它的指令集就是上述的 operation, 需要不同的 gas ![[Pasted image 20250416121513.png]]
但是, gas 并不是固定的价格, 用户可以自定义自己的 gas, 1 gas 为 1 gwei, 2 gwei 等等
整个网络中 gas 的平均价格实际上由需求量所决定
## `gas price`, `gas limit`, `maxGasFee`
- `gas price` 就是你自己定义的 1 gas 价格
- `gas limit` 是数量限制, 限制了该合约可以花费的最多 gas 数量
- `maxGasFee` 等于 `gas price` $\times$ `gas limit` ( london 升级前)
你提交 gas 的时候, 你可以设置 `maxGasFee`, 也就是你愿意支付的 gas 费用, 这个费用是你愿意支付的最高费用, 多余的费用会退还给你
## London 升级
2021 年 8 月 5 日, 以太坊网络进行了一次重要的升级, 叫做 London 升级
这次升级引入了 EIP-1559, 使得 gas 的价格不再是固定的, 而是根据网络的需求量来动态调整
 ![[Pasted image 20250416123134.png]]

- 每个块有一个 `base fee`, 这个 `base fee` 是根据网络的需求量来动态调整的, 用户无法决定,如果网络拥堵, `base fee` 就会增加, 如果网络不拥堵, `base fee` 就会减少
- `tip` 是用户给的小费, 小费越多, 矿工就越愿意优先打包你的交易
## `maxFeePerGas`
由于 London 升级, `maxGasFee` 无法通过 `gas limit` 来决定每单元 gas 的价格, 因此出现了 `maxFeePerGas`
它是你愿意支付的最高费用, 也就是 `base fee` + `tip` 的最大值 ![[Pasted image 20250416130053.png]]
