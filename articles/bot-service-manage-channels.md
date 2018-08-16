---
title: 配置在一个或多个通道上运行的机器人 | Microsoft Docs
description: 了解如何使用 Bot Framework 门户配置在一个或多个通道上运行的机器人。
keywords: 机器人通道, 配置, cortana, facebook messenger, kik, slack, Skype, Azure 门户
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: cb682bf77f801c98d00deffa0fc63249962248cd
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352916"
---
# <a name="connect-a-bot-to-channels"></a>将机器人连接到通道

通道连接 Bot Framework 和通信应用。 配置机器人以连接到希望在其上可用的通道。 例如，连接到 Skype 通道的机器人可以添加到联系人列表，用户可以在 Skype 中与其进行交互。 

通道包括许多常用服务，如 [Cortana](bot-service-channel-connect-cortana.md)、[Facebook Messenger](bot-service-channel-connect-facebook.md)、[Kik](bot-service-channel-connect-kik.md) 和 [Slack](bot-service-channel-connect-slack.md) 等其他服务。 [Skype](https://dev.skype.com/bots) 和网上聊天为预配置。 

在 [Azure 门户](https://portal.azure.com)中，连接到通道的过程简单而快速。

## <a name="get-started"></a>入门

对于大多数通道，必须提供通道配置信息以在通道上运行机器人。 大多数通道要求机器人在通道上具有帐户，而其他通道（如 Facebook Messenger）则要求机器人还需在通道上注册一个应用程序。

若要配置机器人以连接到通道，请完成以下步骤：

1. 登录到 <a href="https://portal.azure.com" target="_blank">Azure 门户</a>。
1. 选择要配置的机器人。
3. 在“机器人服务”边栏选项卡，单击“机器人管理”下的“通道”。
4. 单击要添加到机器人的通道图标。

![连接到通道](~/media/channels/connect-to-channels.png)

配置通道后，该通道上的用户便可以开始使用机器人。

## <a name="publish-a-bot"></a>发布机器人

发布过程对于每个通道都是不同的。

[!INCLUDE [publishing](~/includes/snippet-publish-to-channel.md)]

