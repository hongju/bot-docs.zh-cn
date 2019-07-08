---
title: 向消息添加资讯卡附件 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for .NET 向消息添加富卡。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 51bdc5e52bd147747e9d068fc4721ca4b782ef27
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464512"
---
# <a name="add-rich-card-attachments-to-messages"></a>向消息添加资讯卡附件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

用户与机器人之间的消息交换可以包含一个或多个以列表或轮播方式呈现的富卡。 

<a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> 对象的 `Attachments` 属性包含一组 <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a> 对象，表示消息中的富卡和媒体附件。 

> [!NOTE]
> 有关如何向消息添加媒体附件的信息，请参阅[向消息添加媒体附件](bot-builder-dotnet-add-media-attachments.md)。

## <a name="types-of-rich-cards"></a>富卡类型

Bot Framework 目前支持八个类型的富卡： 

| 卡类型 | 说明 |
|----|----|
| <a href="/adaptive-cards/get-started/bots">自适应卡片</a> | 一种可以包含文本、语音、图像、按钮和输入字段的任意组合的可自定义卡片。 请参阅[每个通道的支持](/adaptive-cards/get-started/bots#channel-status)。  |
| [动画卡][animationCard] | 一种可播放动态 GIF 或短视频的卡。 |
| [音频卡][audioCard] | 一种可播放音频文件的卡。 |
| [英雄卡][heroCard] | 一种通常包含单个大图像、一个或多个按钮和文本的卡。 |
| [缩略图卡][thumbnailCard] | 一种通常包含单个缩略图图像、一个或多个按钮和文本的卡。 |
| [收据卡][receiptCard] | 一种让机器人能够向用户提供收据的卡。 它通常包含要包括在收据中的项目列表、税款和总计信息以及其他文本。 |
| [登录卡][signinCard] | 一种让机器人能够请求用户登录的卡。 它通常包含文本和一个或多个按钮，用户可以单击这些按钮来启动登录进程。 |
| [视频卡][videoCard] | 一种可播放视频的卡。 |

> [!TIP]
> 若要以列表格式显示多个富卡，请将活动的 `AttachmentLayout` 属性设置为“list”。 若要以轮播格式显示多个富卡，请将活动的 `AttachmentLayout` 属性设置为“carousel”。 如果通道不支持轮播格式，它将以列表格式显示富卡，即使 `AttachmentLayout` 属性指定“carousel”也是如此。

## <a name="process-events-within-rich-cards"></a>处理富卡中的事件

若要处理富卡中的事件，请定义 `CardAction` 对象以指定当用户单击按钮或点击卡的某个部分时应发生的情况。 每个 `CardAction` 对象包含以下属性：

| 属性 | Type | 说明 | 
|----|----|----|
| Type | 字符串 | 操作类型（下表中指定的某个值） |
| 标题 | 字符串 | 按钮的标题 |
| 映像 | 字符串 | 按钮的图像 URL |
| 值 | 字符串 | 执行指定类型的操作所需的值 |

> [!NOTE]
> 自适应卡片中的按钮不是使用 `CardAction` 对象创建的，而是使用<a href="http://adaptivecards.io" target="_blank">自适应卡片</a>定义的架构创建的。 有关如何向“自适应卡”添加按钮的示例，请参阅[将自适应卡添加到消息](#adaptive-card)。

此表列出了 `CardAction.Type` 的有效值，并为每个类型介绍 `CardAction.Value` 的预期内容：

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | 要在内置浏览器中打开的 URL |
| imBack | 要发送到机器人的消息文本（来自单击按钮或点击卡的用户）。 通过托管会话的客户端应用程序，所有会话参与者都可看到此消息（从用户到机器人）。 |
| postBack | 要发送到机器人的消息文本（来自单击按钮或点击卡的用户）。 某些客户端应用程序可能会在消息源中显示此文本，所有会话参与者都可看到该文本。 |
| call | 格式如下的电话呼叫的目标：电话:123123123123  |
| playAudio | 要播放的音频的 URL |
| playVideo | 要播放的视频的 URL |
| showImage | 要显示的图像的 URL |
| downloadFile | 要下载的文件的 URL |
| signin | 要启动的 OAuth 流的 URL |

## <a name="add-a-hero-card-to-a-message"></a>向消息添加英雄卡

英雄卡通常包含单个大图像、一个或多个按钮和文本。 

此代码示例演示如何创建包含三个以轮播格式呈现的英雄卡的答复消息： 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>向消息添加缩略图卡

缩略图卡通常包含单个缩略图图像、一个或多个按钮和文本。 

此代码示例演示如何创建包含两个以列表格式呈现的缩略图卡的答复消息： 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>向消息添加收据卡

收据卡让机器人能够向用户提供收据。 它通常包含要包括在收据、税款和总信息以及其他文本中的项列表。 

此代码示例演示如何创建包含收据卡的答复消息： 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>向消息添加登录卡

登录卡让机器人能够请求用户登录。 它通常包含文本和一个或多个按钮，用户可以单击这些按钮来启动登录进程。 

此代码示例演示如何创建包含登录卡的答复消息：

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> 向消息添加自适应卡

自适应卡片可包含文本、语音、图像、按钮和输入域的任意组合。 自适应卡使用<a href="http://adaptivecards.io" target="_blank">自适应卡</a>中指定的 JSON 格式创建而成，这让你可以完全控制卡内容和格式。 

若要使用 .NET 创建自适应卡，请安装 `AdaptiveCards` NuGet 包。 然后，利用<a href="http://adaptivecards.io" target="_blank">自适应卡</a>站点中的信息了解自适应卡架构、探索自适应卡元素，并查看 JSON 示例，以便用于创建不同组合和复杂性的卡。 此外，可以使用交互式可视化工具来设计自适应卡有效负载并预览卡输出。

此代码示例演示如何创建包含用于日历提醒的自适应卡片的消息： 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

生成的卡包含三个文本块、一个输入字段（选择列表）和三个按钮：

![自适应卡片日历提醒](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>其他资源

- [使用 Channel Inspector 预览功能][inspector]
- <a href="http://adaptivecards.io" target="_blank">自适应卡片</a>
- [活动概述](bot-builder-dotnet-activities.md)
- [创建消息](bot-builder-dotnet-create-messages.md)
- [向消息添加媒体附件](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
- <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment 类</a>

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md
