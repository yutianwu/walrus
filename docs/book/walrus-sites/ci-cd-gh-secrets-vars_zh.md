# 准备您的部署凭据

要允许 GitHub Action 部署您的 Walrus Site，它需要能够代表您签署交易。这需要安全地为其提供您的私钥和相应的公共地址。

本指南将向您展示如何：

1. 从您的 Sui 钱包或 CLI 导出私钥。
1. 正确格式化密钥并将其作为 `SUI_KEYSTORE` 密钥添加到您的 GitHub 仓库中。
1. 将匹配的公共地址作为 `SUI_ADDRESS` 变量添加到您的 GitHub 仓库中。

## 先决条件

在开始之前，您必须安装 `sui` 二进制文件。如果您尚未安装，请遵循官方 [Sui 安装指南](https://docs.sui.io/guides/developer/getting-started/sui-install)。

## 导出您的私钥

> **最佳实践**：建议为每个 GitHub 工作流使用专用的 Sui 地址，而不是在不同项目或用途间重复使用地址。这提供了更好的安全隔离，并有助于避免多个工作流尝试同时使用相同 gas 币时可能发生的 [gas 币等价](https://docs.sui.io/guides/developer/sui-101/avoid-equivocation)问题。

---

{{#tabs }}
{{#tab name="从 Sui CLI" }}

要使用命令行导出私钥：

1. 在终端中运行以下命令生成新密钥：

   ```sh
   sui keytool generate ed25519 # 或 secp256k1 或 secp256r1
   ```

1. 此命令在您的当前目录中创建一个名为 `<SUI_ADDRESS>.key`（例如，`0x123...abc.key`）的文件。文件名是您的新 Sui 地址。
1. 此文件的内容是 `base64WithFlag` 格式的私钥。这是您需要用于 `SUI_KEYSTORE` 密钥的值。
1. 您现在拥有用于 `SUI_ADDRESS` 变量的地址（来自文件名）和用于 `SUI_KEYSTORE` 密钥的密钥（来自文件内容）。

> **关于现有密钥的注意事项**
> 如果您希望使用已拥有的密钥，可以在 `~/.sui/sui_config/sui.keystore` 文件中找到它。此文件包含所有密钥的 JSON 数组。要找到特定密钥的地址，您需要使用 `sui keytool unpack "<来自 sui.keystore 的 base64 密钥>"` 命令。

{{#endtab }}
{{#tab name="从 Slush 钱包" }}

如果您通过 Slush 浏览器扩展管理密钥，建议使用此方法。

1. 打开您的 Slush 扩展并选择您想要用于部署的账户。确保复制相应的 Sui 地址，因为您稍后将需要它用于 `SUI_ADDRESS` 变量。
1. 导航到账户管理屏幕并选择**出私钥**。
1. 复制提供的私钥（它将是 bech32 格式，以 `suiprivkey` 开头）。
1. 使用 `sui keytool convert <suiprivkey...>` 命令将您的密钥转换为所需的 Base64 格式。将您复制的密钥粘贴到 `suiprivkey...` 的位置：

   ```sh
   sui keytool convert `suiprivkey...`
   ```

1. 该命令将产生类似以下的输出：

   ```text
   ╭────────────────┬──────────────────────────────────────────────────────────────────────────╮
   │ bech32WithFlag │  suiprivkey............................................................  │
   │ base64WithFlag │  A...........................................                            │
   │ hexWithoutFlag │  ................................................................        │
   │ scheme         │  ed25519                                                                 │
   ╰────────────────┴──────────────────────────────────────────────────────────────────────────╯
   ```

   复制 **`base64WithFlag`** 值。这是您将用于 `SUI_KEYSTORE` 密钥的内容。

{{#endtab }}
{{#endtabs }}

---

## 为您的地址充值

在 GitHub Action 可以部署您的站点之前，您生成的地址需要用 SUI 代币（用于网络 gas 费用）和 WAL 代币（用于存储站点数据）充值。获取这些代币的方法在测试网和主网之间有所不同。

---

{{#tabs }}
{{#tab name="测试网充值" }}

1. **获取 SUI 代币**：使用[官方 Sui 水龙头](https://faucet.sui.io/)获取免费的测试网 SUI。
1. **获取 WAL 代币**：通过运行 `walrus get-wal` CLI 命令或访问 [stake-wal.wal.app](https://stake-wal.wal.app/?network=testnet) 将网络设置为测试网并使用"获取 WAL"按钮，以 1:1 的汇率将您的新测试网 SUI 兑换为测试网 WAL。

{{#endtab }}
{{#tab name="主网充值" }}

对于主网部署，您需要从交易所获取 SUI 和 WAL 代币并将它们转移到您的部署地址。您还可以查看 Slush 钱包的代币交换到 WAL 和入金服务（可用性可能因地区而异）。

{{#endtab }}
{{#endtabs }}

---

## 将凭据添加到 GitHub

现在，让我们将密钥和地址添加到您的 GitHub 仓库。

1. 在 Web 浏览器中导航到您的 GitHub 仓库。
1. 单击**设置**选项卡（位于仓库的顶部导航栏中）。
1. 在左侧边栏中，单击**密钥和变量**，然后选择**Actions**。
1. 您将看到两个选项卡：**密钥**和**变量**。从**密钥**选项卡开始。
1. 单击**新建仓库密钥**按钮。
1. 将密钥命名为 `SUI_KEYSTORE`。
1. 在**值**字段中，粘贴您之前复制的 `Base64 Key with Flag`。它必须格式化为包含单个字符串的 JSON 数组：

   ```json
   ["AXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"]
   ```

1. 单击**添加密钥**保存它。

   ```admonish warning
   确保将密钥库格式化为具有单个字符串元素的 JSON 数组，而不仅仅是原始密钥值。完全按照上面显示的包含方括号和引号。
   ```

1. 接下来，切换到**变量**选项卡并单击**新建仓库变量**。
1. 将变量命名为 `SUI_ADDRESS`。
1. 在**值**字段中，粘贴与您的私钥对应的 Sui 地址（例如：`0x123abc...def789`）。
1. 单击**添加变量**保存它。

```admonish danger title="安全提醒"
永远不要分享您的私钥或将其提交到版本控制。GitHub 密钥已加密，只有您的工作流可以访问，但始终验证您是否正确添加了密钥。
```

---

有关在 GitHub Actions 中管理密钥和变量的更多信息，请查看官方 GitHub 文档：

- [关于密钥](https://docs.github.com/en/actions/concepts/security/about-secrets)
- [在 GitHub Actions 中使用密钥](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)