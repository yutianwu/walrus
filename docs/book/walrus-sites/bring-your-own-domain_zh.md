# 使用自己的域名

在上一节中，我们学习了如何[部署 Walrus Sites 门户](./portal_zh.md)。现在，您可能希望在特定域名下拥有您的 Walrus Site，而不遵循 `https://<walrus-site-domain>.<portal-domain>` 的默认命名约定。

例如，您可能想要使用像 `https://example.com` 这样的域名，而不是 `https://example.wal.app`，其中 `example.com` 是您可以从任何域名注册商购买的经典 DNS 域名。它将指向您门户的 IP 和端口。

最后，您需要配置您的门户，使其只接受针对您自己站点的请求，其他子域名（即其他 Walrus Sites）将不会被提供服务。

您需要遵循的步骤是：

1. [部署本地门户](./portal_zh.md)并在您的 `.env.local` 中包含这些额外的环境变量：

    ```bash
    LANDING_PAGE_OID_B36=34kgqayivjvwuwtk6lckk792g4qhpnimt58eol8i0b2tdgb0y # 示例 b36 ID
    BRING_YOUR_OWN_DOMAIN=true
    ```

    `LANDING_PAGE_OID_B36` 应该是您的 Walrus Site 的 b36 对象 ID。

    将 `BRING_YOUR_OWN_DOMAIN` 设置为 true，以确保门户只为您的域名提供服务，而不为其他站点提供服务。即 `https://<some-site>.example.com` 应该抛出 404 错误。

1. 配置您的 DNS 提供商，使您的域名指向您门户的 IP 和端口。

就是这样！现在您在自己的域名下拥有了一个 Walrus Site。