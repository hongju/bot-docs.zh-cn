---
title: 从机器人接收消息 | Microsoft Docs
description: 了解如何使用 Direct Line API v1.1 从机器人接收消息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d9f1821767e5bd26c9a8bfdf3927f257077f0e79
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298231"
---
# <a name="receive-messages-from-the-bot"></a>从机器人接收消息

> [!IMPORTANT]
> 本文介绍如何使用 Direct Line API 1.1 从机器人接收消息。 如果要在客户端应用程序和机器人之间创建新连接，请改用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-receive-activities.md)。

使用 Direct Line 1.1 协议时，客户端必须轮询 `HTTP GET` 接口以接收消息。 

## <a name="retrieve-messages-with-http-get"></a>通过 HTTP GET 检索消息

若要检索特定聊天的消息，请向 `api/conversations/{conversationId}/messages` 终结点发出 `GET` 请求，根据需要指定 `watermark` 参数以指示客户端看到的最新消息。 即使没有包含消息，也会在 JSON 响应中返回更新的 `watermark` 值。

以下代码片段提供了“获取消息”请求和响应的示例。 “获取消息”响应包含 `watermark` 作为 [MessageSet](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object) 的属性。 客户端应通过增加 `watermark` 值来按页浏览提供的消息，直到不返回任何消息。 

### <a name="request"></a>请求

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>响应

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>计时注意事项

尽管 Direct Line 是一个具有潜在计时间隙的多部分协议，但协议和服务旨在轻松简单地生成可靠的客户端。 发送到“获取消息”中的 `watermark` 属性很可靠。 只要客户端逐字重播水印，将不会遗漏任何消息。

客户端应选择匹配其预期用途的轮询间隔。

- 服务到服务应用程序通常使用 5 秒或 10 秒的轮询间隔。

- 面向客户端的应用程序通常使用 1 秒的轮询间隔，并在客户端发送的每条消息之后 ~300ms 发出额外的请求（以快速检索机器人的响应）。 应根据机器人的速度和传输时间调整此 300ms 延迟。

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [身份验证](bot-framework-rest-direct-line-1-1-authentication.md)
- [启动聊天](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [向机器人发送消息](bot-framework-rest-direct-line-1-1-send-message.md)