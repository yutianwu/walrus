# 客户端守护进程模式和 HTTP API

除了 CLI 和 JSON 模式，Walrus 客户端还提供*守护进程模式*。在此模式下，它运行一个简单的 Web 服务器，分别以*聚合器*和*发布者*角色提供 HTTP 接口来存储和读取 blob。我们还提供[公共聚合器和发布者服务](#public-services)以试用 Walrus HTTP API，无需运行本地客户端。请参阅操作员文档中如何[操作聚合器或发布者](../operator-guide/aggregator_zh.md)。

## HTTP API 使用

对于以下示例，我们假设您分别将 `AGGREGATOR` 和 `PUBLISHER` 环境变量设置为您所需的聚合器和发布者。例如，Mysten Labs 在 Walrus 测试网上运行的实例（有关更多公共选项，请参见[下面](#public-services)）：

```sh
AGGREGATOR=https://aggregator.walrus-testnet.walrus.space
PUBLISHER=https://publisher.walrus-testnet.walrus.space
```

```admonish tip title="API 规范"
Walrus 聚合器和发布者在路径 `/v1/api` 处公开其 API 规范。您可以在浏览器中查看，例如，在 <https://aggregator.walrus-testnet.walrus.space/v1/api>。
```

### 存储

您可以通过简单的 HTTP PUT 请求与守护进程交互。例如，使用 [cURL](https://curl.se)，您可以使用发布者或守护进程存储 blob，如下所示：

```sh
# 存储字符串 `some string` 1 个存储 epoch
curl -X PUT "$PUBLISHER/v1/blobs" -d "some string"
# 存储文件 `some/file` 5 个存储 epoch
curl -X PUT "$PUBLISHER/v1/blobs?epochs=5" --upload-file "some/file"
# 存储文件 `some/file` 并将 blob 对象发送到 $ADDRESS
curl -X PUT "$PUBLISHER/v1/blobs?send_object_to=$ADDRESS" --upload-file "some/file"
# 将文件 `some/file` 存储为可删除的 blob，而不是永久的，并将 blob 对象发送到 $ADDRESS
curl -X PUT "$PUBLISHER/v1/blobs?deletable=true&send_object_to=$ADDRESS" --upload-file "some/file"
```

存储 HTTP API 端点以 JSON 格式返回有关存储的 blob 的信息。当首次存储 blob 时，`newlyCreated` 字段包含有关新 blob 的信息：

```sh
$ curl -X PUT "$PUBLISHER/v1/blobs" -d "some other string"
{
  "newlyCreated": {
    "blobObject": {
      "id": "0xe91eee8c5b6f35b9a250cfc29e30f0d9e5463a21fd8d1ddb0fc22d44db4eac50",
      "registeredEpoch": 34,
      "blobId": "M4hsZGQ1oCktdzegB6HnI6Mi28S2nqOPHxK-W7_4BUk",
      "size": 17,
      "encodingType": "RS2",
      "certifiedEpoch": 34,
      "storage": {
        "id": "0x4748cd83217b5ce7aa77e7f1ad6fc5f7f694e26a157381b9391ac65c47815faf",
        "startEpoch": 34,
        "endEpoch": 35,
        "storageSize": 66034000
      },
      "deletable": false
    },
    "resourceOperation": {
      "registerFromScratch": {
        "encodedLength": 66034000,
        "epochsAhead": 1
      }
    },
    "cost": 132300
  }
}
```

返回的信息是 [Sui blob 对象](../dev-guide/sui-struct_zh.md)的内容。

当发布者找到具有相同 blob ID 和足够有效期的认证 blob 时，它返回 `alreadyCertified` JSON 结构：

```sh
$ curl -X PUT "$PUBLISHER/v1/blobs" -d "some other string"
{
  "alreadyCertified": {
    "blobId": "M4hsZGQ1oCktdzegB6HnI6Mi28S2nqOPHxK-W7_4BUk",
    "event": {
      "txDigest": "4XQHFa9S324wTzYHF3vsBSwpUZuLpmwTHYMFv9nsttSs",
      "eventSeq": "0"
    },
    "endEpoch": 35
  }
}
```

`event` 字段返回可用于在 Sui 浏览器上或使用 Sui SDK 查找创建 Sui Blob 对象的交易的 [Sui 事件 ID](../dev-guide/sui-struct_zh.md)。

### 读取

可以使用其 blob ID 通过 HTTP GET 从聚合器或守护进程读取 blob。例如，以下 cURL 命令读取 blob 并将其写入输出文件：

```sh
curl "$AGGREGATOR/v1/blobs/<some blob ID>" -o <some file name>
```

或者，您可以使用 cURL 命令在终端中打印 blob 的内容：

```sh
curl "$AGGREGATOR/v1/blobs/<some blob ID>"
```

```admonish tip title="内容嗅探"
现代浏览器将尝试嗅探此类资源的内容类型，通常能很好地推断媒体的内容类型。但聚合器故意防止此类嗅探推断出危险的可执行类型，如 JavaScript 或样式表类型。
```

也可以通过使用 Sui blob 对象或共享 blob 的对象 ID 来读取 blob。例如，以下 cURL 命令下载与具有特定对象 ID 的 Sui blob 对应的 blob：

```sh
curl "$AGGREGATOR/v1/blobs/by-object-id/<object-id>" -o <some file name>
```

通过对象 ID 下载 blob 允许使用属性来设置一些 HTTP 头。聚合器识别属性键 `content-disposition`、`content-encoding`、`content-language`、`content-location`、`content-type` 和 `link`，当存在时，在相应的 HTTP 头中返回值。

### Quilt HTTP API

Walrus 支持将多个 blob 作为称为 [quilt](./quilt_zh.md) 的单个单元存储和检索。发布者和聚合器支持 *quilt* 相关操作。

**注意** Quilt API 在 Walrus *1.29* 及更高版本中受支持。

#### 存储 quilt

```admonish tip
存储常规 blob 时可用的所有查询参数（请参见上面的[存储](#store)部分）也可以在存储 quilt 时使用。
```

使用以下发布者 API 将多个 blob 存储为 quilt：

```sh
# 存储两个文件 `document.pdf` 和 `image.png`，分别使用自定义标识符 `contract-v2` 和
# `logo-2024`。
curl -X PUT "$PUBLISHER/v1/quilts?epochs=5" \
  -F "contract-v2=@document.pdf" \
  -F "logo-2024=@image.png"

# 使用 Walrus 原生元数据存储两个文件。元数据分配给具有
# 相同标识符的 blob。注意：`_metadata` 必须用作 Walrus 原生元数据的字段名。
curl -X PUT "$PUBLISHER/v1/quilts?epochs=5" \
  -F "quilt-manual=@document.pdf" \
  -F "logo-2025=@image.png" \
  -F '_metadata=[
    {"identifier": "quilt-manual", "tags": {"creator": "walrus", "version": "1.0"}},
    {"identifier": "logo-2025", "tags": {"type": "logo", "format": "png"}}
  ]'
```

```admonish warning
标识符在 quilt 中必须是唯一的。
```

```admonish info
由于标识符不能以 `_` 开头，字段名 `_metadata` 为 Walrus 原生元数据保留，不会与用户定义的标识符冲突。有关完整的标识符限制，请参阅 [Quilt 文档](./quilt_zh.md)。
```

quilt 存储 API 返回 JSON 响应，其中包含有关存储的 quilt 的信息，包括 quilt ID（blobId）和可用于稍后检索特定 blob 的单个 blob 补丁 ID。以下示例显示了命令和响应（实际的 JSON 输出作为单行返回；这里为了可读性进行了格式化）：

```console
$ curl -X PUT "http://127.0.0.1:31415/v1/quilts?epochs=1" \
  -F "walrus.jpg=@./walrus-33.jpg" \
  -F "another_walrus.jpg=@./walrus-46.jpg"
{
  "blobStoreResult": {
    "newlyCreated": {
      "blobObject": {
        "id": "0xe6ac1e1ac08a603aef73a34328b0b623ffba6be6586e159a1d79c5ef0357bc02",
        "registeredEpoch": 103,
        "blobId": "6XUOE-Q5-nAXHRifN6n9nomVDtHZQbGuAkW3PjlBuKo",
        "size": 1782224,
        "encodingType": "RS2",
        "certifiedEpoch": null,
        "storage": {
          "id": "0xbc8ff9b4071927689d59468f887f94a4a503d9c6c5ef4c4d97fcb475a257758f",
          "startEpoch": 103,
          "endEpoch": 104,
          "storageSize": 72040000
        },
        "deletable": false
      },
      "resourceOperation": {
        "registerFromScratch": {
          "encodedLength": 72040000,
          "epochsAhead": 1
        }
      },
      "cost": 12075000
    }
  },
  "storedQuiltBlobs": [
    {
      "identifier": "another_walrus.jpg",
      "quiltPatchId": "6XUOE-Q5-nAXHRifN6n9nomVDtHZQbGuAkW3PjlBuKoBAQDQAA"
    },
    {
      "identifier": "walrus.jpg",
      "quiltPatchId": "6XUOE-Q5-nAXHRifN6n9nomVDtHZQbGuAkW3PjlBuKoB0AB7Ag"
    }
  ]
}
```

#### 读取 quilt

有两种方式通过聚合器 API 从 quilt 检索 blob：

```admonish info
目前，每个请求只能检索一个 blob。尚不支持在单个请求中从 quilt 批量检索多个 blob。
```

##### 通过 quilt 补丁 ID

quilt 中的每个 blob 都有唯一的补丁 ID。您可以使用其补丁 ID 检索特定的 blob：

```sh
# 使用其 quilt 补丁 ID 检索 blob。
curl "$AGGREGATOR/v1/blobs/by-quilt-patch-id/6XUOE-Q5-nAXHRifN6n9nomVDtHZQbGuAkW3PjlBuKoBAQDQAA" \
```

```admonish tip
QuiltPatchId 可以从存储 quilt 输出（如上所示）或使用 [`list-patches-in-quilt`](./client-cli_zh.md#batch-storing-blobs-with-quilt) CLI 命令获得。
```

##### 通过 quilt ID 和标识符

您还可以使用 quilt ID 和 blob 的标识符检索 blob：

```sh
# 从 quilt 检索标识符为 `walrus.jpg` 的 blob。
curl "$AGGREGATOR/v1/blobs/by-quilt-id/6XUOE-Q5-nAXHRifN6n9nomVDtHZQbGuAkW3PjlBuKo/walrus.jpg" \
```

两种方法都在响应体中返回原始 blob 字节。元数据（如 blob 标识符和标签）作为 HTTP 头返回：

- `X-Quilt-Patch-Identifier`：quilt 中 blob 的标识符
- `ETag`：用于缓存目的的补丁 ID 或 quilt ID
- 来自 blob 标签的其他自定义头（如果配置）

## 使用公共聚合器或发布者 {#public-services}

对于某些用例（例如，公共网站），或者只是试用 HTTP API，需要公共可访问的聚合器和/或发布者。在 Walrus 测试网上，许多实体运行公共聚合器和发布者。在主网上，没有未经身份验证的公共发布者，因为它们消耗 SUI 和 WAL。

请参阅以下子节了解[主网上的公共聚合器](#mainnet)和测试网上的公共[聚合器](#aggregators-testnet)和[发布者](#publishers-testnet)。我们还以 [JSON 格式提供操作员列表](../assets/operators.json)。

<!-- markdownlint-disable proper-names -->
### 主网

以下是 Walrus 主网上已知公共聚合器的列表；它们定期检查，但每个都可能仍然暂时不可用：

{{ #mainnet.aggregators }}
{{ /mainnet.aggregators }}

### 测试网

#### 聚合器 {#aggregators-testnet}

以下是 Walrus 测试网上已知公共聚合器的列表；它们定期检查，但每个都可能仍然暂时不可用：

{{ #testnet.aggregators }}
{{ /testnet.aggregators }}

#### 发布者 {#publishers-testnet}

以下是 Walrus 测试网上已知公共发布者的列表；它们定期检查，但每个都可能仍然暂时不可用：

{{ #testnet.publishers }}
{{ /testnet.publishers }}
<!-- markdownlint-enable proper-names -->

其中大多数默认将请求限制为 10 MiB。如果您想上传更大的文件，您需要[运行自己的发布者](../operator-guide/aggregator_zh.md#local-daemon)或使用 [CLI](./client-cli_zh.md)。