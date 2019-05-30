---
title: 向消息添加媒体 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK 向消息添加媒体。
keywords: 媒体, 消息, 图像, 音频, 视频, 文件, MessageFactory, 富卡, 消息, 自适应卡, 英雄卡, 建议的操作
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 844e3d65794e814dc8a1d16e2b7cf0e7a0e75fab
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215399"
---
# <a name="add-media-to-messages"></a>向消息添加媒体

[!INCLUDE[applies-to](../includes/applies-to.md)]

用户与机器人之间的消息交换可以包含媒体附件，例如图像、视频、音频和文件。 Bot Framework SDK 支持向用户发送富消息的任务。 若要确定某个通道（Facebook、Skype、Slack 等）支持的富消息的类型，请查看该通道的文档，了解存在哪些限制。

有关可用卡的示例，请参阅[设计用户体验](../bot-service-design-user-experience.md)。

## <a name="send-attachments"></a>发送附件

若要向用户发送内容（如图像或视频），可以向消息添加附件或附件列表。

有关可用卡的示例，请参阅[设计用户体验](../bot-service-design-user-experience.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Activity` 对象的 `Attachments` 属性包含一组 `Attachment` 对象，表示媒体附件和附加到消息的富卡。 若要向消息添加媒体附件，请为 `reply` 活动创建 `Attachment` 对象（以前是在活动外使用 `CreateReply()` 创建的），并设置 `ContentType`、`ContentUrl` 和 `Name` 属性。

此处显示的源代码基于[处理附件](https://aka.ms/bot-attachments-sample-code)示例。

若要创建回复消息，请定义文本，然后设置附件。 将附件分配到回复的操作对于每个附加类型来说是相同的，但不同附件的设置和定义方法是不同的，如以下代码片段所示。 以下代码为内联附件设置回复：

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=108-109)]

接下来，我们查看附件的类型。 首先是内联附件：

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=165-176)]

然后是上传的附件：

**Bots/AttachmentsBot.cs** [!code-csharp[uploaded attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=179-215)]

最后是 Internet 附件：

**Bots/AttachmentsBot.cs** [!code-csharp[online attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=218-227)]


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处显示的源代码基于 [JS 处理附件](https://aka.ms/bot-attachments-sample-code-js)示例。

若要使用附件，请在机器人中包括以下库：

**bots/attachmentsBot.js** [!code-javascript[attachments libraries](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=4)]

若要创建回复消息，请定义文本，然后设置附件。 将附件分配到回复的操作对于每个附加类型来说是相同的，但不同附件的设置和定义方法是不同的，如以下代码片段所示。 以下代码为内联附件设置回复：

**bots/attachmentsBot.js** [!code-javascript[attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=119,128-129)]

若要向用户发送单个内容，如图片或视频，可以采用多种不同的方式发送媒体。 首先，用作内联附件：

**bots/attachmentsBot.js** [!code-javascript[inline attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=170-179)]

然后是上传的附件：

**bots/attachmentsBot.js** [!code-javascript[uploaded attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=197-215)]

最后是包含在 URL 中的 Internet 附件：

**bots/attachmentsBot.js** [!code-javascript[internet attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=184-191)]

---

如果附件是图像、音频或视频，连接器服务将以一种让[通道](bot-builder-channeldata.md)可以在会话中呈现附件的方式将此附件数据传递给通道。 如果附件是文件，该文件 URL 将呈现为该会话中的超链接。

## <a name="send-a-hero-card"></a>发送英雄卡

除了简单的图像或视频附件，还可以附加英雄卡  ，这样可以将图像和按钮组合在一个对象中，并将其发送给用户。 大多数文本字段支持 Markdown，但支持可能因通道而异。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要撰写含英雄卡和按钮的消息，可以将 `HeroCard` 附加到消息中。 

此处显示的源代码基于[处理附件](https://aka.ms/bot-attachments-sample-code)示例。

**Bots/AttachmentsBot.cs** [!code-csharp[Hero card](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=39-62)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要撰写含英雄卡和按钮的消息，可以将 `HeroCard` 附加到消息中。 

此处显示的源代码基于 [JS 处理附件](https://aka.ms/bot-attachments-sample-code-js)示例。

**bots/attachmentsBot.js** [!code-javascript[hero card](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=148-164)]

---

## <a name="process-events-within-rich-cards"></a>处理资讯卡中的事件

若要处理富卡中的事件，请使用 card action  对象指定当用户单击按钮或点击卡的某个部分时应发生的情况。 每个卡片操作都有类型和值。  

若要正常运行，请为卡上的每个可点击项目指定一种操作类型。 下表列出并描述了可用的操作类型以及应该存在于相关联的值属性中的内容。

| Type | 说明 | 值 |
| :---- | :---- | :---- |
| openUrl | 在内置浏览器中打开一个 URL。 | 要打开的 URL。 |
| imBack | 向机器人发送一条消息，并在聊天中发布一个可见的响应。 | 要发送的消息文本。 |
| postBack | 向机器人发送一条消息，可能不在聊天中发布一个可见的响应。 | 要发送的消息文本。 |
| call | 拨打电话。 | 电话呼叫的目标采用 `tel:123123123123` 格式。 |
| playAudio | 播放音频。 | 要播放的音频的 URL。 |
| playVideo | 播放视频。 | 要播放的视频的 URL。 |
| showImage | 显示图像。 | 要显示的图像的 URL。 |
| downloadFile | 下载一个文件。 | 要下载的文件的 URL。 |
| signin | 启动 OAuth 登录过程。 | 要启动的 OAuth 流的 URL。 |

## <a name="hero-card-using-various-event-types"></a>使用各种事件类型的英雄卡

以下代码显示了使用各种富卡事件的示例。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

如需所有可用卡片的示例，请参阅 [C# 卡片示例](https://aka.ms/bot-cards-sample-code)。

**Cards.cs** [!code-csharp[hero cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=27-40)]

**Cards.cs** [!code-csharp[cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=91-100)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

如需所有可用卡片的示例，请参阅 [JS 卡片示例](https://aka.ms/bot-cards-js-sample-code)。

**dialogs/mainDialog.js** [!code-javascript[hero cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=213-225)]

**dialogs/mainDialog.js** [!code-javascript[sign in cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=266-272)]

---

## <a name="send-an-adaptive-card"></a>发送自适应卡
自适应卡片和 MessageFactory 用于发送丰富消息（包括文本、图像、视频、音频和文件）来与用户通信。 但是，两者存在一些差异。 

首先，只有某些通道支持自适应卡片，有些通道可能只是部分支持自适应卡片。 例如，如果在 Facebook 中发送自适应卡片，则按钮可能不起作用，而文本和图像可正常使用。 MessageFactory 只是 Bot Framework SDK 中的一个帮助器类，可以自动完成创建步骤，并受大多数通道的支持。 

其次，自适应卡片以卡片格式传送消息，通道确定卡片的布局。 MessageFactory 传送消息的格式取决于通道，除非在附件中包含自适应卡片，否则不一定采用卡片格式。 

若要查找有关自适应卡片通道支持的最新信息，请参阅<a href="http://adaptivecards.io/designer/">自适应卡片设计器</a>。

> [!NOTE]
> 应该使用机器人将使用的通道测试此功能，以确定这些通道是否支持自适应卡。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用自适应卡片，请务必添加 `AdaptiveCards` NuGet 包。

此处显示的源代码基于[使用卡片](https://aka.ms/bot-cards-sample-code)示例：

**Cards.cs** [!code-csharp[adaptive cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=13-25)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用自适应卡片，请务必添加 `adaptivecards` npm 包。

此处显示的源代码基于 [JS 使用卡片](https://aka.ms/bot-cards-js-sample-code)示例。 

在这里，自适应卡片存储在其自己的文件中，并包含在我们的机器人中：

**resources/adaptiveCard.json** [!code-json[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/resources/adaptiveCard.json)]

然后，使用 CardFactory 来创建它：

**dialogs/mainDialog.js** [!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=177-179)]

---

## <a name="send-a-carousel-of-cards"></a>发送采用轮播布局的卡

消息还可以包括采用轮播布局的多个附件，此布局并排放置附件并允许用户滚动。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处显示的源代码基于[卡片示例](https://aka.ms/bot-cards-sample-code)。

首先，创建回复并将附件定义为列表。

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=61-66)]

然后，添加附件。 在这里，我们一次添加一个附件，但你可以根据自己的偏好来添加卡片，对列表随意进行操作。

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=105-113)]

添加附件以后，即可发送回复，就像发送任何其他内容一样。

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=117-118)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处显示的源代码基于 [JS 卡片示例](https://aka.ms/bot-cards-js-sample-code)。

若要发送采用轮播布局的卡片，请在发送回复时将附件设置为数组，并将布局类型定义为 `Carousel`：

**dialogs/mainDialog.js** [!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=104-116)]

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>其他资源

有关可用卡的示例，请参阅[设计用户体验](../bot-service-design-user-experience.md)。

有关架构的详细信息，请参阅 [Bot Framework 卡片架构](https://aka.ms/botSpecs-cardSchema)以及“Bot Framework 活动架构”的[“消息活动”部分](https://aka.ms/botSpecs-activitySchema#message-activity)。

| 代码示例 | C# | JS |
| :------ | :----- | :---|
| 卡片 | [C# 示例](https://aka.ms/bot-cards-sample-code) | [JS 示例](https://aka.ms/bot-cards-js-sample-code) |
| 附件 | [C# 示例](https://aka.ms/bot-attachments-sample-code) | [JS 示例](https://aka.ms/bot-attachments-sample-code-js) |
| 建议的操作 | [C# 示例](https://aka.ms/SuggestedActionsCSharp) | [JS 示例](https://aka.ms/SuggestedActionsJS) |

如需其他示例，请参阅 [GitHub](https://aka.ms/bot-samples-readme) 上的 Bot Builder 示例存储库。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [添加按钮以指导用户操作](./bot-builder-howto-add-suggested-actions.md)
