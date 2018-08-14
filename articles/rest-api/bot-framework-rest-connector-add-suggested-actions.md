---
title: 向消息添加建议的操作 | Microsoft Docs
description: 了解如何使用 Bot Connector 服务向消息添加建议的操作。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d162bdd3f34848b16380317c776f445bc4611157
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298204"
---
# <a name="add-suggested-actions-to-messages"></a>向消息添加建议的操作
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> 若要了解各种通道如何呈现建议的操作，请参阅 [Channel Inspector][channelInspector]。

## <a name="send-suggested-actions"></a>发送建议的操作

若要向消息添加建议的操作，请设置 [Activity][Activity] 的 `suggestedActions` 属性，以指定表示要呈现给用户的按钮的 [CardAction][CardAction] 对象列表。 

以下请求向用户发送一条显示三个建议操作的消息。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "I have colors in mind, but need your help to choose the best one.",
    "inputHint": "expectingInput",
    "suggestedActions": {
        "actions": [
            {
                "type": "imBack",
                "title": "Blue",
                "value": "Blue"
            },
            {
                "type": "imBack",
                "title": "Red",
                "value": "Red"
            },
            {
                "type": "imBack",
                "title": "Green",
                "value": "Green"
            }
        ]
    },
    "replyToId": "5d5cdc723"
}
```

当用户点击其中一个建议的操作时，机器人将从用户收到包含相应操作的 `value` 的消息。

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)

[channelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object

[SuggestedAction]: bot-framework-rest-connector-api-reference.md#suggestedactions-object