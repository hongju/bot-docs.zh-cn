---
title: Bot Builder SDK for Node.js 的示例机器人 | Microsoft Docs
description: 探索大量示例机器人，借助 Bot Builder SDK for Node.js 启动机器人开发。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0eb14f4bf52a293d3405762c786259f771aecad6
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298177"
---
# <a name="bot-builder-sdk-for-nodejs-samples"></a>Bot Builder SDK for Node.js 示例

这些示例演示了以任务为中心的机器人，展示如何利用 Bot Builder SDK for Node.js 中的功能。 这些示例可助你快速入门，构建具有丰富功能的强大机器人。

## <a name="get-the-samples"></a>获取示例
要获取示例，请使用 Git 克隆 [ BotBuilder-Samples ](https://github.com/Microsoft/BotBuilder-Samples) GitHub 存储库。

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

使用 Bot Builder SDK for Node.js 构建的示例机器人组织在 Node 目录中。

此外可以在 GitHub 上查看这些示例，并将其直接部署到 Azure。

## <a name="core"></a>核心
这些示例展示了用于构建丰富且功能强大的机器人的基本技术。

示例 | Description
------------ | ------------- 
[发送附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-SendAttachment) | 一个向用户发送简单的媒体附件（图像）的示例机器人。 
[接收附件](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ReceiveAttachment) | 一个接收用户发送的附件并下载它们的示例机器人。 
[创建新聊天](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CreateNewConversation)  | 一个使用先前存储的用户地址开始新聊天的示例机器人。
[获取聊天的成员](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-GetConversationMembers) | 一个检索聊天的成员列表并检测何时发生更改的示例机器人。 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLine) | 一个使用 Direct Line API 进行相互通信的示例机器人和自定义客户端。 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-DirectLineWebSockets) | 一个使用 Direct Line API 和 WebSocket 进行相互通信的示例机器人和自定义客户端。 
[多对话](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-MultiDialogs) | 一个显示各种对话的示例机器人。
[状态 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-State) | 一个跟踪聊天的上下文的无状态示例机器人。
[自定义状态 API](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-CustomState) | 一个使用自定义存储提供程序跟踪聊天上下文的无状态示例机器人。
[ChannelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) | 一个使用 ChannelData 将本机元数据发送到 Facebook 的示例机器人。
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-AppInsights) | 一个将遥测数据记录到 Application Insights 实例的示例机器人。

## <a name="search"></a>搜索
此示例演示如何在数据驱动型机器人中利用 Azure 搜索。

示例 | Description
------------ | -------------
[Azure 搜索](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search) | 两个示例智能机器人，可帮助用户在大量内容中导航。


## <a name="cards"></a>卡片
这些示例展示了如何发送 Bot Framework 中的资讯卡。

示例 | Description
------------ | -------------
[资讯卡](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-RichCards) | 一个可以发送几种不同类型的资讯卡的示例机器人。
[卡轮播](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/cards-CarouselCards) | 一个使用轮播布局在单个消息中发送多张资讯卡的示例机器人。

## <a name="intelligence"></a>Intelligence
这些示例展示了如何将人工智能功能添加到使用必应和 Microsoft 认知服务 API 的智能机器人。

示例 | Description
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-LUIS) | 一个使用 LuisDialog 与 LUIS.ai 应用程序集成的示例机器人。
[图像标题](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-ImageCaption) | 一个使用 Microsoft 认知服务视觉 API 获取图像标题的示例机器人。
[语音转文本](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SpeechToText)  | 一个使用必应语音 API 从音频获取文本的示例机器人。
[类似产品](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-SimilarProducts) | 一个使用必应图像搜索 API 查找外观相似产品的示例机器人。 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/intelligence-Zummer) | 一个使用必应搜索 API 查找维基百科文章的示例机器人。

## <a name="reference-implementation"></a>参考实现
此示例旨在展示端到端方案。 如果希望在机器人中实现更复杂的功能，那么这是一个很好的代码片段源。


示例 | Description
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-ContosoFlowers) | 一个使用 Bot Framework 的众多功能的示例机器人。

