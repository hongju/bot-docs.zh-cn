---
title: 启动会话 | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 启动会话。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 6754935070fefb890c3d5b7bd90f1c1a4c5d401d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000294"
---
# <a name="start-a-conversation"></a>启动会话

Direct Line 会话由客户端显式打开，只要机器人和客户端参与并拥有有效凭据，就可以运行。 打开会话时，机器人和客户端可能会发送消息。 多个客户端可以连接到一个给定会话，并且每个客户端都可以代表多个用户参与。

## <a name="open-a-new-conversation"></a>打开新会话

若要使用机器人打开一个新会话，请发出以下请求：

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

以下代码片段提供了“启动会话”请求和响应的示例。

### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>响应

如果请求成功，响应将包含会话 ID、令牌和指示令牌过期之前的秒数的值以及客户端可能用于[通过 WebSocket 流接收活动](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)的流 URL。

```http
HTTP/1.1 201 Created
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800,
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
}
```

通常，启动会话请求用于打开一个新会话，如果新会话成功启动，则会返回 HTTP 201 状态代码。 但是，如果客户端使用启动会话操作在先前用于启动对话的 `Authorization` 标头中提交带有 Direct Line 令牌的启动会话请求，则将返回 HTTP 200 状态代码以指示请求可接受但没有创建会话（因为它已经存在）。

> [!TIP]
> 有 60 秒的时间连接到 WebSocket 流 URL。 如果在此期间无法建立连接，则可以[重新连接会话](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)以生成新的流 URL。

## <a name="start-conversation-versus-generate-token"></a>启动会话和生成令牌

启动会话操作 (`POST /v3/directline/conversations`) 类似于[生成令牌](bot-framework-rest-direct-line-3-0-authentication.md#generate-token)操作 (`POST /v3/directline/tokens/generate`)，因为这两个操作都返回可用于访问单个会话的 `token`。 但是，启动会话操作也会启动会话，与机器人联系并创建 WebSocket 流 URL，而生成令牌操作则不会执行这些操作。 

如果打算立即启动会话，请使用启动会话操作。 如果计划将令牌分发给客户端并希望它们启动会话，请改用[生成令牌](bot-framework-rest-direct-line-3-0-authentication.md#generate-token)操作。 

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [身份验证](bot-framework-rest-direct-line-3-0-authentication.md)
- [通过 WebSocket 流接收活动](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket)
- [重新连接到会话](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)