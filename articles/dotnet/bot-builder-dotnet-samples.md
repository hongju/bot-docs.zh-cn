---
title: Bot Builder SDK for .NET 的示例机器人 | Microsoft Docs
description: 探索可以使用 Bot Builder SDK for .NET 帮助启动机器人开发的示例机器人。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/2018
ms.openlocfilehash: 7e19ec2c3e523003c0831544e42eeb88765d8317
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352856"
---
::: moniker range="azure-bot-service-3.0"

# <a name="bot-builder-sdk-for-net-samples"></a>Bot Builder SDK for .NET 示例

这些示例展示了以任务为中心的机器人，这些机器人展示了如何利用 Bot Builder SDK for .NET 的特性。 这些示例可助你快速入门，构建具有丰富功能的强大机器人。

## <a name="get-the-samples"></a>获取示例
若要获取示例，请使用 Git 克隆 [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 存储库。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

使用 Bot Builder SDK for .NET 构建的示例机器人在 CSharp 目录中进行组织。

还可以在 GitHub 上查看示例并直接将它们部署到 Azure。

## <a name="core"></a>核心
这些示例展示了构建丰富且功能强大的机器人的基本技术。

示例 | Description
------------ | ------------- 
[发送附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | 用于向用户发送简单媒体附件（图像）的示例机器人。 
[接收附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | 接收用户发送的附件并下载它们的示例机器人。 
[创建新会话](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | 使用先前存储的用户地址启动新会话的示例机器人。
[获取会话的成员](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | 检索会话的成员列表并检测它何时发生变化的示例机器人。 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | 使用 Direct Line API 相互通信的示例机器人和自定义客户端。 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | 使用 Direct Line API 和 WebSockets 相互通信的示例机器人和自定义客户端。 
[多对话框](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | 显示各种对话框的示例机器人。
[状态 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | 可跟踪会话上下文的无状态示例机器人。
[自定义状态 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | 使用自定义存储提供程序跟踪会话上下文的无状态示例机器人。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | 使用 ChannelData 将本机元数据发送到 Facebook 的示例机器人。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | 将遥测数据记录到 Application Insights 实例的示例机器人。

## <a name="search"></a>搜索
此示例演示如何在数据驱动的机器人中利用 Azure 搜索。

示例 | Description
------------ | -------------
[Azure 搜索](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | 帮助用户导航大量内容的两个示例机器人。


## <a name="cards"></a>卡
这些示例显示了如何在 Bot Framework 中发送富卡。

示例 | Description
------------ | -------------
[富卡](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | 可以发送几种不同类型富卡的示例机器人。
[卡轮播](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | 使用轮播布局在单个消息中发送多个富卡的示例机器人。

## <a name="intelligence"></a>Intelligence
这些示例显示了如何使用必应和 Microsoft 认知服务 API 将人工智能功能添加到机器人。

示例 | Description
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | 使用 LuisDialog 集成 LUIS.ai 应用程序的示例机器人。
[图像标题](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | 使用 Microsoft 认知服务视觉 API 获取图像标题的示例机器人。
[语音转文本](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | 使用必应语音 API 从音频获取文本的示例机器人。
[类似产品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | 使用必应图像搜索 API 查找类似产品的示例机器人。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | 使用必应搜索 API 查找维基百科文章的示例机器人。

## <a name="reference-implementation"></a>引用实现
此示例旨在展示端到端方案。 如果希望在机器人中实现更复杂的功能，那么这是一个很好的代码片段源。


示例 | Description
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | 使用 Bot Framework 的许多功能的示例机器人。

::: moniker-end

::: moniker range="azure-bot-service-4.0"
# <a name="bot-builder-sdk-v4-net-samples"></a>Bot Builder SDK v4 .NET 示例
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

这些示例展示了以任务为中心的机器人，这些机器人展示了如何利用 Bot Builder SDK for .NET 的特性。 这些示例可助你快速入门，构建具有丰富功能的强大机器人。 

注意：SDK v4 正在积极开发中，因此应仅限试验使用。 

若要获取示例，请使用 Git 克隆 [botbuilder-dotnet](https://github.com/Microsoft/botbuilder-dotnet) GitHub 存储库。
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
cd botbuilder-dotnet\samples-final
```
使用 Bot Builder SDK for .NET 构建的示例机器人在 samples-final 目录中进行组织。


::: moniker-end

