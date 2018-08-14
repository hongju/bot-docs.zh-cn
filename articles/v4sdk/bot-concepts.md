---
title: 通道和 Bot Connector 服务 | Microsoft Docs
description: Bot Builder SDK 的关键概念。
keywords: 活动, 会话
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/28/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99c9267fe41734a41243461fafa98c7116162984
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298443"
---
# <a name="channels-and-the-bot-connector-service"></a>通道和 Bot Connector 服务

通道是指用户用于连接到机器人的终结点，例如 Facebook、Skype、电子邮件、Slack 等。通过 [Azure 门户](https://portal.azure.com)配置的 Bot Connector 服务将机器人连接到这些通道，并促进机器人与用户之间的通信。 

除了 Bot Connector 服务提供的标准通道，你还可以使用 [Direct Line](bot-builder-howto-direct-line.md) 作为通道，将机器人连接到你自己的客户端应用程序。

可以借助 Bot Connector 服务规范化机器人发送到通道的消息，采用与通道无关的方式开发机器人。 这涉及将其从 Bot Builder 架构转换为通道架构。 但是，如果通道不支持 Bot Builder 架构的所有方面，该服务会尝试将消息转换为通道支持的格式。 例如，如果机器人向短信通道发送的消息中包含一张带有操作按钮的卡，Connector 可能会将该卡作为一个图像发送，并包含这些操作作为消息文本中的链接。

## <a name="activities-and-conversations"></a>活动和聊天


Bot Connector 服务使用 JSON 在机器人与用户之间交换信息，Bot Builder SDK 将此信息包装在特定于语言的活动对象中。 在讨论[与机器人的交互](bot-builder-basics.md#interaction-with-your-bot)时提到了[活动类型](../bot-service-activities-entities.md)，最常见的活动类型是消息，但其他活动类型也很重要。 其中包括聊天更新、联系人关系更新、删除用户数据、聊天结束、键入、消息回应以及用户可能永远不会看到的其他几个特定于机器人的活动。 有关这些活动以及它们何时发生的详细信息，请参阅我们的活动参考内容。

每个回合及其相关活动均属于逻辑**聊天**，它表示一个或多个机器人与特定用户或用户组之间的交互。 聊天特定于通道，并且具有该通道唯一的 ID。

## <a name="next-steps"></a>后续步骤

你已熟悉机器人的一些关键概念，现在让我们深入了解机器人可能使用的各种聊天形式。

> [!div class="nextstepaction"]
> [聊天形式](bot-builder-conversations.md)
