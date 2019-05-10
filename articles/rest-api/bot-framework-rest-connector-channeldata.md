---
title: 实现特定于通道的功能 | Microsoft Docs
description: 了解如何使用 Bot Connector API 实现特定于通道的功能。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d69013c721552483cfd38b204936cb1c7f508f82
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64564007"
---
# <a name="implement-channel-specific-functionality"></a>实现特定于通道的功能

某些通道提供的功能仅使用[消息文本和附件](bot-framework-rest-connector-create-messages.md)无法实现。 若要实现特定于通道的功能，可以将本机元数据传递给 [Activity][Activity] 对象的 `channelData` 属性中的通道。 例如，机器人可以使用 `channelData` 属性来指示 Telegram 发送贴纸或指示 Office365 发送电子邮件。

本文介绍如何使用消息活动的 `channelData` 属性来实现此通道特定的功能：

| 通道 | 功能 |
|----|----|
| 电子邮件 | 发送和接收包含正文、主题和重要性元数据的电子邮件 |
| Slack | 发送全保真的 Slack 消息 |
| Facebook | 本机发送 Facebook 通知 |
| Telegram | 执行 Telegram 特定操作，例如共享语音备忘录或贴纸 |
| Kik | 发送和接收本机 Kik 消息 | 

> [!NOTE]
> [Activity][Activity] 对象的 `channelData` 属性的值是 JSON 对象。 JSON 对象的结构将根据所执行的通道和功能而变化，如下所述。 

## <a name="create-a-custom-email-message"></a>创建自定义电子邮件

若要创建电子邮件，请将 [Activity][Activity] 对象的 `channelData` 属性设置为包含以下属性的 JSON 对象：

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

此代码片段显示了自定义电子邮件的 `channelData` 属性示例。

```json
"channelData":
{
    "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
    "subject": "Super awesome message subject",
    "importance": "high",
    "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
}
```

## <a name="create-a-full-fidelity-slack-message"></a>创建完全保真的 Slack 消息

若要创建完全保真的 Slack 消息，请将 [Activity][Activity] 对象的 `channelData` 属性设置为 JSON 对象，该对象指定 <a href="https://api.slack.com/docs/messages" target="_blank">Slack 消息</a>、<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack 附件</a>和/或 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按钮</a>。 

> [!NOTE]
> 若要在 Slack 消息中支持按钮，必须在[将机器人连接](../bot-service-manage-channels.md)到 Slack 通道时启用“交互式消息”。

此代码片段显示了自定义 Slack 消息的 `channelData` 属性示例。

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

当用户单击 Slack 消息中的按钮时，机器人将收到一条答复消息，其中 `channelData` 属性中填充了 `payload` JSON 对象。 `payload` 对象指定原始消息的内容，标识单击的按钮，并标识单击该按钮的用户。 

此代码片段显示了当用户单击 Slack 消息中的按钮时，机器人收到的消息中 `channelData` 属性的示例。

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

机器人可以按[常规方式](bot-framework-rest-connector-send-and-receive-messages.md#create-reply)回复此消息，也可以将其答复直接发布到由 `payload` 对象的 `response_url` 属性指定的终结点。 要了解何时以及如何将答复发布到 `response_url`，请参阅 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按钮</a>。 

## <a name="create-a-facebook-notification"></a>创建 Facebook 通知

若要创建 Facebook 通知，请将 [Activity][Activity] 对象的 `channelData` 属性设置为指定以下属性的 JSON 对象： 

| 属性 | 说明 |
|----|----|
| notification_type | 通知的类型（例如 REGULAR、SILENT_PUSH 和 NO_PUSH）。
| attachment | 附件（用于指定图像、视频或其他多媒体类型）或模板化附件（如收据）。 |

> [!NOTE]
> 要详细了解 `notification_type` 属性和`attachment` 属性的格式和内容，请参阅 <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">Facebook API 文档</a>。 

此代码片段显示了Facebook 收据附件的 `channelData` 属性示例。

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>创建 Telegram 消息

若要创建实现特定于 Telegram 的操作的消息，例如共享语音备忘录或贴纸，请将 [Activity][Activity] 对象的 `channelData` 属性设置为指定以下属性的 JSON 对象： 

| 属性 | 说明 |
|----|----|
| method | 要调用的 Telegram 机器人 API 方法。 |
| parameters | 已指定的方法的参数。 |

支持以下 Telegram 方法： 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

要详细了解这些 Telegram 方法及其参数，请参阅 <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">Telegram 机器人 API 文档</a>。

> [!NOTE]
> <ul><li><code>chat_id</code> 参数可通用于所有 Telegram 方法。 如果未将 <code>chat_id</code> 指定为参数，则框架将为你提供 ID。</li>
> <li>不要内联传递文件内容，而是使用 URL 和媒体类型指定文件，如下面的示例所示。</li>
> <li>在机器人从 Telegram 通道接收的每条消息中，<code>channelData</code> 属性都将包含机器人之前发送的消息。</li></ul>

此代码片段显示了一个指定单个 Telegram 方法的 `channelData` 属性的示例。

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

此代码片段显示了一个指定一组 Telegram 方法的 `channelData` 属性的示例。

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>创建本机 Kik 消息

若要创建本机 Kik 消息，请将 [Activity][Activity] 对象的 `channelData` 属性设置为指定以下属性的 JSON 对象： 

| 属性 | 说明 |
|----|----|
| messages | 一组 Kik 消息。 有关 Kik 消息格式的详细信息，请参阅 <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">Kik 消息格式</a>。 |

此代码片段显示了本机 Kik 消息的 `channelData` 属性示例。

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="additional-resources"></a>其他资源

- [活动概述](bot-framework-rest-connector-activities.md)
- [创建消息](bot-framework-rest-connector-create-messages.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)
- [使用 Channel Inspector 预览功能](../bot-service-channel-inspector.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
