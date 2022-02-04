---
layout: article
title:  "Rollup/zk-Rollup以及zkVM/zkEVM"
date:   2022-2-2 10:00:00 +0800
tags: Blockchain Ethereum ZKP Layer-2 Rollup zk-Rollup
categories: Blockchain
---

自从[EIP-1559](https://eips.ethereum.org/EIPS/eip-1559)生效后，Ethereum Layer-1 Mainnet已经成为了普通用户用不起的贵族链。在上个月的一次实验中，我尝试了一把在Layer-1上调用一次古早的合约，大概花了100美刀左右的交易费。

显然，目前天价的交易费以及有限的Throughput已经成为了限制Ethereum继续发展的两大难题。幸运的是，**rollup**技术的发展给社区展现了一种似乎可以一招解决两大难题的绝世武学。

简单的来说，顾名思义，rollup，就是把一堆的transaction rollup到一个新的transaction。然后，通过某种神奇的技术，使得Ethereum Mainnet只需要验证这个新的transaction，就可以保证被Rollup之前的若干的Transaction的确定性，正确性，完整性。举个简单的例子，想象一个交学费的场景。目前状况是每个学生(i.e. Account)都需要通过学校交费系统((i.e. Ethereum Mainnet)单独的将自己的学费转账(Transfer)给学校教务处。假如，现在计算机系有1000个学生，那么系统就要处理1000笔转账的交易(Transaction)。系里新来一位叫做*Prof.Rollup*的教授，他在学生中很有号召力。基于他的个人魅力(神奇的魔法)，**私下里**让所有的的学生把钱通过**某种方式**先转给他。当*Prof.Rollup*收集到系里所有学生的学费之后，然后他通过构造一条transaction把所有的学费交给教务处。这个例子大概就是一个Rollup场景的简单抽象。我们把学校的交费系统看作为Ethereum Mainnet Layer-1，那么私下里收集学生的学费就是所谓的**Layer-2**进行的了。通常，我们把Layer-1上的交易称为on-chain transaction，把在Layer-2进行上的交易称为off-chain transaction。

Rollup的好处是显而易见的，假如每次把**N**个transaction rollup成一个，那么对于Mainnet来说，处理一条交易，实际效果等同于多处理了之前**N**个交易。此刻系统实际吞吐量也等于**N*MAX_TPS**。同样的对于用户来说，同样一条交易Mainnet Transaction Fee实际上是被**N**个用户同时负担的。那么理论上，用户需要的实际交易费用也只有之前的**1/N+c**。这里的**c**代表rollup服务提供商收取的交易费，这个值是远远小于layer-1上交易所需要的交易费用的。

当然，在实现layer-2 rollup时，会遇到很多的困难，有很多的细节有待商榷。比如:

- 怎么保证*Prof.Rollup*一定会把学费交给教务处呢？ (Layer-2 交易的安全性)
- 怎么保证*Prof.Rollup*会把全部的学费都交给教务处呢？(Layer-2 交易的完整性)
- *Prof.Rollup*什么时候才会把学费打给教务处呢？(Layer-2 到Layer-1 跨链交易的时效性问题)
- 等等..

目前，在市场上主要有两种比较热门的rollup的实现方式，分别是**Optimism-Rollup**，和**ZK-Rollup**

## Optimism-Rollup

- Arbitrum
  - 乐观模式，Layer-2完全兼容EVM，通过设置挑战时间来保证跨链交易，目前是one week。

## ZK-Rollup

ZK-Rollup利用了Zero-Knowledge Proof的技术来实现rollup。ZKP里面的细节比较多，在这里我们不展开描述。核心点是ZK-Rollup主要利用了ZK-SNARKs中Verification Time相对较快的特性，来保证layer-1上的Miner可以很快的验证大量交易的准确性。

但是正因为ZKP的一些特性，ZK-Rollup相比于Optimism-Rollup，目前发展的并没有那么完备。

我们知道ZK-SNARKs的计算的基础来自于一个电路Circuit，可以转化为R1CS的形式，从而转化为QAP问题。如果我们想对于一个Problem/Computation/Function使用ZK-SNARKs，那么首先第一步我们就要把这个问题转化为一个Circuit。对于简单的计算问题来说，将问题转化为电路的形式并不是特别的困难。但是对于Ethereum上各种支持图灵完备的智能合约的function来说这个问题就变得非常的棘手。主要因为：

- 转化生成的电路是静态的，虽然电路中有可以复用的部分(Garget)，但是每个Problem/Computation/Function都要构造新的电路。
- 对于通用计算问题，构造出来的电路需要的gate数量可能是惊人的高。这意味着ZK-Rollup可能需要非常长的时间来生成Proof。
- 对于目前EVM架构下的某些计算问题，生成其电路是非常困难的。

前两个问题属于General的ZK-SNARKs应用都会遇到的问题，目前已经有一些的研究人员/公司正在尝试解决这个问题。比如AleoHQ的创始人Howard Wu提出的[DIZK](https://www.usenix.org/conference/usenixsecurity18/presentation/wu)通过分布式的Cluster来并发计算Proof，以及Scroll的创始人[Zhang Ye](https://twitter.com/yezhang1998)，提出的[PipeZK](https://www.microsoft.com/en-us/research/publication/pipezk-accelerating-zero-knowledge-proof-with-a-pipelined-architecture/)通过ASIC来加速ZK计算。

对于第三个问题，属于Ethereum中的专用问题。我们知道在Ethereum中，一条调用合约Function的Transaction的执行是依赖于EVM的。EVM会将Transaction中的函数调用，转换成opcodes，保存在Stack中逐条执行。在执行这些opcodes时，本质上是在执行geth中对应的库函数，部分细节可以参考之前的[blog](http://www.hsyodyssey.com/blockchain/2021/07/25/ethereum_txn.html)。那么如果我们想把一个Transaction的合约调用转换成一个电路的话，本质上我们就要基于这个函数调用过程中执行过的opcodes来生成电路。目前的问题是，EVM在设计的时候并没有考虑到将来会被ZK-SNARKs这一问题，所以在它包含的140个opcode中有些是难以构造电路的。

于是，在目前的ZK-Rollup的解决方案中，大部分**仅支持基础的转账操作**，而**不能支持**通用的图灵完备的计算。也就是说，目前的ZK-Rollup的解决方案都是不完整的，不能完整的发挥Ethereum图灵完备合约的特性，Layer-2又回到了仅支持Token转账的时代。

为了解决这个问题，使得Layer-2能支持像现在的Layer-1一样的功能，目前的技术主要在朝向两个方向发展，1. 构建ZK-SNARKs兼容的zkEVM，2.提出新的VM来兼容ZK-SNARKs，并想办法与Ethereum兼容。

### 构建兼容EVM的zkEVM

这种方案好处在于，开发人员可以继续使用solidity来构建智能合约。如何构造电路，完全交给底层的ZK-EVM来完成。ZK-EVM和现有的EVM是完全兼容的，现有的Ethereum Contract都可以直接移植到Layer-2上来使用。这种方案对现有的以太坊生态圈非常的友好，社区的合约开发人员在开发时，不需要学习什么额外的新知识，零门槛上手。

这种方案的难点在于如何把现有的EVM, OPcode抽象成电路。目前正在研究这个路线的有下面两个团队，Scroll和Polygon Hermez。

- Scroll
- **[Polygon Hermez](https://docs.hermez.io/#start-here-for-hermez-10-documentation)**
  - General
    - Hermez 1.0: Support Ethereum Token transfer.
    - Hermez 2.0: Recreating all the EVM opcodes (Seem they have the same goal with Scroll).

### 提出新的VM/编程语言

这种方案完全甩开了EVM的包袱，重新设计对ZK友好的VM以及对应的Programming Language。由于没有了包袱，所以这种方案开发起来比较快，目前进展最快的应该是StareWare团队开发的Cairo语言。

但是基于这种解决方案Zk-Rollup，不能在Layer-2上直接使用Layer-1上已经编写好的Solidity合约。需要合约开发人员重新学习一门新的语言来编写合约。
- **StareWare**
  - General
    - 专用语言: [Cairo](https://cairo-lang.org/docs/)
      - StarkNet uses the Cairo programing language both for infrastructure and for writing StarkNet contracts.
    - 专用硬件加速Proof生成
- **Zksync**
  - General
    - 专用硬件加速Proof生成(FPGA)
  - Con't
    - 并不支持所有EVM opcodes

从我个人的角度来说，这个方案不能算非常友好的。首先以太坊社区在Solidity生态建设方面已经做的非常的完善，有大量的已经审计过的合约可供开发人员参考，有大量的生产工具可供使用。重新构造一个新的编程语言的社区还是相对困难的。第二，现在需要学习一门新语言的成本太高了，尤其是学习相比C++/Java/Python更加小众的智能合约开发语言。同时新的编程语言，为了不和现有的开发语言完全一致而设计的新的语法，语法糖，很容易让开发人员头大。所以我更看好类似Scroll/Polygon Hermez这种项目的发展。

### 其他ZK-Related Project

在阅读资料时发现的一个项目AleoHQ，看上去是一个原生支持ZKP的图灵完备的新的公链项目，由Berkeley的团队开发。Founder Howard Wu实力很强，是最早的Libsnarks的开发者之一。具体的细节我还在了解之中。

- **[AleoHQ](https://github.com/AleoHQ)**, Developed by Howard Wu (UC Berkeley)
  - General
    - Aleo is the first decentralized, open-source platform to enable both private and programmable applications.
    - Strong privacy guarantees.
    - 专用语言: [Leo](https://github.com/AleoHQ/leo)
      - Leo converts a developer's high-level code into zero knowledge circuits.
    - snarkOS:
      - Aleo runs on a decentralized operating system for private applications called snarkOS. (*HAN:Seems like zkGeth?*)
    - Core Part: ZEXE (Zero Knowledge EXEcution)

## Related Papers

- Zexe: Enabling Decentralized Private Computation
- DIZK: A Distributed Zero Knowledge Proof System, [[link]](https://www.usenix.org/conference/usenixsecurity18/presentation/wu)

## Reference

- EIP-1559: Fee market change for ETH 1.0 chain, [[link]](https://eips.ethereum.org/EIPS/eip-1559)