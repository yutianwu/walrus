# 安装站点构建器

本节描述设置 Walrus Sites 的 `site-builder` 工具并为开发准备环境所需的步骤。

## 先决条件

在开始之前，请确保您

- 已安装最新版本的 [Rust](https://www.rust-lang.org/tools/install)；
- 已遵循所有 [Walrus 设置说明](../usage/setup_zh.md)。

然后，按照这些额外的设置步骤。

## 安装

与 `walrus` 客户端 CLI 工具类似，我们目前为 macOS（Intel 和 Apple CPU）、Ubuntu 和 Windows 提供 `site-builder` 客户端二进制文件：

### 主网二进制文件

| OS      | CPU                   | Architecture |
|---------|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| Ubuntu  | Intel 64bit           | [`site-builder-mainnet-latest-ubuntu-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-mainnet-latest-ubuntu-x86_64)                |
| MacOS   | Apple Silicon         | [`site-builder-mainnet-latest-macos-arm64`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-mainnet-latest-macos-arm64)                    |
| MacOS   | Intel 64bit           | [`site-builder-mainnet-latest-macos-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-mainnet-latest-macos-x86_64)                  |
| Windows | Intel 64bit           | [`site-builder-mainnet-latest-windows-x86_64.exe`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-mainnet-latest-windows-x86_64.exe)      |

### 测试网二进制文件

| OS      | CPU                   | Architecture |
|---------|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| Ubuntu  | Intel 64bit           | [`site-builder-testnet-latest-ubuntu-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-testnet-latest-ubuntu-x86_64)                |
| MacOS   | Apple Silicon         | [`site-builder-testnet-latest-macos-arm64`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-testnet-latest-macos-arm64)                    |
| MacOS   | Intel 64bit           | [`site-builder-testnet-latest-macos-x86_64`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-testnet-latest-macos-x86_64)                  |
| Windows | Intel 64bit           | [`site-builder-testnet-latest-windows-x86_64.exe`](https://storage.googleapis.com/mysten-walrus-binaries/site-builder-testnet-latest-windows-x86_64.exe)      |

```admonish title="Windows"
我们现在也为 Windows 提供预构建的二进制文件。但是，其余的大部分说明假设基于 UNIX 的系统的目录结构、命令等。如果您使用 Windows，您可能需要调整其中大部分内容。
```

您可以从我们的 Google Cloud Storage (GCS) 存储桶下载最新构建（正确设置 `$SYSTEM` 变量）：

### 主网 curl 请求

``` sh
SYSTEM= # 将此设置为您的系统：ubuntu-x86_64, ubuntu-x86_64-generic, macos-x86_64, macos-arm64, windows-x86_64.exe
curl https://storage.googleapis.com/mysten-walrus-binaries/site-builder-mainnet-latest-$SYSTEM -o site-builder
chmod +x site-builder
```

### 测试网 curl 请求

``` sh
SYSTEM= # 将此设置为您的系统：ubuntu-x86_64, ubuntu-x86_64-generic, macos-x86_64, macos-arm64, windows-x86_64.exe
curl https://storage.googleapis.com/mysten-walrus-binaries/site-builder-testnet-latest-$SYSTEM -o site-builder
chmod +x site-builder
```

要能够简单地运行 `site-builder`，请将二进制文件移动到包含在您的 `$PATH` 环境变量中的任何目录。标准位置包括 `/usr/local/bin/`、`$HOME/bin/` 或 `$HOME/.local/bin/`。

```admonish note
站点构建器将在当前目录、`$XDG_HOME/walrus/sites-config.yaml` 和 `$HOME/walrus/sites-config.yaml` 目录中查找默认配置文件 `sites-config.yaml`。
如果您想明确使用不同的 `sites-config.yaml`，请使用 `--config` 标志指定配置文件的路径。
```

完成此操作后，您应该能够在终端中简单地输入 `site-builder`。

```terminal
$ site-builder
Usage: site-builder [OPTIONS] <COMMAND>

Commands:
  publish  Publish a new site on Sui
  update   Update an existing site
  convert  Convert an object ID in hex format to the equivalent Base36
               format
  sitemap  Show the pages composing the Walrus site at the given object ID
  help     Print this message or the help of the given subcommand(s)

  ⋮
```

## 配置

`site-builder` 工具需要配置文件才能工作。此文件名为 `sites-config.yaml`，如下所示：

```yaml
contexts:
  testnet:
    # module: site
    # portal: wal.app
    package: 0xf99aee9f21493e1590e7e5a9aea6f343a1f381031a04a732724871fc294be799
    staking_object: 0xbe46180321c30aab2f8b3501e24048377287fa708018a5b7c2792b35fe339ee3
    general:
       wallet_env: testnet
       walrus_context: testnet # 假设 Walrus CLI 设置包含 "testnet" 上下文的多配置。
       walrus_package: 0xd84704c17fc870b8764832c535aa6b11f21a95cd6f5bb38a9b07d2cf42220c66
       # wallet_address: 0x1234...
       # rpc_url: https://fullnode.testnet.sui.io:443
       # wallet: /path/to/.sui/sui_config/client.yaml
       # walrus_binary: /path/to/walrus
       # walrus_config: /path/to/testnet/client_config.yaml
       # gas_budget: 500000000
  mainnet:
    # module: site
    # portal: wal.app
    package: 0x26eb7ee8688da02c5f671679524e379f0b837a12f1d1d799f255b7eea260ad27
    staking_object: 0x10b9d30c28448939ce6c4d6c6e0ffce4a7f8a4ada8248bdad09ef8b70e4a3904
    general:
       wallet_env: mainnet
       walrus_context: mainnet # 假设 Walrus CLI 设置包含 "mainnet" 上下文的多配置。
       walrus_package: 0xfdc88f7d7cf30afab2f82e8380d11ee8f70efb90e863d1de8616fae1bb09ea77
       # wallet_address: 0x1234...
       # rpc_url: https://fullnode.mainnet.sui.io:443
       # wallet: /path/to/.sui/sui_config/client.yaml
       # walrus_binary: /path/to/walrus
       # walrus_config: /path/to/mainnet/client_config.yaml
       # gas_budget: 500000000

default_context: mainnet
```

如您所见，配置文件非常简单。您可以在此定义不同的上下文及其配置，只有包 ID 字段是必需的，代表 Walrus Sites 智能合约的 Sui 对象 ID。您可以在 `mainnet` 分支上的 [Walrus Sites 仓库](https://github.com/MystenLabs/walrus-sites/tree/mainnet)中找到包的最新版本。

```admonish danger title="Walrus Sites 稳定分支"
Walrus Sites 的稳定分支是 `mainnet`。
确保您始终从那里拉取最新更改。
```

您可以在运行 `site-builder` 命令时使用 `--config` 标志定义 `sites-config.yaml` 文件的位置，如下所示：

``` sh
site-builder --config /path/to/sites-config.yaml publish <build-directory-of-a-site>
```

但是，如果您不喜欢一遍又一遍地重复相同的标志，将配置文件放在[默认位置](./tutorial-install_zh.md#admonition-note)之一总是更容易的。

从仓库下载 `sites-config.yaml` 文件，并将其放置在上述默认位置之一。为了说明，我们将使用 `~/.config/walrus` 目录，如下所示：

```sh
curl https://raw.githubusercontent.com/MystenLabs/walrus-sites/refs/heads/mainnet/sites-config.yaml -o ~/.config/walrus/sites-config.yaml
```

您现在已准备好开始使用 Walrus Sites！🎉