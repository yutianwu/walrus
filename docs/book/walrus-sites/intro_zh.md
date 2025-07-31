# Walrus 站点介绍

*Walrus 站点*是使用 Sui 和 Walrus 作为底层技术的"网"站。它们是展示如何使用 Walrus 构建新的令人兴奋的去中心化应用程序的典型示例。任何人都可以构建和部署 Walrus 站点并使其可供全世界访问！有趣的是，本文档本身就作为 Walrus 站点在 <https://docs.wal.app/walrus-sites/intro.html> 上提供（如果您还没有在那里的话）。

在高层次上，以下是一些最令人兴奋的功能：

- 发布站点不需要管理服务器或复杂配置；只需提供源文件（由您喜欢的 Web 框架生成），使用[站点构建器工具](./overview_zh.md#the-site-builder)将它们发布到 Walrus 站点，就完成了！
- 站点可以从普通的 Sui 对象链接。此功能使得，例如，创建一个 NFT 集合，其中*每个单独的 NFT* 都有一个*专门为其量身定制的个人网站*。
- Walrus 站点由 Sui 上的地址拥有，并且由于 Sui 灵活的编程模型，可以交换、共享和更新。这意味着，除其他事项外，Walrus 站点可以利用 [SuiNS](https://suins.io/) 名称系统拥有人类可读的名称。不再需要处理 DNS！
- 由于 Walrus 的去中心化和极高的数据可用性，没有因任何原因删除您的站点的风险。
- 由于它们存在于 Walrus 上，这些站点不能在传统意义上拥有后端，因此可以被认为是"静态"站点。但是，开发者可以与 Sui 兼容的钱包集成并利用 Sui 的可编程性为 Walrus 站点添加后端功能！

## 展示

为了给您一个 Walrus 站点如何工作的非常高层次的视图，让我们看一个例子：Sui 上的一个简单 NFT 集合，它有一个在 Walrus 站点上托管的前端 dApp 来铸造 NFT，并且其中*每个 NFT* 都有一个*特定的、个性化的 Walrus 站点*。

您可以在 <https://flatland.wal.app/> 查看铸币页面。该站点通过 Walrus 站点*门户* <https://wal.app> 提供给您的浏览器。虽然门户的操作在[后面的部分](./portal_zh.md)中解释，但现在考虑可以有许多门户（由任何想要拥有自己门户的人托管，甚至在 `localhost` 上）。此外，门户的唯一功能是检索构成站点的元数据（来自 Sui）和资源文件（来自 Walrus）。

如果您有一个带有一些 SUI 的 Sui 钱包，您可以尝试从站点"铸造一个新的 Flatlander"。这会从集合中创建一个 NFT 并向您显示两个链接：一个到浏览器，一个到"Flatlander 站点"。后一个站点是一个特殊的 Walrus 站点页面，仅为该 NFT 存在，并具有基于 NFT 内容的特殊特征（背景颜色、图像...）。

此每个 NFT 站点的 URL 看起来像这样：
`https://flatland.wal.app/0x811285f7bbbaad302e23a3467ed8b8e56324ab31294c27d7392dac649b215716`。
您会注意到域名仍然是 `wal.app`，但路径是一个长且看起来随机的字符串。这个字符串实际上是 NFT 对象 ID 的[十六进制](https://simple.wikipedia.org/wiki/Hexadecimal)编码，即 [0x811285f7b...][flatlander]。此路径对每个 NFT 都是唯一的，用于获取其对应页面的元数据和资源文件。然后根据 NFT 的特征渲染页面。

总结：

- Walrus 站点通过门户提供服务；在这种情况下，`https://wal.app`。可以有许多门户，任何人都可以托管一个。
- URL 上的子域指向 Sui 上的特定对象，允许浏览器获取和渲染站点资源。此指针应该是 SuiNS 名称，例如 `https://flatland.wal.app` 中的 `flatland`。
- 每个路径都可以映射到 Sui 上的特定对象，允许浏览器根据 NFT 的特征获取和渲染页面。

想知道这种魔法是如何实现的吗？阅读[技术概述](./overview_zh.md)！如果您只是想开始尝试 Walrus 站点，请查看[教程](./tutorial_zh.md)。

[flatlander]: https://suiscan.xyz/mainnet/object/0x811285f7bbbaad302e23a3467ed8b8e56324ab31294c27d7392dac649b215716