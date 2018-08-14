---
title: 向消息添加输入提示 | Microsoft Docs
description: 了解如何使用 Bot Connector 服务向消息添加输入提示。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 66c6bc20013ff2de82e29af76e9c99898c8b13d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298202"
---
# <a name="add-input-hints-to-messages"></a>向消息添加输入提示
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

通过为消息指定输入提示，在将消息传送到客户端后你可以指示机器人是接受、期待还是忽略用户输入。 对于许多通道，这使客户端可以相应地设置用户输入控件的状态。 例如，如果消息的输入提示指示机器人忽略用户输入，则客户端可以关闭麦克风并禁用输入框以防止用户提供输入。

## <a name="accepting-input"></a>接受输入

若要指示机器人被动地准备好接收输入但未等待用户的响应，请在表示消息的 [Activity][Activity] 对象中将 `inputHint` 属性设置为 acceptingInput。 在许多通道上，这将导致客户端的输入框启用并且麦克风关闭，但仍可供用户访问。 例如，如果用户按住麦克风按钮，Cortana 将打开麦克风以接受来自用户的输入。 

以下示例显示了发送消息并指定机器人正在接受输入的请求。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "Here's a picture of the house I was telling you about.",
    "inputHint": "acceptingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="expecting-input"></a>期待输入

若要指示机器人等待用户的响应，请在表示消息的 [Activity][Activity] 对象中将 `inputHint` 属性设置为 expectingInput。 在许多通道上，这将导致客户端的输入框启用并且麦克风打开。 

以下示例显示了发送消息并指定机器人期待输入的请求。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "What is your favorite color?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="ignoring-input"></a>忽略输入
 
若要指示机器人尚未准备好接收用户的输入，请在表示消息的 [Activity][Activity] 对象中将 `inputHint` 属性设置为 ignoringInput。 在许多通道上，这将导致客户端的输入框禁用并且麦克风关闭。 

以下示例显示了发送消息并指定机器人忽略输入的请求。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "Please hold while I perform the calculation.",
    "inputHint": "ignoringInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object