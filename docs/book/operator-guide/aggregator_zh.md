# 运营聚合器或发布器
<!-- TODO (WAL-118): Add further details and example cache setup. -->

本页描述如何运行暴露 [HTTP API](../usage/web-api_zh.md) 的 Walrus 聚合器或发布器。

## 本地启动守护进程 {#local-daemon}

您可以通过 `walrus` 二进制文件运行本地 Walrus 守护进程。有三个不同的命令：

- `walrus aggregator` 启动一个"聚合器"，提供从 Walrus 读取 blob 的 HTTP 接口。
- `walrus publisher` 启动一个"发布器"，提供在 Walrus 中存储 blob 的 HTTP 接口。
- `walrus daemon` 在同一地址和端口上提供聚合器和发布器的组合功能。

聚合器不执行任何链上操作，只需要指定监听地址：

```sh
walrus aggregator --bind-address "127.0.0.1:31415"
```

发布器和守护进程执行链上操作，因此需要一个具有足够 SUI 和 WAL 余额的 Sui 测试网钱包。为了处理许多并行请求而不发生对象冲突，它们从 1.4.0 版本开始创建内部子钱包，这些子钱包由主钱包提供资金。这些子钱包保存在使用 `--sub-wallets-dir` 参数指定的目录中；可以使用任何现有目录。如果该目录已包含子钱包，它们将被重复使用。

默认情况下，会创建并资助 8 个子钱包。这可以通过 `--n-clients` 参数更改。对于简单的本地测试，通常 1 或 2 个子钱包就足够了。

例如，您可以使用以下命令运行一个带有单个子钱包的发布器，该子钱包存储在 Walrus 配置目录中：

```sh
PUBLISHER_WALLETS_DIR=~/.config/walrus/publisher-wallets
mkdir -p "$PUBLISHER_WALLETS_DIR"
walrus publisher \
  --bind-address "127.0.0.1:31416" \
  --sub-wallets-dir "$PUBLISHER_WALLETS_DIR" \
  --n-clients 1
```

将 `publisher` 替换为 `daemon` 以在同一地址和端口上运行聚合器和发布器。

```admonish warning
虽然聚合器不执行 Sui 链上操作，因此不消耗 gas，但发布器确实在链上执行操作，会消耗 SUI 和 WAL 代币。因此，重要的是确保只有授权方可以访问它，或采取其他措施来管理 gas 成本，特别是在未来的主网部署中。
```

默认情况下，[存储 blob](../usage/web-api_zh.md#store) 请求限制为 10 MiB；您可以通过 `--max-body-size` 选项增加此限制。[存储拼接](../usage/web-api_zh.md#storing-quilts) 请求默认限制为 100 MiB，可以使用 `--max-quilt-body-size` 选项增加。

### 守护进程指标

服务默认导出可通过 `curl http://127.0.0.1:27182/metrics` 访问的指标端点。可以使用 `--metrics-address <METRICS_ADDRESS>` CLI 选项进行更改。

### 示例 systemd 配置

以下是一个聚合器节点的示例，它托管一个 HTTP 端点，可用于通过 Web 从 Walrus 获取数据。

聚合器进程通过如上所述的 `walrus` 客户端二进制文件运行。它可以通过多种方式运行，一个例子是通过 systemd 服务：

```ini
[Unit]
Description=Walrus Aggregator

[Service]
User=walrus
Environment=RUST_BACKTRACE=1
Environment=RUST_LOG=info
ExecStart=/opt/walrus/bin/walrus --config /opt/walrus/config/client_config.yaml aggregator --bind-address 0.0.0.0:9000
Restart=always

LimitNOFILE=65536
```

## 发布器操作和配置

我们在这里列出一些关于发布器如何处理 Sui 上的资金和对象的重要细节。

### 子钱包数量和上传并发性

如上所述，发布器使用子钱包来允许并行存储 blob。默认情况下，发布器使用 8 个子钱包，这意味着它可以同时处理 8 个 blob 存储 HTTP 请求。

为了运营高性能和高并发的发布器，以下选项可能有用。

- `--n-clients <NUM>` 选项创建多个独立的钱包，用于执行并发的 Sui 链操作。增加此数量以允许更多并行上传。请注意，更高的数量最初也需要更多的 SUI 和 WAL 代币，以分配给更多钱包。

- `--max-concurrent-requests <NUM>` 确定可以处理多少并发请求，包括 Sui 操作（受客户端数量限制）以及上传。超过此数量后，更多请求会排队等待，直到达到 `--max-buffer-size <NUM>`，之后请求会被拒绝并返回 HTTP 429 代码。

### 子钱包中的 SUI 代币管理

每个子钱包都需要资金来与链交互和购买存储。因此，后台进程会定期检查子钱包是否有足够的资金。在稳定状态下，每个子钱包将有 0.5-1.0 SUI 和 WAL 的余额。代币补充的数量和触发器可以通过 CLI 参数配置。

要调整如何处理补充，您可以使用 `--refill-interval <REFILL_INTERVAL>`、`--gas-refill-amount <GAS_REFILL_AMOUNT>`、`--wal-refill-amount <WAL_REFILL_AMOUNT>` 和 `--sub-wallets-min-balance <SUB_WALLETS_MIN_BALANCE>` 参数。

### 创建的链上 `Blob` 对象的生命周期

Walrus 中的每个存储操作都会在 Sui 上创建一个 `Blob` 对象。此 blob 对象表示对关联数据的（部分）所有权，并允许某些数据管理操作（例如，在可删除 blob 的情况下）。

当发布器代表客户端存储 blob 时，`Blob` 对象最初由存储 blob 的子钱包拥有。然后，根据配置，可能出现以下情况：

- 如果客户端请求存储 blob 并指定 `send_object_to` 查询参数（参见[相关部分](../usage/web-api_zh.md#store)的示例），那么 `Blob` 对象会被转移到指定的地址。这是客户端取回其数据创建的对象的一种方式。
- 如果未指定 `send_object_to` 查询参数，则可能出现两种情况：
  - 默认情况下，子钱包将新创建的 blob 对象转移到主钱包，以便所有这些对象都保存在那里。可以通过设置 `--burn-after-store` 标志来更改此行为，然后 blob 对象会立即被删除。
  - 但是，请注意此标志*不会影响* `send_object_to` 查询参数的使用：无论此标志的状态如何，如果在 PUT 请求中指定了 `send_object_to` 查询参数，发布器都会将创建的对象发送到该参数中的地址。

### 高级发布器用途

"认证发布器"的设置和使用在[单独的部分](./auth-publisher_zh.md)中介绍。