# 创建新的区块

> 在Linera中，提出区块和验证区块的责任是分开的。

尽管所有链的验证方式相同，但Linera协议根据生成新区块的方式定义了几种类型的链。

- 最简单且延迟最低的链称为单所有者链。
- Linera SDK目前不支持的其他类型的链包括权限链和公共链（更多详细信息请参阅[白皮书](https://linera.io/whitepaper)）。

> 对于大多数链（除了公共链），Linera验证器不需要彼此交换消息。

相反，链所有者的钱包（也称为`linera`客户端）通过提出区块并向验证器主动提供任何额外所需的数据来推动系统进展。例如，客户端命令如`transfer`、`publish-bytecode`或`open-chain`执行多个步骤来追加包含令牌转移、应用程序发布或链创建操作的区块：

- Linera客户端创建一个新的区块，包含所需的操作和新的传入消息（如果有）。它还包含最近区块的哈希，以指定其父区块。客户端将新区块发送给所有验证器。
- 验证器验证区块，即检查区块是否满足上述条件，并向客户端发送加密签名，表示他们投票追加新区块。但前提是他们之前在相同高度上没有为其他区块投票！
- 客户端理想情况下收到每个验证器的投票，但只需获得多数票（例如三分之二）即可：这构成了一个“证书”，证明该区块已确认。客户端将证书发送给每个验证器。
- 验证器“执行”区块：他们通过应用所有消息和操作更新他们对链最新状态的视图，如果生成任何跨链消息，他们将这些消息发送给适当的工作节点。

为了保证区块中每个传入消息确实是由另一个链发送的，验证器在第二步只会为已执行发送消息的区块投票。然而，如果收到一个有效的区块证书，其中包含尚未看到的消息，他们仍然会接受并执行该区块。证书证明了大多数其他验证器已看到该消息，因此必须是正确的。

对于单所有者链，客户端必须小心实施，以确保它们从不在同一高度提议多个区块。否则，链可能会被卡住：一旦两个冲突的区块各自获得足够多的验证器签名，就无法收集到任何一个区块的多数票。

在未来，我们预计大多数用户即使是他们链的唯一所有者也将使用权限链。权限链具有两个确认步骤而不是一个，但不可能意外使链无法扩展。它们还允许用户委托某些管理任务给第三方，特别是在处理时代更改时（例如当验证器重新配置时）。