# Walrus 上传中继

## 概述

Walrus 的一个目标是使 dApp 能够在其最终用户的浏览器中 `存储` 到 Walrus，这些浏览器具有中低等的机器规格（移动设备、低功耗笔记本电脑等）。由于需要大量网络连接来将所有切片上传到所有分片，在实践中，通过直接与存储节点通信的浏览器内进程实现这种基于浏览器的场景是困难的。

上传中继是一个可下载的程序，社区成员、Mysten Labs 和/或 dApp 编写者本身可以在面向互联网的主机上运行，以帮助代表最终用户将 blob 切片存储到存储节点，从而减轻浏览器资源消耗并启用基于 Web 的 `存储` 操作。

```admonish tip title="公共上传中继"
Mysten Labs 运行两个公开可用的上传中继，一个用于 `testnet`，一个用于 `mainnet`。
您可以在以下地址找到它们：

- `testnet`：`https://upload-relay.testnet.walrus.space`
- `mainnet`：`https://upload-relay.mainnet.walrus.space`
```

## 设计

### 概述

在高层次上，客户端使用上传中继存储 blob 的过程如下：

1. 首先，客户端在本地编码 blob 并在 Sui 上注册它；
1. 然后，客户端通过 HTTP POST 请求将 blob 发送到上传中继的 blob-relay 端点（`/v1/blob-upload-relay`）；
1. 上传中继然后编码 blob，将切片发送到存储节点，收集存储确认证书，并将其发送回客户端；
1. 最后，客户端使用确认证书在 Sui 上认证 blob。

因此，上传中继*不会*执行任何链上操作，只是帮助客户端将其 blob 的切片分发到存储节点。

这种客户端和上传中继之间的流程已在 Walrus CLI、Rust SDK 和 TS SDK 中实现，因此开发人员不需要自己实现它。为了完整性，以下我们讨论该服务是如何实现的、如何付费的以及客户端如何使用它。

### 上传中继操作

上传中继可以通过两种方式操作：

- **免费服务**：它接受来自客户端的带有 blob 的简单 HTTP POST 请求，并免费将它们中继到存储节点。
- **付费服务**（对于公共中继）：在此配置中，上传中继需要*小费*来中继 blob。小费可由中继操作员用于覆盖运行基础设施的成本并在服务上获得收益。目前，我们允许恒定小费和与未编码数据大小线性相关的小费。

上传中继暴露一个小费配置端点（`/v1/tip-config`），返回小费配置。例如：

```json
{
  "send_tip": {
    "address": "0x1234...",
    "kind": {
      "const": 105
    }
  }
}
```

上述配置指定每个存储操作需要 `105 MIST`（任意值）的小费，支付给设定的地址（`0x1234..`）。请注意，即使对于免费上传中继，也会提供此配置（返回 `"no_tip"`）。

### 支付小费

此步骤只有在中继需要小费时才必要。

要支付小费，客户端按以下步骤进行：

1. 它计算 `blob_digest = SHA256(blob)`
1. 它生成一个随机 `nonce`，并对其进行哈希 `nonce_digest = SHA256(nonce)`
1. 它计算 `unencoded_length = blob.len()`

然后，它创建一个 PTB，其中第一个输入（`input 0`）是 `blob_digest || nonce_digest || unencoded_length` 的 `bcs` 编码表示。这将稍后被中继用于认证存储请求的发送者。在同一个 PTB 中，客户端将适当的小费金额转移到中继的钱包（也通过 `/v1/tip-config` 端点找到）。通常，客户端也会在此交易中注册 blob。这不是强制性的，blob 可以在另一个交易中注册，但通常将所有这些操作一起执行是方便且便宜的。

交易执行后，客户端保留交易 ID `tx_id`、`nonce` 和 `blob_id`，这些都是下一阶段所需的。

注意：中继将对支付小费的交易强制执行新鲜度检查（目前默认为 1 小时，但每个中继可以独立配置此设置）。

### 向上传中继发送数据

请参阅上传中继的完整 OpenAPI 规范以获取完整详细信息（[yaml](https://github.com/mystenlabs/walrus/tree/main/crates/walrus-upload-relay/upload_relay_openapi.yaml)、[html](https://github.com/mystenlabs/walrus/tree/main/crates/walrus-upload-relay/upload_relay_openapi.html)）

本质上，客户端向中继上的 `/v1/blob-upload-relay` API 端点发送 POST 请求，请求体中包含要存储的 blob 的字节。

这些额外参数必须在 URL 的查询字符串中指定：

- **blob_id**（必需）：要存储的 blob 的 blob ID。示例：
  `blob_id=E7_nNXvFU_3qZVu3OH1yycRG7LZlyn1-UxEDCDDqGGU`
- **tx_id**（如果中继需要小费则必需）：将小费转移到中继的交易的交易 ID（Base58 编码）。示例：
  `tx_id=EmcFpdozbobqH61w76T4UehhC4UGaAv32uZpv6c4CNyg`
- **nonce**（如果中继需要小费则必需）：`nonce`，添加到上面创建的交易输入的哈希的原像，作为不带填充的 Base64 URL 编码字符串。
  `nonce=rw8xIuqxwMpdOcF_3jOprsD9TtPWfXK97tT_lWr1teQ`
- **deletable_blob_object**（如果 blob 注册为可删除则必需）：如果要存储的 blob 是可删除的，那么客户端必须指定 Blob 的对象 ID（作为十六进制字符串）。如果 blob 要作为永久 blob 存储，则不应指定此参数。示例：
  `deletable_blob_object=0x56ae1c86e17db174ea002f8340e28880bc8a8587c56e8604a4fa6b1170b23a60`
- **encoding_type**（可选）：用于 blob 的编码类型。默认值，也是目前唯一的值，是 `RS2`。

## 接收证书

一旦中继完成 blob 存储，它将从存储节点收集确认证书。

然后，它将向客户端发送响应，包含已存储 blob 的 `blob_id` 以及 `confirmation_certificate`。客户端然后可以使用此证书在链上认证 blob。

## 安装

### 下载二进制文件

如果您想下载预构建的二进制文件以手动运行 `walrus-upload-relay`，您需要从[发布页面](https://github.com/MystenLabs/walrus/releases)下载它。请注意，`walrus-upload-relay` 不会自己守护化，需要某些监督进程来确保启动时启动、在失败时重启等。

### Docker

`walrus-upload-relay` 的 docker 镜像在 Docker Hub 上可用，名为 `mysten/walrus-upload-relay`。

```sh
docker run -it --rm mysten/walrus-upload-relay --help
```

### 从源代码构建

当然，如果您想从源代码构建，这也总是一个选择。`walrus-upload-relay` 的源代码在 [GitHub](https://github.com/MystenLabs/walrus) 上可用，位于 `crates/walrus-upload-relay` 子目录中。从 Walrus 仓库的根目录运行以下命令应该会为您提供一个工作的二进制文件。

```sh
cargo build --release --bin walrus-upload-relay
./target/release/walrus-upload-relay --help
```

## 配置

请注意，`walrus-upload-relay` 需要一些配置才能启动。以下是一个示例，说明您如何放置配置，使其在调用 Docker 运行中继时可达。为了下面示例的目的，我们假设：

- `$HOME/.config/walrus/walrus_upload_relay_config.yaml` 存在于主机上，并包含 `walrus-upload-relay` 的配置，如[配置部分](#relay-specific-configuration)中所述。
- `$HOME/.config/walrus/client_config.yaml` 存在于主机上，并包含 Walrus 客户端配置，如 [CLI 设置部分](../usage/setup_zh.html#configuration)中指定的。
- 端口 3000 可用于中继绑定（将此更改为您想要从主机暴露的任何端口）。

```sh
docker run \
  -p 3000:3000 \
  -v $HOME/.config/walrus/walrus_upload_relay_config.yaml:/opt/walrus/walrus_upload_relay_config.yaml \
  -v $HOME/.config/walrus/client_config.yaml:/opt/walrus/client_config.yaml \
  mysten/walrus-upload-relay \
    --context testnet \
    --walrus-config /opt/walrus/client_config.yaml \
    --server-address 0.0.0.0:3000 \
    --relay-config /opt/walrus/walrus_upload_relay_config.yaml
```

### 中继特定配置

以下是 `walrus-upload-relay` 配置文件的示例：

```yaml
{{ #include ../setup/walrus_upload_relay_config_example.yaml }}
```

目前，选项如下：

- `tip_config`：要支付的小费配置。它可以是 `!no_tip`，用于免费服务，或 `!send_tip`，用于配置请求的小费。在这种情况下：
  - `address`：包含上传中继所有者的十六进制编码地址，小费应发送到该地址。
  - `kind`：应支付的小费类型。它可以是 `!const`，每次存储的恒定小费，或 `!linear`，与未编码 blob 大小的线性小费。
- `tx_freshness_threshold_secs`：支付小费的交易被认为有效的最长时间（秒）。
- `tx_max_future_threshold_secs`：支付小费的交易在未来被认为有效的最长时间。这只是为了考虑一些时钟偏差。