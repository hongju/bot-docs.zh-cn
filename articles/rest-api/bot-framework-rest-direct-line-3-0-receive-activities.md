---
title: 从机器人接收活动 | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 从机器人接收活动。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: dd5e81ba3feaba09e60011c138dcbe1537144b5a
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/12/2019
ms.locfileid: "59541003"
---
# <a name="receive-activities-from-the-bot"></a>从机器人接收活动

使用 Direct Line 3.0 协议，客户端可以通过 `WebSocket` 流接收活动或通过发出 `HTTP GET` 请求来检索活动。 

## <a name="websocket-vs-http-get"></a>WebSocket 和 HTTP GET

流式处理 WebSocket 有效地将消息推送到客户端，而 GET 接口可使客户端显式请求消息。 尽管 WebSocket 机制由于其效率通常是首选机制，但 GET 机制对于无法使用 WebSocket 的客户端非常有用。 

并非所有[活动类型](bot-framework-rest-connector-activities.md)都可通过 WebSocket 和 HTTP GET 获得。 下表介绍了使用 Direct Line 协议的客户端的各种活动类型的可用性。

| 活动类型 | 可用性 | 
|----|----|
| message | HTTP GET 和 WebSocket |
| typing | 仅 WebSocket |
| conversationUpdate | 未通过客户端发送/接收 |
| contactRelationUpdate | Direct Line 不受支持 |
| endOfConversation | HTTP GET 和 WebSocket |
| 所有其他活动类型 | HTTP GET 和 WebSocket |

## <a id="connect-via-websocket"></a> 通过 WebSocket 流接收活动

当客户端发送[启动会话](bot-framework-rest-direct-line-3-0-start-conversation.md)请求以打开与机器人的会话时，该服务的响应包括客户端随后可以通过 WebSocket 连接的 `streamUrl` 属性。 流 URL 是预先授权的，因此客户端通过 WebSocket 连接的请求不需要 `Authorization` 标头。

下面的示例演示使用 `streamUrl` 通过 WebSocket 进行连接的请求。

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

该服务以状态代码 HTTP 101（“交换协议”）响应。

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>接收消息

通过 WebSocket 连接后，客户端可能会从 Direct Line 服务接收以下类型的消息：

- 包含 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) 的消息，其中包含一个或多个活动和水印（如下所述）。
- 一条空消息，Direct Line 服务使用该消息来确保连接仍然有效。
- 其他类型，以便稍后进行定义。 这些类型由 JSON 根中的属性标识。

`ActivitySet` 包含机器人和会话中所有用户发送的消息。 以下示例显示包含单个消息的 `ActivitySet`。

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>处理消息

客户端应该跟踪它在每个 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) 中接收的 `watermark` 值，以便它可以使用水印来保证在丢失连接并且需要[重新连接到会话](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)时不丢失任何消息。 如果客户端收到 `ActivitySet`，其中 `watermark` 属性为 `null` 或缺失，则应忽略该值并且不覆盖以前收到的水印。

客户端应忽略从 Direct Line 服务接收的空消息。

客户端可以向 Direct Line 服务发送空消息以验证连接。 Direct Line 服务将忽略它从客户端接收的空消息。

Direct Line 服务可能会在某些条件下强行关闭 WebSocket 连接。 如果客户端尚未收到 `endOfConversation` 活动，则可以[生成新 WebSocket 流 URL](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)，该 URL 可用于重新连接到会话。 

WebSocket 流包含实时更新和最近收到的消息（因为已发出通过 WebSocket 连接的调用），但它不包括在最近的 `POST` 到 `/v3/directline/conversations/{id}` 之前发送的消息。 若要检索之前在会话中发送的消息，请使用 `HTTP GET`，如下所述。

## <a id="http-get"></a> 使用 HTTP GET 检索活动

无法使用 WebSocket 的客户端可以通过 `HTTP GET` 检索活动。

若要检索特定聊天的消息，请向 `/v3/directline/conversations/{conversationId}/activities` 终结点发出 `GET` 请求，根据需要指定 `watermark` 参数以指示客户端看到的最新消息。 

以下代码片段提供了 Get Conversation Activities 请求和响应的示例。 Get Conversation Activities 响应包含作为 [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object) 的属性的 `watermark`。 客户端应通过增加 `watermark` 值来浏览可用活动，直到不返回任何活动。

### <a name="request"></a>请求

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>响应

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>计时注意事项

大多数客户端都想要保留完整的消息历史记录。 尽管 Direct Line 是一个具有潜在计时间隙的多部分协议，但协议和服务旨在使生成可靠的客户端变得轻松简单。

- 在 WebSocket 流和 Get Conversation Activities 响应中发送的 `watermark` 属性是可靠的。 客户端将不会遗漏任何消息，只要它逐字重播水印。

- 当客户端启动会话并连接到 WebSocket 流时，在 POST 之后但在打开套接字之前发送的任何活动都将在新活动之前重播。

- 当客户端在连接到 WebSocket 流后发出 Get Conversation Activities 请求（以刷新历史记录）时，活动可能会在两个通道中重复。 客户端应该跟踪所有已知的活动 ID，以便它们能够拒绝重复的活动（如果出现重复活动）。

使用 `HTTP GET` 轮询的客户端应选择与其预期用途相匹配的轮询间隔。

- 服务到服务应用程序通常使用 5 秒或 10 秒的轮询间隔。

- 面向客户端的应用程序通常使用 1 秒的轮询间隔，并在客户端发送每条消息之后很快发出单个额外的请求（以快速检索机器人的响应）。 此延迟可以低至 300 毫秒，但应该根据机器人的速度和传输时间进行调整。 在延长的时间段，轮询不应超过每秒一次的频率。

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [身份验证](bot-framework-rest-direct-line-3-0-authentication.md)
- [启动会话](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [重新连接到会话](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [向机器人发送活动](bot-framework-rest-direct-line-3-0-send-activity.md)
- [结束会话](bot-framework-rest-direct-line-3-0-end-conversation.md)
