# 宣布测试网 v2

发布时间：2025-01-16

```admonish warning
此博客文章以原始形式显示，可能包含不再准确的信息。
一些损坏的链接可能已被更新或删除。
```

我们今天重新部署 Walrus 测试网以整合各种改进，包括一些向后不兼容的变更。请确保按照[设置部分](../usage/setup.md)中描述的方式获取最新的二进制文件和配置。

请注意，之前测试网实例上的所有 blob 数据都已被清除。所有 blob 都需要重新上传到新的测试网实例，包括 Walrus Sites。此外，有一个新版本的 WAL 代币，所以您之前的 WAL 代币将不再有效。要使用测试网 v2，您需要获取新的 WAL 代币。

在以下部分中，我们描述了显著的变更和现有 Walrus Sites 所需的操作。

## 时期持续时间

时期持续时间已从一天增加到两天，以强调此持续时间与 Sui 时期不同（在主网上，时期可能会持续多周）。此外，blob 可以存储的最大时期数已从 200 减少到 183（对应一年）。

`walrus store` 命令现在还支持 `--epochs max` 标志，这将为最大时期数存储 blob。请注意，`--epochs` 标志现在是必需的。

## 新功能

除了对合约和存储节点服务的许多改进之外，最新的 Walrus 版本还带来了几个面向用户的改进。

- `walrus store` 命令现在支持一次存储多个文件。与单独存储每个文件相比，这更快、更经济，因为交易可以通过 [PTB](https://docs.sui.io/concepts/transactions/prog-txn-blocks) 进行批处理。值得注意的是，这与许多 shell 提供的 glob 模式兼容，所以您可以运行诸如 `walrus store *.png --epochs 100` 之类的命令来存储当前目录中的所有 PNG 文件。
- `walrus` CLI 现在支持使用 `walrus share`、`walrus store --share` 和 `walrus fund-shared-blob` 命令创建、资助和扩展*共享 blob*。共享 blob 是集体管理和资助的 blob 的一个例子。有关更多详细信息，请参见[共享 blob 部分](../usage/client-cli.md#shared-blobs)。

## 新的 WAL 代币

随着 Walrus 的重新部署，我们也部署了一个全新的 WAL 合约。这意味着您不能在新的测试网实例中使用之前测试网实例的任何 WAL 代币。您需要通过[测试网 WAL 水龙头](../usage/setup.md#testnet-wal-faucet)请求新的 WAL 代币。

## 向后不兼容的变更

完全重新部署的一个原因是允许我们进行一些向后不兼容的变更。其中许多与合约相关，因此对用户不太可见。但是，有一些变更可能会影响您。

### 配置文件

存储节点和客户端的配置文件格式已更改。请确保使用最新版本的配置文件，请参见[配置部分](../usage/setup.md#configuration)。

### CLI 选项

`walrus` CLI 的几个 CLI 选项已更改。值得注意的是，所有选项的"简短"变体（例如，`-e` 而不是 `--epochs`）都已被删除，以防止与新选项的未来混淆。此外，`--epochs` 标志现在是 `walrus store` 命令的必需项（这也会影响 [JSON API](../usage/json-api.md)）。

请参考 CLI 帮助（`walrus --help` 或 `walrus <command> --help`）了解更多详细信息。

### HTTP API

存储节点以及聚合器和发布器的 HTTP API 的路径、请求和响应格式已更改。请参考 [HTTP API](../usage/web-api.md) 部分了解更多详细信息。

## 对现有 Walrus Sites 的影响和所需操作

Walrus Sites 合约没有更改，这意味着 Sui 上的所有相应对象仍然有效。但是，资源现在指向新测试网上尚不存在的 blob ID。修复现有站点的最简单方法是使用 `--force` 标志简单地更新它们：

```sh
site-builder update --epochs <时期数> --force <站点路径> <现有站点对象>
```

## 新的 Move 合约和文档

作为 Walrus 新测试网版本的一部分，Move 智能合约已更新；部署的版本可以在 [`walrus-docs` 仓库](https://github.com/MystenLabs/walrus-docs/tree/main/contracts)中找到。