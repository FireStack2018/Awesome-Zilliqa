# Zilliqa官网文档中文版

Scilla是一种智能合约中级语言的简称，它是为[Zilliqa](https://zilliqa.com/)而开发的。Scilla以智能合约的高安全性为设计理念。

Scilla在智能合约的基础上加入了一个新的架构，通过在语言层面针对性修复已知漏洞，使应用程序更不易受到攻击。此外，Scilla的基础架构将使应用程序本身更安全并且易于进行形式验证。

该语言的语法正逐步规范中，并将嵌进[Coq proof assistant](https://coq.inria.fr/) - 一种用于程序性能的机械化证明的最先进工具。Coq基于先进的依赖型理论，拥有大量的数学工具库。它先前已成功应用于实现认证（即完全机械验证）的编译器、并发和分布式应用程序、及区块链相关方面。

Zilliqa - 运行Scilla合约的底层区块链平台，旨在实现可扩展性。它采用分片的思想来验证并发交易。Zilliqa有一种名为Zilling的token ，简称ZIL。Zilliqa上运行智能合约需要消耗ZIL。

## 发展状况

Scilla正在积极研究和开发中，因此本文档中部分描述可能会发生变化。Scilla目前提供了一个二进制解释器，它已集成到两个Scilla指定的Web-IDE中。[尝试Scilla](./Scilla/尝试Scilla.md)对这两个IDE的功能进行了详细介绍。

请注意，Scilla尚未实现类型检查器，因此无法保证用Scilla编写的合约的类型安全性。

## Resources

可以通过相关资源来了解Scilla和Zilliqa，如下：

- **Scilla**

  [Scilla设计构思：第1部分（为什么我们需要一种新语言？）](https://blog.zilliqa.com/%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E8%AF%AD%E8%A8%80scilla%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%95%85%E4%BA%8B-2c2927bfbc94)

- **Zilliqa**

  [Zilliqa设计构思：第1部分（网络分片）](https://blog.zilliqa.com/zilliqa%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%9E%84%E6%80%9D-483839031cf0)

  [Zilliqa设计构思：第2部分（共识机制）](https://blog.zilliqa.com/zilliqa-%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%9E%84%E6%80%9D-4ca8a3f61f99)

  [Zilliqa设计构思：第3部分（使共识更高效）](https://blog.zilliqa.com/zilliqa-%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%9E%84%E6%80%9D-e9a340a72bec)

## 目录

- [关于](./README.md)
- [Scilla设计原理](./Scilla/Scilla设计原理.md)
- [尝试Scilla](./Scilla/尝试Scilla.md)
- [Scilla入门](./Scilla/Scilla入门.md)
- Scilla进阶
  - [Scilla合约架构](./Scilla/Scilla合约架构.md)
  - [原生数据类型&操作](./Scilla/原生数据类型&操作.md)
  - [代数数据类型](./Scilla/代数数据类型.md)
- [编译器介绍](./Scilla/编译器介绍.md)
- [JSON-RPC接口文档](../JSON-RPC_cn/JSON-RPC_cn.md)

