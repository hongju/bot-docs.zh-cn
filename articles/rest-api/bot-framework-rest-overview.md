---
title: Bot Framework REST API | Microsoft Docs
description: 开始使用可用于构建机器人和连接到机器人的客户端的 Bot Framework REST API。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: eace4852feaadcc180f7329261990fc8eaf7f522
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224262"
---
# <a name="bot-framework-rest-apis"></a>Bot Framework REST API
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

借助 Bot Framework REST API，可以生成使用 <a href="https://dev.botframework.com/" target="_blank">Bot Framework 门户</a>中配置的通道交换消息、存储和检索状态数据的机器人，并将自己的客户端应用程序连接到机器人。 所有 Bot Framework 服务都通过 HTTPS 使用行业标准 REST 和 JSON。

## <a name="build-a-bot"></a>生成机器人

可以使用 Bot Connector 服务与 Bot Framework 门户中配置的通道交换消息，从而使用任何编程语言创建机器人。 

> [!TIP]
> Bot Framework 提供了可用于在 C# 或 Node.js 中生成机器人的客户端库。 若要使用 C# 生成机器人，请使用 [Bot Framework SDK for C#](../dotnet/bot-builder-dotnet-overview.md)。 若要使用 Node.js 生成机器人，请使用 [Bot Framework SDK for Node.js](../nodejs/index.md)。 

若要了解使用 Bot Connector 服务生成机器人的详细信息，请参阅[关键概念](bot-framework-rest-connector-concepts.md)。

## <a name="build-a-client"></a>生成客户端

可使用 Direct Line API 启用自己的客户端应用程序以与机器人进行通信。 Direct Line API 实现了一种使用标准机密/令牌模式并提供稳定架构的身份验证机制，即使机器人更改其协议版本也是如此。 若要了解有关使用 Direct Line API 启用客户端与机器人之间通信的详细信息，请参阅[关键概念](bot-framework-rest-direct-line-3-0-concepts.md)。 