---
title: 使用 Bot Connector 服务创建机器人 | Microsoft Docs
description: 使用 Bot Connector 服务创建机器人。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 57babac9594118c12805ff9023cf7086e526a273
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997927"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>使用 Bot Connector 服务创建机器人
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [机器人服务](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Bot Connector 服务使机器人能够使用行业标准 REST 和基于 HTTPS 的 JSON 与 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>中配置的通道交换消息。 本教程将指导你完成从 Bot Framework 获取访问令牌并使用 Bot Connector 服务与用户交换消息的过程。

## <a id="get-token"></a> 获取访问令牌

> [!IMPORTANT]
> 如果尚未这样做，则必须向 Bot Framework [注册机器人](../bot-service-quickstart-registration.md)以获取其应用 ID 和密码。 你将需要机器人的 AppID 和密码以获取访问令牌。

若要与 Bot Connector 服务通信，必须使用以下格式在每个 API 请求的 `Authorization` 标头中指定访问令牌： 

```http
Authorization: Bearer ACCESS_TOKEN
```

可以通过发出 API 请求获取机器人的访问令牌。

### <a name="request"></a>请求

若要请求可用于对发送到 Bot Connector 服务的请求进行身份验证的访问令牌，请发出以下请求，将 **MICROSOFT-APP-ID** 和 **MICROSOFT-APP-PASSWORD** 替换为向 Bot Framework [注册](../bot-service-quickstart-registration.md)机器人时获得的应用 ID 和密码。

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>响应

如果请求成功，你将收到 HTTP 200 响应，该响应指定访问令牌及其到期时间的相关信息。 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> 有关 Bot Connector 服务中的身份验证的更多详细信息，请参阅[身份验证](bot-framework-rest-connector-authentication.md)。

## <a name="exchange-messages-with-the-user"></a>与用户交换消息

聊天是用户和机器人之间交换的一系列消息。 

### <a name="receive-a-message-from-the-user"></a>接收来自用户的消息

当用户发送消息时，Bot Framework Connector 会向你[注册](../bot-service-quickstart-registration.md)机器人时指定的终结点发出请求。 请求正文是 [Activity][Activity] 对象。 以下示例显示了当用户向机器人发送简单消息时机器人收到的请求正文。 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>回复用户的消息

当机器人的终结点收到表示来自用户的消息的 `POST` 请求（即 `type` = **message**）时，请使用该请求中的信息为响应创建 [Activity][Activity] 对象。

1. 将 **conversation** 属性设置为用户消息中 **conversation** 属性的内容。
2. 将 **from** 属性设置为用户消息中 **recipient** 属性的内容。
3. 将 **recipient** 属性设置为用户消息中 **from** 属性的内容。
4. 根据需要设置 **text** 和 **attachments** 属性。

使用传入请求中的 `serviceUrl` 属性[标识机器人应该用于发出响应的基本 URI](bot-framework-rest-connector-api-reference.md#base-uri)。 

若要发送响应，请将 [Activity][Activity] 对象`POST`到 `/v3/conversations/{conversationId}/activities/{activityId}`，如以下示例所示。 此请求的正文是 [Activity][Activity] 对象，它提示用户选择可用的约会时间。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。 

> [!IMPORTANT]
> 如此示例所示，你发送的每个 API 请求的 `Authorization` 标头必须包含单词 **Bearer**，后跟[从 Bot Framework 获取](#get-token)的访问令牌。

若要发送另一条消息，使用户能够通过单击按钮选择可用的约会时间，请将另一个请求 `POST` 到同一终结点：

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>后续步骤

在本教程中，你从 Bot Framework 获取了访问令牌，并使用 Bot Connector 服务与用户交换消息。 可以使用 [Bot Framework Emulator](../bot-service-debug-emulator.md) 测试和调试机器人。 如果想要与其他人共享机器人，则需要将其[配置](../bot-service-manage-channels.md)为在一个或多个通道上运行。


[Activity]: bot-framework-rest-connector-api-reference.md#activity-object