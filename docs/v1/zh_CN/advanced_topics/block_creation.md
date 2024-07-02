# 4.4. 创建新区块

> 在Linera中，创建新区块与验证区块是分离的。

虽然所有微链都以同样方式验证，但是由于区块生产方式不同，Linera协议中定义了几种不同类型的微链：

- 最简单、延迟最低的微链是*单所有者*链。
- *许可链*和*公开链*暂未支持(参见[白皮书](https://linera.io/whitepaper)查看更多内容)。

> 除了*公开链*，Linera验证器不需要为其他微链交换消息。

取而代之的是，链所有者的钱包(即`linera`客户端)通过创建区块，以及主动向验证器提供所有所需数据来确保系统正常运行。例如，`transfer`, `publish-bytecode`, 或`open-chain`这些客户端命令行通过执行多个步骤，向微链添加包含转账、应用发布或微链创建的区块：

- Linera客户端创建一个包含所需操作和新接收消息（如果有的话）的新区块。新区块中也包含最新的已认证区块哈希，作为父区块。客户端将区块发送给所有验证器。
- 验证器验证区块，即检查区块是否满足上面(译者注：这里的上面指前面的区块验证章节)列出的条件，然后向客户端发送一个密码学签名，表示验证器投票同意将该区块添加到微链上。如果此前验证器已经在同样高度为其他区块投票，那么验证器不应该再为本区块投票。
- 理想情况下，客户端应该能够收到每个验证器的一次投票，但是仅需要其中的一部分投票(2/3)便可达成共识：这样就形成一个证书，表示区块已经被确认。客户端将已经确认的证书发送给每个验证器。
- 验证器“执行”区块：验证器将区块中的全部消息和操作应用到最新状态上，并更新视图，如果有跨链消息产生，验证器将消息发送到合适的工作节点处理。

在第二步中，验证器仅为那些已经执行成功的区块*投票*，以确保区块中的每条消息都是由其他微链发送。然而，当验证器收到一个认区块证书，其中包含它未见过的消息时，验证器将会接受并*执行*该区块。区块证书意味着其他验证器已经见过并验证过区块内的消息，因此可以证明消息都是正确的。

单所有者链的情况下，客户端需要精心实现，确保不会在同一个高度创建多个区块，否则，微链将会停止：一旦两个冲突的区块中的每一个都被足够多的验证者签名，就不可能为任何一个区块收集到足够的选举票，从而无法达成最终共识。

将来，我们预期大部分用户都会使用*许可链*，即使他们创建的微链只有链所有者。与单所有者链的一步确认不同，许可链将会通过两个步骤确认区块，从而避免微链被冲突区块停止。许可链也允许用户将特定管理任务委托给到第三方，从而帮助改变epoch(即重新配置验证器)。