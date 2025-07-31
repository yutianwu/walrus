# 操作

## Blob 编码和 blob ID

Walrus 在存储节点上以[编码形式](../design/encoding.md)存储 blob，并通过它们的 *blob ID* 引用 blob。blob ID 是根据 blob 的内容和 Walrus 配置确定性地派生的。具有相同内容的两个文件的 blob ID 将是相同的。

您可以使用命令在本地派生文件的 blob ID：`walrus blob-id <file path>`。

## 存储

Walrus 可用于通过原生客户端 API 或发布器**存储 blob**。

```admonish danger title="公共访问"
**存储在 Walrus 中的所有 blob 都是公开的，所有人都可以发现。** 因此，您不得使用 Walrus 存储任何包含机密或私人数据的内容，除非采取额外措施保护机密性。
```

在底层，在 Sui 和存储节点上都会发生许多操作：

- 客户端或发布器编码 blob 并派生出标识 blob 的 *blob ID*。这是一个通常编码为 URL 安全 base64 字符串的 `u256`。
- 在 Sui 上执行交易以从系统对象购买一些存储，然后*注册占用此存储的 blob ID*。客户端 API 返回 *Sui blob 对象 ID*。交易使用 SUI 购买存储并支付 gas。
- blob 的编码片段分发到所有存储节点。它们各自签署收据。
- 签名收据被聚合并提交到 Sui blob 对象以*认证 blob*。认证 blob 会发出带有 blob ID 和可用性期间的 Sui 事件。

一旦相应的 Sui blob 对象在最后一步中得到认证，blob 就被认为在 Walrus 上可用。存储操作涉及的步骤可以由二进制客户端执行，或者由通过 HTTP 接受和发布 blob 的发布器执行。

Walrus 目前允许存储最大大小的 blob，可以通过 [`walrus info`](../usage/client-cli.md#walrus-system-information) CLI 命令确定。最大 blob 大小目前为 13.3&nbsp;GiB。您可以通过将较大的 blob 拆分为较小的块来存储它们。

Blob 存储一定数量的*时期*，如存储时指定的那样。Walrus 存储节点确保在这些时期内读取成功。当前主网使用 2 周的时期持续时间。

## 读取

Walrus 也可用于通过提供其 blob ID 在存储后**读取 blob**。
读取通过执行以下步骤来完成：

- 读取 Sui 上的系统对象以确定 Walrus 存储节点委员会。
- 查询多个存储节点以获取 blob 元数据和它们存储的片段。
- 从恢复的片段重构 blob 并根据 blob ID 进行检查。

读取操作涉及的步骤由二进制客户端或聚合器服务执行，后者公开 HTTP 接口来读取 blob。读取具有极强的弹性，即使多达三分之一的存储节点不可用，也能在所有情况下成功恢复 blob。在大多数情况下，同步完成后，即使三分之二的存储节点宕机，也可以读取 blob。

## 认证可用性

Walrus 可用于使用 Sui **认证 blob 的可用性**。检查这是否发生目前可以通过 3 种不同方式完成：

- 可以使用 Sui SDK 读取来认证在 Sui 上认证 blob ID 时发出的认证 blob 事件。客户端 `walrus blob-status` 命令可用于识别需要检查的事件 ID。
- 可以使用 Sui SDK 读取来认证与 blob ID 对应的 Sui blob 对象，并检查它是否已认证、在到期时期之前且不可删除。
- Sui 智能合约可以读取 Sui 上的 blob 对象（或对它的引用）以检查它是否已认证、在到期时期之前且不可删除。

[Sui 轻客户端](https://github.com/MystenLabs/sui/tree/main/crates/sui-light-client)的底层协议返回发出事件或对象的数字签名证据，并可被离线或非交互式应用程序用作 blob ID 在一定数量时期内可用性的证明。

一旦 blob 得到认证，Walrus 将确保存储节点上始终有足够的片段可用于在指定时期内恢复它。

## 删除

存储的 blob 可以由创建它们的用户可选择设置为可删除。此元数据存储在 Sui blob 对象中，blob 是否可删除包含在认证 blob 事件中。可删除的 blob 可以由 blob 对象的所有者删除，以回收和重用与其关联的存储资源。

如果 Walrus 中不存在 blob 的其他副本，删除 blob 最终将使其无法使用读取命令恢复。但是，如果 Walrus 上存在 blob 的其他副本，删除命令将为调用它的用户回收存储空间，但不会使 blob 不可用，直到所有其他副本都被删除或过期。