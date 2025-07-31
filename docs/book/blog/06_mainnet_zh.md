# 宣布主网

发布时间：2025-03-27

## 去中心化网络：主网

生产级 Walrus 主网现已上线，由超过 100 个存储节点组成的去中心化网络运营。第 1 个时期于 2025 年 3 月 25 日开始。该网络现在可以用于[发布和检索 blob](../usage/interacting.md)，[上传和浏览 Walrus Sites](../walrus-sites/tutorial-publish.md)，以及使用实时[主网 WAL 代币](https://www.walrus.xyz/wal-token)进行[质押和解质押](https://stake-wal.wal.app/)来确定未来的委员会。在主网上，Walrus 安全属性得到保障。Walrus 现在已准备好满足真实应用程序的需求。

Walrus 是[开源的](https://github.com/MystenLabs/walrus)。Walrus 协议的健康状况由一个[独立基金会](https://www.walrus.xyz/)监督，该基金会现在[资源充足](https://www.walrus.xyz/blog/walrus-foundation-fundraising)，可以支持未来的开发和增长。社区正在 [Awesome Walrus!](https://github.com/MystenLabs/awesome-walrus) 下收集 Walrus 的工具、资源和 dapp。

这是一个重要的里程碑，距离一个小团队开始设计 Walrus 协议仅仅 12 个多月。特别感谢通过连续测试网支持 Walrus 开发的存储运营商社区，以及早期集成协议并提供反馈以改进协议的所有社区成员。

## 新功能

除了稳定性和安全性的承诺之外，Walrus 主网版本还带来了一些值得注意的新功能和变更：

- **Blob 属性。** 每个 Sui blob 对象都可以附加多个属性和值，以编码应用程序元数据。聚合器使用此功能返回常见的 HTTP 头。

- **在 Sui 上销毁 blob 对象**。命令行 `walrus` 工具扩展了"销毁" Sui blob 对象以回收相关存储费用的命令，使存储 blob 变得更便宜。

- **CLI 过期时间改进。** 现在在存储 blob 时可以更灵活地表达 blob 过期时间，通过 `--epochs max`、使用 `--earliest-expiry-time` 的 RFC3339 日期，或使用 `--end-epoch` 的具体结束时期。

- **使用 Reed-Solomon 码的 RedStuff。** RedStuff 底层的纠删码从 RaptorQ 更改为 Reed-Solomon (RS) 码。深入的基准测试表明，对于我们的参数集，它们的性能相似，使用 RS 码提供完美的健壮性，即在给定阈值的切片情况下总是可以重构 blob。我们感谢 [Joachim Neu](https://www.jneu.net/) 在这个主题上的广泛反馈。

- **存储节点的 TLS 处理。** 存储节点现在可以配置为提供公开信任的 TLS 证书，例如云提供商和 Let's Encrypt 等公共机构颁发的证书。这允许 JavaScript 客户端直接从 Walrus 存储和检索 blob。

- **发布器的 JWT 认证。** 现在发布器可以配置为仅通过消费可通过任何认证机制分发的 JWT 令牌向经过认证的用户提供服务。

- **广泛的日志记录和指标。** 从存储节点到聚合器和发布器的所有服务都导出可用于构建仪表板的指标，以及多个级别的日志来排除其操作故障。

- **健康端点。** 存储节点和 CLI 包含一个 `health` 命令来检查存储节点的状态和基本信息。这在分配质押以及监控网络时很有用。

- **Walrus Sites 更新。** 主网 Walrus Sites 公共门户将托管在 `wal.app` 域名上。现在 Walrus Sites 支持可删除的 blob，使其更新更加资本高效。运营自己门户的人也可以使用自己的域名来提供 Walrus Site。[质押](https://stake-wal.wal.app/)、[文档](https://docs.wal.app)、[Snowreads](https://snowreads.wal.app) 和 [Flatland](https://flatland.wal.app) Walrus Sites 现在在主网上。

- **补贴合约。** Walrus 主网版本需要 WAL 来存储 blob，但是为了让早期采用者能够试用系统并过渡到它，Walrus 基金会运营一个智能合约来[获取补贴存储](https://github.com/MystenLabs/walrus/tree/main/contracts/subsidies)。CLI 客户端在存储 blob 时会自动使用它。

## 测试网未来计划

当前的 Walrus 测试网将很快被清除并重新启动，以使代码库与主网版本保持一致。今后我们将定期清除测试网，每隔几个月一次。开发者应该使用 Walrus 主网来获得任何级别的稳定性。Walrus 测试网只是为了在新功能部署到生产环境之前对其进行测试。

我们不会在测试网上运营公共门户来提供 Walrus Sites，以减少与免费网络托管相关的成本和事件响应。开发者仍然可以运行指向测试网的本地门户来测试 Walrus Sites。

## 开源 Walrus 代码库

Walrus 代码库，包括 Move 中的所有智能合约、Rust 中的服务和文档，现在在 Apache 2.0 许可证下开源，并托管在 [GitHub](https://github.com/MystenLabs/walrus) 上。由于主要的 Walrus 仓库现在对所有人开放，并包含文档和智能合约，我们正在停用 `walrus-docs` 仓库。

开发者可能会发现 Rust CLI 客户端以及相关的聚合器和发布器服务最感兴趣。这些可以扩展以专门化服务到特定的操作需求。它们也是如何使用 Rust 客户端与 Walrus 接口的良好示例。一个更清洁、更稳定的 Rust SDK 也在开发中。

## 发布器改进和计划

在主网上发布 blob 会消耗真实的 WAL 和 SUI。为了避免运营的不受控制的成本，发布器服务增加了认证请求和为每个用户计算成本的功能。