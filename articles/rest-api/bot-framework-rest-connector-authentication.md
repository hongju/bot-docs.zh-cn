---
title: 对请求进行身份验证 | Microsoft Docs
description: 了解如何在 Bot Connector API 和 Bot State API 中对 API 请求进行身份验证。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 53643f21e2cf1ebdfd84caed38f8f84c330ef71b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298103"
---
# <a name="authentication"></a>身份验证

机器人使用通过安全通道 (SSL/TLS) 的 HTTP 与 Bot Connector 服务进行通信。 当机器人向 Connector 服务发送请求时，它必须包含 Connector 服务可以用来验证其身份的信息。 同样，当 Connector 服务向机器人发送请求时，它必须包含机器人可以用来验证其身份的信息。 本文介绍在机器人和 Bot Connector 服务之间进行的服务级身份验证的身份验证技术和要求。 如果正在编写自己的身份验证代码，必须实现本文所述的安全过程，以便机器人能够与 Bot Connector 服务交换消息。

> [!IMPORTANT]
> 如果正在编写自己的身份验证代码，则必须正确实现所有安全程序。 通过实现本文中的所有步骤，可以降低攻击者能够读取发送到机器人的消息、发送模拟机器人的消息以及窃取密钥的风险。 

如果使用的是 [Bot Builder SDK for .NET](../dotnet/bot-builder-dotnet-overview.md) 或 [Bot Builder SDK for Node.js](../nodejs/index.md)，则不需要实现本文所述的安全程序，因为 SDK 会自动为你完成。 只需使用在[注册](../bot-service-quickstart-registration.md)期间为机器人获得的应用 ID 和密码配置项目，SDK 将处理其余部分。

> [!WARNING]
> 在 2016 年 12 月，v3.1 Bot Framework 安全协议引入了对在令牌生成和验证期间使用的几个值的更改。 在 2017 年秋末，将引入 v3.2 Bot Framework 安全协议，它将再次包含对在令牌生成和验证期间使用的值的更改。
> 有关详细信息，请参阅[安全协议更改](#security-protocol-changes)。

## <a name="authentication-technologies"></a>身份验证技术

四个身份验证技术用于在机器人和 Bot Connector 之间建立信任：

| 技术 | Description |
|----|----|
| **SSL/TLS** | SSL/TLS 适用于所有服务到服务连接。 `X.509v3` 证书用于建立所有 HTTPS 服务的标识。 客户端应始终检查服务证书以确保它们受信任且有效。 （客户端证书不作为此方案的一部分使用。） |
| **OAuth 2.0** | OAuth 2.0 登录到 Microsoft Account (MSA)/AAD v2 登录服务用于生成机器人可用于发送消息的安全令牌。 此令牌是服务到服务令牌；无需用户登录。 |
| **JSON Web 令牌 (JWT)** | JSON Web 令牌用于编码向机器人发送和从中发出的令牌。 客户端应根据本文中所述的要求完全验证他们收到的所有 JWT 令牌。 |
| **OpenID 元数据** | Bot Connector 服务发布一个有效令牌列表，用于在已知静态终结点上将自己的 JWT 令牌签署到 OpenID 元数据。 |

本文介绍如何通过标准 HTTPS 和 JSON 使用这些技术。 不需要特殊 SDK，尽管你可能发现 OpenID 之类的帮助程序很有用。

## <a id="bot-to-connector"></a> 对从机器人到 Bot Connector 服务的请求进行身份验证

若要与 Bot Connector 服务通信，必须使用以下格式在每个 API 请求的 `Authorization` 标头中指定访问令牌： 

```http
Authorization: Bearer ACCESS_TOKEN
```

此图显示 bot-to-connector 身份验证的步骤：

![依次对 MSA 登录服务和机器人进行身份验证](../media/connector/auth_bot_to_bot_connector.png)

> [!IMPORTANT]
> 如果尚未这样做，则必须向 Bot Framework [注册机器人](../bot-service-quickstart-registration.md)以获取其应用 ID 和密码。 将需要机器人的应用 ID 和密码以请求访问令牌。

### <a name="step-1-request-an-access-token-from-the-msaaad-v2-login-service"></a>步骤 1：从 MSA/AAD v2 登录服务请求访问令牌

若要从 MSA/AAD v2 登录服务请求访问令牌，请发出以下请求，将 MICROSOFT-APP-ID 和 MICROSOFT-APP-PASSWORD 替换为向 Bot Framework [注册](../bot-service-quickstart-registration.md)机器人时获得的应用 ID 和密码。

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-msaaad-v2-login-service-response"></a>步骤 2：从 MSA/AAD v2 登录服务响应中获取 JWT 令牌

如果通过 MSA/AAD v2 登录服务对应用程序进行授权，JSON 响应正文将指定访问令牌、其类型及其过期时间（以秒为单位）。 

将令牌添加到请求的 `Authorization` 标头时，必须使用此响应中指定的确切值（即，不转义或编码令牌值）。 访问令牌有效，直到过期。 为了防止令牌过期影响机器人性能，可以选择缓存并主动刷新令牌。

此示例演示来自 MSA/AAD v2 登录服务的响应：

```http
HTTP/1.1 200 OK
... (other headers) 
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>步骤 3：指定请求的 Authorization 标头中的 JWT 令牌

当向 Bot Connector 服务发送 API 请求，使用此格式指定请求的 `Authorization` 标头中的访问令牌：

```http
Authorization: Bearer ACCESS_TOKEN
```

发送到 Bot Connector 服务的所有请求都必须在 `Authorization` 标头中包含访问令牌。 如果令牌正确、未过期且由 MSA/AAD v2 登录服务生成，Bot Connector 服务将授权请求。 需执行额外检查以确保令牌属于发送请求的机器人。

下面的示例演示如何指定请求 `Authorization` 标头中的访问令牌。 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> 只在发送到 Bot Connector 服务的请求的 `Authorization` 标头中指定 JWT 令牌。 不要将令牌发送到非安全通道，也不要将其包含在发送给其他服务的 HTTP 请求中。 从 MSA/AAD v2 登录服务获取的 JWT 令牌类似于密码，应谨慎处理。 拥有令牌的任何人都可以使用它代表机器人执行操作。 

#### <a name="bot-to-connector-example-jwt-components"></a>机器人到 Connector：示例 JWT 组件

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> 在实践中，实际字段可能会有所不同。 创建并验证上面指定的所有 JWT 令牌。

## <a id="connector-to-bot"></a> 对从 Bot Connector 服务到机器人的请求进行身份验证

当 Bot Connector 服务将请求发送到机器人时，它会在请求的 `Authorization` 标头中指定已签名的 JWT 令牌。 通过验证已签名的 JWT 令牌的真实性，机器人可以对来自 Bot Connector 服务的调用进行身份验证。 

此图显示 connector-to-bot 身份验证的步骤：

![对从 Bot Connector 到机器人的调用进行身份验证](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> 步骤 2：获取 OpenID 元数据文档

OpenID 元数据文档指定第二个文档的位置，其中列出了 Bot Connector 服务的有效签名密钥。 若要获取 OpenID 元数据文档，请通过 HTTPS 发出此请求：

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> 这是静态 URL，可以硬编码到应用程序。 

下面的示例演示响应 `GET` 请求返回的 OpenID 元数据文档。 `jwks_uri` 属性指定包含 Bot Connector 服务的有效签名密钥的文档的位置。

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> 步骤 3：获取有效的签名密钥列表

若要获取有效的签名密钥列表，请通过 HTTPS 向 OpenID 元数据文档中的 `jwks_uri` 属性指定的 URL 发出 `GET` 请求。 例如：

```http
GET https://login.botframework.com/v1/.well-known/keys
```

响应正文指定 [JWK 格式](https://tools.ietf.org/html/rfc7517)的文档，还包括每个密钥的其他属性：`endorsements`。 密钥列表相对稳定，并可能会缓存较长时间（默认情况下，在 Bot Builder SDK 中为 5 天）。

每个密钥中的 `endorsements` 属性包含一个或多个字符串，可用于验证在传入请求的 [Activity][Activity] 对象的 `channelId` 属性中定义的通道 ID 是否可信。 需要认可的通道 ID 列表在每个机器人中都可配置。 默认情况下，它将是所有已发布通道 ID 的列表，尽管机器人开发人员可能会通过任何一种方式重写选定通道的 ID 值。 如果需要认可通道 ID：

- 应请求使用通道 ID 发送到机器人的任何 [Activity][Activity] 对象随附 JWT 令牌，并签署有对该通道的认可。 
- 如果不存在认可，则机器人应通过返回 HTTP 403（禁止）状态代码拒绝该请求。

### <a name="step-4-verify-the-jwt-token"></a>步骤 4：验证 JWT 令牌

若要验证由 Bot Connector 服务发送的令牌的真实性，必须从请求的 `Authorization` 标头中提取令牌、分析令牌、验证其内容并验证其签名。 

JWT 分析库适用于许多平台，并且大多数对 JWT 令牌实现安全且可靠的分析，尽管通常情况下必须配置这些库以请求令牌的某些特征（其颁发者、受众等）包含正确的值。 分析令牌时，必须配置分析库或编写你自己的验证来确保令牌满足这些要求：

1. 令牌在 HTTP `Authorization` 标头中使用“持有者”方案发送。
2. 令牌是有效的 JSON，符合 [JWT 标准](http://openid.net/specs/draft-jones-json-web-token-07.html)。
3. 令牌包含值为 `https://api.botframework.com` 的“颁发者”声明。
4. 该令牌包含一个“受众”声明，其值等于机器人的 Microsoft 应用 ID。
5. 该令牌在其有效期内。 行业标准时钟偏差为 5 分钟。
6. 令牌具有有效的加密签名，包含在[步骤 3](#connector-to-bot-step-3) 中检索到的 OpenID 密钥文档中列出的密钥，使用的是[步骤 2](#openid-metadata-document) 中检索到的 Open ID 元数据文档的 `id_token_signing_alg_values_supported` 属性中指定的签名算法。
7. 该令牌包含“serviceUrl”声明，其值与传入请求的 [Activity][Activity] 对象根的 `servieUrl` 属性匹配。 

如果令牌不满足所有这些要求，则机器人应通过返回 HTTP 403（禁止）状态代码拒绝该请求。

> [!IMPORTANT]
> 所有这些要求都很重要，尤其是要求 4 和 6。 如果未能实现所有这些验证要求将使机器人受到攻击，这可能会导致机器人公开其 JWT 令牌。

实施者不应公开禁用发送给机器人的 JWT 令牌的验证的方法。

#### <a name="connector-to-bot-example-jwt-components"></a>Connector 到机器人：示例 JWT 组件

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 在实践中，实际字段可能会有所不同。 创建并验证上面指定的所有 JWT 令牌。

## <a id="emulator-to-bot"></a> 对从 Bot Framework Emulator 到机器人的请求进行身份验证

> [!WARNING]
> 在 2017 年秋末，将引入 v3.2 Bot Framework 安全协议。 此新版本在令牌中包括新的“颁发者”值，这些令牌在 Bot Framework Eumaltor 和机器人之间进行交换。 若要准备进行此更改，以下步骤概述了如何检查 v3.1 和 v3.2 颁发者值。 

[Bot Framework Emulator](../bot-service-debug-emulator.md) 是一个桌面工具，可用于测试机器人的功能。 虽然 Bot Framework Emulator 使用如上所述的相同[身份验证技术](#authentication-technologies)，但它无法模拟真实的 Bot Connector 服务。 相反，它使用在将模拟器连接到机器人时所指定的 Microsoft 应用 ID 和 Microsoft 应用密码，以创建与机器人创建的令牌相同的令牌。 在模拟器向机器人发送请求时，它将在请求的 `Authorization` 标头中指定 JWT 令牌 -- 从本质上讲，使用机器人自己的凭据对请求进行身份验证。 

如果要实现身份验证库，并想要接受来自 Bot Framework Emulator 的请求，则必须添加此附加验证路径。 路径在结构上类似于 [Connector -> Bot](#connector-to-bot) 验证路径，但它使用 MSA 的 OpenID 文档而不是 Bot Connector 的 OpenID 文档。

此图显示 emulator-to-bot 身份验证的步骤：

![对从 Bot Framework Emulator 到机器人的调用进行身份验证](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>步骤 2：获取 MSA OpenID 元数据文档

OpenID 元数据文档指定第二个文档的位置，其中列出了有效签名密钥。 若要获取 MSA OpenID 元数据文档，请通过 HTTPS 发出此请求：

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

下面的示例演示响应 `GET` 请求返回的 OpenID 元数据文档。 `jwks_uri` 属性指定文档的位置，其中包含有效的签名密钥。

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> 步骤 3：获取有效的签名密钥列表

若要获取有效的签名密钥列表，请通过 HTTPS 向 OpenID 元数据文档中的 `jwks_uri` 属性指定的 URL 发出 `GET` 请求。 例如：

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

响应正文指定 [JWK 格式](https://tools.ietf.org/html/rfc7517)的文档。 

### <a name="step-4-verify-the-jwt-token"></a>步骤 4：验证 JWT 令牌

若要验证模拟器发送的令牌的真实性，必须从请求的 `Authorization` 标头提取令牌、分析令牌、验证其内容，并验证其签名。 

JWT 分析库适用于许多平台，并且大多数对 JWT 令牌实现安全且可靠的分析，尽管通常情况下必须配置这些库以请求令牌的某些特征（其颁发者、受众等）包含正确的值。 分析令牌时，必须配置分析库或编写你自己的验证来确保令牌满足这些要求：

1. 令牌在 HTTP `Authorization` 标头中使用“持有者”方案发送。
2. 令牌是有效的 JSON，符合 [JWT 标准](http://openid.net/specs/draft-jones-json-web-token-07.html)。
3. 该令牌包含值为 `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` 或 `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` 的“颁发者”声明。 （检查颁发者值将确保检查安全性协议 v3.1 和 v3.2 颁发者值）
4. 该令牌包含一个“受众”声明，其值等于机器人的 Microsoft 应用 ID。
5. 该令牌包含一个“appid”声明，其值等于机器人的 Microsoft 应用 ID。
6. 该令牌在其有效期内。 行业标准时钟偏差为 5 分钟。
7. 令牌具有有效的加密签名，包含在[步骤 3](#emulator-to-bot-step-3) 中检索到的 OpenID 密钥文档中列出的密钥。

> [!NOTE]
> 要求 5 特定于模拟器验证路径。 

如果令牌不满足所有这些要求，则机器人应通过返回 HTTP 403（禁止）状态代码终止该请求。

> [!IMPORTANT]
> 所有这些要求都很重要，尤其是要求 4 和 7。 如果未能实现所有这些验证要求将使机器人受到攻击，这可能会导致机器人公开其 JWT 令牌。

#### <a name="emulator-to-bot-example-jwt-components"></a>模拟器到机器人：示例 JWT 组件

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> 在实践中，实际字段可能会有所不同。 创建并验证上面指定的所有 JWT 令牌。

## <a name="security-protocol-changes"></a>安全协议更改

> [!WARNING]
> 对 v3.0 安全协议的支持已于 2017 年 7 月 31 日停用。 如果已编写自己的身份验证代码（即，未使用 Bot Builder SDK 创建机器人），则必须通过更新应用程序升级到 v3.1 安全协议，以便使用下面列出的 v3.1 值。 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[机器人到 Connector 身份验证](#bot-to-connector)

#### <a name="oauth-login-url"></a>OAuth 登录 URL

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth 作用域

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[Connector 到机器人身份验证](#connector-to-bot)

#### <a name="openid-metadata-document"></a>OpenID 元数据文档

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>JWT 颁发者

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[模拟器到机器人身份验证](#emulator-to-bot)

#### <a name="oauth-login-url"></a>OAuth 登录 URL

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>OAuth 作用域

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 |  机器人的 Microsoft 应用 ID + `/.default` |

#### <a name="jwt-audience"></a>JWT 受众

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 | 机器人的 Microsoft 应用 ID |

#### <a name="jwt-issuer"></a>JWT 颁发者

| 协议版本 | 有效值 |
|----|----|
| v3.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| v3.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>OpenID 元数据文档

| 协议版本 | 有效值 |
|----|----|
| v3.1 和 v3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>其他资源

- [对 Bot Framework 身份验证进行故障排除](../bot-service-troubleshoot-authentication-problems.md)
- [JSON Web 令牌 (JWT) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html)
- [JSON Web 签名 (JWS) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04)
- [JSON Web 密钥 (JWK) RFC 7517](https://tools.ietf.org/html/rfc7517)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
