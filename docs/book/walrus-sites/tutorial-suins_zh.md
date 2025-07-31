# 设置 SuiNS 名称

```admonish note
使用 b36 子域名浏览站点（例如 `https://1lupgq2auevjruy7hs9z7tskqwjp5cc8c5ebhci4v57qyl4piy.wal.app`）
在使用 `wal.app` 门户时不再可能。如果您使用本地服务器或替代门户，仍支持 b36 子域名。

要使用 `wal.app` 门户浏览 walrus 站点，请
[改用 SuiNS 名称](./tutorial-suins_zh.md#获取-suins-名称)。
```

Walrus Sites 需要使用 SuiNS 名称（这类似于 Sui 的 DNS）为 Walrus Site 分配人类可读的名称。为此，您只需获取一个您喜欢的 SuiNS 名称，并将其指向 Walrus Site 的对象 ID（由 `publish` 或 `update` 命令提供）。

让我们一步一步来做。

## 获取 SuiNS 名称

- 导航到 <https://suins.io>（主网）或 <https://testnet.suins.io>（测试网）并使用您的钱包购买域名。例如，`walrusgame`（这个特定的名称已被占用，请选择另一个您喜欢的）。

  ```admonish note
  目前，您只能选择由字母 `a-z` 和数字 `0-9` 组成的名称，但不能包含特殊字符（例如 `-`）。
  ```

## 将 SuiNS 名称映射到 Walrus Site

现在，您可以设置 SuiNS 名称指向您的 Walrus Site 的地址。为此，请转到 [SuiNS.io](https://suins.io/account/my-names) 或测试网的 [Testnet.SuiNS.io](https://testnet.suins.io/account/my-names) 的"您拥有的名称"部分，点击您想要映射的名称上方的"三个点"菜单图标，然后点击"Link To Walrus Site"。在栏中粘贴 Walrus Site 的对象 ID，检查是否正确，然后点击"Apply"。

批准交易后，我们现在可以浏览 <https://walrusgame.wal.app>！

``` admonish warning title="向后兼容性"
如果您之前将 SuiNS 域名链接到 Walrus Site，您可能记得点击"Link To Wallet Address"按钮而不是"Link To Walrus Site"按钮。这些旧链接仍然有效，但我们建议对所有新站点和旧站点的更新使用上述程序。门户将首先检查域名是否使用"Link to Walrus Site"链接，如果未设置，它将回退到检查"Link to Wallet Address"。
```