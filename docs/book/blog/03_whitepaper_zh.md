# 宣布官方 Walrus 白皮书

发布时间：2024-09-17

```admonish warning
此博客文章以原始形式显示，可能包含不再准确的信息。
一些损坏的链接可能已被更新或删除。
```

6月，Mysten Labs 宣布了 Walrus，一个新的去中心化安全 blob 存储设计，并推出了一个开发者预览版，目前存储了超过 12TiB 的数据。[Breaking the Ice](https://info.breakingtheice.sui.io/) 聚集了超过 200 名开发者构建利用去中心化存储的应用程序。

现在是时候揭示项目的下一个阶段了：Walrus 将成为一个独立的去中心化网络，拥有自己的实用代币 WAL，该代币将在网络的运营和治理中发挥关键作用。Walrus 将通过使用 WAL 代币的委托权益证明机制由存储节点运营。一个独立的 Walrus 基金会将鼓励 Walrus 的发展和采用，并支持其用户和开发者社区。

今天，我们发布了 Walrus [白皮书](../walrus.pdf)（也在 [GitHub](https://github.com/MystenLabs/walrus/blob/main/docs/book/walrus.pdf) 上），提供了更多详细信息，包括：

- Walrus 使用的编码方案和读/写操作，以确保安全性和高效扩展到数百和数千个存储节点，包括与作为 Walrus 操作协调层的 Sui 区块链的交互。
- 跨时期的存储节点重新配置，以及协议如何确保 Walrus 上可用的 blob 在长期内保持可用。
- 基于 WAL 代币的 Walrus 代币经济学，包括质押和质押奖励的结构，如何处理和分配每个时期的存储定价和付款，以及关键系统参数的治理。
- 前瞻性设计选项，例如挑战和审计存储节点的廉价机制，确保具有更高服务质量的读取选项（可能需要付费），以及赋予轻节点有意义地为协议健壮性做出贡献、提供读取服务并获得奖励的设计。

白皮书专注于 Walrus 的稳态设计方面。有关项目的更多详细信息，例如时间表、社区参与机会、如何作为存储节点加入网络，以及轻节点的计划，将在后续文章中分享。

要成为这一旅程的一部分：

- 在 [Twitter](https://x.com/WalrusProtocol) 上关注我们
- 加入我们的 [Discord](https://discord.com/invite/walrusprotocol)
- 在 Walrus 上[构建应用程序](../index.md)
- [发布 Walrus Site](../walrus-sites/intro.md) 并分享它