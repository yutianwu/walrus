# 配置站点构建器

配置 `site-builder` 工具很简单，但需要小心确保一切正常工作。

`site-builder` 工具需要配置文件来了解在 Sui 上使用哪个包、使用哪个钱包、gas 预算和其他操作细节。其中大部分通过合理的默认值进行抽象化，因此您应该不需要修改它们。但是，为了完整起见，我们在这里提供所有配置选项的详细信息。

## 最小配置

配置文件预期位于[默认位置](../usage/setup_zh.md#config-custom-path)之一，也可以通过 `--config` 标志指向其他位置。对于您的首次运行，应该足以在不明确指定配置的情况下调用 `site-builder`，如果您彻底遵循了[安装步骤](./tutorial-install_zh.md#configuration)，配置已经配置好了。

如果由于任何原因您没有将 `walrus` 添加到 `$PATH`，请确保配置指向二进制文件的指针，见下文。

## 其他选项

如果您想对站点构建器的行为有更多控制，可以在配置文件中自定义以下变量：

- `package`：Sui 上 Walrus Sites 包的对象 ID。这必须始终在配置中指定，并且在 `./sites-config.yaml` 中已经适当配置。
- `portal`：将通过其查看站点的门户名称；这只影响 CLI 的输出，而不影响其他任何内容（默认：`wal.app`）。所有 Walrus Sites 都可以通过任何门户访问，而不管此设置如何。
- `general`：这些是可以通过 CLI 和配置配置的一般选项：
  - `rpc_url`：要使用的 Sui RPC 节点的 URL。如果未设置，`site-builder` 将从钱包推断。
  - `wallet`：指向要使用的 Sui 钱包的指针。默认情况下，它使用系统范围的钱包（来自 `sui client addresses` 的钱包）。
  - `walrus_binary`：指向 `walrus` 二进制文件的指针。默认情况下，这预期从 `$PATH` 运行。
  - `walrus_config`：`walrus` 客户端二进制文件的配置，请参阅[相关章节](../usage/setup_zh.md)。
  - `gas_budget`：交易花费的最大 gas 量（默认：500M MIST）。

```admonish note
偶尔会发生包升级。这将要求您更新配置文件中的 `package` 字段。
```