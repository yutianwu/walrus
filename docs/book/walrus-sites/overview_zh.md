# 技术概述

在以下部分中，我们深入探讨 Walrus Sites 的技术规范。

## 高级图景

Walrus Sites 由 Sui 和 Walrus 启用。Walrus Site 的资源（`html`、`css`、`js`、图像等）存储在 Walrus 上，而它的主要入口点是存储在 Sui 上的对象，该对象包含站点的元数据并指向 Walrus blob ID。

### Sui 上的 Walrus Sites 对象

Walrus `Site` 在 Sui 上表示为一个非常简单的对象：

``` move
struct Site has key, store {
    id: UID,
    name: String,
}
```

与此站点关联的资源然后作为 `Resource` 类型的[动态字段](https://docs.sui.io/concepts/versioning#dynamic-fields)添加到此对象：

``` move
struct Resource has store, drop {
    path: String,
    // 包含此资源字节的 walrus blob id
    blob_id: u256,
    ⋮
}
```

重要的是，每个资源包含：

- 资源的 `path`，例如 `/index.html`（所有路径始终表示为从根 `/` 开始）；
- `blob_id`，这是可以找到资源的 Walrus blob ID；以及
- 稍后将在文档中解释的其他字段。

这些 `Resource` 动态字段使用 `ResourcePath` 类型的结构作为键

``` move
struct ResourcePath has copy, store, drop {
    path: String,
}
```

此结构只是保存路径的字符串（`/index.html`）；拥有单独的类型确保我们不会与其他动态字段发生命名空间冲突，这些字段可能由其他包添加。

要查看实际情况，请查看[浏览器中的 Walrus Site](https://suiscan.xyz/testnet/object/0xd20b90149409ba5d005d4a2cd981db9494bc3cdb2f04c47ca1af98dd8f71610a)，并检查其动态字段。

### 站点渲染路径

给定 Walrus Site 的 Sui 对象 ID，很容易通过查看动态字段来查找组成它的资源，然后使用 `Resource` 结构中包含的 blob ID 从 Walrus 获取这些资源。

因此，唯一未决的问题是如何在客户端上执行这些查找。可能有几种方法：

- 拥有接受 Sui 对象 ID 和可能路径请求的服务器，代表客户端执行此解析，将资源作为标准 HTML 响应提供回来。
- 使用具有 Web 浏览器和 Walrus Sites 工作方式知识的客户端上的自定义应用程序，可以在本地执行此解析。
- 基于服务工作者的混合方法，其中能够执行解析的服务工作者从门户安装在浏览器中。

所有这些方法都是可行的（例如，第一种已用于 IPFS 网关中的类似应用程序），并且有权衡。

目前，我们提供服务器端和基于服务工作者的方法。

### 浏览和域隔离

我们必须确保当通过门户浏览多个站点时，例如托管在 <https://wal.app> 的门户，这些站点是隔离的。隔离对于安全性是必要的，并确保浏览器中的钱包连接按预期工作。

为此，我们为每个 Walrus Site 提供门户域的特定*子域*。例如，Flatland mint dApp 托管在 <https://flatland.wal.app>，其中子域 `flatland` 通过 SuiNS 唯一关联到 Walrus Site 的对象 ID。

Walrus Sites 也在*没有* SuiNS 的情况下工作：当通过自托管门户或通过支持 Base36 域解析的门户访问站点时，可以通过使用站点的 Sui 对象 ID 的 Base36 编码作为子域来浏览站点。

``` admonish danger
`wal.app` 门户不支持 Base36 域解析；因此，<https://58gr4pinoayuijgdixud23441t55jd94ugep68fsm72b8mwmq2.wal.app> 不是有效的 URL。如果您想访问这样的站点，您应该使用另一个门户或[托管自己的门户](./portal_zh.md#running-a-local-portal)。
```

选择 Base36 有两个原因，这是由子域标准强制的：

1. 子域最多可以有 63 个字符，而十六进制编码的 Sui 对象 ID 需要 64 个。
1. 子域是大小写*不敏感*的，排除了其他流行的编码，例如 Base64 或 Base58。

## Walrus Site 门户

### 门户类型

如前所述，我们提供两种类型的门户来浏览 Walrus Sites：

- 服务器端门户，它是解析 Walrus Site 并将其返回给浏览器的服务器。服务器端门户托管在 <https://wal.app>。此门户仅支持与 SuiNS 名称关联的站点。
- 服务工作者门户，它是安装在浏览器中并解析 Walrus Site 的服务工作者。服务工作者门户不再托管在特定域上，但代码仍然维护，以便任何人都可以使用。

我们现在更详细地展示如何使用通用门户在客户端浏览器中渲染 Walrus Site。根据不同类型的门户，一些细节可能有所不同，但总体图景保持不变。

### Walrus Site 的端到端解析

以下步骤都引用以下图：

![Walrus Site 解析](../assets/walrus-sites-portal-diagram.svg)

- **站点发布**（步骤 0）：站点开发者使用 [`site-builder`](#the-site-builder) 或使用发布者发布 Walrus Site。假设开发者使用 SuiNS 名称 `dapp.sui` 指向创建的 Walrus Site 的对象 ID。
- **浏览开始**（步骤 1）：客户端在浏览器中浏览 `dapp.wal.app/index.html`。
- **站点解析**（步骤 2-6）：门户解释其*origin* `dapp.wal.app`，并为 `dapp.sui` 进行 SuiNS 解析，获得相关的对象 ID。使用对象 ID，它然后获取对象的动态字段（还检查[重定向](./portal_zh.md)）。从动态字段中，它选择 `/index.html` 的字段，并提取其 Walrus blob ID 和头部（参见[头部的高级部分](./routing_zh.md)）。
- **Blob 获取**（步骤 7-11）：给定 blob ID，门户查询 Walrus 聚合器以获取 blob 数据。
- **返回响应**（步骤 12-13）：现在门户有了 `/index.html` 的字节及其头部，它可以制作一个响应，然后由浏览器渲染。

这些步骤对浏览器之后可能查询的所有资源执行（例如，如果 `/index.html` 指向 `assets/cat.png`）。

## 站点构建器

为了促进 Walrus Sites 的创建，我们提供"站点构建器"工具。站点构建器负责使用正确的结构在 Sui 上创建 Walrus Sites 对象，并将站点资源存储到 Walrus。有关设置和使用说明，请参阅[教程](./tutorial_zh.md)。