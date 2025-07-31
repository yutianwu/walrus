# 指定头部、路由和元数据

在其基本配置中，Walrus Sites 通过门户提供静态资产。但是，许多现代 Web 应用程序需要更高级的功能，如自定义头部和客户端路由。

因此，site-builder 可以读取 `ws-resource.json` 配置文件，您可以在其中直接指定资源头部和路由规则。

## `ws-resources.json` 文件

此文件可选择放置在站点目录的根目录中，并且*不会*与站点资源一起上传（换句话说，该文件不是生成的 Walrus Site 的一部分，也不会由门户提供服务）。

如果您不想使用此默认位置，可以在运行 `deploy`、`publish` 或 `update` 命令时使用 `--ws-resources` 标志指定配置文件的路径。

该文件采用 JSON 格式，如下所示：

``` JSON
{
  "headers": {
    "/index.html": {
      "Content-Type": "text/html; charset=utf-8",
      "Cache-Control": "max-age=3500"
    }
  },
  "routes": {
    "/*": "/index.html",
    "/accounts/*": "/accounts.html",
    "/path/assets/*": "/assets/asset_router.html"
  },
  "metadata": {
      "link": "https://subdomain.wal.app",
      "image_url": "https://www.walrus.xyz/walrus-site",
      "description": "This is a walrus site.",
      "project_url": "https://github.com/MystenLabs/walrus-sites/",
      "creator": "MystenLabs"
  },
  "site_name": "My Walrus Site",
  "object_id": "0xe674c144119a37a0ed9cef26a962c3fdfbdbfd86a3b3db562ee81d5542a4eccf",
  "ignore": ["/private/*", "/secret.txt", "/images/tmp/*"]
}
```

```admonish note
`ws-resources.json` 文件期望字段名采用 `snake_case` 格式
```

我们现在详细描述配置文件的六个部分：`headers`、`routes`、`metadata`、`site_name`、`object_id` 和 `ignore` 部分。

## 指定 HTTP 响应头部

`headers` 部分允许您为特定资源指定自定义 HTTP 响应头部。`headers` 对象中的键是资源的路径，值是对应于门户将附加到响应的头部的键值对列表。

例如，在上面的配置中，文件 `index.html` 将使用 `Content-Type` 头部设置为 `text/html; charset=utf-8` 和 `Cache-Control` 头部设置为 `max-age=3500` 来提供服务。

此机制允许您控制资源交付的各个方面，如缓存、编码和内容类型。

```admonish
资源路径始终表示为从根目录 `/` 开始。
```

### 默认头部

默认情况下，不需要指定头部，可以省略 `ws-resources.json` 文件。site-builder 将自动尝试根据文件扩展名推断 `Content-Type` 头部，并将 `Content-Encoding` 设置为 `identity`（无转换）。

如果无法推断内容类型，`Content-Type` 将设置为 `application/octet-stream`。

这些默认值将被 `ws-resources.json` 文件中指定的任何头部覆盖。

## 指定客户端路由

`routes` 部分允许您为站点指定客户端路由规则。当您想使用单页应用程序 (SPA) 框架（如 React 或 Angular）时，这很有用。

`routes` 对象中的配置是从路由键到资源路径的映射。

**`routes` 键**是形式为 `/path/to/some/*` 的路径模式，其中 `*` 字符代表通配符。

```admonish
目前，通配符*只能在路径末尾指定*。
因此，`/path/*` 是有效路径，而 `/path/*/to` 和 `*/path/to/*` 则无效。
```

**`routes` 值**是当路由键匹配时应提供服务的资源路径。

```admonish danger title="重要"
值中的路径**必须**是有效的资源路径，这意味着它们必须存在于站点的资源中。如果用户尝试创建指向不存在资源的路由，Walrus sites 合约将**中止**。
```

简单的路由算法如下：

- 每当在站点资源中*找不到*资源路径时，门户会尝试将路径与 `routes` 匹配。
- 然后对所有匹配的路由进行*字典序排序*，并选择*最长*的匹配。
- 然后提供与此最长匹配对应的资源。

```admonish
换句话说，门户将*始终*提供存在的资源，如果不存在，则提供路由中*最长匹配前缀*对应的资源。
```

回顾上面的示例：

``` JSON
"routes": {
  "/*": "/index.html",
  "/path/*": "/accounts.html",
  "/path/assets/*": "/assets/asset_router.html"
}
```

将发生以下匹配：

- 浏览 `/any/other/test.html` 将提供 `/index.html`；
- 浏览 `/path/test.html` 将提供 `/accounts.html`，因为它比前一个匹配更具体；
- 同样，浏览 `/path/assets/test.html` 将提供 `/assets/asset_router.html`。

`/index.html`、`/accounts.html` 和 `/assets/asset_router.html` 都是 Sui 上 Walrus Sites 对象的现有资源。

## 在钱包和浏览器中显示站点对象

通过使用 `metadata` 字段，可以使您的站点对象在 Sui 浏览器和钱包上显示相关信息，从而使其更加美观。这对于添加有关站点的人类可读信息很有用。例如，您可以添加站点主页的链接、站点徽标的图像等。

如您从上面的示例中看到的，这些字段对应于 [Sui Display 对象标准](https://docs.sui.io/standards/display#display-properties)建议的基本属性集。

```JSON
"metadata": {
      "link": "https://subdomain.wal.app",
      "image_url": "https://www.walrus.xyz/walrus-site",
      "description": "This is a walrus site.",
      "project_url": "https://github.com/MystenLabs/walrus-sites/",
      "creator": "MystenLabs"
  }
```

所有元数据字段都是可选的，如果不需要可以省略。site-builder CLI 中指定了一些默认值，用户可以覆盖这些值。

建议像这样使用上述字段：

- `link`：添加指向您站点主页的链接。
- `image_url`：包含将在钱包中显示的站点徽标的 URL。例如，您可以放置指向站点 favicon 的链接。
- `description`：添加站点的简要描述。
- `project_url`：如果您的站点是开源的，请添加指向站点 GitHub 仓库的链接。
- `creator`：添加您的公司、团体或站点的个人创建者的名称。

## 站点名称 (`site_name`)

您可以通过 `ws-resources.json` 文件中的可选 `site_name` 字段为您的 Walrus Site 设置名称。

```admonish note
如果您还通过 `--site-name` cli 标志提供了站点名称，cli 标志将优先于 `ws-resources.json` 中的 `site_name` 字段，后者将被覆盖。
```

```admonish note
如果根本没有提供站点名称（既没有通过 `--site-name` cli 标志，也没有通过 `ws-resources.json` 中的 `site_name` 字段），则将使用默认名称。
```

## 站点对象 ID (`object_id`)

`ws-resources.json` 文件中的可选 `object_id` 字段存储已部署的 Walrus Site 的 Sui 对象 ID。

**在部署中的作用：**

`site-builder deploy` 命令主要使用此字段来标识现有站点以进行更新。
如果存在有效的 `object_id`，`deploy` 将针对该站点进行修改。
如果此字段缺失（且未使用 `--object-id` CLI 标志），`deploy` 将发布新站点。
如果成功，该命令会自动在您的 `ws-resources.json` 中填充此 `object_id` 字段。

## 忽略文件上传

您可以使用可选的 `ignore` 字段来排除某些文件或文件夹的发布。当您想将开发文件、秘密或临时资产排除在最终构建之外时，这很有用。

`ignore` 字段是要跳过的资源路径列表。每个模式必须以 `/` 开头，并可以以 `*` 结尾以指示通配符匹配。

例如：

```json
"ignore": [
  "/private/*",
  "/secret.txt",
  "/images/tmp/*"
]
```

此配置将跳过 `/private/` 和 `/images/tmp/` 目录内的所有文件，以及特定文件 `/secret.txt`。

> 通配符仅在路径的**最后位置**受支持（例如，`/foo/*` 有效，但 `/foo/*/bar` 无效）。