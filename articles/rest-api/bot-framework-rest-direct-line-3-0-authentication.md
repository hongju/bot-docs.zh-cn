---
title: 身份验证 | Microsoft Docs
description: 了解如何在 Direct Line API v3.0 中对 API 请求进行身份验证。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 18a5815a96e2052a54c48f6af211d8b28e20d983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297948"
---
# <a name="authentication"></a>身份验证

客户端可以使用从 Bot Framework 门户中的 [Direct Line 通道配置页获取](../bot-service-channel-connect-directline.md)的机密或使用在运行时获得的令牌来对 Direct Line API 3.0 的请求进行身份验证。 应使用以下格式在每个请求的 `Authorization` 标头中指定机密或令牌： 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>机密和令牌

Direct Line 机密是可用于访问属于关联的机器人的任何会话的主密钥。 机密还可用于获取令牌。 机密不会过期。 

Direct Line 令牌是可用于访问单个会话的密钥。 令牌会过期，但可以进行刷新。 

如果你要创建服务到服务应用程序，在 Direct Line API 请求的 `Authorization` 标头中指定机密可能是最简单的方法。 如果你要编写客户端在 Web 浏览器或移动应用中运行的应用程序，你可能想要交换你的机密以获取令牌（仅适用于单个会话，且在不刷新的情况下会过期）并在 Direct Line API 请求的 `Authorization` 标头中指定令牌。 选择最适合你的安全模型。

> [!NOTE]
> 你的 Direct Line 客户端凭据与你的机器人的凭据是不同的。 这让你能够独立修改你的密钥，并使你能够在不泄漏你的机器人的密码的情况下共享客户端令牌。 

## <a name="get-a-direct-line-secret"></a>获取 Direct Line 机密

可以在 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>中通过 Direct Line 通道配置页为你的机器人[获取 Direct Line 机密](../bot-service-channel-connect-directline.md)：

![Direct Line 配置](../media/direct-line-configure.png)

## <a id="generate-token"></a> 生成 Direct Line 令牌

若要生成可用于访问单个会话的 Direct Line 令牌，首先从 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>中的 Direct Line 通道配置页获取 Direct Line 机密。 然后，发出交换 Direct Line 机密的请求，以获取 Direct Line 令牌：

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

在此请求的 `Authorization` 标头中，将 SECRET 替换为 Direct Line 机密的值。

以下代码片段提供了 Generate Token 请求和响应的示例。

### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>响应

如果请求成功，响应将包对单个会话有效的 `token` 以及指示令牌过期之前的秒数的 `expires_in` 值。 为了使令牌仍然可用，必须在其过期前[刷新令牌](#refresh-token)。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

### <a name="generate-token-versus-start-conversation"></a>生成令牌和启动会话

生成令牌操作 (`POST /v3/directline/tokens/generate`) 类似于[启动会话](bot-framework-rest-direct-line-3-0-start-conversation.md)操作 (`POST /v3/directline/conversations`)，因为这两个操作都返回可用于访问单个会话的 `token`。 但是，与启动会话操作不同的是，生成令牌操作不会启动会话、不会联系机器人、也不会创建流式处理 WebSocket URL。 

如果计划将令牌分发给客户端并希望它们启动会话，请使用生成令牌操作。 如果打算立即启动会话，请改用[启动会话](bot-framework-rest-direct-line-3-0-start-conversation.md)操作。

## <a id="refresh-token"></a> 刷新 Direct Line 令牌

只要 Direct Line 令牌未过期，就可以无次数限制地刷新它。 无法刷新已过期的令牌。 若要刷新 Direct Line 令牌，请发出此请求： 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

在此请求的 `Authorization` 标头中，将 TOKEN_TO_BE_REFRESHED 替换为你想要刷新的 Direct Line 令牌。

以下代码片段提供了 Refresh Token 请求和响应的示例。

### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>响应

如果请求成功，响应将包含新的 `token`（它对与以前令牌相同的会话有效）以及 `expires_in` 值（它指示新令牌过期之前的秒数）。 为使新令牌仍然可用，必须在其过期前[刷新令牌](#refresh-token)。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [将机器人连接到 Direct Line](../bot-service-channel-connect-directline.md)