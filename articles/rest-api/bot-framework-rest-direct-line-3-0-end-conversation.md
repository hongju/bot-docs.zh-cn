---
title: 结束会话 | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 结束会话。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ac984609acfdd8f85088bd47ccded1f45e953b2c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298104"
---
# <a name="end-a-conversation"></a>结束会话

无论是客户端还是机器人，都可以通过发送 endOfConversation [活动](bot-framework-rest-connector-activities.md)来指示 Direct Line 会话结束。 

## <a name="send-an-endofconversation-activity"></a>发送 endOfConversation 活动

endOfConversation 活动结束机器人和客户端之间的通信。 发送 endOfConversation 活动后，客户端可能仍使用 `HTTP GET` [检索消息](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)，但客户端和机器人都不可以向会话发送任何其他消息。 

若要结束会话，只需发出 POST 请求发送 endOfConversation 活动。

### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
    "type": "endOfConversation",
    "from": {
        "id": "user1"
    }
}
```

### <a name="response"></a>响应

如果请求成功，响应将包含已发送活动的 ID。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0004"
}
```

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [身份验证](bot-framework-rest-direct-line-3-0-authentication.md)
- [向机器人发送活动](bot-framework-rest-direct-line-3-0-send-activity.md)
