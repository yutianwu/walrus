# 编写您的工作流

现在您已经配置了密钥和变量，可以创建 GitHub 工作流来使用官方的"部署 Walrus Site"GitHub Action 自动化您的 Walrus Site 部署。

## 关键要求：预构建的站点目录

部署 Walrus Site action 操作**已构建的站点目录**。该 action 不构建您的站点 - 它将现有的静态文件部署到 Walrus。

这意味着：

- 如果您的站点由可立即部署的静态文件（HTML、CSS、JS）组成，您可以直接使用该 action
- 如果您的站点需要构建步骤（React、Vue、Svelte 等），您**必须**在调用部署 action 之前在工作流中包含构建步骤

## 使用部署 Walrus Site GitHub Action

该 action（`MystenLabs/walrus-sites/.github/actions/deploy`）需要以下输入：

- **`SUI_ADDRESS`**：您的 Sui 地址（GitHub 变量）
- **`SUI_KEYSTORE`**：您的 base64 格式私钥（GitHub 密钥）  
- **`DIST`**：您的**已构建**站点目录的路径

可选输入包括：

- **`SUI_NETWORK`**：目标网络（`mainnet` 或 `testnet`，默认为 `mainnet`）
- **`EPOCHS`**：保持站点存储的 epoch 数量（默认为 `5`）
- **`WALRUS_CONFIG`**：自定义 Walrus 配置（如果未提供则下载默认配置）
- **`SITES_CONFIG`**：自定义站点配置（如果未提供则下载默认配置）
- **`WS_RESOURCES`**：`ws-resources.json` 文件的完整路径（默认为 `DIST/ws-resources.json`）
- **`GITHUB_TOKEN`**：当站点资源发生变化时启用自动创建拉取请求

### 关于 `GITHUB_TOKEN`

`GITHUB_TOKEN` 输入对于跟踪站点资源的变化特别有用。当您部署站点时，site-builder 创建或更新跟踪站点数据的 `ws-resources.json` 文件。如果此文件在部署期间发生变化，该 action 可以自动创建包含更新文件的拉取请求。

要使用此功能：

1. 在工作流中设置 `GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}`
1. 将这些[权限](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token)添加到您的工作流：

   ```yaml
   permissions:
     contents: write
     pull-requests: write
   ```

```admonish note
如果您不提供 `GITHUB_TOKEN`，该 action 仍将成功部署您的站点，但不会为资源文件更改创建拉取请求。
```

### 新站点与现有站点

了解工作流的行为取决于您是否有现有站点：

**新站点（没有现有的 `object_id`）：**

- 在 Sui 上创建新的 Walrus Site。
- 在您的 `DIST` 目录中生成 `ws-resources.json` 文件（或更新由 `WS_RESOURCES` 指定的文件）。
- 此文件包含您新创建站点的 `object_id`。
- 如果提供了具有正确权限的 `GITHUB_TOKEN`，则创建包含这些更改的拉取请求。

**现有站点（有 `object_id`）：**

- 使用 Walrus-Sites 资源文件中的现有 `object_id` 来更新同一站点。
- 只有在部署期间资源文件发生变化时才创建 PR。
- 这适用于您之前是否使用 GitHub Actions 或使用 CLI 手动部署 - 只要 `object_id` 存在于您的 `WS_RESOURCES` 文件中（默认为 `DIST/ws-resources.json`）

```admonish tip
`ws-resources.json` 文件至关重要 - 它告诉 action 要更新哪个站点。确保合并第一次运行的 PR，以便将来的部署更新同一站点而不是创建新站点。
```

```admonish tip
一旦您的站点部署并拥有 `object_id`，您可以将其与 SuiNS 名称链接，使您的站点可在 `<suins>.wal.app` 访问。请参阅[设置 SuiNS 名称](./tutorial-suins_zh.md)了解如何设置的详细信息。
```

## 创建您的工作流

1. 在您的仓库中创建 `.github/workflows/deploy-site.yml`
1. 添加检出步骤以获取您的仓库文件
1. 如果您的站点需要构建，添加构建步骤：
   - 添加构建环境设置（Node.js 等）
   - 添加构建命令
1. 设置您的 `DIST` 路径：
   - 对于需要构建的站点：指向您的构建输出目录（例如，`dist/`、`build/`）
   - 对于静态站点：直接指向您的静态文件目录
1. 使用您配置的密钥和变量添加部署 Walrus Site action

关键是确保您的 `DIST` 路径指向包含应该发布到 Walrus 的最终可部署静态文件的目录。

## 示例工作流

- [deploy-snake.yml](https://github.com/MystenLabs/walrus-sites/blob/main/.github/workflows/deploy-snake.yml) - 简单静态站点部署
- [deploy-vite-react-ts.yml](https://github.com/MystenLabs/walrus-sites/blob/main/.github/workflows/deploy-vite-react-ts.yml) - 带构建步骤的 React 应用程序

---

有关创建和配置 GitHub 工作流的更多信息，请查看[官方 GitHub Actions 文档](https://docs.github.com/en/actions/writing-workflows)。