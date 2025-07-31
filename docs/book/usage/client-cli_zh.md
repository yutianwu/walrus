# 使用 Walrus 客户端

`walrus` 二进制文件可用于作为客户端与 Walrus 交互。有关先决条件、安装和配置，请参阅[设置章节](./setup_zh.md)。

详细的使用信息（包括可用命令的完整列表）可以通过以下方式查看：

```sh
walrus --help
```

`walrus` 的每个子命令也可以使用 `--help` 调用以打印其特定参数及其含义。

如果您的配置文件中有多个*上下文*（如[设置页面](./setup_zh.md#configuration)中包含的默认文件），您可以通过 `--context` 选项为每个命令指定上下文。

您可以使用 `walrus completion` 生成 bash/zsh/fish 完成脚本，并将其放在适当的目录中，如 `~/.local/share/bash-completion/completions`。

## Walrus 系统信息

有关 Walrus 系统的信息可通过 `walrus info` 命令获得。它提供了当前系统参数的概览，如当前纪元、系统中存储节点和分片的数量、最大 blob 大小以及存储 blobs 的当前 WAL 成本：

```console
$ walrus info

Walrus system information

Epochs and storage duration
Current epoch: 1
Start time: 2025-03-25 15:00:24.408 UTC
End time: 2025-04-08 15:00:24.408 UTC
Epoch duration: 14days
Blobs can be stored for at most 53 epochs in the future.

Storage nodes
Number of storage nodes: 103
Number of shards: 1000

Blob size
Maximum blob size: 13.6 GiB (14,599,533,452 B)
Storage unit: 1.00 MiB

Storage prices per epoch
(Conversion rate: 1 WAL = 1,000,000,000 FROST)
Price per encoded storage unit: 0.0001 WAL
Additional price for each write: 20,000 FROST

...
```

```admonish tip title="纪元持续时间"
主网上的纪元持续时间为 2 周。有关主网和测试网的其他参数，请参见[此处](./networks_zh.md#network-parameters)。
```

```admonish tip title="FROST 和 WAL"
FROST 是 WAL 的较小单位，类似于 SUI 的 MIST。转换也与 SUI 相同：`1 WAL = 1 000 000 000 FROST`。
```

其他信息，如编码参数和大小、BFT 系统信息，以及当前和（如果已选择）下一个委员会中存储节点的信息，包括它们的节点 ID 和质押及分片分布，可以通过各种子命令查看，详细信息请参见 `walrus info --help`。请注意，之前的 `--dev` 选项已被 `all` 子命令替换。

存储节点的健康状况可以通过 `walrus health` 命令检查。此命令采用不同的选项来选择要检查的节点（详细信息请参见 `walrus health --help`）。例如，`walrus health --committee` 检查所有当前委员会成员的状态。

## 存储 blobs

```admonish danger title="公共访问"
**存储在 Walrus 中的所有 blobs 都是公共的，所有人都可发现。** 因此，您不得使用 Walrus 存储包含机密或私人数据的任何内容，除非采取额外措施保护机密性。
```

```admonish warning
必须确保只有单个进程使用 Sui 钱包进行写操作（存储或删除）。同时使用客户端的多个实例时，每个实例都必须指向不同的钱包。请注意，可以使用单个 `walrus store` 命令存储多个 blobs。
```

在 Walrus 上存储一个或多个 blobs 可以通过以下命令实现：

```sh
walrus store <FILES> --epochs <EPOCHS>
```

必须设置强制性 CLI 参数来指定 blob 的生命周期。目前有三种方式：

1. `--epochs <EPOCHS>` 选项表示 blob 应存储的纪元数。blob 可存储的纪元数有上限，为 53，对应两年。除了正整数，您还可以使用 `--epochs max` 将 blob 存储最大纪元数。请注意，结束纪元定义为当前纪元加上指定的纪元数。
1. `--earliest-expiry-time <EARLIEST_EXPIRY_TIME>` 选项采用 RFC3339 格式的日期（例如，"2024-03-20T15:00:00Z"）或更宽松的格式（例如，"2024-03-20 15:00:00"），并确保 blob 在可能的情况下在该日期之后过期。
1. `--end-epoch <END_EPOCH>` 选项为 blob 指定特定的结束纪元。

```admonish warning title="结束纪元"
blob 在其结束纪元*开始时*过期。例如，结束纪元为 `314` 的 blob 将在纪元 `314` 开始时变得不可用。这样做的一个后果是，在纪元变更前立即使用 `--epochs 1` 存储 blob 时，它将过期并几乎立即变得不可用。请注意，只有在 blobs 尚未过期时才能[延长](#extending-the-lifetime-of-a-blob)其生命周期。
```

您可以存储单个文件或多个文件，用空格分隔。值得注意的是，这与 glob 模式兼容；例如，`walrus store *.png --epochs <EPOCHS>` 将存储当前目录中的所有 PNG 文件。

默认情况下，该命令将 blob 存储为*永久* blob，尽管这将在不久的将来发生变化。有关可删除 blobs 的更多详细信息，请参见[可删除 blobs 部分](#reclaiming-space-via-deletable-blobs)。此外，默认情况下会创建拥有的 `Blob` 对象。可以将其包装为共享对象，任何人都可以资助和扩展，请参见[共享 blobs 部分](#shared-blobs)。

存储 blob 时，客户端执行许多自动优化，包括以下内容：

- 如果 blob 已作为*永久 blob* 存储在 Walrus 上足够的纪元数，该命令不会再次存储它。可以使用 `--force` CLI 选项覆盖此行为，该选项再次存储 blob 并在 Sui 上创建属于钱包地址的新 blob 对象。
- 如果用户的钱包有合适大小和持续时间的存储资源，则（重新）使用它而不是购买新的。
- 如果 blob 已在 Walrus 上认证但作为*可删除* blob 或没有足够的纪元数，该命令跳过向存储节点发送编码的 blob 数据，只收集可用性证书

```admonish tip title="成本"
我们有一个关于成本考虑的[单独页面](../dev-guide/costs_zh.md)。
```

### 使用 Walrus 上传中继

```admonish note title="最低 CLI 版本"
Walrus 上传中继功能仅在 Walrus CLI 版本 *v1.29* 或更高版本中可用。
```

Walrus 上传中继是第三方服务，可以帮助带宽和网络能力有限的客户端（例如浏览器）在 Walrus 上存储 blobs。

链上资产管理（购买存储、注册和认证 blobs）仍在客户端上进行；上传中继只是获取未编码的 blob，对其进行编码，并将 slivers 发送到存储节点，最后返回证书。详细信息请参见这些文档的 [Walrus 上传中继部分](../operator-guide/upload-relay_zh.md)。

当使用 `walrus store` 命令存储 blobs 时（以及[将 blobs 存储为 quilt](#storing-blobs-as-a-quilt) 时），您可以使用带有 URL 的 `--upload-relay` 标志来指定 CLI 使用的上传中继服务器。

```admonish tip title="小费"
上传中继是第三方服务，可能需要费用或"小费"。此小费可能是每个存储的 blob 的恒定 SUI 金额，或取决于存储的 blob 的大小。Walrus CLI 将显示上传中继需要多少小费，并在继续之前要求确认。

有关小费如何计算和支付的技术详情在[此处](../operator-guide/upload-relay_zh.md)。
```

## 查询 blob 状态

blob 的状态可以通过以下命令之一查询：

```sh
walrus blob-status --blob-id <BLOB_ID>
walrus blob-status --file <FILE>
```

这返回 blob 是否存储及其可用期。如果您使用 `--file` 选项指定文件，CLI 重新编码文件内容并在检查状态之前派生 blob ID。

当 blob 可用时，`blob-status` 命令还返回 `BlobCertified` Sui 事件 ID，该 ID 由交易 ID 和交易发出的事件中的序列号组成。此事件的存在证明了 blob 的可用性。

## 读取 blobs

从 Walrus 读取 blobs 可以通过以下命令实现：

```sh
walrus read <some blob ID>
```

默认情况下，blob 数据写入标准输出。`--out <OUT>` CLI 选项可用于指定输出文件名。`--rpc-url <URL>` 可用于指定要使用的 Sui RPC 节点，而不是钱包配置中设置的或默认的节点。

## 延长 blob 的生命周期

回想一下，当您存储 blob 时，必须指定其[结束纪元](#storing-blobs)。通过指定结束纪元，您确保 blob 将通过 `walrus read`（或其他对 Walrus 的 SDK 访问）可用，直到达到该结束纪元。

Walrus blob 生命周期可以*只要 blobs 未过期*就可以使用 `walrus extend --blob-obj-id <blob object id> ...` 命令延长。常规单地址拥有的 blobs 和[共享 blobs](#shared-blobs) 都可以延长。共享 blobs 可以由任何人延长，但拥有的 blobs 只能由其所有者延长。延长共享 blob 时，您需要提供 `--shared` 标志以通知命令该 blob 是共享的。

请注意，为了延长 blob，需要 blob 的*对象 ID*，不需要 blob ID。有关 blob 延长的更多信息，请参见 `walrus extend --help`。

## 通过可删除 blobs 回收空间

默认情况下，`walrus store` 上传一个永久 blob，在其到期纪元之后可用。即使上传者也不能提前删除它。但是，可选地，store 命令可以使用 `--deletable` 标志调用，以指示 blob 可以在其到期之前由表示 blob 的 Sui blob 对象的所有者删除。可删除 blobs 在认证它们的 Sui 事件中被如此指示，其他人不应依赖其可用性。

可删除的 blob 可以使用以下命令删除：

```sh
walrus delete --blob-id <BLOB_ID>
```

可选地，delete 命令可以通过指定 `--file <PATH>` 选项从文件派生 blob ID 调用，或使用 `--object-id <SUI_ID>` 删除指定 Sui blob 对象中的 blob。

在删除 blob 之前，除非指定 `--yes` 选项，否则 `walrus delete` 命令将要求确认。

`delete` 命令回收与已删除 blob 关联的存储对象，该对象被重新用于存储新 blobs。删除操作提供了管理存储成本和重新使用存储的灵活性。

删除操作对隐私的效用有限：它只从当前纪元存储节点和后续纪元存储节点删除 slivers，如果没有其他用户上传相同 blob 的副本。如果 Walrus 中存在相同 blob 的另一个副本，删除操作不会使 blob 不可下载，`walrus read` 调用将下载它。删除完成后，CLI 检查 blob 的更新状态以查看它在 Walrus 中是否仍然可访问（除非指定 `--no-status-check` 选项）。但是，即使不是，公共 blob 的副本可能被用户缓存或下载，这些副本不会被删除。

```admonish danger title="删除仅回收空间"
**存储在 Walrus 中的所有 blobs 都是公共的，所有人都可发现。** 如果其他用户可能在 Walrus 上存储了 blob 的其他副本，`delete` 命令将不会删除 slivers。它不会从缓存、过去存储节点的 slivers 或用户在 blob 被删除之前可能制作的副本中删除 blobs。
```

## 共享 blobs

*共享 blobs* 是包装"标准" `Blob` 对象的共享 Sui 对象，任何人都可以资助并延长其生命周期。更多详细信息，请参见[共享 blobs 合约](https://github.com/MystenLabs/walrus/tree/main/contracts/walrus/sources/system/shared_blob.move)。

您可以使用 `walrus share` 命令从您拥有的现有 `Blob` 对象创建共享 blob：

```sh
walrus share --blob-obj-id <BLOB_OBJ_ID>
```

生成的共享 blob 可以通过添加 `--amount` 直接资助，或者您可以使用 `walrus fund-shared-blob` 命令资助现有的共享 blob。此外，您可以通过向 `walrus store` 命令添加 `--share` 选项立即共享新创建的 blob。

共享 blobs 只能包含永久 blobs，因此不能在其到期之前删除。

有关 blob 延长的更多信息，请参见[此部分](#extending-the-lifetime-of-a-blob)。

## 使用 quilts 批量存储 blobs

**注意：** *quilt* 功能仅在 Walrus 版本 *v1.29* 或更高版本中可用。

```admonish warning
- quilt 中的 blobs 通过 `QuiltPatchId` 检索，而不是其标准 `BlobId`。此 ID 基于 quilt 中的所有 blobs 生成，因此如果将 blob 移动到不同的 quilt，blob 的 `QuiltPatchId` 将发生变化。
- 标准 blob 操作如 `delete`、`extend` 或 `share` 不能针对 quilt 内的单个 blobs；它们必须应用于整个 quilt。
```

为了有效存储大量小 blobs，Walrus 提供了 Quilt。它将多个 blobs 批处理为单个存储单元，显著减少开销和成本。您可以在 [Quilt](./quilt_zh.md) 中找到该功能的更详细概述。

您可以使用一组专门的 `walrus` 子命令与 quilts 交互。

### 将 Blobs 存储为 Quilt

要将多个文件批处理存储为单个 quilt，请使用 `store-quilt` 命令。您可以通过两种方式指定要包含的文件：

```admonish warning
您必须确保 quilt 中的所有标识符都是唯一的，否则操作将失败。标识符是用于从 quilt 中检索单个 blobs 的唯一名称。
```

#### 使用 `--paths`

递归存储一个或多个目录中的所有文件。每个文件的文件名将用作其在 quilt 中的唯一标识符。支持正则表达式从多个路径上传，与 `walrus store` 命令相同。

与常规 `store` 命令一样，您可以使用 `--epochs`、`--earliest-expiry-time` 或 `--end-epoch` 指定存储持续时间。

```sh
walrus store-quilt --epochs <EPOCHS> --paths <path-to-directory-1> <path-to-directory-2> <path-to-blob>
```

#### 使用 `--blobs`

将 blobs 列表指定为 JSON 对象。这为您提供了更多控制，允许您为每个文件设置自定义 `identifier` 和 `tags`。如果 `identifier` 为 `null` 或省略，则使用文件名。

```sh
walrus store-quilt \
  --blobs '{"path":"<path-to-blob-1>","identifier":"walrus","tags":{"color":"grey","size":"medium"}}' \
          '{"path":"<path-to_blob-2>","identifier":"seal","tags":{"color":"grey","size":"small"}}' \
  --epochs <EPOCHS>
```

### 从 Quilt 读取 Blobs

您可以从 quilt 中检索单个 blobs（正式为"patches"），而无需下载整个 quilt。`read-quilt` 命令允许您通过其标识符、标签或唯一补丁 ID 查询特定 blobs。

要通过其标识符读取 blobs，请使用 `--identifiers` 标志：

```sh
walrus read-quilt --out <download dir> \
  --quilt-id 057MX9PAaUIQLliItM_khR_cp5jPHzJWf-CuJr1z1ik --identifiers walrus.jpg another-walrus.jpg
```

quilt 中的 blobs 可以根据其标签进行访问和过滤。例如，如果您有存储在 quilt 中的动物图像集合，每个都标有物种标签，如"species=cat"，您可以使用以下命令下载标记为猫的**所有**图像：

```sh
# 读取所有带有标签"size: medium"的 blobs
walrus read-quilt --out <download dir> \
  --quilt-id 057MX9PAaUIQLliItM_khR_cp5jPHzJWf-CuJr1z1ik --tag species cat
```

您还可以使用其 QuiltPatchId 读取 blob，可以使用 [`list-patches-in-quilt`](#list-patches-in-a-quilt) 检索。

```sh
walrus read-quilt --out <download dir> \
  --quilt-patch-ids GRSuRSQ_hLYR9nyo7mlBlS7MLQVSSXRrfPVOxF6n6XcBuQG8AQ \
  GRSuRSQ_hLYR9nyo7mlBlS7MLQVSSXRrfPVOxF6n6XcBwgHHAQ
```

### 列出 Quilt 中的补丁

要查看 quilt 中包含的所有补丁以及其标识符和 QuiltPatchIds，请使用 `list-patches-in-quilt` 命令。

```sh
walrus list-patches-in-quilt 057MX9PAaUIQLliItM_khR_cp5jPHzJWf-CuJr1z1ik
```

## Blob 对象和 blob ID 实用程序

命令 `walrus blob-id <FILE>` 可用于派生任何文件的 blob ID。blob ID 是文件的承诺，具有相同 ID 的任何 blob 都将解码为相同的内容。blob ID 是 256 位数字，在某些 Sui 浏览器上表示为十进制大数。命令 `walrus convert-blob-id <BLOB_ID_DECIMAL>` 可用于将其转换为命令行工具和其他 API 使用的 base64 URL 安全编码。

`walrus list-blobs` 命令列出当前账户拥有的所有未过期 Sui blob 对象，包括其 blob ID、对象 ID 和有关过期和可删除状态的元数据。`--include-expired` 选项还列出过期的 blob 对象。

与 blob 对象关联的 Sui 存储成本可以通过销毁 Sui blob 对象来回收。这不会导致 Walrus blob 被删除，但意味着诸如延长其生命周期、删除它或修改属性等操作不再可用。`walrus burn-blobs --object-ids <BLOB_OBJ_IDS>` 命令可用于销毁特定的 blobs 对象 ID 列表。`--all` 标志销毁用户账户下的所有 blobs，`--all-expired` 销毁用户账户下的所有过期 blobs。

## Blob 属性

Walrus 允许将一组键值属性对与 blob 对象关联。虽然键和值可以是任意字符串以适应 dapps 的任何需求，但在通过聚合器提供 blobs 时，特定键会转换为 HTTP 标头。每个聚合器可以通过 `--allowed-headers` CLI 选项决定允许哪些标头；可以通过 `walrus aggregator --help` 查看默认值。

命令

```sh
walrus set-blob-attribute <BLOB_OBJ_ID> --attr "key1" "value1" --attr "key2" "value2"
```

分别将属性 `key1` 和 `key2` 设置为值 `value1` 和 `value2`。命令 `walrus get-blob-attribute <BLOB_OBJ_ID>` 返回与 blob ID 关联的所有属性。最后，

```sh
walrus remove-blob-attribute-fields <BLOB_OBJ_ID> --keys "key1,key2"
```

删除列出的键的属性（用逗号或空格分隔）。blob 对象的所有属性可以通过命令 `walrus remove-blob-attribute <BLOB_OBJ_ID>` 删除。

请注意，属性与 Sui 上的 blob 对象 ID 关联，而不是与 Walrus 上的 blob 本身关联。这意味着通过删除属性回收存储的 gas。还意味着相同的 blob 内容对于相同 blob ID 的不同 blob 对象可能具有不同的属性。

## 更改默认配置

使用 `--config` 选项指定[配置位置](../usage/setup_zh.md#configuration)的自定义路径。

`--wallet <WALLET>` 参数可用于指定非标准 Sui 钱包配置文件。`--gas-budget <GAS_BUDGET>` 参数可用于更改命令允许使用的最大 Sui 金额（以 MIST 为单位）。

## 日志和指标

`walrus` CLI 允许多个级别的日志记录，可以通过环境变量开启：

```sh
RUST_LOG=walrus=trace walrus info
```

默认情况下启用 `info` 级别日志，但 `debug` 和 `trace` 可以提供对命令执行或失败方式的更深入理解。