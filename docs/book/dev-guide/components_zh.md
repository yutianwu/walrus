# 组件

从开发者的角度来看，一些 Walrus 组件是 Sui 上的对象和智能合约，而一些组件是 Walrus 特定的二进制文件和服务。通常，Sui 用于管理 blob 和存储节点的元数据，而 Walrus 特定的服务用于存储和读取 blob 内容，这些内容可能非常大。

Walrus 在 Sui 上定义了许多对象和智能合约：

- 一个共享的*系统对象*记录和管理当前的存储节点委员会。
- *存储资源*代表可用于存储 blob 的空存储空间。
- *Blob 资源*代表正在注册并认证为已存储的 blob。
- 对这些对象的更改会发出*Walrus 相关事件*。

Walrus 系统对象 ID 可以在 Walrus `client_config.yaml` 文件中找到（参见[配置](../usage/setup.md#configuration)）。您可以使用任何 Sui 浏览器查看其内容，以及探索 blob 对象的内容。有关这些的更多信息，请参见 [Walrus Sui 结构快速参考](sui-struct.md)。

Walrus 还由许多 Walrus 特定的服务和二进制文件组成：

- 客户端（二进制文件）可以在本地执行，提供[命令行界面（CLI）](../usage/client-cli.md)、[JSON API](../usage/json-api.md)和 [HTTP API](../usage/web-api.md) 来执行 Walrus 操作。
- 聚合器服务允许通过 HTTP 请求读取 blob。
- 发布器服务用于将 blob 存储到 Walrus。
- 一组存储节点存储编码的 blob。这些节点构成了 Walrus 的去中心化存储基础设施。

聚合器、发布器和其他服务使用客户端 API 与 Walrus 交互。使用 Walrus 的服务的最终用户通过自定义服务、聚合器或发布器与存储进行交互，这些服务公开 HTTP API，避免了需要在本地运行二进制客户端的需要。