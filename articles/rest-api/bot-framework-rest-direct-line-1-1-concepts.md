---
title: Bot Framework Direct Line API 1.1 中的重要概念 | Microsoft Docs
description: 了解 Bot Framework Direct Line API 1.1 中的重要概念。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: f37d8e9215b0a2cd640431f237d1b8c53fad576b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297645"
---
# <a name="key-concepts-in-direct-line-api-11"></a>Direct Line API 1.1 中的重要概念

可以使用 Direct Line API 在机器人和自己的客户端应用程序之间启用通信。 

> [!IMPORTANT]
> 本文介绍 Direct Line API 1.1 中的重要概念，并提供有关相关开发人员资源的信息。 如果要在客户端应用程序和机器人之间创建新连接，请改用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-concepts.md)。

## <a name="authentication"></a>身份验证

可以使用从 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>中的 Direct Line 通道配置页获取的**机密**或使用在运行时获得的**令牌**来对 Direct Line API 1.1 请求进行身份验证。  有关详细信息，请参阅[身份验证](bot-framework-rest-direct-line-1-1-authentication.md)。

## <a name="starting-a-conversation"></a>开始聊天

Direct Line 聊天由客户端显式打开，只要机器人和客户端参与并拥有有效凭据，就可以运行。 有关详细信息，请参阅[开始聊天](bot-framework-rest-direct-line-1-1-start-conversation.md)。

## <a name="sending-messages"></a>发送消息

使用 Direct Line API 1.1，客户端可以通过发出 `HTTP POST` 请求向机器人发送消息。 客户端可以按请求发送单个消息。 有关详细信息，请参阅[向机器人发送消息](bot-framework-rest-direct-line-1-1-send-message.md)。

## <a name="receiving-messages"></a>接收消息

使用 Direct Line API 1.1，客户端可以通过使用 `HTTP GET` 请求进行轮询来接收消息。 为响应每个请求，客户端可以在 `MessageSet` 中从机器人接收多个消息。 有关详细信息，请参阅[从机器人接收消息](bot-framework-rest-direct-line-1-1-receive-messages.md)。

## <a name="developer-resources"></a>开发人员资源

### <a name="client-library"></a>客户端库

Bot Framework 提供了一个客户端库，以便于通过 C# 访问 Direct Line API 1.1。 若要在 Visual Studio 项目中使用客户端库，请安装 `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">v1.x NuGet 包</a>。 

作为使用 C# 客户端库的替代方法，可以使用 <a href="https://docs.botframework.com/en-us/restapi/directline/swagger.json" target="_blank">Direct Line API 1.1 Swagger 文件</a>以所选的语言生成自己的客户端库。

### <a name="web-chat-control"></a>网上聊天控件 

Bot Framework 提供了一个控件，用于将 Direct-Line 驱动的机器人嵌入到客户端应用程序中。 有关详细信息，请参阅 <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework 网上聊天控件</a>。
