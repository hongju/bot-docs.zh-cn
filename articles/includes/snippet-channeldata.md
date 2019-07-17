---
ms.openlocfilehash: c19287b38a2c807e6675af2c3f7e1824eb7eab8e
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230772"
---
某些通道提供无法仅使用消息文本和附件实现的功能。 若要实现特定于通道的功能，可以将本机元数据传递给活动对象的_通道数据_属性。 例如，机器人可使用通道数据属性来指示 Telegram 发送贴纸或指示 Office365 发送电子邮件。

本文介绍如何使用消息活动的通道数据属性来实现此通道特定的功能：

| 通道 | 功能 |
|----|----|
| 电子邮件 | 发送和接收包含正文、主题和重要性元数据的电子邮件 |
| Slack | 发送全保真的 Slack 消息 |
| Facebook | 本机发送 Facebook 通知 |
| Telegram | 执行 Telegram 特定操作，例如共享语音备忘录或贴纸 |
| Kik | 发送和接收本机 Kik 消息 |

> [!NOTE]
> 活动对象的通道数据属性的值是 JSON 对象。
> 因此，本文中的示例显示了各种方案中 `channelData` JSON 属性的预期格式。
> 要使用 .NET 创建 JSON 对象，请使用 `JObject` (.NET) 类。

## <a name="create-a-custom-email-message"></a>创建自定义电子邮件

若要创建电子邮件，请将活动对象的通道数据属性设置为包含以下属性的 JSON 对象：

| 属性 | 说明 |
|----|----|
| bccRecipients | 添加到邮件“Bcc(密件抄送)”字段的用分号 (;) 分隔的电子邮件地址字符串。 |
| ccRecipients | 添加到邮件“Cc(抄送)”字段的用分号 (;) 分隔的电子邮件地址字符串。 |
| htmlBody | 用于指定电子邮件正文的 HTML 文档。 要了解受支持的 HTML 元素和特性，请参阅通道的相关文档。 |
| importance | 电子邮件的重要性级别。 有效值为 high、normal 和 low    。 默认值为 normal  。 |
| subject | 电子邮件的主题。 要了解字段要求，请参阅通道的相关文档。 |
| toRecipients | 添加到邮件“收件人”字段的用分号 (;) 分隔的电子邮件地址字符串。 |

> [!NOTE]
> 机器人通过“电子邮件”通道从用户处接收的消息可能包含通道数据属性，该属性中填充了一个如上所述的 JSON 对象。

此代码片段显示了自定义“电子邮件”消息的 `channelData` 属性示例。

```json
"channelData": {
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

## <a name="create-a-full-fidelity-slack-message"></a>创建全保真的 Slack 消息

若要创建完全保真的 Slack 消息，请将活动对象的通道数据属性设置为 JSON 对象，该对象指定 <a href="https://api.slack.com/docs/messages" target="_blank">Slack 消息</a>、<a href="https://api.slack.com/docs/message-attachments" target="_blank">Slack 附件</a>和/或 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按钮</a>。

> [!NOTE]
> 要使用户能在 Slack 消息中使用按钮，必须在[将机器人连接](../bot-service-manage-channels.md)到 Slack 通道时启用“交互式消息”  。

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

当用户单击 Slack 消息中的按钮时，机器人将收到一条答复消息，其中的通道数据属性填充了 `payload` JSON 对象。
`payload` 对象指定原始消息的内容，标识单击的按钮，并标识单击该按钮的用户。

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

机器人可以按常规方式回复此消息，也可以将其答复直接发布到由 `payload` 对象的 `response_url` 属性指定的终结点。
要了解何时以及如何将答复发布到 `response_url`，请参阅 <a href="https://api.slack.com/docs/message-buttons" target="_blank">Slack 按钮</a>。

可使用以下 JSON 创建动态按钮。

```json
{
    "text": "Would you like to play a game ? ",
    "attachments": [
        {
            "text": "Choose a game to play!",
            "fallback": "You are unable to choose a game",
            "callback_id": "wopr_game",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "game",
                    "text": "Chess",
                    "type": "button",
                    "value": "chess"
                },
                {
                    "name": "game",
                    "text": "Falken's Maze",
                    "type": "button",
                    "value": "maze"
                },
                {
                    "name": "game",
                    "text": "Thermonuclear War",
                    "style": "danger",
                    "type": "button",
                    "value": "war",
                    "confirm": {
                        "title": "Are you sure?",
                        "text": "Wouldn't you prefer a good game of chess?",
                        "ok_text": "Yes",
                        "dismiss_text": "No"
                    }
                }
            ]
        }
    ]
}
```

若要创建交互式菜单，请使用以下 JSON：

```json
{
    "text": "Would you like to play a game ? ",
    "response_type": "in_channel",
    "attachments": [
        {
            "text": "Choose a game to play",
            "fallback": "If you could read this message, you'd be choosing something fun to do right now.",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "callback_id": "game_selection",
            "actions": [
                {
                    "name": "games_list",
                    "text": "Pick a game...",
                    "type": "select",
                    "options": [
                        {
                            "text": "Hearts",
                            "value": "menu_id_hearts"
                        },
                        {
                            "text": "Bridge",
                            "value": "menu_id_bridge"
                        },
                        {
                            "text": "Checkers",
                            "value": "menu_id_checkers"
                        },
                        {
                            "text": "Chess",
                            "value": "menu_id_chess"
                        },
                        {
                            "text": "Poker",
                            "value": "menu_id_poker"
                        },
                        {
                            "text": "Falken's Maze",
                            "value": "menu_id_maze"
                        },
                        {
                            "text": "Global Thermonuclear War",
                            "value": "menu_id_war"
                        }
                    ]
                }
            ]
        }
    ]
}
```

## <a name="create-a-facebook-notification"></a>创建 Facebook 通知

若要创建 Facebook 通知，请将活动对象的通道数据属性设置为指定以下属性的 JSON 对象：

| 属性 | 说明 |
|----|----|
| notification_type | 通知的类型（例如 REGULAR、SILENT_PUSH 和 NO_PUSH）    。
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

若要创建实现特定于 Telegram 的操作的消息，例如共享语音备忘录或贴纸，请将活动对象的通道数据属性设置为指定以下属性的 JSON 对象： 

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
> <li>在机器人从 Telegram 通道接收的每条消息中，<code>ChannelData</code> 属性都将包含机器人之前发送的消息。</li></ul>

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

若要创建本机 Kik 消息，请将活动对象的通道数据属性设置为指定以下属性的 JSON 对象：

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

## <a name="create-a-line-message"></a>创建 LINE 消息

若要创建一条消息以实现特定于 LINE 的消息类型，例如便签、模板或特定于 LINE 的操作类型（例如打开手机摄像头），请将活动对象的通道数据属性设置为可指定 LINE 消息类型和操作类型的 JSON 对象。 

| 属性 | 说明 |
|----|----|
| type | LINE 操作/消息类型名称 |

支持以下 LINE 消息类型：
* 便签
* Imagemap 
* 模板（按钮、确认、轮播） 
* Flex 

以下 LINE 操作可以在消息类型为 JSON 对象的操作字段中指定： 
* 回发 
* 消息 
* URI 
* Datetimerpicker 
* 照相机 
* 本机照片 
* 位置 

若要详细了解这些 LINE 方法及其参数，请参阅 [LINE 机器人 API 文档](https://developers.line.biz/en/docs/messaging-api/)。 

此代码片段显示的示例介绍了 `channelData` 属性（用于指定通道消息类型 `ButtonTemplate`）和 3 个操作类型：相机、cameraRoll、Datetimepicker。 

```json
"channelData": { 
    "type": "ButtonsTemplate", 
    "altText": "This is a buttons template", 
    "template": { 
        "type": "buttons", 
        "thumbnailImageUrl": "https://example.com/bot/images/image.jpg", 
        "imageAspectRatio": "rectangle", 
        "imageSize": "cover", 
        "imageBackgroundColor": "#FFFFFF", 
        "title": "Menu", 
        "text": "Please select", 
        "defaultAction": { 
            "type": "uri", 
            "label": "View detail", 
            "uri": "http://example.com/page/123" 
        }, 
        "actions": [{ 
                "type": "cameraRoll", 
                "label": "Camera roll" 
            }, 
            { 
                "type": "camera", 
                "label": "Camera" 
            }, 
            { 
                "type": "datetimepicker", 
                "label": "Select date", 
                "data": "storeId=12345", 
                "mode": "datetime", 
                "initial": "2017-12-25t00:00", 
                "max": "2018-01-24t23:59", 
                "min": "2017-12-25t00:00" 
            } 
        ] 
    } 
}
```

## <a name="additional-resources"></a>其他资源

- [实体和活动类型](../bot-service-activities-entities.md)
- [Bot Framework Activity 架构](https://aka.ms/botSpecs-activitySchema)
