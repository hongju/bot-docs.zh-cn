---
title: Bot Connector 服务和 Bot State 服务中的重要概念 | Microsoft Docs
description: 了解 Bot Framework 的 Bot Connector 服务和 Bot State 服务中的重要概念。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 20ae641a23399f5ee10aed9b31c4521f355903cf
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998945"
---
# <a name="key-concepts"></a>关键概念

可使用 Bot Connector 服务和 Bot State 服务通过多个通道（如 Skype、电子邮件、Slack 等）与用户进行通信。 本文介绍了 Bot Connector 服务和 Bot State 服务中的重要概念。

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境，该 API 可能会在将来的版本中弃用。 建议更新机器人代码以使用内存中存储进行测试，或者将 Azure 扩展之一用于生产机器人。 有关详细信息，请参阅针对 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 实现的“管理状态数据”主题。

## <a name="bot-connector-service"></a>Bot Connector 服务

Bot Connector 服务使机器人能够与 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>中配置的通道交换消息。 它通过 HTTPS 使用行业标准 REST 和 JSON，并通过 JWT 持有者令牌进行身份验证。 有关如何使用 Bot Connector 服务的详细信息，请参阅[身份验证](bot-framework-rest-connector-authentication.md)以及本部分中的其余文章。

### <a name="activity"></a>活动

Bot Connector 服务通过传递 [Activity][Activity] 对象在机器人和通道（用户）之间交换信息。 最常见的活动类型是“消息”，但是还有其他活动类型可用于将各种类型的信息传达给机器人或通道。 有关 Bot Connector 服务中“活动”的详细信息，请参阅[活动概述](bot-framework-rest-connector-activities.md)。

## <a name="bot-state-service"></a>Bot State 服务

Bot State 服务使机器人能够在特定聊天的上下文中存储和检索与用户、聊天或特定用户关联的状态数据。 它通过 HTTPS 使用行业标准 REST 和 JSON，并通过 JWT 持有者令牌进行身份验证。 使用 Bot State 服务保存的任何数据将存储在 Azure 中并进行静态加密。

Bot State 服务仅在与 Bot Connector 服务结合时有用。 也就是说，它仅能用于存储与机器人使用 Bot Connector 服务进行的聊天相关的状态数据。 有关如何使用 Bot State 服务的详细信息，请参阅[身份验证](bot-framework-rest-connector-authentication.md)和[管理状态数据](bot-framework-rest-state.md)。

## <a name="authentication"></a>身份验证

Bot Connector 服务和 Bot State 服务均可通过 JWT 持有者令牌实现身份验证。 有关如何验证机器人发送到 Bot Framework 的出站请求，以及如何验证机器人从 Bot Framework 接收的入站请求的详细信息，请参阅[身份验证](bot-framework-rest-connector-authentication.md)。 

## <a name="client-libraries"></a>客户端库

Bot Framework 提供了可用于在 C# 或 Node.js 中生成机器人的客户端库。 

- 若要使用 C# 生成机器人，请使用 [Bot Builder SDK for C#](../dotnet/bot-builder-dotnet-overview.md)。 
- 若要使用 Node.js 生成机器人，请使用 [Bot Builder SDK for Node.js](../nodejs/index.md)。 

除了对 Bot Connector 服务和 Bot State 服务进行建模之外，每个 Bot Builder SDK 还提供了一个强大的系统，用于构建封装聊天逻辑的对话、简单事情的内置提示（如是/否、字符串、数字和枚举）、对强大 AI 框架的内置支持，例如 <a href="https://www.luis.ai/" target="_blank">LUIS</a> 等等。 

> [!NOTE]
> 作为使用 C# SDK 或 Node.js SDK 的替代方法，可使用 <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/ConnectorAPI.json" target="_blank">Bot Connector Swagger 文件</a>和 <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/StateAPI.json" target="_blank">Bot State Swagger 文件</a>以所选的语言生成自己的客户端库。

## <a name="additional-resources"></a>其他资源

通过查看本部分中的文章，从[身份验证](bot-framework-rest-connector-authentication.md)开始，了解有关使用 Bot Connector 服务和 Bot State 服务生成机器人的详细信息。 如果遇到问题或对 Bot Connector 服务和 Bot State 服务有任何建议，请参阅[支持](../bot-service-resources-links-help.md)了解可用资源列表。 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object