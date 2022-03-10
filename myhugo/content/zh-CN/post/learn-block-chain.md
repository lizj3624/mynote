---
title: "区块链技术快速入门"
date: 2022-03-10T09:16:59+08:00
tags:
   - 区块链 
   - blockchain 
categories:
   - 区块链 
   - blockchain 
toc: true
---

> 知乎一网友写快速学习区块链技术的文章，从读书学习构建区块链技术体系，然后去实践，我感觉非常不错，再次收藏一下。
> 对格式、文章结构、部分引用url略有改动，[原文](https://www.zhihu.com/question/51047975/answer/1077996076?utm_source=wechat_session&utm_medium=social&utm_oi=1164288090926637056&utm_content=group2_Answer&utm_campaign=shareopn)

2019年从互联网后端开发工程师转型为区块链工程师。一个月时间系统学习区块链技术，参与到链研发工作当中。下面来谈谈自己的学习心得，希望能够帮助到你。

关于「如何高效的自学一项新技能」，我通常分为两步走：
1. 搜集高质量学习材料和工具。   
2. 应用合适的学习方法。

区块链经过18、19年的火热发展后，网络上大家都能轻易的找到各种各样的高质量学习资料。此外，本问题的大部分答案都是围绕「学习什么」，因此，下文我将重点介绍「如何学习」的方法论。

## **一、泛读通读，建立框架，「不求甚解」**

尽管区块链和其他的计算机技术没有本质上的区别，但有一个很重要的差异是，区块链技术涉及的技术面非常广。事实上，当我们把[中本聪](https://www.zhihu.com/search?q=%E4%B8%AD%E6%9C%AC%E8%81%AA&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1077996076%7D)的区块链技术拆解后可发现，区块链技术是由20世纪提出的一些老技术和知识的组合而成的，换句话说，区块链的创新在于老技术的组合创新，正正体现了区块链技术的系统之美。也正是如此，区块链技术涉及知识面非常广，其中包括：分布式系统、[拜占庭问题](https://www.zhihu.com/search?q=%E6%8B%9C%E5%8D%A0%E5%BA%AD%E9%97%AE%E9%A2%98&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1077996076%7D)、密码学、数据结构、P2P网络等技术，以及博弈论、经济学等思想。

如果你一开始就针对某一方面孤立学习，不但无法系统化的学习，并且容易迷失在繁多复杂的新概念当中。因此，我建议的方法：**以泛读通读的方式，先建立知识框架，对区块链有个大致的认识**。

区块链是伴随比特币而产生的，因此要搞明白区块链，首先需要了解比特币：

- 普林斯顿大学课程 [Bitcoin and Cryptocurrency Technologies](https://www.coursera.org/learn/cryptocurrency)
- [《精通比特币》](https://github.com/lizj3624/MasterBitcoin2CN/blob/master/SUMMARY.md)
- [比特币白皮书](http://lixiaolai.com/#/bitcoin-whitepaper-cn-en-translation/Bitcoin-Whitepaper-EN-CN.html)

除此之外，对区块链技术的演变也要有一定的了解。作为区块链2.0的代表的以太坊是同样值得学习：

- [以太坊白皮书](https://github.com/lizj3624/mynote/blob/master/blockchain/ETH/%E4%BB%A5%E5%A4%AA%E5%9D%8A%E7%99%BD%E7%9A%AE%E4%B9%A6-zh.md)
- [以太坊开发入门指南](https://learnblockchain.cn/2017/11/20/whatiseth/)
- [精通以太坊（中文版）](https://github.com/lizj3624/ethereum_book)
- [《区块链技术指南》](https://github.com/yeasy/blockchain_guide/blob/master/SUMMARY.md)

记住！你不需要一字一句全部读完，遇到不懂的概念和知识点记录下来和忽略。在这个过程中，配合搜索引擎，你需要不断的去思考和回答以下几个问题：

- 区块链、比特币和以太坊是什么？它们的工作原理大概是如何的？比特币和以太坊的区别？
- 它们具备什么性质？包含哪些关键的技术点？
- 区块链的发明目的是解决什么问题？除此以外，还能解决什么问题？
- 区块链具备什么优缺点？

完成这一步后，你已经对区块链和比特币有一个相对宏观、整体的认识，并初步建立了一个属于你自己的知识框架，此框架在后面的学习中会起到重要的作用。
也许你会觉得你建立的框架并不正确和完整，但不要紧，随着你的认识加深，不断的修正和完善框架。学习本身就是一个螺旋式上升的过程。

## 二、从外到内，逐一突破

建立知识框架之后，下一步你需要做的就是丰富和完善它。对于第一步中遗留的那些「似懂非懂」的概念和知识点，便可以在这个环节中逐一突破。

简单来说，你要做的便是：**主动学习——快速定位你存在疑惑的概念和知识点，用一切办法来攻克它。**

在这个阶段中， 除了回顾上面推荐的材料以外， 你需要广泛的搜索，不局限任何的形式。高质量的搜索结果依赖于准确的问题定义，
因此在查询过程中也逐渐帮助定义清楚你的疑惑。此外推荐两个关于区块链技术的中文社区：

- [深入浅出区块链](https://learnblockchain.cn/)
- [比特币布道者](http://btc.mom/)

当你在攻克某一个知识点和概念的过程中，你一定会遇到其他新的知识点，此时你便可以顺藤摸瓜，把相关的知识一并学习吸收。
值得注意的是，在这个过程中需要把握好知识扩展的度，避免过度分散注意力，重点还是关注原来的疑惑本身，以目标为导向。至于如何把握，我的建议是：

- 判断新遇到的概念是否直接影响到你理解原来的概念？若是，务必一切办法攻克它；
- 新概念在整个框架（知识体系）中是否占有很重要的地位？还是说只是某个知识点延伸出来的小分支？
  比如：在你了解P2P网络时遇到新概念分布式哈希表（DHT），此概念只不过是P2P网络中用于定位特定节点或数据的一个技术手段，显然可暂时不需要深入了解；
- 要理解新知识点的要求是否远超过当前的知识储备？若是，你也可以暂且先放一放，如：[EVM原理](https://cnodejs.org/topic/5aeecba802591040485bab2a)。

重点关注的范围还是围绕区块链的工作原理相关的概念为主，如：1）如何处理交易和记账；2）如何产生区块及达成共识；如何验证和储存状态等过程中的重要概念。此外，不必过分追求技术的实现细节。

而关于如何判断是否真正理解透彻，我建议使用「[费曼方法](https://zhuanlan.zhihu.com/p/152547764)，
尝试用自己的方式，用通熟易懂的语言描述清楚。当解决完成相关的概念后，把它们重新放在框架里，不断修正和完善框架。

完成这一步后，我相信你已经对区块链及两个应用的大部分重要概念都理解通透了，基本概念如：
- 去中心化
- [共识机制](https://yeasy.gitbook.io/blockchain_guide/06_bitcoin/consensus)
- 工作量证明PoW、PoS
- 非对称加密
- 硬（软）分叉
- 双花
- 智能合约
- Merkle Tree(默克尔树)
- 51%攻击

## 三、从点到面，构建知识网络

进入第三部以后，你的区块链技术算是入门了。基于你所建立的框架，你已经有能力去理解之前晦涩难懂的概念。接下来，你便可以进一步扩大区块链技术的广度和深度，如：

- 其他的区块链项目，如：Filecoin、Fabric、EOS等
- 不同类型的共识算法
- 零知识证明
- 区块链的可扩展性方案
- 智能合约的编写
- ……

当你学习上述新的知识的过程中，你需要刻意的去思考和构建知识间的「**联系**」——此知识和别的知识有什么关系？是如何关联一起的？

> **知识的本质永远不是信息本身，而是信息之间的联系。正是这种联系，涌现出了超越单个信息点总和的「系统性」。**

**而区块链技术的创新本身也恰恰是「系统性」。** 在这个过程中，我主要的使用如下的系统性方法

### 对比
将解决同一个问题的不同技术手段归纳整理在一起，多维度进行对比，找出共性和异性。比如：PoW与PoS之间的区别？

![01](./bc-01.jpg)
> PoW vs PoS

### 分类
目前解决区块链的可行性方案有哪些潜在的研发方向？具体有哪些技术手段？

![02](./bc-02.jpg)
> 区块链技术可扩展方案

### 提炼
尝试用最精炼的语言貌似一类相关的知识点，比如比特币的核心原理：

> 1. 中本聪使用非对称加密解决电子货币的所有权问题；
> 2. 用区块时间戳解决交易的存在性问题；
> 3. 用分布式账本解决剔除第三方机构后交易的验证问题；
> 4. 用工作量证明和最长链约定来保证节点状态的一致性，已解决「双花」问题。

### 架构
尝试对系统中的关键模块和模块间的关系进行抽象，并绘制成架构图，如：区块链的分层架构。

![03](./bc-03.jpg)
> 区块链分层架构

### 流程
也可以将根据信息流将不同的知识点串联在一起，绘制成流程图。如：以太坊交易打包流程。

![04](./bc-04.jpg)
> 以太坊交易打包流程，来源：CSDN

总之，你需要想尽一些的办法，将知识点关联在一起，逐渐结构化、系统化。

## **四、实践是检验真理的唯一标准**

到这一步后，你掌握的区块链技术的知识体系逐步成型，接下来需要做的便是将技术落地到应用中。

首先，尝试在本地搭建比特币、以太坊的测试网络，和做不同类型的交易交易。对于以太坊，你还可以部署和调用智能合约等等。

- [以太坊本地私有链开发环境搭建](https://link.zhihu.com/?target=https%3A//ethfans.org/posts/ethereum-private-network-bootstrap)

开始编写更加复杂的Dapp应用：

- [Solidity语言文档](https://link.zhihu.com/?target=http%3A//www.tryblockchain.org/)
- [Web3.JS接口文档](https://link.zhihu.com/?target=http%3A//web3.tryblockchain.org/)
- [Truffle框架文档](https://link.zhihu.com/?target=http%3A//truffle.tryblockchain.org/)
- [Open Zeppelin框架文档](https://link.zhihu.com/?target=http%3A//zeppelin.tryblockchain.org/)
- [Ethereum Smart Contract Security Best Practices](https://link.zhihu.com/?target=https%3A//consensys.github.io/smart-contract-best-practices/)
- [Ethereum Voting Dapp](https://link.zhihu.com/?target=https%3A//github.com/maheshmurthy/ethereum_voting_dapp)
- [React Ethereum Dapp Example](https://link.zhihu.com/?target=https%3A//github.com/leopoldjoy/react-ethereum-dapp-example)

在此环节，你的主要目标是熟悉并掌握开发Dapp的相关技能和工具。

## 五、Code As Documentation

最后一步，选择一个你感兴趣的项目，阅读它的源码，了解底层技术的实现原理，将理论与实践进一步融会贯通。
关于项目的选择，我个人建议是以太坊，至今为止，以太坊的应用面还是最广的，受到各大互联网公司的青睐。

![05](./bc-05.jpg)

至于如何阅读和学习以太坊的源码，个人建议结合以太坊的黄皮书对比阅读学习。可参考：

- [以太坊黄皮书-中文版](https://github.com/wanshan1024/ethereum_yellowpaper/blob/master/ethereum_yellow_paper_cn.pdf)
- [以太坊代码剖析](https://mindcarver.cn/)
- [解读以太坊黄皮书](https://github.com/lizj3624/mynote/tree/master/blockchain/ETH)
- [以太坊源代码分析](https://www.jianshu.com/u/78a086870bbe)
- [Go Ethereum Code Analysis](https://github.com/ZtesoftCS/go-ethereum-code-analysis)

## *七、最后的最后**

区块链行业真处于高速发展的时候，作为区块链从业人员，不仅仅要掌握技术，还需要时刻掌握行业动态，挖掘其他有价值的项目，把握认知变现的机会。

- [《区块链革命》](https://book.douban.com/subject/26871829/)
- [《货币的非国家化》](https://book.douban.com/subject/2155342/)

希望以上答案可以对你有所帮助！
