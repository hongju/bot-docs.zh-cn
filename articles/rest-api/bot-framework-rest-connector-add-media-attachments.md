---
title: 向消息添加媒体附件 | Microsoft Docs
description: 了解如何使用 Bot Connector 服务向消息添加媒体附件。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 39247ec3be4da7129989041269e930de8fa766ae
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297970"
---
# <a name="add-media-attachments-to-messages"></a>向消息添加媒体附件
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

机器人和通道通常会交换文本字符串，但某些通道也支持交换附件，这样机器人就可向用户发送更丰富的消息。 例如，机器人可以发送媒体附件（例如，图像、视频、音频、文件）和[富卡](bot-framework-rest-connector-add-rich-cards.md)。 本文介绍如何使用 Bot Connector 服务向消息添加媒体附件。

> [!TIP]
> 若要确定通道支持的附件的类型和数量以及通道如何呈现附件，请参阅[通道检查器][ChannelInspector]。

## <a name="add-a-media-attachment"></a>添加媒体附件  

若要向消息添加媒体附件，请创建 [Attachment][Attachment] 对象，设置 `name` 属性，将 `contentUrl` 属性设置为媒体文件的 URL，并将 `contentType` 属性设置为合适的媒体类型（例如，image/png、audio/wav、video/mp4）。 然后在表示消息的 [Activity][Activity] 对象中，在 `attachments` 数组中指定 [Attachment][Attachment] 对象。 

以下示例显示了一个请求，该请求可以发送包含文本和单个图像附件的消息。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "http://aka.ms/Fo983c",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

对于支持图像的内联二进制文件的通道，可以将 `Attachment` 的 `contentUrl` 属性设置为图像的 base64 二进制文件（例如，data:image/png;base64,iVBORw0KGgo…）。 通道将在消息的文本字符串旁显示图像或图像的 URL。

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

可以通过为图像文件使用上述相同的过程，将视频文件或音频文件附加到消息中。 根据通道的不同，视频和音频可以内联播放，也可以显示为链接。

> [!NOTE] 
> 机器人也可能会收到包含媒体附件的消息。
> 例如，如果通道允许用户上传要分析的照片或要存储的文档，则机器人收到的消息可能包含附件。

## <a name="add-an-audiocard-attachment"></a>添加 AudioCard 附件

添加 [AudioCard](bot-framework-rest-connector-api-reference.md#audiocard-object) 或 [VideoCard](bot-framework-rest-connector-api-reference.md#videocard-object) 附件与添加媒体附件相同。 例如，以下 JSON 显示了如何在媒体附件中添加音频卡片。

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
    "attachments": [
    {
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

通道收到此附件后，将开始播放音频文件。 例如，如果用户通过单击“暂停”按钮与音频交互，则该通道将使用看起来像下面这样的 JSON 向机器人发送回调：

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

媒体事件名称“media/pause”将出现在 `activity.name` 字段中。 参考下表了解所有媒体事件名称的列表。

| 事件 | Description |
| ---- | ---- |
| **media/next** | 客户端跳到下一个媒体 |
| **media/pause** | 客户端暂停播放媒体 |
| **media/play** | 客户端开始播放媒体 |
| **media/previous** | 客户端跳到上一个媒体 |
| **media/resume** | 客户端继续播放媒体 |
| **media/stop** | 客户端停止播放媒体 |

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)
- [向消息添加富卡](bot-framework-rest-connector-add-rich-cards.md)
- [通道检查器][ChannelInspector]

[ChannelInspector]: ../bot-service-channel-inspector.md

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
