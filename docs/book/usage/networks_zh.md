# 可用网络

Walrus 主网在 Sui 主网上运行生产质量的存储网络。Walrus 测试网在 Sui 测试网上运行，用于在新功能升级到主网之前测试 Walrus 的新功能。最后，开发人员可以运行本地 Walrus 和 Sui 网络进行测试。

## 网络参数

主网和测试网的一些重要固定系统参数汇总在下表中：

| 参数                                           | 主网     | 测试网   |
|------------------------------------------------|----------|----------|
| Sui 网络                                       | 主网     | 测试网   |
| 分片数量                                       | 1000     | 1000     |
| 纪元持续时间                                   | 2 周     | 1 天     |
| 可购买存储的最大纪元数                         | 53       | 53       |

许多其他参数，包括系统容量和价格，都是动态的。这些存储在系统对象中（请参见[关于系统和质押对象的文档](../dev-guide/sui-struct_zh.md#system-and-staking-information)），可以使用各种工具（如 [Walruscan 浏览器](https://walruscan.com/)）查看。

## 主网配置

### 主网参数

Walrus 主网的客户端参数为：

```yaml
{{ #include ../setup/client_config_mainnet.yaml }}
```

如果您希望探索 Walrus 合约，它们的包 ID 如下：

- WAL 包：`0x356a26eb9e012a68958082340d4c4116e7f55615cf27affcff209cf0ae544f59`
- Walrus 包：`0xfdc88f7d7cf30afab2f82e8380d11ee8f70efb90e863d1de8616fae1bb09ea77`
- 补贴包：`0xd843c37d213ea683ec3519abe4646fd618f52d7fce1c4e9875a4144d53e21ebc`

由于这些是从上述对象 ID 自动推断的，因此无需手动将它们输入到 Walrus 客户端配置文件中。最新发布的包 ID 也可以在 [GitHub 上 `contracts` 目录](https://github.com/MystenLabs/walrus/tree/main/contracts)子目录中的 `Move.lock` 文件中找到。

[设置页面](./setup_zh.md#configuration)上描述的配置文件包括主网和测试网配置。如果您*只*想要主网配置，可以下载[仅主网配置文件](../setup/client_config_mainnet.yaml)。

## 测试网配置

```admonish danger title="关于 Walrus 测试网的免责声明"
所有交易都在 Sui 测试网上执行，使用没有价值的测试网 WAL 和 SUI。存储的状态**可以且将会**在任何时候被清除，可能没有任何警告。不要依赖此测试网用于任何生产目的，它没有可用性或持久性保证。

测试网上的新功能可能会破坏已部署的测试网应用程序，因为这是我们测试新更新的网络，包括与生态系统应用程序的兼容性。

另请参阅提供此测试网的[测试网服务条款](../legal/testnet_tos_zh.md)。
```

### 先决条件：Sui 钱包和测试网 SUI {#prerequisites}

与 Walrus 交互需要有效的 Sui 测试网钱包和一定数量的 SUI 代币。设置这个的正常方式是通过 Sui CLI；请参见 Sui 文档中的[安装说明](https://docs.sui.io/guides/developer/getting-started/sui-install)。如果您不想安装 Sui CLI，也可以使用 `walrus generate-sui-wallet --network testnet` 为测试网生成新的 Sui 钱包。

安装 Sui CLI 后，您需要通过运行 `sui client` 设置测试网钱包，这会提示您设置新配置。确保将其指向 Sui 测试网，您可以使用 `https://fullnode.testnet.sui.io:443` 的全节点。更多详细信息请参见 [Sui 文档](https://docs.sui.io/guides/developer/getting-started/connect)。

如果您已经配置了 Sui 钱包，可以直接设置测试网环境（如果您还没有）：

```sh
sui client new-env --alias testnet --rpc https://fullnode.testnet.sui.io:443
```

并将活动环境切换到它：

```sh
sui client switch --env testnet
```

之后，您应该得到类似这样的结果（除了 `testnet` 行外，其他都是可选的）：

```terminal
$ sui client envs
╭──────────┬─────────────────────────────────────┬────────╮
│ alias    │ url                                 │ active │
├──────────┼─────────────────────────────────────┼────────┤
│ devnet   │ https://fullnode.devnet.sui.io:443  │        │
│ localnet │ http://127.0.0.1:9000               │        │
│ testnet  │ https://fullnode.testnet.sui.io:443 │ *      │
│ mainnet  │ https://fullnode.mainnet.sui.io:443 │        │
╰──────────┴─────────────────────────────────────┴────────╯
```

最后，确保您至少有一个至少 1 SUI 的 gas 代币。您可以从 [Sui 测试网水龙头](https://faucet.sui.io/?network=testnet)获得一个（您可以通过 `sui client active-address` 命令找到您的地址）。

几秒钟后，您应该看到您的新 SUI 代币：

```terminal
$ sui client gas
╭─────────────────┬────────────────────┬──────────────────╮
│ gasCoinId       │ mistBalance (MIST) │ suiBalance (SUI) │
├─────────────────┼────────────────────┼──────────────────┤
│ 0x65dca966dc... │ 1000000000         │ 1.00             │
╰─────────────────┴────────────────────┴──────────────────╯
```

如果没有指定其他路径，Walrus 将使用系统范围的钱包。如果您想使用不同的 Sui 钱包，可以在 [Walrus 配置文件](./setup_zh.md#configuration)中指定或在[运行 CLI](./interacting_zh.md) 时指定。

### 测试网参数

Walrus 测试网的配置参数包含在[设置页面](./setup_zh.md#configuration)上描述的配置文件中。如果您*只*想要测试网配置，可以获取[仅测试网配置文件](../setup/client_config_testnet.yaml)。参数为：

```yaml
{{ #include ../setup/client_config_testnet.yaml }}
```

当前的测试网包 ID 可以在 [GitHub 上 `testnet-contracts` 目录](https://github.com/MystenLabs/walrus/tree/main/testnet-contracts)子目录中的 `Move.lock` 文件中找到。

### 测试网 WAL 水龙头

Walrus 测试网使用测试网 WAL 代币购买存储和质押。测试网 WAL 代币没有价值，可以通过以下命令以 1:1 的汇率兑换一些同样没有价值的测试网 SUI 代币：

```sh
walrus get-wal
```

您可以通过检查 Sui 余额来确认您已收到测试网 WAL：

```sh
$ sui client balance
╭─────────────────────────────────────────╮
│ Balance of coins owned by this address  │
├─────────────────────────────────────────┤
│ ╭─────────────────────────────────────╮ │
│ │ coin  balance (raw)     balance     │ │
│ ├─────────────────────────────────────┤ │
│ │ Sui   8869252670        8.86 SUI    │ │
│ │ WAL   500000000         0.50 WAL    │ │
│ ╰─────────────────────────────────────╯ │
╰─────────────────────────────────────────╯
```

默认情况下，0.5 SUI 兑换 0.5 WAL，但可以使用 `--amount` 选项兑换不同数量的 SUI（值以 MIST/FROST 为单位），并且可以通过 `--exchange-id` 选项使用特定的 SUI/WAL 兑换对象。`walrus get-wal --help` 命令提供有关这些的更多信息。

## 运行本地 Walrus 网络

除了公开部署的 Walrus 网络外，您还可以在本地机器上部署 Walrus 测试床进行本地测试。您需要做的就是运行在 Walrus [git 代码存储库](https://github.com/MystenLabs/walrus)中找到的脚本 `scripts/local-testbed.sh`。更多使用信息请参见 `scripts/local-testbed.sh -h`。

该脚本生成您在[运行 Walrus 客户端](./client-cli_zh.md)时可以使用的配置，并打印该配置文件的路径。

此外，可以启动本地 grafana 实例来可视化存储节点收集的指标。这可以通过 `cd docker/grafana-local; docker compose up` 完成。这应该与默认存储节点配置一起工作。

请注意，虽然此测试床的 Walrus 存储节点在您的本地机器上运行，但默认使用 Sui Devnet 来部署和与合约交互。要完全本地运行测试床，只需[使用 `sui start --with-faucet --force-regenesis` 启动本地网络](https://docs.sui.io/guides/developer/getting-started/local-network)（需要 `sui` 为 `v1.28.0` 或更高版本）并在启动 Walrus 测试床时指定 `localnet`。