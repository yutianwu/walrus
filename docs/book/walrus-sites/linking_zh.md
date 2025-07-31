# 从和到 Walrus Sites 的链接

``` admonish warning title="Walrus Sites 链接目前不可用"
此功能目前在服务器端门户上不可用。因此，如果您在 <https://wal.app> 上浏览 Walrus Site，您将无法使用 Walrus Sites 链接。

我们正在努力启用此功能。敬请期待！
```

Walrus Sites 中的链接*几乎*与您在常规网站中期望的那样工作。我们在这里指定一些细节。

## 链接到同一站点内的资源

相对和绝对链接（`href="/path/to/resource.html"`）照常工作。

## 链接到 Web 上的资源

链接到 Web 上的资源（`href="https://some.cdn.example.com/stylesheet.css"`）也照常工作。

## 链接到其他 Walrus Sites 中的资源

这里是有点不同的部分。假设有一些图像可以在 `https://gallery.wal.app/walrus_arctic.webp` 浏览，您想从自己的 Walrus Site 链接它。

但是，请记住，`https://wal.app` 只是可能许多门户中的一个。即，相同的资源可以从本地门户（`http://gallery.localhost:8080/walrus_arctic.webp`）浏览。因此，您如何以*门户无关的方式*链接资源？这对于互操作性、可用性和尊重用户的门户选择很重要。

### 解决方案：Walrus Sites 链接

``` admonish warning
此功能仅适用于基于服务工作者的门户。
```

我们通过让门户解释在 Web 上通常无效的特殊链接并重定向到门户本身中相应的 Walrus Sites 资源来解决这个问题。

考虑上面的示例，其中资源 `/walrus_arctic.webp` 从具有 SuiNS 名称 `gallery` 的 Walrus Site 浏览，该名称指向对象 ID `abcd123…`（Base36 编码）。然后，门户无关的链接是：`https://gallery.suiobj/walrus_arctic.webp`。要固定对象 ID 而不是 SuiNS 名称，您可以使用 `https://abcd123….suiobj/walrus_arctic.webp`。

另一种可能性是直接指向资源的 Walrus *blob ID*，并让浏览器"嗅探"内容类型。这适用于图像，例如，但不适用于脚本或样式表。例如，要指向 blob ID（例如，包含图像）`qwer5678…`，使用 URL `https://blobid.walrus/qwer5678…`。

使用这样的链接，门户将提取 blob ID 并将请求重定向到它正在使用的聚合器以获取 blob。