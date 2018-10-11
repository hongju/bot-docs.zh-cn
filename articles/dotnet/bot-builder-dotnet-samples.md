---
title: Bot Builder SDK for .NET 的示例机器人 | Microsoft Docs
description: 探索可以使用 Bot Builder SDK for .NET 帮助启动机器人开发的示例机器人。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/13/2018
ms.openlocfilehash: 9f80fc551cac2f1994b0d398cd640d2ad3d78e61
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707373"
---
# <a name="bot-builder-sdk-for-net-samples"></a>Bot Builder SDK for .NET 示例

::: moniker range="azure-bot-service-3.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

这些示例展示了以任务为中心的机器人，这些机器人展示了如何利用 Bot Builder SDK for .NET 的特性。 这些示例可助你快速入门，构建具有丰富功能的强大机器人。

## <a name="get-the-samples"></a>获取示例
若要获取示例，请使用 Git 克隆 [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 存储库。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

使用 Bot Builder SDK for .NET 构建的示例机器人在 CSharp 目录中进行组织。

此外可以在 GitHub 上查看这些示例，并将其直接部署到 Azure。

## <a name="core"></a>核心
这些示例展示了用于构建丰富且功能强大的机器人的基本技术。

示例 | Description
------------ | ------------- 
[发送附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | 一个向用户发送简单的媒体附件（图像）的示例机器人。 
[接收附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | 一个接收用户发送的附件并下载它们的示例机器人。 
[创建新聊天](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | 一个使用先前存储的用户地址开始新聊天的示例机器人。
[获取聊天的成员](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | 一个检索聊天的成员列表并检测何时发生更改的示例机器人。 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | 一个使用 Direct Line API 进行相互通信的示例机器人和自定义客户端。 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | 一个使用 Direct Line API 和 WebSocket 进行相互通信的示例机器人和自定义客户端。 
[多对话](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | 一个显示各种对话的示例机器人。
[状态 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | 一个跟踪聊天的上下文的无状态示例机器人。
[自定义状态 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | 一个使用自定义存储提供程序跟踪聊天上下文的无状态示例机器人。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | 一个使用 ChannelData 将本机元数据发送到 Facebook 的示例机器人。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | 一个将遥测数据记录到 Application Insights 实例的示例机器人。

## <a name="search"></a>搜索
此示例演示如何在数据驱动型机器人中利用 Azure 搜索。

示例 | Description
------------ | -------------
[Azure 搜索](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | 两个示例智能机器人，可帮助用户在大量内容中导航。


## <a name="cards"></a>卡片
这些示例展示了如何发送 Bot Framework 中的资讯卡。

示例 | Description
------------ | -------------
[资讯卡](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | 一个可以发送几种不同类型的资讯卡的示例机器人。
[卡轮播](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | 一个使用轮播布局在单个消息中发送多张资讯卡的示例机器人。

## <a name="intelligence"></a>Intelligence
这些示例展示了如何将人工智能功能添加到使用必应和 Microsoft 认知服务 API 的智能机器人。

示例 | Description
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | 一个使用 LuisDialog 与 LUIS.ai 应用程序集成的示例机器人。
[图像标题](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | 一个使用 Microsoft 认知服务视觉 API 获取图像标题的示例机器人。
[语音转文本](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | 一个使用必应语音 API 从音频获取文本的示例机器人。
[类似产品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | 一个使用必应图像搜索 API 查找外观相似产品的示例机器人。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | 一个使用必应搜索 API 查找维基百科文章的示例机器人。

## <a name="reference-implementation"></a>参考实现
此示例旨在展示端到端方案。 如果希望在机器人中实现更复杂的功能，那么这是一个很好的代码片段源。


示例 | Description
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | 一个使用 Bot Framework 的众多功能的示例机器人。

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot Builder 示例存储库中的示例演示了以任务为中心的机器人，这些机器人展示了如何利用用于 .NET 的 SDK 中提供的功能。 这些示例可助你快速入门，构建具有丰富功能的强大机器人。 有关示例列表和其他信息，请参阅[自述](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md)文件。

若要获取示例，请使用 Git 克隆 [botbuilder-samples](https://github.com/Microsoft/botbuilder-samples) GitHub 存储库。
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
```

::: moniker-end

