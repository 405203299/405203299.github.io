## 测试虚拟机间的 gas 计量和确定性执行概述

Linera 是为需要保证性能的无限活跃用户的 Web3 应用程序优化的去中心化基础设施。

Linera 协议的核心理念是在一组验证者中并行运行许多轻量级区块链，称为**微链**。

## Linera 工作原理

在 Linera 中，用户钱包操作其自己的微链。链的所有者可以选择何时向链中添加新的区块以及区块中包含的内容。这样的链称为**用户链**。

用户可以向他们的链中添加新的区块，以处理来自其他链的**入站消息**，或者执行安全的**操作**，例如将资产转移给另一个用户。

重要的是，验证者确保所有新区块是**有效**的。例如，转账操作必须起始于具有足够资金的账户；入站消息必须确实来自另一条链。验证者对每条链的区块都以相同的方式进行验证。

Linera 的一个**应用程序**是一个定义了自己状态和操作的 Wasm 程序。用户可以发布字节码并在一条链上初始化一个应用程序，它将自动部署到所有需要它的链上，并在每条链上有独立的状态。

为了确保跨链协调，应用程序可以依赖于异步的**跨链消息**。消息负载是应用程序特定的，对系统的其余部分是不透明的。

```ignore
                               ┌───┐     ┌───┐     ┌───┐
                       Chain A │   ├────►│   ├────►│   │
                               └───┘     └───┘     └───┘
                                                     ▲
                                           ┌─────────┘
                                           │
                               ┌───┐     ┌─┴─┐     ┌───┐
                       Chain B │   ├────►│   ├────►│   │
                               └───┘     └─┬─┘     └───┘
                                           │         ▲
                                           │         │
                                           ▼         │
                               ┌───┐     ┌───┐     ┌─┴─┐
                       Chain C │   ├────►│   ├────►│   │
                               └───┘     └───┘     └───┘
```

单条链上可以存在的应用数量没有限制。在同一条链上，应用程序可以像往常一样通过同步调用进行**组合**。

当前的 Linera SDK 使用**Rust**作为源语言来创建 Wasm 应用程序。它依赖于常规的 Rust 工具链，使得 Rust 程序员可以在他们喜欢的环境中工作。

> ## Linera 如何与现有的多链基础设施相比？
>
> Linera 是第一个旨在支持并行运行多条链的基础设施，特别是一种由用户钱包操作的任意数量的**用户链**。
>
> 在传统的多链基础设施中，每条链通常在单独的一组验证者中运行完整的区块链协议。创建新链或在链之间交换消息都是昂贵的。因此，总链条数量通常是有限的。有些链可能专门用于特定的用例，称为“应用链”。
>
> 相比之下，Linera 针对大量用户链进行了优化：
>
> - 用户只在需要时在其链上创建区块；
> - 创建微链不需要验证者入职；
> - 所有链具有相同的安全级别；
> - 微链使用验证者内部网络高效通信；
> - 验证者内部分片化（类似常规的网络服务），可以通过添加或移除内部工作节点弹性调整其容量。
>
> > 除了用户链，[Linera 协议](https://linera.io/whitepaper) 还设计支持其他类型的微链，如“权限”链和“公共”链。“公共”链由验证者操作，从这个角度来看，它们类似于传统的区块链。“权限”链用于临时用户之间的交互，如原子交换。

## 为什么在 Linera 之上构建？

我们认为许多高价值的应用场景目前无法通过现有的 Web3 基础设施实现，因为这些基础设施在同时为**许多活跃用户**提供服务时面临挑战（费用不可预测、延迟等）。

需要处理由许多同时用户创建的时间敏感交易的应用示例包括：

- 实时微支付和微奖励，
- 社交数据源，
- 实时拍卖系统，
- 回合制游戏，
- 软件、数据管道或 AI 训练管道的版本控制系统。

轻量级用户链在提供弹性可扩展性方面非常重要，但它们还有其他好处。由于用户链的区块较少，因此在 Linera 中，用户链的全节点将嵌入到用户的钱包中，通常部署为浏览器扩展程序。

这意味着连接到钱包的 Web UI 将能够直接查询用户链的状态（无需 API 提供者、无轻客户端），使用熟悉的框架（如 React/GraphQL）。此外，钱包还可以利用全节点来增强安全性，包括向用户显示有意义的确认消息。

## Linera 的当前开发状态如何？

Linera 的[参考开源实现](https://github.com/linera-io/linera-protocol)正在积极开发中。它已经包含一个 Web3 SDK，具备原型化简单的 Web3 应用程序并在同一台机器上进行本地测试并部署到 Devnet 所需的功能。

可以基于嵌入 Wasm 的 GraphQL 服务构建 Web UI（可能是响应式的），并在浏览器中进行本地测试。

我们当前 Web3 SDK 的主要限制包括：

- Web UI 需要查询充当钱包的本地 HTTP 服务。这种设置仅用于临时和测试：将来，Web UI 将像往常一样安全地连接到安装为浏览器扩展的钱包。
- 目前本手册仅记录了用户链。最近添加了“权限”链（即“临时”链）。对公共链的支持正在进行中。

Linera 的主要开发工作流除了 SDK 外，可以分解如下。

### 核心协议

- [x] 用户链
- [x] 权限链（仅核心协议）
- [x] 跨链消息
- [x] 跨链发布/订阅通道（初始版本）
- [x] 字节码发布
- [x] 应用程序创建
- [x] 验证者重新配置
- [x] 支持 gas 费用
- [x] 支持存储费用和存储限制
- [x] 外部服务（称为“水龙头”）帮助用户创建其第一条链
- [x] 权限链（添加操作访问控制、原子交换演示等）
- [ ] 避免重复从存储加载链状态
- [ ] 可供系统和用户应用程序使用的 Blob 存储（泛化/替换字节码存储）
- [ ] 支持将用户链轻松引入新应用程序（去除接受请求的需求）
- [ ] 替换发布/订阅通道为数据流（去除接受订阅的需求）
- [ ] 允许链客户端控制其跟踪（惰性/主动）和执行（不执行所有跟踪的链）
- [ ] 多签名事件以便未来与外部链的桥接
- [ ] 公共链（添加领导选举、收件箱约束等）
- [ ] 交易脚本
- [ ] 支持动态分片分配
- [ ] 支持归档链
- [ ] 所有利益相关者的 Tokenomics 和激励
- [ ] 在管理链上的治理（例如 DPoS、验证者入职）
- [ ] 无需许可的审计协议

### Wasm VM 集成

- [x] 支持 Wasmer VM
- [x] 支持 Wasmtime VM（实验性）
- [x] 测试虚拟机间的 gas 计量和确定性执行
- [x] 在同一链上组合 Wasm 应用程序
- [x] 支持非阻塞（但确定性）的对存储的调用
- [x] 支持在 Wasm 中的只读 GraphQL 服务
- [x] 支持模拟系统 API
- [x] 改进主机/客户桩生成，以使模拟更容易
- [ ] 支持在浏览器中运行 Wasm 应用程序

### 存储

- [x] 基于键值存储抽象的对象管理库（“linera-views”）
- [x] 支持 Rocksdb
- [x] 实验性支持 DynamoDb
- [x] 为 GraphQL 派生宏
- [x] 支持 ScyllaDb
- [x] 使库对用户完全可扩展（需要更好的 GraphQL 宏）
- [x] 用于测试目的的内存存储服务
- [x] 支持 Web 存储（IndexedDB）
- [ ] 性能基准和改进（包括更快的状态哈希）
- [ ] 更好的配置管理
- [ ] 本地写前日志
- [ ] 选定主数据库的生产级支持
- [ ] 调试工具
- [ ] 使存储库在 Linera 之外易于使用

### 验证者基础设施

- [x] 简单的 TCP/UDP 网络（仅用于基准测试）
- [x] GRPC 网络
- [x] 支持固定内部分片的基本前端（称为代理）
- [x] 可观察性
- [x] CI 中的 Kubernetes 支持
- [x] 使用云提供商部署
- [ ] 水平可扩展的前端（称为代理）
- [ ] 动态分片分配
- [ ] 云集成以演示弹性扩展

### Web3 SDK

- [x] 合同和服务接口的特性
- [x] 支持单元测试
- [x] 支持集成测试
- [x] 本地 GraphQL 服务以查询和浏览系统状态
- [x] 本地 GraphQL 服务以查询和浏览应用程序状态
- [x] 使用 GraphQL mutations 执行操作和创建区块
- [x] 合同和服务接口的 ABI
- [x] 允许消息发送者支付消息执行费用
- [ ] 钱包作为浏览器扩展（无 VM）
- [ ] 钱包作为浏览器扩展（带 Wasm VM）
- [ ] 更容易与 EVM 链通信
- [ ] 绑定以使用 Wasm 的原生加密原语
- [ ] 允许应用程序支付用户费用
- [ ] 允许应用程序使用权限链和公共链
