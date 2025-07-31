# 佣金和治理

虽然大多数节点参数可以使用 StorageNodeCap 更改，并根据节点配置中的值自动更新，但合约升级和提取存储节点佣金的授权是单独处理的，以确保存储节点上的热钱包不需要被授权进行这些操作。

我们强烈建议所有节点操作员为这些操作指定安全钱包，这些钱包不存储在存储节点机器上。

本页解释如何更新被授权进行治理（即合约升级）和佣金操作的实体，以及如何执行这些操作。

## 更新佣金接收者和被授权进行治理的实体

可以指定任意对象作为治理/佣金操作的能力，或者指定地址被授权。

```admonish warning
一旦设置，只有被授权的实体（基于地址或能力）可以再次更改授权，所以只将其设置为您控制的地址/对象，并确保它们保持可访问。
```

要设置接收佣金和执行治理操作的授权，您可以在 CLI 中使用 `walrus` 二进制文件，或者您可以使用[节点操作 Web 界面](https://stake-wal.wal.app/node-operations)。

### 使用 CLI

以下假设您已经[正确设置](../usage/setup_zh.md)了 `walrus` 二进制文件，使用当前被授权执行这些操作的钱包。如果这是第一次更新被授权的实体，这将是您用来设置存储节点的钱包。要指定不在标准位置的钱包和/或配置，您可以使用 `--wallet` 和 `--config` 命令行参数指定它们。

请注意，佣金和治理的被授权实体是独立的，即它们不需要设置为相同的地址或对象。

```sh
NODE_ID=            # 将此设置为您的节点 ID。
COMMISSION_AUTHORIZED_ADDRESS= # 将此设置为您控制的安全钱包地址。
GOVERNANCE_AUTHORIZED_ADDRESS= # 将此设置为您控制的安全钱包地址。
walrus node-admin set-commission-authorized --node-id $NODE_ID --address $COMMISSION_AUTHORIZED_ADDRESS
walrus node-admin set-governance-authorized --node-id $NODE_ID --address $GOVERNANCE_AUTHORIZED_ADDRESS
```

除了使用 `--address` 标志指定被授权地址外，还可以使用 `--object` 标志将任意对象指定为能力：

```sh
NODE_ID=           # 将此设置为您的节点 ID。
COMMISSION_AUTHORIZED_OBJECT= # 将此设置为您在安全钱包中拥有的对象的 ID。
GOVERNANCE_AUTHORIZED_OBJECT= # 将此设置为您在安全钱包中拥有的对象的 ID。
walrus node-admin set-commission-authorized --node-id $NODE_ID --object $COMMISSION_AUTHORIZED_OBJECT
walrus node-admin set-governance-authorized --node-id $NODE_ID --object $GOVERNANCE_AUTHORIZED_OBJECT
```

### 使用 Web 界面

转到[质押 dApp 上的操作员面板](https://stake-wal.wal.app/node-operations)，连接您的钱包，通过下拉菜单选择您的节点或粘贴您的节点 ID。然后选择 `Set Commission Receiver` 或 `Set Governance Authorized` 并按照步骤发送交易。

## 收取佣金

要收取您的佣金，您可以再次使用 CLI 或 Web 界面。要使用 CLI，请确保 `walrus` 配置为使用被授权的钱包并运行以下命令：

```sh
NODE_ID=           # 将此设置为您的节点 ID。
walrus node-admin --node-id $NODE_ID collect-commission
```

要使用 Web 界面，转到[质押 dApp 上的操作员面板](https://stake-wal.wal.app/node-operations)，连接您的钱包，通过下拉菜单选择您的节点或粘贴您的节点 ID。然后选择 `Collect Commission` 并按照步骤发送交易。

## 合约升级

Walrus 中的合约升级通过基于法定人数的投票系统管理。这确保升级只有在节点操作员中有足够共识后才被应用。该过程需要计算提议升级的摘要并对其进行投票。

### 为基于法定人数的升级投票

当提议合约升级时，提议者通常会与其他节点操作员分享提议升级的代码。例如，如果 Walrus 基金会提议升级，它将在 [Walrus GitHub 仓库](https://github.com/MystenLabs/walrus)中分享包含提议更改的分支中的特定提交哈希，更改位于 `testnet-contracts/walrus` 或 `mainnet-contracts/walrus` 目录中，这取决于是否正在升级测试网或主网合约。

投票需要在一个纪元内完成，否则委员会会更改，投票需要重复。要为升级投票，请完成以下步骤。

#### 计算升级摘要

操作员应计算要升级的包的包摘要。这里重要的是使用相同的编译器版本并指定正确的 Sui 网络。如果您使用标准的 [Walrus 配置](../usage/setup_zh.html#configuration)，当使用 `--context` 标志指定 Walrus 网络时，Sui 网络将自动选择，使用最新的 `walrus` 版本将确保编译器版本在所有投票者中一致。

要计算提议升级的摘要，您应该使用 `walrus node-admin` 命令。假设您当前的工作目录是 Walrus 仓库的根目录，并且您已经检出了正确的提交，您将使用以下命令进行测试网升级：

```sh
walrus node-admin package-digest --package-path testnet-contracts/walrus --context testnet
```

以及以下用于主网升级：

```sh
walrus node-admin package-digest --package-path mainnet-contracts/walrus --context mainnet
```

该命令将输出一个摘要（作为十六进制和 Base64），您可以使用它来验证提议并为升级投票。

#### 使用 Web 界面为升级投票

使用 Web 界面为升级投票是最简单的方式，也允许您使用浏览器支持的任何钱包（例如硬件钱包）。要通过 Web 界面投票：

1. 转到[质押 dApp 上的操作员面板](https://stake-wal.wal.app/node-operations)。
1. 连接您的钱包并选择您的节点。
1. 导航到"Vote on Upgrade"部分。
1. 粘贴上一步的 Base64 包摘要。
1. 按照提示提交您的投票。

#### 使用 CLI 为升级投票

要使用 CLI 为升级投票，请确保您的 `walrus` 二进制文件配置了被授权的钱包，并且您在 Walrus 仓库根目录的正确分支上。然后为测试网运行以下命令：

```sh
NODE_ID=   # 将此设置为您的节点 ID。
PACKAGE_PATH=testnet-contracts/walrus
UPGRADE_MANAGER_OBJECT_ID=0xc768e475fd1527b7739884d7c3a3d1bc09ae422dfdba6b9ae94c1f128297283c
walrus node-admin vote-for-upgrade \
    --node-id $NODE_ID \
    --upgrade-manager-object-id $UPGRADE_MANAGER_OBJECT_ID \
    --package-path $PACKAGE_PATH \
    --context testnet
```

以及主网升级：

```sh
NODE_ID=   # 将此设置为您的节点 ID。
PACKAGE_PATH=mainnet-contracts/walrus
UPGRADE_MANAGER_OBJECT_ID=0xc42868ad4861f22bd1bcd886ae1858d5c007458f647a49e502d44da8bbd17b51
walrus node-admin vote-for-upgrade \
    --node-id $NODE_ID \
    --upgrade-manager-object-id $UPGRADE_MANAGER_OBJECT_ID \
    --package-path $PACKAGE_PATH \
    --context mainnet
```

#### 升级完成

一旦法定人数的节点操作员（按分片数量）已经投票赞成提议，合约升级就可以最终确定。这通常由提议者使用调用相应函数来授权、执行和提交升级的 PTB 完成。然后，根据升级的不同，在下一个纪元开始时或立即通过调用 `walrus` 包的 `init` 模块中的 `migrate` 函数，将系统和质押对象迁移到新版本。升级和迁移都可以使用 `walrus-deploy` 二进制文件执行。