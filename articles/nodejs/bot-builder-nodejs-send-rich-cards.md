---
title: 向消息添加资讯卡附件 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 发送有吸引力的交互式丰富卡片。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cba67dc4da5a0b505b4f91f9cbf7fbc0a47b8974
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404805"
---
# <a name="add-rich-card-attachments-to-messages"></a>向消息添加资讯卡附件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]


> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

多个通道（例如 Skype 和 Facebook）支持使用交互式按钮（用户单击这些按钮可以启动某个操作）将丰富的图形卡片发送到用户。 SDK 提供多个消息与卡片生成器类用于创建和发送卡片。 Bot Framework 连接器服务将使用通道原生的架构来呈现这些卡片，并支持跨平台通信。 如果通道（例如 SMS）不支持卡片，Bot Framework 会尽量向用户呈现合理的体验。 

## <a name="types-of-rich-cards"></a>丰富卡片的类型 
Bot Framework 目前支持八个类型的富卡： 

| 卡类型 | 说明 |
|------|------|
| <a href="/adaptive-cards/get-started/bots">自适应卡片</a> | 一种可以包含文本、语音、图像、按钮和输入字段的任意组合的可自定义卡片。  请参阅[每个通道的支持](/adaptive-cards/get-started/bots#channel-status)。 |
| [动画卡片][animationCard] | 一种可播放动态 GIF 或短视频的卡。 |
| [音频卡片][audioCard] | 一种可播放音频文件的卡。 |
| [英雄卡片][heroCard] | 一种通常包含单个大图像、一个或多个按钮和文本的卡。 |
| [缩略图卡片][thumbnailCard] | 一种通常包含单个缩略图图像、一个或多个按钮和文本的卡。|
| [收据卡片][receiptCard] | 一种让机器人能够向用户提供收据的卡。 它通常包含要包括在收据中的项目列表、税款和总计信息以及其他文本。 |
| [登录卡片][signinCard] | 一种让机器人能够请求用户登录的卡。 它通常包含文本和一个或多个按钮，用户可以单击这些按钮来启动登录进程。 |
| [视频卡片][videoCard] | 一种可播放视频的卡。 |

## <a name="send-a-carousel-of-hero-cards"></a>发送轮播的 Hero 卡片
以下示例显示了一家虚构 T 恤公司的机器人，其中演示了如何在用户讲出“展示衬衫”后发送轮播的卡片。 

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```
此示例使用 [Message][Message] 类来生成轮播内容。  
轮播内容由 [HeroCard][heroCard] 类的列表构成，这些类包含触发商品购买活动的图像、文本和单个按钮。  
单击“购买”按钮会触发消息发送，因此，我们需要添加另一个对话来捕获按钮单击动作。  

## <a name="handle-button-input"></a>处理按钮输入

每当收到以“buy”或“add”开头，并后接包含单词“shirt”的内容的消息时，将触发 `buyButtonClick` 对话。 该对话首先使用几个正则表达式来查找用户请求的衬衫颜色和可选尺寸。
这样就提高了灵活性，使你可以支持按钮单击，以及用户讲述的，类似于“请将一件大号灰色衬衫添加到我的购物车”的自然语言消息。
如果颜色有效但尺寸未知，机器人会提示用户在将商品添加到购物车之前，从列表中选择一个尺寸。 机器人获得所需的所有信息后，会将商品放到使用 **session.userData** 保存的购物车中，然后向用户发送确认消息。

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = { 
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }   
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 

> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user.   
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  

-->
## <a name="add-a-message-delay-for-image-downloads"></a>针对图像下载添加消息延迟
某些通道往往会在向用户显示消息之前下载图像，因此，如果发送一条包含图像的消息，紧接着发送一条不带图像的消息，则有时你会看到用户馈送内容中的消息发生翻转。 为了尽量减少出现这种情况的可能性，请确保图像来自内容分发网络 (CDN)，并避免使用过大的图像。 在极端的情况下，甚至需要在包含图像的消息与其后面的内容之间插入 1-2 秒的延迟。 可以调用 **session.sendTyping()** 在开始延迟之前发送一个键入指示符，使用户觉得这种延迟符合常态。 

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework 实施某种形式的批处理，以尝试避免机器人提供的多个消息显示失序。 <!-- Unfortunately, not all channels can guarantee this. --> 当机器人向用户发送多个回复时，每条消息将自动分组到一批，并以一组的形式传送给用户，以保持消息的原始顺序。 在每次调用 **session.send()** 之后、初始化下一个 **send()** 调用之前，此自动批处理默认会等待 250 毫秒。

可以配置消息批处理延迟。 若要禁用 SDK 的自动批处理逻辑，请将默认延迟设置为较大的数字，并结合传送该批后调用的回调来手动调用 **sendBatch()** 。

## <a name="send-an-adaptive-card"></a>发送自适应卡片

自适应卡片可以包含文本、语音、图像、按钮和输入域的任意组合。 自适应卡片是使用<a href="http://adaptivecards.io" target="_blank">自适应卡片</a>中指定的 JSON 格式创建的，因此你可以完全控制卡片的内容和格式。 

若要使用 Node.js 创建自适应卡片，请利用<a href="http://adaptivecards.io" target="_blank">自适应卡片</a>站点中的信息了解自适应卡片的架构，探索自适应卡片元素，并查看 JSON 示例（可用于创建具有不同组合形式和复杂性的卡片）。 此外，可以使用交互式可视化工具来设计自适应卡片有效负载和预览卡片输出。

此代码示例演示如何创建包含用于日历提醒的自适应卡片的消息： 

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

生成的卡包含三个文本块、一个输入字段（选择列表）和三个按钮：

![自适应卡片日历提醒](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>其他资源

* [使用 Channel Inspector 预览功能][inspector]
* <a href="http://adaptivecards.io" target="_blank">自适应卡片</a>
* [AnimationCard][animationCard]
* [AudioCard][audioCard]
* [HeroCard][heroCard]
* [ThumbnailCard][thumbnailCard]
* [ReceiptCard][receiptCard]
* [SigninCard][signinCard]
* [VideoCard][videoCard]
* [消息][Message]
* [如何发送附件](bot-builder-nodejs-send-receive-attachments.md)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.animationcard.html 

[audioCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.audiocard.html 

[heroCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html

[thumbnailCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html 

[receiptCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html 

[signinCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.signincard.html 

[videoCard]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.videocard.html

[inspector]: ../bot-service-channel-inspector.md
