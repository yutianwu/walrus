# 发布 Walrus Site

现在一切都已安装和配置完成，您应该能够开始发布您的第一个 Walrus Site！

## 选择站点的源材料

`site-builder` 通过将任何 Web 框架产生的文件目录上传到 Walrus 并将相关元数据添加到 Sui 来工作。此目录应在其根目录中有一个名为 `index.html` 的文件，这将是 Walrus Site 的入口点。

有一个非常有用的 [example-Walrus-sites](https://github.com/MystenLabs/example-walrus-sites) 仓库，包含多种类型的站点，您可以用作参考。

为了简单起见，我们将首先发布最简朴的站点，即 `walrus-snake` 游戏。

首先，克隆示例仓库：

``` sh
git clone https://github.com/MystenLabs/example-walrus-sites.git && cd example-walrus-sites
```

## 发布站点

由于我们已将 `walrus` 和 `site-builder` 二进制文件和配置放置在其默认位置，发布 `./walrus-snake` 站点就像调用发布命令一样简单：

``` sh
site-builder deploy ./walrus-snake --epochs 1
```

``` admonish tip
根据网络，一个 epoch 的持续时间可能会有所不同。目前在 Walrus 测试网上，一个 epoch 的持续时间是一天。在主网上，一个 epoch 的持续时间是两周。
```

输出的结尾应该如下所示：

``` txt
Execution completed
Resource operations performed:
  - created resource /.DS_Store with blob ID PwNzE9_a9anYb8AZysafQZGqd4h0scsTGhzF2GPsWmQ
  - created resource /Oi-Regular.ttf with blob ID KUTTV_95_c68oQhaRP97tDPOYu0vqCWiGL7mzOq1faU
  - created resource /file.svg with blob ID oUpm044qBN1rkyIJYvMB4dUj6bRe3QEvJAN-cvlIFmk
  - created resource /index.html with blob ID AR03hvxSlyfYl-7MhXct4y3rnIIGPHdnjiIF03BK_XY
  - created resource /walrus.svg with blob ID xK8K1Q5khrl3eBT4jEiB-L_gyShEIOVWti8DcAoEjtw
The site routes were modified

Created new site: test site
New site object ID: 0xe674c144119a37a0ed9cef26a962c3fdfbdbfd86a3b3db562ee81d5542a4eccf
To browse the site, you have the following options:
        1. Run a local portal, and browse the site through it: e.g. http://5qs1ypn4wn90d6mv7d7dkwvvl49hdrlpqulr11ngpykoifycwf.localhost:3000
           (more info: https://docs.wal.app/walrus-sites/portal.html#running-the-portal-locally)
        2. Use a third-party portal (e.g. wal.app), which will require a SuiNS name.
           First, buy a SuiNS name at suins.io (e.g. example-domain), then point it to the site object ID.
           Finally, browse it with: https://example-domain.wal.app
```

``` admonish note
请记住，选项 2 仅在 `mainnet` 上可用。
```

此输出告诉您，对于文件夹中的每个文件，都创建了一个新的 Walrus blob，以及相应的 blob ID。此外，它打印 Sui 上 Walrus Site 对象的对象 ID（因此您可以在浏览器中查看并使用它来设置 SuiNS 名称），最后是您可以浏览站点的 URL。
deploy 命令还会将这个新的站点对象 ID 保存到 ws-resources.json

请注意，我们这里隐式使用默认的 `sites-config.yaml` 作为我们之前在[安装部分](./tutorial-install_zh.md)设置的站点构建器的配置。配置文件是必要的，以确保 `site-builder` 知道 Walrus Sites 逻辑的正确 Sui 包。

有关 `site-builder` 配置的更多详细信息，可以在[高级配置](./builder-config_zh.md)部分找到。

## 更新站点

现在假设您想更新站点的内容，例如将标题从"eat all the blobs!"更改为"Glob all the Blobs!"。

首先，在 `./walrus-snake/index.html` 文件中进行此编辑。

然后，您可以通过再次运行 `deploy` 命令来更新现有站点。deploy 命令将使用存储在 ws-resources.json 中的站点对象 ID（来自初始部署）来识别要更新的站点。您无需手动指定对象 ID：

``` sh
site-builder deploy --epochs 1 ./walrus-snake
```

这次的输出应该是：

``` txt
Execution completed
Resource operations performed:
  - deleted resource /index.html with blob ID LVLk9VSnBrEgQ2HJHAgU3p8IarKypQpfn38aSeUZzzE
  - created resource /index.html with blob ID pcZaosgEFtmP2d2IV3QdVhnUjajvQzY2ev8d9U_D5VY
The site routes were left unchanged

Site object ID: 0xe674c144119a37a0ed9cef26a962c3fdfbdbfd86a3b3db562ee81d5542a4eccf
To browse the site, you have the following options:
        1. Run a local portal, and browse the site through it: e.g. http://2ql9wtro4xf2x13pm9jjeyhhfj28okawz5hy453hkyfeholy6f.localhost:3000
           (more info: https://docs.wal.app/walrus-sites/portal.html#running-the-portal-locally)
        2. Use a third-party portal (e.g. wal.app), which will require a SuiNS name.
           First, buy a SuiNS name at suins.io (e.g. example-domain), then point it to the site object ID.
           Finally, browse it with: https://example-domain.wal.app
```

与站点首次发布时相比，我们可以看到现在执行的唯一操作是删除旧的 `index.html`，并用较新的替换它。

浏览到提供的 URL 应该反映更改。您已更新站点！

```admonish note
您使用的钱包必须是 Walrus Site 对象的*所有者*才能更新它。
```

```admonish danger title="延长现有站点的到期日期"
要延长先前存储的站点的到期日期，请使用带有 `--check-extend` 标志的 `update` 命令。使用此标志，site-builder 将强制检查组成站点的所有 Walrus blob 的状态，并延长在 `--epochs` epoch 之前到期的 blob。这对于确保站点中的所有资源在相同数量的 epoch 内可用很有用。
```