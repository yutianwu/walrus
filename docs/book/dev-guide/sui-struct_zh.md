# Sui 结构

本节是可选的，支持高级用例，利用 Sui 智能合约与 Walrus 智能合约的交互。

您可以纯粹通过客户端 CLI 以及提供的 JSON 或 HTTP API 与 Walrus 交互，而无需直接在 Sui 上查询或执行交易。但是，Walrus 使用 Sui 来管理其元数据，智能合约开发者可以在 Sui 上读取有关 Walrus 系统以及存储的 blob 的信息。

[Walrus 系统合约的 Move 代码](https://github.com/MystenLabs/walrus/tree/main/contracts)可用，您可以找到一个实现包装 blob 的[示例智能合约](https://github.com/MystenLabs/walrus/tree/main/docs/examples)。以下部分提供了对合约的进一步见解以及如何在您自己的 Sui 智能合约中使用 Walrus 对象的概述。

## Blob 和存储对象

Walrus blob 表示为 `Blob` 类型的 Sui 对象。blob 首先被注册，表示存储节点应该预期来自 Blob ID 的片段被存储。然后 blob 被认证，表示已存储足够数量的片段以保证 blob 的可用性。当 blob 被认证时，其 `certified_epoch` 字段包含它被认证的时期。

`Blob` 对象始终与 `Storage` 对象关联，为 blob 的存储保留足够长时间的足够空间。认证的 blob 在底层存储资源保证存储的期间内可用。

具体来说，`Blob` 和 `Storage` 对象具有以下字段，可以通过 Sui SDK 读取：

```move
/// Reservation for storage for a given period, which is inclusive start, exclusive end.
public struct Storage has key, store {
    id: UID,
    start_epoch: u32,
    end_epoch: u32,
    storage_size: u64,
}

/// The blob structure represents a blob that has been registered to with some storage,
/// and then may eventually be certified as being available in the system.
public struct Blob has key, store {
    id: UID,
    registered_epoch: u32,
    blob_id: u256,
    size: u64,
    encoding_type: u8,
    // Stores the epoch first certified if any.
    certified_epoch: option::Option<u32>,
    storage: Storage,
    // Marks if this blob can be deleted.
    deletable: bool,
}
```

与这些对象关联的公共函数可以在相应的 [`storage_resource` move 模块](https://github.com/MystenLabs/walrus/tree/main/contracts/walrus/sources/system/storage_resource.move)和 [`blob` move 模块](https://github.com/MystenLabs/walrus/tree/main/contracts/walrus/sources/system/blob.move)中找到。存储资源可以在时间和数据容量上拆分和合并，并可以在用户之间转移，允许创建复杂的合约。

## 事件

Walrus 广泛使用[自定义 Sui 事件](https://github.com/MystenLabs/walrus/blob/main/contracts/walrus/sources/system/events.move)来通知存储节点有关存储的 Blob 和网络状态的更新。应用程序也可以使用 Sui RPC 设施来观察 Walrus 相关事件。

当 blob 首次注册时，会发出 `BlobRegistered` 事件，通知存储节点它们应该预期与其 Blob ID 关联的片段。最终当 blob 被认证时，会发出包含有关 blob ID 和 blob 被删除后的时期信息的 `BlobCertified`。在该时期之前，blob 保证可用。

`BlobCertified` 事件的 `deletable` 设置为 false 且 `end_epoch` 在未来表示 blob 将可用到此时期。为 blob ID 发出此事件的轻客户端证明构成具有此 blob ID 的数据的可用性证明。当可删除的 blob 被删除时，会发出 `BlobDeleted` 事件。

当存储节点检测到错误编码的 blob 时会发出 `InvalidBlobID` 事件。任何尝试读取此类 blob 的人都保证也会检测到它是无效的。

系统级事件如 `EpochChangeStart` 和 `EpochChangeDone` 表示时期之间的转换。相关事件如 `ShardsReceived`、`EpochParametersSelected` 和 `ShardRecoveryStart` 表示与时期转换、分片迁移和时期参数相关的存储节点级事件。

## 系统和质押信息

[Walrus 系统对象](https://github.com/MystenLabs/walrus/blob/main/contracts/walrus/sources/system/system_state_inner.move)包含有关可用和已使用存储的元数据，以及以 FROST 为单位的每 KiB 存储的存储价格。系统对象内的委员会结构可用于读取当前时期号以及有关委员会的信息。时期之间的委员会变更由一组[质押合约](https://github.com/MystenLabs/walrus/tree/main/contracts/walrus/sources/staking)管理，这些合约实现基于 WAL 代币的完整委托权益证明系统。