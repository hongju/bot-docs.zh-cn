---
title: 向消息添加媒体 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK 向消息添加媒体。
keywords: 媒体, 消息, 图像, 音频, 视频, 文件, MessageFactory, 富卡, 消息, 自适应卡, 英雄卡, 建议的操作
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 87862e8fcbfa357c27a1c8fac0e8dd71d9bc2998
ms.sourcegitcommit: bd4f9669c0d26ac2a4be1ab8e508f163a1f465f3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2018
ms.locfileid: "47430326"
---
# <a name="add-media-to-messages"></a>向消息添加媒体

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

用户与机器人之间的消息交换可以包含媒体附件，例如图像、视频、音频和文件。 Bot Builder SDK 支持向用户发送富消息的任务。 若要确定某个通道（Facebook、Skype、Slack 等）支持的富消息的类型，请查看该通道的文档，了解存在哪些限制。 如需可用卡的列表，请参阅[设计用户体验](../bot-service-design-user-experience.md)。 

## <a name="send-attachments"></a>发送附件

若要向用户发送内容（如图像或视频），可以向消息添加附件或附件列表。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 对象的 `Attachments` 属性包含一组 `Attachment` 对象，表示媒体附件和附加到消息的富卡。 若要向消息添加媒体附件，请为 `message` 活动创建 `Attachment` 对象，并设置 `ContentType`、`ContentUrl` 和 `Name` 属性。 `Activity` 对象的 `Attachments` 属性包含一组 `Attachment` 对象，表示媒体附件和附加到消息的富卡。 若要向消息添加媒体附件，请使用 `Attachment` 方法为 `message` 活动创建 `Attachment` 对象，并设置 `ContentType`、`ContentUrl`、`Name` 属性。 此处显示的源代码基于[处理附件](https://aka.ms/bot-attachments-sample-code)示例。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };
    
// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处显示的源代码基于 [JS 处理附件](https://aka.ms/bot-attachments-sample-code-js)示例。
若要向用户发送单个内容，如图片或视频，可以发送包含在 URL 中的媒体：

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// Call function to get an attachment.
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

如果附件是图像、音频或视频，连接器服务将以一种让[通道](bot-builder-channeldata.md)可以在会话中呈现附件的方式将此附件数据传递给通道。 如果附件是文件，该文件 URL 将呈现为该会话中的超链接。

## <a name="send-a-hero-card"></a>发送英雄卡

除了简单的图像或视频附件，还可以附加英雄卡，这样可以将图像和按钮组合在一个对象中，并将其发送给用户。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要撰写含英雄卡和按钮的消息，可以将 `HeroCard` 附加到消息中。 此处显示的源代码基于[处理附件](https://aka.ms/bot-attachments-sample-code)示例。 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要撰写含英雄卡和按钮的消息，可以将 `HeroCard` 附加到消息中。 此处显示的源代码基于 [JS 处理附件](https://aka.ms/bot-attachments-sample-code-js)示例：

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```
---

## <a name="process-events-within-rich-cards"></a>处理资讯卡中的事件

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

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

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
自适应卡片和 MessageFactory 用于发送丰富消息（包括文本、图像、视频、音频和文件）来与用户通信。 但是，两者存在一些差异。 

首先，只有某些通道支持自适应卡片，有些通道可能只是部分支持自适应卡片。 例如，如果在 Facebook 中发送自适应卡片，则按钮可能不起作用，而文本和图像可正常使用。 MessageFactory 只是 Bot Builder SDK 中的一个帮助器类，可以自动完成创建步骤，并受大多数通道的支持。 

其次，自适应卡片以卡片格式传送消息，通道确定卡片的布局。 MessageFactory 传送消息的格式取决于通道，除非在附件中包含自适应卡片，否则不一定采用卡片格式。 

若要查找有关自适应卡通道支持的最新信息，请参阅<a href="http://adaptivecards.io/visualizer/">自适应卡可视化工具</a>。

若要使用自适应卡，请务必添加 `Microsoft.AdaptiveCards` NuGet 包。 


> [!NOTE]
> 应该使用机器人将使用的通道测试此功能，以确定这些通道是否支持自适应卡。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处显示的源代码基于[使用自适应卡](https://aka.ms/bot-adaptive-cards-sample-code)示例：

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处显示的源代码基于 [JS 使用自适应卡](https://aka.ms/bot-adaptive-cards-js-sample-code)示例：

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
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
可以在此处找到以下卡的示例代码：[C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code) 自适应卡：[C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code)，附件：[C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js)，以及建议的操作：[C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS)。 如需其他示例，请参阅 [GitHub](https://github.com/Microsoft/BotBuilder-Samples) 上的 Bot Builder 示例存储库。
