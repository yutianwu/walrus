# 已知限制

Walrus Sites 可以用于部署几乎任何形式的传统静态 web2 网站，为现代浏览器构建。然而，开发者在创建或将网站移植到 Walrus Sites 时应该记住一些限制。

## 无秘密值

Walrus Sites 是完全公开访问的，因为元数据存储在 Sui 上，站点内容存储在 Walrus 上。因此，开发者*绝不能*在站点内存储秘密值。

我们再次强调，任何此类后端特定操作（存储秘密值、身份验证等）都可以通过利用与 Sui 区块链和 Sui 兼容钱包的集成来实现。

## 存在最大重定向深度

Walrus Site 可以执行的连续重定向次数由门户限制（参见[门户配置](./portal_zh.md)）。此措施确保加载 Walrus Site 不会导致无限加载循环。

不同的门户可以根据需要设置此限制。托管在 <https://wal.app> 的门户的最大重定向深度为 3。

## 存在特殊的 `__wal__` 路径

`/__wal__/*` 路径下的任何内容都由门户保留用于健康检查和其他门户特定操作。如果您在站点内使用此路径，该路径下提供的资源将无法正确加载。

## Service-worker 门户限制

以下限制仅适用于基于 service worker 的门户。

``` admonish tip
如果您需要支持下面列出的任何功能，您应该使用服务器端门户。
```

### Service-worker 门户无法提供基于 service worker 的站点

Service-worker 门户利用客户端浏览器中的 service worker 来执行基本操作：

1. 从 Sui 读取站点元数据；
1. 从 Walrus 获取页面内容；以及
1. 向浏览器提供内容。

因此，通过 service-worker 门户访问的站点本身不能使用 service worker。即您不能"堆叠" service worker！从 Walrus Site 内安装 service worker 将导致站点功能失常和用户体验不佳。

### iOS Sui 移动钱包无法与 service-worker 门户配合使用

```admonish warning
此限制**仅适用于基于 service worker 的门户**。如果您需要访问支持此功能的站点，您应该使用服务器端门户。
```

由于 WebKit 引擎的限制，service worker 无法在 iOS 的应用内浏览器中加载。因此，通过 service worker 门户访问的 Walrus Sites 无法在 iOS 上的 Sui 兼容钱包应用内使用。因此，Sui 钱包目前无法在 iOS 上的 service-worker 门户上使用。但是，请注意，在 iOS 上通过任何浏览器*浏览* Walrus Site 仍然是可能的。

鉴于您决定使用 service-worker 门户作为访问您站点的主要入口，为了为 iOS 用户（以及其他不支持 service worker 的浏览器上的用户）提供无缝体验，建议重定向到服务器端门户（<https://wal.app>）。每当 iOS 钱包上的用户浏览 Walrus Site 时，重定向将自动将他们带到 `<site_name>.wal.app` 服务器端门户。这样，用户仍然可以使用钱包。

### Service worker 门户不支持渐进式 Web 应用程序 (PWA)

```admonish warning
此限制**仅适用于基于 service worker 的门户**。如果您需要访问支持此功能的站点，您应该使用服务器端门户。
```

在当前设计下，service-worker 门户无法用于访问渐进式 Web 应用程序 (PWA)。

service-worker 门户的两个特性阻止了对 PWA 的支持：

- 由于需要注册 service worker 才能使页面工作，PWA 的清单文件无法直接由浏览器加载。
- 每个源只能注册一个 service worker。因此，注册 PWA 的 service worker 会移除 Walrus Sites service worker，破坏 Walrus Sites 的功能。

请注意，服务器端门户不存在这些限制。但是，目前我们支持两种技术：Walrus Sites 必须能够从 service-worker 门户和服务器端门户加载，因此必须使用限制性更强的功能集构建。有关更多详细信息，请参阅 [PWA 的安装要求](https://en.wikipedia.org/wiki/Progressive_web_app#Installation_criteria)。