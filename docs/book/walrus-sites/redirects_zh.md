# 将对象重定向到 Walrus Sites

我们在[概述](./overview_zh.md)中看到了 Sui 上的 Walrus Site 对象是什么样的。现在我们将讨论如何确保*一组任意对象*都可以绑定到一个特定的、可能是唯一的 Walrus Site。

## 目标

考虑一个 NFT 集合，比如由 <https://flatland.wal.app> 发布的集合。正如我们在那里展示的，每个铸造的 NFT 都有自己的 Walrus Site，可以根据 NFT 本身的内容（例如颜色）进行个性化。我们如何实现这一点呢？

## 重定向链接

解决方案很简单：我们在 NFT 的 [`Display`](https://docs.sui.io/standards/display#sui-utility-objects) 属性中添加一个"重定向"。每当通过门户浏览 NFT 的对象 ID 时，门户将检查 NFT 的 `Display`，如果遇到 `walrus site address` 键，它将获取位于相应对象 ID 的 Walrus Site。

### Move 中的重定向

实际上，在创建 NFT 的 `Display` 时，您可以包含指向要使用的 Walrus Site 的键值对。

``` move
...
const VISUALIZATION_SITE: address = @0x901fb0...;
display.add(b"walrus site address".to_string(), VISUALIZATION_SITE.to_string());
...
```

### 如何根据 NFT 更改站点？

上面的代码只会在浏览 NFT 的对象 ID 时打开指定的 Walrus Site。我们如何确保可以使用 NFT 的属性来个性化站点？

这需要在 `VISUALIZATION_SITE` 中完成：由于子域名仍然指向 NFT 的对象 ID，加载的 Walrus Site 可以在 JavaScript 中检查其 `origin`，并使用子域名来确定 NFT，从链上获取它，并使用其内部字段来修改显示的站点。

有关端到端示例，请参阅 `flatland` [仓库](https://github.com/MystenLabs/example-walrus-sites/tree/main/flatland)。