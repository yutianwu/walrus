# Walrus Sites 门户

我们使用术语"门户"来指代任何用于访问和浏览 Walrus Sites 的技术。
如[概述](./overview_zh.md#站点渲染路径)中所述，我们预见三种类型的门户：

1. 服务器端门户；
1. 自定义本地应用程序；以及
1. 在浏览器中运行的基于 service-worker 的门户。

目前，只有一个服务器端门户在 <https://wal.app> 提供服务。

```admonish warning
我们正在停用测试网门户！从现在开始，您只能访问位于 <https://wal.app> 的主网门户。
```

```admonish note title="Service Worker 的托管"
Service-worker 门户不再托管，但您仍然可以在本地运行它。其代码在 `walrus-sites` 仓库中可用。有关更多信息，请参阅[运行本地门户](#运行本地门户)。
```

```admonish danger title="Walrus Sites 稳定分支"
Walrus Sites 的稳定分支是 `mainnet`。
```

## 运行本地门户

在本地运行门户可能很有用，例如，如果您想在不访问外部门户的情况下浏览 Walrus Sites，或用于开发目的。

让我们首先克隆 `walrus-sites` 仓库：

```bash
git clone https://github.com/MystenLabs/walrus-sites.git
cd walrus-sites
```

确保您在稳定分支上：

``` sh
git checkout mainnet
```

接下来，我们将看到如何配置门户以支持我们需要的功能。

### 配置

门户配置通过两个关键元素管理：

- 环境变量：基本功能所必需的。
- 常量文件：高级自定义的可选项。

#### 环境变量

环境变量在每个门户目录根目录的 `.env.local` 文件中设置。
要运行一个简单的门户实例，您可以使用 `.env.<network>.example` 文件中指定的环境变量：

##### 主网服务器门户环境变量

```sh
cp ./portal/server/.env.mainnet.example ./portal/server/.env.local
```

##### 测试网服务器门户环境变量

```sh
cp ./portal/server/.env.testnet.example ./portal/server/.env.local
```

同样，如果您想运行 service-worker 门户，可以将 `.env.<network>.example` 文件复制到 `portal/worker` 目录中的 `.env.local`。

##### 主网 Service Worker 门户环境变量

```sh
cp ./portal/worker/.env.mainnet.example ./portal/worker/.env.local
```

##### 测试网 Service Worker 门户环境变量

```sh
cp ./portal/worker/.env.testnet.example ./portal/worker/.env.local
```

要进行更详细的配置，您可以修改 `.env.local` 文件以满足您的需求。
作为参考，以下是环境变量的定义：

```admonish note
服务器门户代码包含可以使用环境变量启用或禁用的额外功能。例如，您可以通过将 `ENABLE_BLOCKLIST` 变量设置为 `true` 或 `false` 来启用或禁用阻止列表功能。这有助于管理门户的行为。例如，如果您托管一个公开访问的门户，您可能希望避免提供不是由您发布的站点。
```

- `AGGREGATOR_URL`：将从 Walrus 获取站点资源的 Walrus 聚合器的 URL。

- `AMPLITUDE_API_KEY`：如果您想为服务器分析启用 [Amplitude](https://amplitude.com/)，请提供它。

- `EDGE_CONFIG`：如果您在 Vercel 上托管门户，可以使用 [Edge Config][edge-config] 来阻止某些 SuiNS 子域名或 b36 对象 ID。

- `EDGE_CONFIG_ALLOWLIST`：类似于阻止列表，但允许某些子域名使用高级 RPC URL 列表。

- `ENABLE_ALLOWLIST`：启用允许列表功能。

- `ENABLE_BLOCKLIST`：启用阻止列表功能。

- `ENABLE_SENTRY`：启用 Sentry 错误跟踪。

- `ENABLE_VERCEL_WEB_ANALYTICS`：启用 Vercel web 分析。

- `LANDING_PAGE_OID_B36`：着陆页 Walrus Site 的 b36 对象 ID。即您访问 `localhost:3000` 时得到的页面。

- `PORTAL_DOMAIN_NAME_LENGTH`：如果您将门户连接到域名，请指定域名的长度。例如 `example.com` 的长度为 11。

- `PREMIUM_RPC_URL_LIST`：当站点属于允许列表时使用的 RPC URL 列表。

- `RPC_URL_LIST`：当站点不属于允许列表时使用的 RPC URL 列表。

- `SENTRY_AUTH_TOKEN`：如果您启用 Sentry 错误跟踪，请提供您的 Sentry 身份验证令牌。

- `SENTRY_DSN`：如果您启用 Sentry 错误跟踪，请提供您的 Sentry DSN。

- `SENTRY_TRACES_SAMPLE_RATE`：如果您启用 Sentry 错误跟踪，请提供跟踪的采样率。

- `SITE_PACKAGE`：Walrus Site 包 ID。根据您使用的网络，您必须指定正确的包 ID。

- `SUINS_CLIENT_NETWORK`：SuiNS 客户端的网络。

- `B36_DOMAIN_RESOLUTION_SUPPORT`：定义是否支持 b36 域名解析。否则站点将不会被提供服务。

#### 常量

您可以在 `portal/common/lib` 目录中找到 `constants.ts` 文件。它保存了门户的关键配置参数。通常，您不需要修改这些，但如果需要，以下是每个参数的解释：

- `MAX_REDIRECT_DEPTH`：门户在停止之前将遵循的[重定向](./redirects_zh.md)次数。
- `SITE_NAMES`：硬编码的 `name: objectID` 映射，用于覆盖 SuiNS 名称。仅用于开发。使用时风险自负，可能使某些具有合法 SuiNS 名称的站点无法使用。

- `FALLBACK_PORTAL`：这仅与 service worker 门户相关。后备门户应该是在某些浏览器不支持 service worker 的情况下使用的服务器端门户。

### 部署门户

要在本地运行门户，您可以使用 Docker 容器或本地开发环境。

您可以通过 Docker 运行门户以进行快速设置，或者如果您想修改代码或为项目做出贡献，可以使用本地开发环境。

#### Docker

首先，确保您的系统上安装了 Docker。

```sh
docker --version
```

如果未安装，请按照 [Docker 网站][get-docker] 上的说明操作。

然后，使用以下命令构建 Docker 镜像：

```sh
docker build -f portal/docker/server/Dockerfile -t server-portal . --build-arg ENABLE_SENTRY=false --no-cache
```

最后，运行 Docker 容器：

```sh
docker run -p 3000:3000 server-portal --env-file ./portal/server/.env.local
```

在 `localhost:3000` 浏览站点。

#### 本地开发

这需要安装 [`bun`](https://bun.sh/) 工具：

检查是否安装了 bun：

``` sh
bun --version
```

如果未安装，运行以下命令：

```sh
curl -fsSL https://bun.sh/install | bash
```

安装依赖项：

```sh
cd portal
bun install
```

要运行服务器端门户：

```sh
bun run server
```

要运行 service-worker 门户：

```sh
bun build:worker
bun run worker
```

要提供其中一个门户的服务。通常，您会发现它在 `localhost:3000`（对于服务器端门户）或 `localhost:8080`（对于 service worker）提供服务（但请查看 serve 命令的输出）。

## 下一步

作为下一步，您可以为门户带来您自己的域名。如果您想使用 Walrus Sites 在自定义域名下托管您的站点，这很有用。在[下一节][own-domain]中了解如何执行此操作。

[get-docker]: https://docs.docker.com/get-docker/
[edge-config]: https://vercel.com/docs/edge-config
[own-domain]: ./bring-your-own-domain_zh.md