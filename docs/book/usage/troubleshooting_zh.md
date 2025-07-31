# 故障排除

```admonish tip title="调试日志"
您可以通过设置环境变量 `RUST_LOG=walrus=debug` 来启用 Walrus 的调试日志。
```

## 最新二进制文件

在采取任何其他步骤之前，请确保您拥有[最新的 `walrus` 二进制文件](./setup_zh.md#installation)。如果您在不同位置有多个版本，使用 `which walrus` 查找实际将使用的二进制文件。

## 旧硬件或不兼容的虚拟机

我们的标准 Ubuntu 二进制文件已知在某些旧硬件和某些虚拟化环境中会导致问题。如果您遇到"Illegal instruction (core dumped)" 等错误，请[安装](./setup_zh.md#installation) `ubuntu-x86_64-generic` 版本，该版本专门编译以与几乎所有物理和虚拟 x86-64 CPU 兼容。

## 正确的 Sui 网络配置

如果您收到"the specified Walrus system object does not exist"等错误，请确保您的钱包设置为正确的 Sui 网络（根据您的需要选择主网或测试网），并且您使用最新的[配置](./setup_zh.md#configuration)。

## 最新的 Walrus 配置

Walrus 测试网定期清除，需要更新到最新的二进制文件和配置。如果您收到"could not retrieve enough confirmations to certify the blob"等错误，您可能正在使用过时的配置，指向不活跃的 Walrus 系统。在这种情况下，请使用最新的[配置](./setup_zh.md#configuration)更新您的配置文件，并确保 CLI 使用预期的配置。

```admonish tip
当设置 `RUST_LOG=info` 时，`walrus` 客户端二进制文件在开始执行时会打印有关使用的配置的信息，包括 Walrus 配置文件和 Sui 钱包的路径。
```