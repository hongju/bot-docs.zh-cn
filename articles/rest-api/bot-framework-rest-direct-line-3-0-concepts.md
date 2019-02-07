---
title: Bot Framework Direct Line API 3.0 中的重要概念 | Microsoft Docs
description: 了解 Bot Framework Direct Line API 3.0 中的重要概念。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/06/2019
ms.openlocfilehash: 94ef9cc221e67f4f3762eb7a1a006a915e3c5307
ms.sourcegitcommit: fd60ad0ff51b92fa6495b016e136eaf333413512
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/06/2019
ms.locfileid: "55764095"
---
# <a name="key-concepts-in-direct-line-api-30"></a>Direct Line API 3.0 中的重要概念

可使用 Direct Line API 在机器人和自己的客户端应用程序之间实现通信。 本文介绍 Direct Line API 3.0 中的重要概念，同时介绍了相关的开发人员资源。

## <a name="authentication"></a>身份验证

可通过以下方式对 Direct Line API 3.0 请求进行身份验证：使用从 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>的 Direct Line 通道配置页中获取的密钥，或使用在运行时获得的令牌。 有关详细信息，请参阅[身份验证](bot-framework-rest-direct-line-3-0-authentication.md)。

## <a name="starting-a-conversation"></a>开始聊天

Direct Line 聊天由客户端显式发起，只要机器人和客户端参与并拥有有效凭据，就可以运行。 有关详细信息，请参阅[开始聊天](bot-framework-rest-direct-line-3-0-start-conversation.md)。

## <a name="sending-messages"></a>发送消息

使用 Direct Line API 3.0，客户端可通过发出 `HTTP POST` 请求向机器人发送消息。 客户端可为每个请求发送一条消息。 有关详细信息，请参阅[向机器人发送活动](bot-framework-rest-direct-line-3-0-send-activity.md)。

## <a name="receiving-messages"></a>接收消息

使用 Direct Line API 3.0，客户端可通过 `WebSocket` 流或通过发出 `HTTP GET` 请求来接收机器人发出的消息。 使用其中任一技术，客户端都可一次接收多条来自机器人的消息，作为 `ActivitySet` 的一部分。 有关详细信息，请参阅[从机器人接收活动](bot-framework-rest-direct-line-3-0-receive-activities.md)。

## <a name="developer-resources"></a>开发人员资源

### <a name="client-libraries"></a>客户端库

Bot Framework 提供了客户端库，帮助用户通过 C# 和 Node.js 访问 Direct Line API 3.0。 

- 要在 Visual Studio 项目中使用 .NET 客户端库，请安装 `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">NuGet 包</a>。 

- 要使用 Node.js 客户端库，请使用 <a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a> 安装 `botframework-directlinejs` 库（或<a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">下载</a>源）。

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>代码示例

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder-Samples</a> GitHub 存储库包含多个示例，它们演示了如何结合使用 Direct Line API 3.0 与 C# 和 Node.js。

| 示例 | 语言 | 说明 |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">Direct Line 机器人示例</a> | C# | 使用 Direct Line API 相互通信的示例机器人和自定义客户端。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">Direct Line 机器人示例（使用客户端 Websocket）</a> | C# | 使用 Direct Line API 和 Websocket 相互通信的示例机器人和自定义客户端。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">Direct Line 机器人示例</a> | JavaScript | 使用 Direct Line API 相互通信的示例机器人和自定义客户端。 |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">Direct Line 机器人示例（使用客户端 Websocket）</a> | JavaScript | 使用 Direct Line API 和 Websocket 相互通信的示例机器人和自定义客户端。 |

::: moniker-end

### <a name="web-chat-control"></a>网上聊天控件 

Bot Framework 提供了一个控件，用于将 Direct-Line 驱动的机器人嵌入到客户端应用程序中。 有关详细信息，请参阅 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework 网上聊天控件</a>。
