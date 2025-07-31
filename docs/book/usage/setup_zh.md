# 设置

我们为 macOS（Intel 和 Apple CPU）和 Ubuntu 提供了预编译的 `walrus` 客户端二进制文件，它支持不同的使用模式（请参阅[下一章节](./interacting_zh.md)）。本章描述了 Walrus 客户端的[先决条件](#prerequisites)、[安装](#installation)和[配置](#configuration)。

Walrus 是基于 Apache 2 许可证的开源软件，也可以通过 cargo 从 Rust 源代码构建和安装。

```admonish info title="Walrus 网络"
本页描述了如何连接到 Walrus **主网**。有关所有 Walrus 网络的概述，请参阅[可用网络](./networks_zh.md)。
```

## 先决条件：Sui 钱包、SUI 和 WAL {#prerequisites}

```admonish tip title="快速钱包设置"
如果您只想为 Walrus 设置一个新的 Sui 钱包，可以在[安装 Walrus](#installation) 后使用 
`walrus generate-sui-wallet --network mainnet` 命令生成一个。
您仍然需要获得一些 SUI 和 WAL 代币，但不必安装 Sui CLI。
```

与 Walrus 交互需要一个有效的 Sui 钱包，其中包含一定数量的 SUI 和 WAL 代币。设置的正常方式是通过 Sui CLI；请参阅 Sui 文档中的[安装说明](https://docs.sui.io/guides/developer/getting-started/sui-install)。

安装 Sui CLI 后，您需要通过运行 `sui client` 来设置钱包，它会提示您设置新配置。确保将其指向 Sui 主网，您可以使用 `https://fullnode.mainnet.sui.io:443` 的完整节点。有关更多详细信息，请参阅 [Sui 文档](https://docs.sui.io/guides/developer/getting-started/connect)。

之后，您应该得到类似这样的内容（除了 `mainnet` 行外，其他都是可选的）：

```terminal
$ sui client envs
╭──────────┬─────────────────────────────────────┬────────╮
│ alias    │ url                                 │ active │
├──────────┼─────────────────────────────────────┼────────┤
│ devnet   │ https://fullnode.devnet.sui.io:443  │        │
│ localnet │ http://127.0.0.1:9000               │        │
│ testnet  │ https://fullnode.testnet.sui.io:443 │        │
│ mainnet  │ https://fullnode.mainnet.sui.io:443 │ *      │
╰──────────┴─────────────────────────────────────┴────────╯
```

确保您至少有一个燃料币，至少 1 SUI。

```terminal
$ sui client gas
╭─────────────────┬────────────────────┬──────────────────╮
│ gasCoinId       │ mistBalance (MIST) │ suiBalance (SUI) │
├─────────────────┼────────────────────┼──────────────────┤
│ 0x65dca966dc... │ 1000000000         │ 1.00             │
╰─────────────────┴────────────────────┴──────────────────╯
```

最后，要在 Walrus 上发布数据块，您需要一些主网 WAL 来支付存储和上传成本。您可以通过各种中心化或去中心化交易所购买 WAL。

如果未指定其他路径，Walrus 将使用系统范围的钱包。如果您想使用不同的 Sui 钱包，可以在 [Walrus 配置文件](#configuration)中指定，或在[运行 CLI](./interacting_zh.md) 时指定。

## 安装

我们目前为 macOS（Intel 和 Apple CPU）、Ubuntu 和 Windows 提供 `walrus` 客户端二进制文件。Ubuntu 版本很可能也适用于其他 Linux 发行版。

| 操作系统 | CPU                    | 架构                                                                                                                         |
| -------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Ubuntu   | Intel 64位             | [`ubuntu-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-mainnet-latest-ubuntu-x86_64)                 |
| Ubuntu   | Intel 64位（通用）     | [`ubuntu-x86_64-generic`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-mainnet-latest-ubuntu-x86_64-generic) |
| Ubuntu   | ARM 64位               | [`ubuntu-aarch64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-mainnet-latest-ubuntu-aarch64)               |
| MacOS    | Apple Silicon          | [`macos-arm64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-mainnet-latest-macos-arm64)                     |
| MacOS    | Intel 64位             | [`macos-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-mainnet-latest-macos-x86_64)                   |
| Windows  | Intel 64位             | [`windows-x86_64.exe`](https://storage.googleapis.com/mysten-walrus-binaries/walrus-mainnet-latest-windows-x86_64.exe)       |

### 通过脚本安装 {#nix-install}

要下载并将 `walrus` 安装到您的 `"$HOME"/.local/bin` 目录，请在终端中运行以下命令之一，然后按照屏幕上的说明操作。如果您使用 Windows，请参阅 [Windows 特定说明](#windows-install)或 [`suiup` 安装](#suiup-install)（实验性）

```sh
# 使用最新主网版本进行首次安装。
curl -sSf https://install.wal.app | sh

# 改为安装最新测试网版本。
curl -sSf https://install.wal.app | sh -s -- -n testnet

# 更新现有安装（覆盖之前的 walrus 版本）。
curl -sSf https://install.wal.app | sh -s -- -f
```

确保 `"$HOME"/.local/bin` 目录在您的 `$PATH` 中。

完成后，您应该能够在终端中使用 `walrus` 命令运行 Walrus。

您可以查看使用说明如下（有关更多详细信息，请参阅[下一章节](./interacting_zh.md)）：

```terminal
$ walrus --help
Walrus client

Usage: walrus [OPTIONS] <COMMAND>

Commands:
⋮
```

```admonish tip
我们最新的 Walrus 二进制文件也可在 Walrus 本身上获得，即在 <https://bin.wal.app>，
例如，<https://bin.wal.app/walrus-mainnet-latest-ubuntu-x86_64>。
请注意，由于 DoS 保护，可能无法使用 `curl` 或 `wget` 下载二进制文件。
```

### 在 Windows 上安装 {#windows-install}

要将 `walrus` 下载到您的 Microsoft Windows 计算机，请在 PowerShell 中运行以下命令。

```PowerShell
(New-Object System.Net.WebClient).DownloadFile(
  "https://storage.googleapis.com/mysten-walrus-binaries/walrus-testnet-latest-windows-x86_64.exe",
  "walrus.exe"
)
```

然后，您需要将 `walrus.exe` 放在您的 `PATH` 中的某个位置。

```admonish title="Windows"
请注意，其余大部分说明假设基于 UNIX 的系统的目录结构、命令等。
如果您使用 Windows，可能需要调整其中大部分内容。
```

### 通过 suiup 安装（实验性） {#suiup-install}

`suiup` 是一个用于安装和管理在 Sui 生态系统中工作的 CLI 工具不同版本的工具，包括 `walrus` CLI。按照 [`suiup` 文档](https://github.com/MystenLabs/suiup?tab=readme-ov-file#installation)中描述的方式安装 `suiup` 后，您可以安装 `walrus` CLI 的特定版本：

```sh
suiup install walrus@testnet # 安装最新的测试网版本
suiup install walrus@mainnet # 安装最新的主网版本
suiup install walrus@testnet-v1.27.1 # 安装特定版本
```

### GitHub 发布

您可以在 [GitHub](https://github.com/MystenLabs/walrus/releases) 上找到包含发布说明的所有版本。只需下载适合您系统的存档并提取 `walrus` 二进制文件。

### 通过 Cargo 安装

您也可以通过 Cargo 安装 Walrus。例如，要安装最新的主网版本：

```sh
cargo install --git https://github.com/MystenLabs/walrus --branch mainnet walrus-service --locked
```

您也可以指定特定标签（例如，`--tag mainnet-v1.18.2`）或提交（例如，`--rev b2009ac73388705f379ddad48515e1c1503fc8fc`）来代替 `--branch mainnet`。

### 从源代码构建

Walrus 是在 Apache 2 许可证下发布的开源软件。代码在 <https://github.com/MystenLabs/walrus> 的 `git` 仓库中开发。

主网和测试网的最新版本分别在 `mainnet` 和 `testnet` 分支下可用，最新版本在 `main` 分支下。我们欢迎问题报告和错误修复。按照 `README.md` 文件中的说明从源代码构建和使用 Walrus。

## 配置

Walrus 客户端需要了解存储 Walrus 系统和质押信息的 Sui 对象，请参阅[开发者指南](../dev-guide/sui-struct_zh.md#system-and-staking-information)。这些需要在文件 `~/.config/walrus/client_config.yaml` 中配置。此外，可以指定 `subsidies` 对象，该对象将补贴使用客户端购买的存储。

您可以通过以下配置访问测试网和主网。请注意，此示例 Walrus CLI 配置引用了 Sui 配置的标准位置（`"~/.sui/sui_config/client.yaml"`）。

```yaml
{{ #include ../setup/client_config.yaml }}
```

<!-- markdownlint-disable code-fence-style -->
~~~admonish tip
获得最新配置的最简单方法是直接从 Walrus 下载：

```sh
curl https://docs.wal.app/setup/client_config.yaml -o ~/.config/walrus/client_config.yaml
```
~~~
<!-- markdownlint-enable code-fence-style -->

### 自定义路径（可选） {#config-custom-path}

默认情况下，Walrus 客户端将在当前目录、`$XDG_CONFIG_HOME/walrus/`、`~/.config/walrus/` 或 `~/.walrus/` 中查找 `client_config.yaml`（或 `client_config.yml`）配置文件。但是，您可以将文件放在任何地方并为其命名任何您喜欢的名称；在这种情况下，您需要在运行 `walrus` 二进制文件时使用 `--config` 选项。

### 高级配置（可选）

配置文件目前支持每个上下文的以下参数：

```yaml
# 这些是唯一的必填字段。这些对象特定于特定的 Walrus 
# 部署，但随后不会随时间变化。
system_object: 0x2134d52768ea07e8c43570ef975eb3e4c27a39fa6396bef985b5abc58d03ddd2
staking_object: 0x10b9d30c28448939ce6c4d6c6e0ffce4a7f8a4ada8248bdad09ef8b70e4a3904

# subsidies 对象允许客户端使用补贴合约购买存储
# 这将减少获得存储资源和扩展数据块的成本，并且
# 为质押池的奖励添加补贴。
subsidies_object: 0xb606eb177899edc2130c93bf65985af7ec959a2755dc126c953755e59324209e

# 您可以在此定义 Sui 钱包配置的自定义路径。如果未设置或为 `null`
#（默认），钱包从 `./sui_config.yaml`（相对于您当前的工作目录）配置，
# 或者按此顺序从 `~/.sui/sui_config/client.yaml` 的系统范围钱包配置。
# `active_env` 和 `active_address` 都可以省略，在这种情况下使用 Sui 钱包中的值。
wallet_config:
  # 钱包配置文件的路径。
  path: ~/.sui/sui_config/client.yaml
  # 可选的 `active_env` 用于覆盖配置文件中列出的任何 `active_env`。
  active_env: mainnet
  # 可选的 `active_address` 用于覆盖配置文件中列出的任何 `active_address`。
  active_address: 0x...

# 以下参数可用于调整客户端的网络行为。玩弄这些值没有风险。
# 在最坏的情况下，由于超时或其他网络错误，您可能无法存储/读取数据块。
{{ #include ../setup/client_config_example.yaml:8: }}
```