---
title: 发送和接收消息 | Microsoft Docs
description: 了解如何使用 Bot Connector 服务来发送和接收消息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 60d96297ea4bdc6ba920b4f8f990fabb0af8b8d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297058"
---
# <a name="send-and-receive-messages"></a>发送和接收消息

Bot Connector 服务可让机器人跨多个通道（例如 Skype、电子邮件、Slack 等）通信。 它通过从机器人到通道以及从通道到机器人的[活动](bot-framework-rest-connector-activities.md)中继来简化机器人与用户之间的通信。 每个活动包含用于将消息路由到相应目标的信息，以及有关消息创建者、消息上下文和消息接收者的信息。 本文介绍如何使用 Bot Connector 服务在通道上的机器人与用户之间交换**消息**活动。 

## <a id="create-reply"></a> 回复消息

### <a name="create-a-reply"></a>创建回复 

当用户向机器人发送消息时，机器人收到的消息是 **message** 类型的 [Activity][Activity] 对象。 若要创建用户消息的回复，请创建一个新的 [Activity][Activity] 对象，并首先设置以下属性：

| 属性 | 值 |
|----|----|
| conversation | 将此属性设置为用户消息中 `conversation` 属性的内容。 |
| from | 将此属性设置为用户消息中 `recipient` 属性的内容。 |
| 区域设置 | 将此属性设置为用户消息中 `locale` 属性的内容（如果已指定）。 |
| recipient | 将此属性设置为用户消息中 `from` 属性的内容。 |
| replyToId | 将此属性设置为用户消息中 `id` 属性的内容。 |
| type | 将此属性设置为 **message**。 |

接下来，设置用于指定要向用户传达的信息的属性。 例如，可以设置 `text` 属性来指定要在消息中显示的文本，设置 `speak` 属性来指定机器人要讲出的文本，设置 `attachments` 属性来指定要包含在消息中的媒体附件或丰富卡片。 有关常用消息属性的详细信息，请参阅[创建消息](bot-framework-rest-connector-create-messages.md)。

### <a name="send-the-reply"></a>发送回复

使用传入请求中的 `serviceUrl` 属性来标识机器人在发出响应时应该使用的[基 URI](bot-framework-rest-connector-api-reference.md#base-uri)。 

若要发送回复，请发出以下请求： 

```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

在此请求 URI 中，请将 **{conversationId}** 替换为（回复）活动中 `conversation` 对象的 `id` 属性值，并将 **{activityId}** 替换为（回复）活动中 `replyToId` 属性的值。 将请求正文设置为创建用来表示回复的 [Activity][Activity] 对象。

以下示例演示了一个向用户消息发送简单纯文本回复的请求。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基 URI；机器人发出的请求的基 URI 可能不同。 有关设置基 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
        "name": "Pepper's News Feed"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "Convo1"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "SteveW"
    },
    "text": "My bot's reply",
    "replyToId": "5d5cdc723"
}
```

## <a id="send-message"></a>发送（非回复）消息

机器人发送的大多数消息是对来自用户的消息的回复。 但是，机器人有时需要向聊天活动发送消息，而该消息并不是对来自用户的任何消息的直接回复。 例如，机器人可能需要启动新的聊天主题，或者在聊天结束时发送再见消息。 

若要向聊天活动发送一条并非直接回复来自用户的任何消息的消息，请发出以下请求： 

```http
POST /v3/conversations/{conversationId}/activities
```

在此请求 URI 中，请将 **{conversationId}** 替换为聊天的 ID。 
    
将请求正文设置为创建用来表示回复的 [Activity][Activity] 对象。

> [!NOTE]
> Bot Framework 不会对机器人可以发送的消息数量施加任何限制。 但是，大多数通道都会强制实施限制，以限制机器人在短时间内发送大量的消息。 此外，如果机器人快速连续地发送多条消息，则通道可能无法始终以正确的顺序呈现消息。

## <a name="start-a-conversation"></a>开始聊天

有时，机器人需要与一个或多个用户发起聊天。 若要开始与通道上的某个用户聊天，机器人必须知道自身的帐户信息，以及该用户在该通道上的帐户信息。 

> [!TIP]
> 如果机器人将来可能需要与其用户开始聊天，请缓存用户帐户信息、其他相关信息（例如用户首选项和区域设置），以及服务 URL（用作“开始聊天”请求中的基 URI）。 

若要开始聊天，请发出以下请求： 

```http
POST /v3/conversations
```

将请求正文设置为指定要包含在聊天中的机器人帐户信息和用户帐户信息的 [Conversation][Conversation] 对象。

> [!NOTE]
> 并非所有通道都支持群组聊天。 请查阅通道文档，以确定某个通道是否支持群组聊天，并确定某个通道在一个聊天活动中允许的参与者人数上限。

以下示例演示了开始聊天的请求。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基 URI；机器人发出的请求的基 URI 可能不同。 有关设置基 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/conversations 
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "bot": {
        "id": "12345678",
        "name": "bot's name"
    },
    "isGroup": false,
    "members": [
        {
            "id": "1234abcd",
            "name": "recipient's name"
        }
    ],
    "topicName": "News Alert"
}
```

如果已与指定的用户建立聊天会话，则响应将包含用于标识该聊天的 ID。 

```json
{
    "id": "abcd1234"
}
```

然后，机器人可以使用此聊天 ID 向聊天的用户[发送消息](#send-message)。

## <a name="additional-resources"></a>其他资源

- [活动概述](bot-framework-rest-connector-activities.md)
- [创建消息](bot-framework-rest-connector-create-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ConversationAccount]: bot-framework-rest-connector-api-reference.md#conversationaccount-object
[Conversation]: bot-framework-rest-connector-api-reference.md#conversation-object

