# 认证发布器

我们现在描述认证发布器，它要求存储 blob 的 HTTP 请求必须经过认证。这样的认证发布器可以用作在 Walrus `mainnet` 上需要通过 HTTP 存储的服务的构建块，在这种情况下，"开放"发布器是不可取的（因为发布到 Walrus 的 `SUI` 和 `WAL` 成本）。

## 概述

Walrus 发布器可以配置为要求每个 HTTP 请求都带有 JWT（JSON Web Token）进行用户认证。认证系统确保只有授权的客户端可以存储 blob，并允许通过 JWT 声明对存储参数进行细粒度控制。

认证发布流程在高层次上如下进行：

1. **发布器设置**：发布器操作员：
    - 为发布器的钱包充值足够的 `SUI` 和 `WAL`；
    - 配置发布器只接受认证请求。这需要设置认证 JWT 的算法、JWT 的过期时间和 JWT 认证密钥。
1. **认证通道设置**：发布器操作员设置一个通道，用户可以通过该通道获取 JWT 令牌。此步骤可以通过任何生成有效 JWT 的方式执行，本实现中未提供。
1. **客户端认证**：客户端从上一步设置的通道获取 JWT 令牌。JWT 令牌可以指定与 Walrus 相关的约束，例如 JWT 可用于存储的最大纪元数和要存储的 blob 的最大大小。
1. **发布请求**：客户端使用发布器请求存储 blob。这通过包含在 Authorization Bearer HTTP 标头中的 JWT 的 HTTP PUT 请求完成。
1. **存储到 Walrus**：发布器检查 JWT，并检查存储请求是否符合 JWT 中指定的约束（例如，要存储的 blob 小于授权的最大大小）。
1. （可选）**资产返回**：如果指定，发布器将新创建的 `Blob` 对象返回到请求中设置的 Sui 地址。

### 请求认证流程

我们现在提供一个认证发布器如何在允许用户通过 Web 前端将文件上传到 Walrus 的 Web 应用程序中使用的示例。这在用户*不*需要拥有钱包，因此无法自己将 blob 存储到 Walrus 的情况下特别有用。

- 用户连接到 Web 应用程序，进行认证（例如，通过用户名和密码，因为他们没有钱包）。
- 用户上传要存储的文件，并选择要存储的纪元数。
- Web 应用程序前端将文件大小和纪元数的信息发送到后端。
- 根据此信息，后端计算用户是否被授权存储给定数量的数据达到选定的纪元数，并可能减少用户配额。此记账可以在本地完成，也可以直接在 Sui 上完成。重要的是，此时后端还可以检查上传的成本（以 SUI 和 WAL 计），并检查是否在用户的预算范围内。
- 如果用户被授权存储，后端向前端返回一个带有大小和纪元限制的 JWT 令牌。
- 前端通过 `PUT` 请求将文件发送到发布器，将 JWT 作为 Bearer 令牌设置在 Authorization 标头中。
- 收到后，发布器验证：
  - 使用配置的密钥验证令牌签名；
  - 令牌过期；
  - 令牌唯一性（防止重放攻击）；
  - 根据声明验证上传参数（如果启用）。
- 如果所有检查都成功，发布器将文件存储在 Walrus 上。
- 如果设置，发布器最终将创建的 `Blob` 对象返回到用户指定的地址。

这里一个重要的注意事项是，JWT 令牌本身是"一次性"令牌，不应用于记账目的。即，一个令牌只能用于存储一个文件，之后令牌会被重放抑制系统拒绝。

## 发布器设置

发布器在启动时使用以下命令行参数进行配置：

- `--jwt-decode-secret`：用于验证 JWT 签名的密钥。如果设置，发布器将只存储带有有效 JWT 的 blob。
- `--jwt-algorithm`：用于 JWT 验证的算法（默认为 HMAC）。
- `--jwt-expiring-sec`：JWT 被认为过期的持续时间（秒）。
- `--jwt-verify-upload`：启用根据 JWT 声明验证上传参数。

更多详细信息如下。

### JWT 解码密钥

密钥可以是十六进制字符串，以 `0x` 开头。如果未指定此参数，认证将被禁用。

所有 JWT 令牌都应在声明中将 `jti`（JWT ID）设置为唯一值。JWT 用于重放抑制，即避免恶意用户使用相同的 JWT 多次存储。因此，JWT 创建者必须确保此值在对发布器的所有请求中是唯一的。我们建议使用大的随机数以避免冲突。

### 认证算法

支持以下算法："HS256"、"HS384"、"HS512"、"ES256"、"ES384"、"RS256"、"RS384"、"PS256"、"PS384"、"PS512"、"RS512"、"EdDSA"。默认的 JWT 认证算法将是 HS256。

### JWT 过期

如果参数设置且大于 0，发布器将根据 JWT 令牌中的"发行时间"（`iat`）值检查 JWT 令牌是否过期。

### 上传参数验证

如果设置，发布器将验证请求的上传是否与 JWT 中的声明匹配。这*不会*启用或禁用 JWT 的加密认证；它只是启用或禁用确保 JWT 声明内容与请求的 blob 上传匹配的检查。

具体来说，发布器将：

- 验证查询中的 `epochs` 数与 JWT 中的 `epochs`（如果存在）相同；
- 验证查询中的 `send_object_to` 字段与 JWT 中的 `send_object_to`（如果存在）相同；
- 验证上传文件的大小；
- 验证 `jti` 声明的唯一性。

如果存储请求的来源可信，但发布器可能被不可信的来源联系，则禁用参数验证可能很有用。在这种情况下，JWT 的认证是必要的，但不需要验证上传参数。

### 重放抑制配置

如上所述，发布器支持重放抑制以避免恶意重用 JWT 令牌。

重放抑制支持以下配置参数：

- `--jwt-cache-size`：发布器的 JWT 缓存的最大大小，其中"已使用"JWT 的 `jti` JWT ID 保存到其过期为止。这是缓存中条目数的硬上限，超过此限制后，额外的存储请求将被拒绝。引入此硬边界是为了避免通过缓存对发布器进行 DoS 攻击。
- `--jwt-cache-refresh-interval`：缓存刷新的间隔（秒），过期的 JWT 将被移除（可能为插入额外的 JWT 创建空间）。

## JWT 详细信息

### JWT 字段

当前的认证发布器实现不提供生成 JWT 并将其分发给客户端的方法。这些可以使用任何工具生成（下一节中将给出如何创建 JWT 的示例），只要它们遵守以下约束。

#### 必填字段

- `exp`（过期）：令牌过期的时间戳；
- `jti`（JWT ID）：令牌的唯一标识符，用于防止重放攻击；

#### 可选字段

- `iat`（发行时间）：可选的令牌发行时间戳；
- `send_object_to`：可选的 Sui 地址，新创建的 `Blob` 对象应发送到该地址；
- `epochs`：可选的 blob 应存储的确切纪元数
- `max_epochs`：可选的 blob 可存储的最大纪元数
- `max_size`：可选的 blob 的最大大小（字节）
- `size`：可选的 blob 的确切大小（字节）

注意：`epochs` 和 `max_epochs` 声明不能一起使用，`size` 和 `max_size` 也不能一起使用。在任一情况下同时使用两者都会导致令牌被拒绝。

重要的是，JWT（目前）只能编码要存储的 blob 的大小和纪元信息，而不能编码用户允许消耗的"SUI 和 WAL 数量"。这应该在发放 JWT 之前在后端完成。

### 在后端创建有效的 JWT

我们在这里提供一些关于如何创建可被认证发布器使用的 JWT 的示例。

#### Rust

在 Rust 中，可以使用 [`jsonwebtoken`](https://docs.rs/jsonwebtoken/latest/jsonwebtoken/) crate 来创建 JWT。

在我们的代码中，我们使用以下结构体在发布器中反序列化传入的令牌（请参阅[代码](https://github.com/MystenLabs/walrus/blob/main/crates/walrus-service/src/client/daemon/auth.rs)获取完整版本）。

```rust
pub struct Claim {
    pub iat: Option<i64>,
    pub exp: i64,
    pub jti: String,
    pub send_object_to: Option<SuiAddress>,
    pub epochs: Option<u32>,
    pub max_epochs: Option<u32>,
    pub size: Option<u64>,
    pub max_size: Option<u64>,
}
```

相同的结构体可用于在 Rust 中创建然后编码有效的令牌。这将是类似于：

```rust
use jsonwebtoken::{encode, Algorithm, EncodingKey, Header};

...

let encoding_key = EncodingKey::from_secret("my_secret".as_bytes());
let claim = Claim { /* set here the parameters for the Claim struct above */ };
let jwt = encode(&Header::default(), &claim, &encode_key).expect("a valid claim and key");
```