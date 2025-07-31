# CI/CD

如果您需要自动化 Walrus Sites 的部署，您可以使用 CI/CD 管道。本指南涵盖了为您的站点设置持续部署的基本步骤。

## 入门

按顺序遵循以下内容：

1. **[准备您的部署凭据](./ci-cd-gh-secrets-vars_zh.md)** - 在您的 GitHub 仓库中设置必要的密钥和变量
1. **[编写您的工作流](./ci-cd-gh-workflow_zh.md)** - 创建和配置您的 GitHub Actions 工作流文件

## 其他示例

您可以在这里找到更多 GitHub 工作流的示例：

1. [部署 Walrus 文档的工作流](https://github.com/MystenLabs/walrus/blob/main/.github/workflows/publish-docs.yaml)
1. [Sui Potatoes 部署工作流](https://github.com/sui-potatoes/app/blob/main/.github/workflows/walrus.yml)
1. [社区驱动的 Walrus Sites 出处](https://github.com/zktx-io/walrus-sites-provenance)