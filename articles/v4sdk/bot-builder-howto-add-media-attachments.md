---
title: 向消息添加媒体 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK 向消息添加媒体。
keywords: 媒体, 消息, 图像, 音频, 视频, 文件, 消息工厂, 富消息
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 30a0c463698d9ab7e3b2b0f9ddb0e872f007d1d8
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/04/2018
ms.locfileid: "39515037"
---
# <a name="add-media-to-messages"></a>向消息添加媒体

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

用户与机器人之间的消息交换可以包含媒体附件，例如图像、视频、音频和文件。 Bot Builder SDK 包含一个新的 `Microsoft.Bot.Builder.MessageFactory` 类，旨在简化向用户发送富消息的任务。

## <a name="send-attachments"></a>发送附件

若要向用户发送内容（如图像或视频），可以向消息添加附件或附件列表。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 对象的 `Attachments` 属性包含一组 `Attachment` 对象，表示媒体附件和附加到消息的富卡。 若要向消息添加媒体附件，请为 `message` 活动创建 `Attachment` 对象，并设置 `ContentType`、`ContentUrl` 和 `Name` 属性。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(
    new Attachment { ContentUrl = "imageUrl.png", ContentType = "image/png" }
);

// Send the activity to the user.
await context.SendActivity(activity);
```

消息工厂的 `Attachment` 方法还可以发送附件列表，此列表堆叠在另一个列表上。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and add an attachment.
var activity = MessageFactory.Attachment(new Attachment[]
{
    new Attachment { ContentUrl = "imageUrl1.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl2.png", ContentType = "image/png" },
    new Attachment { ContentUrl = "imageUrl3.png", ContentType = "image/png" }
});

// Send the activity to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要向用户发送单个内容，如图片或视频，可以发送包含在 URL 中的媒体：

```javascript
const {MessageFactory} = require('botbuilder');
let imageOrVideoMessage = MessageFactory.contentUrl('imageUrl.png', 'image/jpeg')

// Send the activity to the user.
await context.sendActivity(imageOrVideoMessage);
```

若要发送附件列表（一个堆叠在另一个上）：<!-- TODO: Convert the hero cards to image attachments in this example. -->

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

let messageWithCarouselOfCards = MessageFactory.list([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

如果附件是图像、音频或视频，连接器服务将以一种让[通道](~/v4sdk/bot-builder-channeldata.md)可以在会话中呈现附件的方式将此附件数据传递给通道。 如果附件是文件，该文件 URL 将呈现为该会话中的超链接。

## <a name="send-a-hero-card"></a>发送英雄卡

除了简单的图像或视频附件，还可以附加英雄卡，这样可以将图像和按钮组合在一个对象中，并将其发送给用户。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要撰写含英雄卡和按钮的消息，可以将 `HeroCard` 附加到消息中：

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "White T-Shirt",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "buy", type: ActionTypes.ImBack, value: "buy")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要向用户发送卡片和按钮，可以将附加 `heroCard` 到消息：

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory} = require('botbuilder');
const {CardFactory} = require('botbuilder');

const message = MessageFactory.attachment(
     CardFactory.heroCard(
        'White T-Shirt',
        ['https://example.com/whiteShirt.jpg'],
        ['buy']
    )
 );

await context.sendActivity(message);
```

---

<!--Lifted from the RESP API documentation-->

富卡包括标题、说明、链接和图像。 消息可以包含多个富卡，以列表格式或轮播格式显示。 Bot Builder SDK 目前支持各种富卡。 若要查看富卡列表和支持它们的通道，请参阅[设计 UX 元素](../bot-service-design-user-experience.md)。

> [!TIP]
> 若要确定通道支持的富卡类型并查看通道如何呈现富卡，请参阅[通道检查器](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-inspector?view=azure-bot-service-4.0)。 有关卡内容限制的信息（例如，最大按钮数或最大标题长度），请参阅通道的文档。

## <a name="process-events-within-rich-cards"></a>处理富卡中的事件

若要处理富卡中的事件，请使用 card action 对象指定当用户单击按钮或点击卡的某个部分时应发生的情况。

若要正常运行，请为卡上的每个可点击项目指定一种操作类型。 此表列出了 card action 对象 type 属性的有效值，并描述了每种类型的 value 属性的预期内容。

| Type | 值 |
| :---- | :---- |
| openUrl | 要在内置浏览器中打开的 URL。 通过打开 URL 响应点击或单击。 |
| imBack | 要发送到机器人的消息文本（来自单击按钮或点击卡的用户）。 通过托管会话的客户端应用程序，所有会话参与者都可看到此消息（从用户到机器人）。 |
| postBack | 要发送到机器人的消息文本（来自单击按钮或点击卡的用户）。 某些客户端应用程序可能会在消息源中显示此文本，所有会话参与者都可看到该文本。 |
| call | 采用 `tel:123123123123` 格式的电话呼叫的目的地。通过发起呼叫来响应点击或单击。|
| playAudio | 要播放的音频的 URL。 通过播放音频响应点击或单击。 |
| playVideo | 要播放的视频的 URL。 通过播放视频响应点击或单击。 |
| showImage | 要显示的图像的 URL。 通过显示图像响应点击或单击。 |
| downloadFile | 要下载的文件的 URL。  通过下载文件响应点击或单击。 |
| signin | 要启动的 OAuth 流的 URL。 通过启动登录响应点击或单击。 |

## <a name="hero-card-using-various-event-types"></a>使用各种事件类型的英雄卡

以下代码显示了使用各种富卡事件的示例。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a Hero card.
var activity = MessageFactory.Attachment(
    new HeroCard(
        title: "Holler Back Buttons",
        images: new CardImage[] { new CardImage(url: "imageUrl.png") },
        buttons: new CardAction[]
        {
            new CardAction(title: "Shout Out Loud", type: ActionTypes.imBack, value: "You can ALL hear me!"),
            new CardAction(title: "Much Quieter", type: ActionTypes.postBack, value: "Shh! My Bot friend hears me."),
            new CardAction(title: "Show me how to Holler", type: ActionTypes.openURL, value: $"https://en.wikipedia.org/wiki/{cardContent.Key}")
        })
    .ToAttachment());

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");
```

```javascript
const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>发送自适应卡

你也可以将自适应卡作为附件发送。 当前并非所有通道都支持自适应卡。 若要查找有关自适应卡通道支持的最新信息，请参阅<a href="http://adaptivecards.io/visualizer/">自适应卡可视化工具</a>。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
若要使用自适应卡，请务必添加 `Microsoft.AdaptiveCards` NuGet 包。


> [!NOTE]
> 应该使用机器人将使用的通道测试此功能，以确定这些通道是否支持自适应卡。

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach an Adaptive Card.
var card = new AdaptiveCard();
card.Body.Add(new TextBlock()
{
    Text = "<title>",
    Size = TextSize.Large,
    Wrap = true,
    Weight = TextWeight.Bolder
});
card.Body.Add(new TextBlock() { Text = "<message text>", Wrap = true });
card.Body.Add(new TextInput()
{
    Id = "Title",
    Value = string.Empty,
    Style = TextInputStyle.Text,
    Placeholder = "Title",
    IsRequired = true,
    MaxLength = 50
});
card.Actions.Add(new SubmitAction() { Title = "Submit", DataJson = "{ Action:'Submit' }" });
card.Actions.Add(new SubmitAction() { Title = "Cancel", DataJson = "{ Action:'Cancel'}" });

var activity = MessageFactory.Attachment(new Attachment(AdaptiveCard.ContentType, content: card));

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {CardFactory} = require("botbuilder");

const message = CardFactory.adaptiveCard({
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.0",
    "type": "AdaptiveCard",
    "speak": "Your flight is confirmed for you from San Francisco to Amsterdam on Friday, October 10 8:30 AM",
    "body": [
        {
            "type": "TextBlock",
            "text": "Passenger",
            "weight": "bolder",
            "isSubtle": false
        },
        {
            "type": "TextBlock",
            "text": "Sarah Hum",
            "separator": true
        },
        {
            "type": "TextBlock",
            "text": "1 Stop",
            "weight": "bolder",
            "spacing": "medium"
        },
        {
            "type": "TextBlock",
            "text": "Fri, October 10 8:30 AM",
            "weight": "bolder",
            "spacing": "none"
        },
        {
            "type": "ColumnSet",
            "separator": true,
            "columns": [
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "San Francisco",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "SFO",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": "auto",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": " "
                        },
                        {
                            "type": "Image",
                            "url": "http://messagecardplayground.azurewebsites.net/assets/airplane.png",
                            "size": "small",
                            "spacing": "none"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "Amsterdam",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "size": "extraLarge",
                            "color": "accent",
                            "text": "AMS",
                            "spacing": "none"
                        }
                    ]
                }
            ]
        },
        {
            "type": "ColumnSet",
            "spacing": "medium",
            "columns": [
                {
                    "type": "Column",
                    "width": "1",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "Total",
                            "size": "medium",
                            "isSubtle": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": 1,
                    "items": [
                        {
                            "type": "TextBlock",
                            "horizontalAlignment": "right",
                            "text": "$1,032.54",
                            "size": "medium",
                            "weight": "bolder"
                        }
                    ]
                }
            ]
        }
    ]
});

// send adaptive card as attachment 
await context.sendActivity({ attachments: [message] })
```

---

## <a name="send-a-carousel-of-cards"></a>发送采用轮播布局的卡

消息还可以包括采用轮播布局的多个附件，此布局并排放置附件并允许用户滚动。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

## <a name="additional-resources"></a>其他资源

[使用 Channel Inspector 预览功能](../bot-service-channel-inspector.md)

---