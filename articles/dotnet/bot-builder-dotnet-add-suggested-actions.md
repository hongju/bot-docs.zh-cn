---
title: 向消息添加建议的操作 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 向消息添加建议的操作。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 63e5f4b8ac86b6b0e29d09796dbe74295bf3e213
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297771"
---
# <a name="add-suggested-actions-to-messages"></a>向消息添加建议的操作
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> 使用 [Channel Inspector][channelInspector] 查看建议的操作在各种通道上的外观和工作方式。

## <a name="send-suggested-actions"></a>发送建议的操作

若要向消息添加建议的操作，请将活动的 `SuggestedActions` 属性设置为 [CardAction][cardAction] 对象的列表，这些对象表示要呈现给用户的按钮。 

此代码示例演示如何创建向用户显示三个建议操作的消息：

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

当用户点击建议的操作之一时，机器人将从用户收到包含相应操作的 `Value` 的消息。

## <a name="additional-resources"></a>其他资源

- [使用 Channel Inspector 预览功能][inspector]
- [活动概述](bot-builder-dotnet-activities.md)
- [创建消息](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">活动类</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 接口</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">CardAction 类</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">SuggestedActions 类</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md


