---
title: 重新连接到聊天 | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 重新连接到聊天。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2c6b3a7e9f0fdc7d5227fc8112cb6f3e330a2bcc
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297830"
---
# <a name="reconnect-to-a-conversation"></a>重新连接到聊天

如果客户端使用 [WebSocket 接口](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)来接收消息但丢失了连接，则可能需要重新连接。 在这种情况下，客户端必须生成一个新的 WebSocket 流 URL，它可以用来重新连接到聊天。

## <a name="generate-a-new-websocket-stream-url"></a>生成新的 WebSocket 流 URL

若要生成可用于重新连接到现有聊天的新 WebSocket 流 URL，请发出此请求： 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

在此请求 URI 中，将 **{conversationId}** 替换为聊天 ID，并将 **{watermark_value}** 替换为水印值（如果提供了 `watermark` 参数）。 `watermark` 参数是可选的。 如果在请求 URI 中指定了 `watermark` 参数，则聊天会从水印处重播，从而保证不会丢失任何消息。 如果请求 URI 中省略了 `watermark` 参数，则仅重播重新连接请求后收到的消息。

以下代码片段提供了重新连接请求和响应的示例。

### <a name="request"></a>请求

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>响应

如果请求成功，则响应将包含聊天 ID、令牌和新的 WebSocket 流 URL。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>重新连接到聊天

客户端必须使用新的 WebSocket 流 URL 在 60 秒内[重新连接到聊天](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)。 如果在此期间无法建立连接，则客户端必须发出另一个重新连接请求以生成新的流 URL。

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [身份验证](bot-framework-rest-direct-line-3-0-authentication.md)
- [通过 WebSocket 流接收活动](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)