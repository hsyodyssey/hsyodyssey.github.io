---
layout: article
title:  "Understanding Ethereum: Starting with go-ethereum Source Code｜理解以太坊: Go-Ethereum 源码剖析"
date:   2022-01-01 10:00:00 +0800
tags: Blockchain Ethereum Go
categories: Blockchain
---

## 前言 (Preface)

### 写作背景

#### 与时俱进的Blockchain

Blockchain 这一概念，最早由中本聪在**比特币白皮书**提出，至今已经过了十几年。随着加密货币价格的飞涨，区块链社区的参与人数不断的增加，大量的专业人士带来了全新的想法。随着新的思想持续的涌入，区块链技术在这十几年中也在不断演化发展。从作为支撑 Bitcoin的分布式账本，**Blockchain**逐渐成为了包括*数据库*，*分布式系统*，*密码学*，*点对点网络*，*编译原理*，*静态软件分析*，*众包*，*经济学*，*货币金融学*，*社会学*在内的等多个学科知识的一个全新技术领域。Blockchain也逐渐从小众的去中心化社区逐渐走向了主流社会的舞台，目前仍是当下**最热度最高**，**技术迭代最快**，**最能引起社会讨论**的技术话题之一。在Blockchain 原生的decentralized 的思想的影响下，市面上绝大多数的Blockchain 系统都已经开源，并以开源的形式持续发布在 Github 上。这就为我们提供了一种的极好的学习Blockchain 技术的方式: 结合文档，基于的Blockchain Systems 的源代码，理解和学习系统的设计思想和实现原理。

#### 为什么要研究以太坊的原理

随着网络基础建设的不断完善，网络带宽增加和通信延迟下降的趋势将会持续。同时，伴随着存储技术和分布式算法的不断发展，未来系统的运行效率持续的提高，逼近硬件极限。这对构建大规模去中心化应用带来更大的确定性。在未来的是五到十年内，云端服务/去中心化系统的性能以及覆盖场景一定还会有很大的提升。另一方面，未来云技术世界会是两极分化的。一极是以大型云计算公司（i.e, Google，MS，Oracle，Snowflake，and Alibaba）为代表的中心化服务商。另一极就是以Blockchain技术作为核心的去中心化的世界。在这个世界中，Ethereum 及其生态系统是当之无愧的领头羊。Ethereum 作为通用型 Public Chain 中的翘楚取得了巨大的成功，成功的构建了稳定强大的生态系统。更重要的是，Ethereum及其生态吸引到了一大批世界上最优秀的工程师和研究人员的持续的输出。不断的将新思想，新理念，新技术引入到Ethereum及其生态中，并且持续的引领整个 Blockchain 生态系统发展。同时，Go-Ethereum作为其优秀的开源实现，已经被广泛的订制，来适应不同的私有/联盟/Layer-2的场景(e.g., Quorum, Binance Smart Chain, Optimism)。因此，研究好Ethereum的原理以及其设计思想，对于想真正掌握好区块链系统的原理，达到可以设计开发区块链系统的水平的开发/研究人员是非常有必要。

#### 本书的写作目的

一个热门的技术是否热门的标志之一是: 是否有不同视角的作者，在不同的技术发展阶段记录下来的文档资料。目前，对于学习者，不管是探究以加密货币导向（Crypto-based）的Bitcoin, 还是了解致力于实现通用Web3.0框架（General-Purpose）的 Ethereum，社区中有丰厚的 high-level 的角度的技术文档来讲述它们的基础概念和设计的思想。比如，技术社区有非常多的资料来讲述什么是梅克尔树 (Merkle Hash Tree)，什么是梅克尔帕特里夏树 (Merkle Patricia Trie)，什么是有向无环图 (Directed acyclic Graph); BFT (Byzantine Fault Tolerance)和 PoW (Proof-Of-Work) 共识算法算法的区别; 以及介绍Blockchain系统为什么可以抵抗双花攻击 (Double-Spending)，或者为什么Ethereum会遇到 DAO Attack (Decentralized autonomous organization) 等具体问题。

但是，现有的资料往往对工程实现的细节往往介绍的不够清晰。对于研究人员和开发人员来说，只了解关键组件的实现细节，或者高度抽象的系统工作流，并不代表着搞清楚 Blockchain 的**工作原理**。反而很容易在一些关键细节上一头雾水，似懂非懂。比如，当我们谈到Blockchain 系统中 Transaction 的生命周期时，在文档中经常会提到，“Miner节点批量地从自己维护的Transaction pool中选择一些Transaction并打包成一个新的 Block 中”。那么究竟 miner 是怎么从网络中获取到Transaction？又是基于什么样的策略从Transaction pool 中选取**多少**Transaction？最终按照又按照什么样的 Order 把 Transaction 打包进区块链中的呢？如何打包成功的 Block 是怎么交互给其他节点呢？在我学习的过程中，尝试去搜索了一下，发现鲜有文章从*整体*的系统工作流的角度出发，以**细粒度**的视角对区块链系统中的具体的实现*细节*进行解析。与数据库系统(Database Management System)相似，Blockchain 系统同样是一个包含网络层，业务逻辑层，任务解析层，存储层的复杂数据管理系统。对它研究同样需要从系统的实现细节出发，从宏观到微观的了解每个执行逻辑的工作流，才能彻底理解和掌握这门技术的秘密。

本系列文章作为我在博士期间学习/研究的记录，将会从 Ethereum 中具体业务的工作的视角出发，在源码的层面，细粒度的解析以太坊系统中各个模块的实现的细节，以及背后的蕴含的技术和设计思想。同时，在阅读源代码中发现的问题也可以会提交Pr来贡献社区。本系列基于的代码库是 Go-ethereum version 1.10.*(after merge)版本。Go-ethereum是以太坊协议的 Go 语言实现版本，目前由以太坊基金会维护。目前除了 Go-ethereum 之外，Ethereum 还有C++, Python，Java, Rust等基于其他语言实现的版本。相比于其他的由社区维护的版本，Go-ethereum 的用户数量最多，开发人员最多，版本更新最频繁，issues 的发现和处理都较快。其他语言的Ethereum实现版本因为用户与开发人员的数量相对较少，更新频率相对较低，隐藏问题出现的可能性更高。因此我们选择从 Go-ethereum 代码库作为我们的主要学习资料。

在合并之后，以太坊信标链和原有的主链进行了合并。原有的主链节点 (Go-ethereum 节点) 进行了功能缩减，放弃了共识相关的功能，仅作为执行层继续在以太坊的生态中发挥至关重要的作用。同时，交易的执行，状态的维护，数据的存储等基本功能还是由执行层进行维护。因此，作为开发和研究人员，了解 Go-ethereum 代码库仍然是十分有意义的。

### 我们为什么要阅读区块链系统的源代码？

1. 关于以太坊细节实现的文档资料相对较少。由于 Ethereum 进行了多次设计上的更新，一些源代码解析的文章中采用的代码已经经历了多次的修改。同时，不少文章在分析细节的时候，浅尝辄止，对一些关键问题没有解析到位。比如，*很多的科普文章都提到，在打包新的Block的时候，miner负责把a batch of transactions从transaction pool中打包到新的block中*。那么我们希望读者思考如下的几个问题：
    - Miner 是基于什么样策略从 Transaction Pool 中选择 Transaction 呢？
    - 被选择的 Transactions 又是以怎样的顺序(Order)被打包到区块中的呢？
    - 在执行 Transaction 的 EVM 是怎么计算 gas used，从而限定 Block 中Transaction 的数量的呢?
    - 剩余的 gas 又是怎么返还给 Transaction Proposer 的呢？
    - EVM 是怎么解释 Contract 的 Message Call 并执行的呢？
    - 在执行 Transaction 中是哪个模块，是怎样去修改 Contract 中的持久化变量呢？
    - Smart Contract 中的持久化变量又是以什么样的形式存储？在什么地方的呢？
    - 当新的 Block 更新到 Blockchain 中时，World State 又是何时怎样更新的呢？
    - 哪些数据常驻内存，哪些数据又需要保存在 Disk 中呢？
  
2. 目前的 Blockchain 系统并没有像数据库系统(DBMS)那样统一的形成系统性的方法论。在 Ethereum 中每个不同的模块中都集成了大量的细节。从源码的角度出发可以了解到很多容易被忽视的细节。简单的说，一个完整的区块链系统至少包含以下的模块:
    - 密码学模块: 加解密，签名，安全hash，Mining
    - 网络模块: P2P节点通信
    - 分布式共识模块: PoW, BFT，PoA
    - 智能合约解释器模块: Solidity编译语言，EVM解释器
    - 数据存储模块: 数据库，Caching，数据存储，Index，LevelDB
    - Log日志模块
    - etc.

而在具体实现中，由于设计理念，以及 go 语言的特性(没有继承派生关系)，Go-ethereum中的模块之间相互调用关系相对复杂。因此，只有通过阅读源码的方式才能更好理解不同模块之间的调用关系，以及业务的 workflow 中执行流程/细节。

-----------------------------------------------------------

## 目录Contents (暂定)

### PART ONE - General Source Code Analysis: Basic Components

- [00_万物的起点: Geth Start](http://www.hsyodyssey.com/blockchain/2022/01/02/geth.html)
- [01_State-based 模型 & Account](http://www.hsyodyssey.com/blockchain/2022/01/02/ethereum-account.html)
- [02_Transaction: 一个Transaction的生老病死](http://www.hsyodyssey.com/blockchain/2022/01/03/ethereum_txn.html)
- [03_从Block到Blockchain: 区块链数据结构的构建]
- [04_一个新节点是怎么加入网络并同步区块的]
- [05_一个网吧老板是怎么用闲置的电脑进行挖矿的]

### PART TWO - General Source Code Analysis: Services

- [10_StateDB的实例是如何构建的]
- [11_Blockchain的数据是如何持久化的]
- [12_Signer: 如何证明Transaction是合法的]
- [13_节点的调用 RPC and IPC]
- [14_深入EVM: 设计与实现]

### PART THREE - Advanced Topics

- [20_结合BFT Consensus 解决拜占庭将军问题]
- [21_从Plasma到Rollup]
- [22_Authenticated data structures Brief]
- [23_Bloom Filter]
- [24_图灵机和停机问题]
- [25_Log-structured merge-tree in Ethereum]
- [26_Concurrency in Ethereum Transaction]
- [27_Zero-knowledge Proof]

### PART FOUR - Ethereum in Practice

- [30_使用geth构建一个私有网络]
- [31_如何编写Solidity语言]
- [32_使用预言机(Oracle)构建随机化的DApp]
- [33_Query On Ethereum Data]
- [34_layer2 in Practice]


### PART FIVE - APPENDIX

- [40_FQA]
- [41_Ethereum System Tunning]
- [42_go-ethereum的开发思想]
- [43_Metrics in Ethereum]
- [44_Golang with Ethereum]

-----------------------------------------------------------

## How to measure the level of understanding of a system？

如何衡量对一个系统的理解程度?

- Level 4: 掌握（Mastering）
  - 在完全理解的基础上，可以设计并编写一个新的系统
- Level 3: 完全理解（Complete Understanding）
  - 在理解的基础上，完全掌握系统各个模块实现的细节
  - 能快速的从系统功能模块定位到其对应的代码库的位置
  - 可以将系统定制化到不同的应用场景
  - 能对系统中的各个模块做出优化
- Level 2: 理解（Understanding）
  - 熟练使用系统的常用 API
  - 了解系统各个模块的调用关系
  - 了解部分核心模块的设计细节
  - 能对系统的部分模块进行简单修改/重构
- Level 1:了解（Brief understanding）
  - 了解系统设计的主要目标
  - 了解系统的应用场景
  - 了解系统的主要功能
  - 可以使用系统的部分的 API

 我们希望读者在阅读完本系列之后，对以太坊的理解能够达到 Level 2 - Level 3 的水平。

## Some Details

- 以太坊是基于 State 状态机模型的区块链系统，交易的结果会直接更新到账户的状态上。因此，Miner 在生成新的区块的时候，会直接调用 EVM 中增加余额的函数，添加区块奖励给自己。因此，与 Bitcoin 不同的是，Ethereum 的区块中，并没有额外增加 Coinbase 的transaction。
- 在core/transaction.go 中, transaction的数据结构是包含了一个time.Time类型的成员变量的。在后续创建一个新的 Transaction 的 newTransaction 函数中，只使用Local time(`time.now()`)对Transaction.time进行初始化。
- uncle block 中打包的 transaction 不会被更新到包含该叔块的主链区块中。
- 不同的合约中的数据会混合的保存在底层的同一个LevelDB instance中。
- LevelDB 中保存的 KV-Pair 是 MPT 的 Node 信息，包括 State Trie 和Storage Trie。
- 在以太坊更新数据的工作流中，通常先调用`Finalise`函数，然后执行`Commit`函数。

### Blockchain System (BCS) VS Database Management System (DBMS)

Blockchain 系统在设计层面借鉴了很多数据库系统中的设计逻辑。

- Blockchain 系统同样也从 Transaction 作为基本操作的载体。Transaction 的执行是一个原子化的操作，只有成功和失败两种状态。
  
## 关键函数

```go
 // 向leveldb中更新Storage 数据
 func WritePreimages(db ethdb.KeyValueWriter, preimages map[common.Hash][]byte)

 // 向Blockchain中添加新的Block，会涉及到StateDB(Memory)/Trie(Memory)/EthDB(Disk)的更新
 func (bc *BlockChain) InsertChain(chain types.Blocks) (int, error)
 func (bc *BlockChain) insertChain(chain types.Blocks, verifySeals, setHead bool) (int, error)

 // insertChain中调用来执行Block中的所有的交易
 func (p *StateProcessor) Process(block *types.Block, statedb *state.StateDB, cfg vm.Config) (types.Receipts, []*types.Log, uint64, error)

 //执行单条Transaction的调用
 func applyTransaction(msg types.Message, config *params.ChainConfig, bc ChainContext, author *common.Address, gp *GasPool, statedb *state.StateDB, blockNumber *big.Int, blockHash common.Hash, tx *types.Transaction, usedGas *uint64, evm *vm.EVM) (*types.Receipt, error)

 // 状态转移函数
 func (st *StateTransition) TransitionDb() (*ExecutionResult, error)

 // 执行合约内function
 func (in *EVMInterpreter) Run(contract *Contract, input []byte, readOnly bool) (ret []byte, err error)

 // opSstore的调用
 func (s *StateDB) SetState(addr common.Address, key, value common.Hash)
 // 被修改的state的值会首先被放在StateObject的dirtyStorage中，而不是直接添加到Trie或者Disk Database中。
 func (s *stateObject) setState(key, value common.Hash)

 // 在Finalizes所有的pending的Storage时候，并且更新到Trie，计算State Trie的Root
 func (s *StateDB) IntermediateRoot(deleteEmptyObjects bool) common.Hash

 // Finalise 当前内存中的Cache.
 func (s *StateDB) Finalise(deleteEmptyObjects bool) 

 // Commit StateDB中的Cache到内存数据库中
 func (s *StateDB) Commit(deleteEmptyObjects bool) (common.Hash, error)

 // 将StateObject中所有的dirtyStorage转存到PendingStorage中，并清空dirtyStorage，并给prefetcher赋值
 func (s *stateObject) finalise(prefetch bool)

 // 更新StorageObject对应的Trie, from Pending Storage
 func (s *stateObject) updateTrie(db Database) Trie

 // 最终获取到新的StateObject的Storage Root
 func (t *Trie) hashRoot() (node, node, error)

 // 用于在内存数据库中保存MPT节点
 func (c *committer) store(n node, db *Database) node

 // 向rawdb对应的数据库写数据(leveldb)
 func (db *Database) Commit(node common.Hash, report bool, callback func(common.Hash)) error

```

## Conclusion

持续更新中，Markdown文件库地址[[link]](https://github.com/hsyodyssey/Understanding-Ethereum-Go-version)

## Reference

- [1] Ethereum Yellow Paper [(Paper Link)](https://ethereum.github.io/yellowpaper/paper.pdf)
- [2] Ethereum/Go-Ethereum [(link)](https://github.com/ethereum/go-ethereum)
- [3] Go-ethereum code analysis [(Link)](https://github.com/ZtesoftCS/go-ethereum-code-analysis)
- [4] Ethereum Improvement Proposals [(link)](https://github.com/ethereum/EIPs)
- [5] Mastering Bitcoin(Second Edition)
- [6] Mastering Ethereum [(link)](https://github.com/ethereumbook/ethereumbook)
- [7] EIP-2718: Typed Transaction Envelope[[link]](https://eips.ethereum.org/EIPS/eip-2718)
- [8] EIP-2930: Optional access lists [[link]](https://eips.ethereum.org/EIPS/eip-2930)
- [9] EIP-1559: Fee market change for ETH 1.0 chain [[link]](https://eips.ethereum.org/EIPS/eip-1559)