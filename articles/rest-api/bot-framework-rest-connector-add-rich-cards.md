---
title: 向消息添加资讯卡附件 | Microsoft Docs
description: 了解如何使用 Bot Connector 服务向消息添加资讯卡。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a662bb24f384d072a162242a4634fe4fe3a4b395
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033449"
---
# <a name="add-rich-card-attachments-to-messages"></a>向消息添加资讯卡附件
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

机器人和通道通常会交换文本字符串，但某些通道也支持交换附件，这样机器人就可向用户发送更丰富的消息。 例如，机器人可以发送资讯卡和媒体附件（例如图像、视频、音频、文件）。 本文介绍如何使用 Bot Connector 服务在消息中添加资讯卡附件。

> [!NOTE]
> 若要了解如何向消息添加媒体附件，请参阅[向消息添加媒体附件](bot-framework-rest-connector-add-media-attachments.md)。

## <a id="types-of-cards"></a>资讯卡类型

资讯卡包括标题、说明、链接和图像。 消息可包含多个资讯卡，以列表格式或轮播格式显示。
Bot Framework 目前支持八种类型的资讯卡： 

| 卡类型 | 说明 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">AdaptiveCard</a> | 一种可自定义卡，可包含文本、语音、图像、按钮和输入域的任意组合。 请参阅[每个通道的支持](/adaptive-cards/get-started/bots#channel-status)。  |
| [AnimationCard][animationCard] | 一种可播放动态 GIF 或短视频的卡。 |
| [AudioCard][audioCard] | 一种可播放音频文件的卡。 |
| [HeroCard][heroCard] | 一种通常包含单个大图像、一个或多个按钮和文本的卡。 |
| [ThumbnailCard][thumbnailCard] | 一种通常包含单个缩略图图像、一个或多个按钮和文本的卡。 |
| [ReceiptCard][receiptCard] | 一种可让机器人向用户提供收据的卡。 它通常包含要包括在收据中的项目列表、税款和总计信息以及其他文本。 |
| [SignInCard][signinCard] | 一种可让机器人请求用户登录的卡。 它通常包含文本和一个或多个按钮，用户可以单击这些按钮来启动登录进程。 |
| [VideoCard][videoCard] | 一种可播放视频的卡。 |

> [!TIP]
> 若要确定通道支持的资讯卡类型并查看通道呈现资讯卡的方式，请参阅 [Channel Inspector][ChannelInspector]。 有关卡内容限制的信息（例如最大按钮数或最大标题长度），请参阅通道文档。

## <a name="process-events-within-rich-cards"></a>处理资讯卡中的事件

若要处理资讯卡中的事件，请使用 [CardAction][CardAction] 对象指定当用户单击按钮或点击卡的某个部分时应发生的情况。 每个 [CardAction][CardAction] 对象都包含以下属性：

| 属性 | Type | 说明 | 
|----|----|----|
| type | 字符串 | 操作类型（下表中指定的某个值） |
| title | 字符串 | 按钮的标题 |
| image | 字符串 | 按钮的图像 URL |
| 值 | 字符串 | 执行指定类型的操作所需的值 |

> [!NOTE]
> 自适应卡片中的按钮不是使用 `CardAction` 对象创建的，而是使用<a href="http://adaptivecards.io" target="_blank">自适应卡片</a>定义的架构创建的。 有关如何向“自适应卡片”添加按钮的示例，请参阅[向消息添加自适应卡片](#adaptive-card)。

此表列出了 [CardAction][CardAction] 对象 `type` 属性的有效值，并描述了每种类型的 `value` 属性的预期内容：

| type | 值 | 
|----|----|
| openUrl | 要在内置浏览器中打开的 URL |
| imBack | 要发送到机器人的消息文本（来自单击按钮或点击卡的用户）。 通过托管会话的客户端应用程序，所有会话参与者都可看到此消息（从用户到机器人）。 |
| postBack | 要发送到机器人的消息文本（来自单击按钮或点击卡的用户）。 某些客户端应用程序可能会在消息源中显示此文本，所有会话参与者都可看到该文本。 |
| call | 格式如下的电话呼叫的目标：电话:123123123123 |
| playAudio | 要播放的音频的 URL |
| playVideo | 要播放的视频的 URL |
| showImage | 要显示的图像的 URL |
| downloadFile | 要下载的文件的 URL |
| signin | 要启动的 OAuth 流的 URL |

## <a name="add-a-hero-card-to-a-message"></a>向消息添加 Hero 卡

若要向消息添加资讯卡附件，请先创建与要添加到消息的[卡类型](#types-of-cards)对应的对象。 然后创建一个 [Attachment][Attachment] 对象，将其 `contentType` 属性设置为卡的媒体类型，将其 `content` 属性设置为创建的用于表示该卡的对象。 在消息的 `attachments` 数组中指定 [Attachment][Attachment] 对象。

> [!TIP]
> 包含资讯卡附件的消息通常不指定 `text`。

某些通道允许将多个资讯卡添加到消息中的 `attachments` 数组。 如果想为用户提供多个选项，此功能非常有用。 例如，如果机器人允许用户预订酒店房间，它可为用户提供一张资讯卡列表，显示出可选房间类型。 每张卡包含与房间类型相对应的图片和设施清单，并且用户可通过点击卡或单击按钮来选择房间类型。

> [!TIP]
> 若要以列表格式显示多个资讯卡，请将 [Activity][Activity] 对象的 `attachmentLayout` 属性设置为“列表”。 若要以轮播格式显示多个资讯卡，请将 [Activity][Activity] 对象的 `attachmentLayout` 属性设置为“轮播”。 如果通道不支持轮播格式，即使 `attachmentLayout` 属性指定“轮播”，它也将以列表格式显示资讯卡。

以下示例显示的请求可发送包含单个 Hero 卡附件的消息。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "https://aka.ms/DuckOnARock",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "https://aka.ms/DuckOnARock",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a id="adaptive-card"></a>向消息添加自适应卡片

自适应卡片可包含文本、语音、图像、按钮和输入域的任意组合。 自适应卡片使用<a href="http://adaptivecards.io" target="_blank">自适应卡片</a>中指定的 JSON 格式创建，让用户能够完全控制卡片内容和格式。 

利用<a href="http://adaptivecards.io" target="_blank">自适应卡片</a>站点中的信息来了解自适应卡片架构，探索自适应卡片元素，并查看可用于创建具有不同组成内容和复杂性的卡的 JSON 样本。 此外，可使用交互式可视化工具设计自适应卡片有效负载并预览卡片输出。

以下示例显示的请求可发送包含用于日历提醒的单个自适应卡片的消息。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "type": "AdaptiveCard",
                "body": [
                    {
                        "type": "TextBlock",
                        "text": "Adaptive Card design session",
                        "size": "large",
                        "weight": "bolder"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Conf Room 112/3377 (10)"
                    },
                    {
                        "type": "TextBlock",
                        "text": "12:30 PM - 1:30 PM"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Snooze for"
                    },
                    {
                        "type": "Input.ChoiceSet",
                        "id": "snooze",
                        "style": "compact",
                        "choices": [
                            {
                                "title": "5 minutes",
                                "value": "5",
                                "isSelected": true
                            },
                            {
                                "title": "15 minutes",
                                "value": "15"
                            },
                            {
                                "title": "30 minutes",
                                "value": "30"
                            }
                        ]
                    }
                ],
                "actions": [
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Snooze"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "I'll be late"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Dismiss"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

生成的卡包含三个文本块、一个输入域（选项列表）和三个按钮：

![自适应卡片日历提醒](../media/adaptive-card-reminder.png)


## <a name="additional-resources"></a>其他资源

- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)
- [向消息添加媒体附件](bot-framework-rest-connector-add-media-attachments.md)
- [Channel Inspector][ChannelInspector]
- <a href="http://adaptivecards.io" target="_blank">自适应卡片</a>

[ChannelInspector]: ../bot-service-channel-inspector.md

[animationCard]: bot-framework-rest-connector-api-reference.md#animationcard-object
[audioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[heroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[thumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[receiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[signinCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[videoCard]: bot-framework-rest-connector-api-reference.md#videocard-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object