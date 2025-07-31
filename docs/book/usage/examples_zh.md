# 示例

作为灵感，我们提供了几个不同编程语言的简单示例，通过各种接口与 Walrus 交互。它们位于 <https://github.com/MystenLabs/walrus/tree/main/docs/examples>，下面将进行描述。

此外，我们在 Walrus 之上构建了实际的应用程序。主要示例是 [Walrus Sites](../walrus-sites/intro_zh.md)，代码可在 <https://github.com/MystenLabs/walrus-sites> 存储库中获得。

有关如何构建静态网站并使用 GitHub actions 将其存储为 Walrus Site 的示例，只需查看我们用于发布这个网站的 [CI 工作流程](https://github.com/MystenLabs/walrus-docs/blob/main/.github/workflows/publish.yaml)。还有一个由社区创建的 [Walrus-Sites GitHub Action](https://github.com/zktx-io/walrus-sites-ga)，您可以使用它非常轻松地使用 GitHub Actions 发布您自己的 Walrus Sites（请注意，这*不是*由 Mysten Labs 的 Walrus 团队创建或官方支持的）。

另请参阅我们现有和即将推出的 [SDK 和其他工具](./sdks_zh.md)列表。

## Python

[Python 示例](https://github.com/MystenLabs/walrus/tree/main/docs/examples/python)文件夹包含许多示例：

- 如何[使用 HTTP API](https://github.com/MystenLabs/walrus/blob/main/docs/examples/python/hello_walrus_webapi.py) 存储和读取 blob。
- 如何[使用 JSON API](https://github.com/MystenLabs/walrus/blob/main/docs/examples/python/hello_walrus_jsonapi.py) 存储、读取和检查 blob 的可用性。检查 blob 的认证说明了读取认证的 Blob Sui 对象（请参见 [Walrus Sui 参考](../dev-guide/sui-struct_zh.md)）。
- 如何[跟踪 Walrus 相关事件](https://github.com/MystenLabs/walrus/blob/main/docs/examples/python/track_walrus_events.py)。

## JavaScript

提供了一个 [JavaScript 示例](https://github.com/MystenLabs/walrus/tree/main/docs/examples/javascript)，展示如何使用 HTTP API 通过 web 表单上传和下载 blob。

## Move

对于更复杂的应用程序，您可能希望与 Walrus 链上对象交互。为此，当前部署的 Walrus 合约包含在[我们的 GitHub 存储库中](https://github.com/MystenLabs/walrus/tree/main/contracts)。

此外，我们提供了一个简单的[示例合约](https://github.com/MystenLabs/walrus/tree/main/docs/examples/move)，它导入并使用 Walrus 对象。