---
title: 配置在一个或多个通道上运行的机器人 | Microsoft Docs
description: 了解如何使用 Bot Framework 门户配置在一个或多个通道上运行的机器人。
keywords: 机器人通道, 配置, cortana, facebook messenger, kik, slack, Skype, Azure 门户
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/22/2018
ms.openlocfilehash: 8c7f10f5f7ce507c367f12b01b4b28ff1cc01c0f
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/05/2018
ms.locfileid: "47418819"
---
# <a name="connect-a-bot-to-channels"></a>将机器人连接到通道

通道连接机器人和通信应用。 配置机器人以连接到希望在其上可用的通道。 通过 Azure 门户配置的 Bot Framework Service 将机器人连接到这些通道，并促进机器人与用户之间的通信。 可以连接到许多常用服务，如 [Cortana](bot-service-channel-connect-cortana.md)、[Facebook Messenger](bot-service-channel-connect-facebook.md)、[Kik](bot-service-channel-connect-kik.md) 和 [Slack](bot-service-channel-connect-slack.md) 等。 [Skype](https://dev.skype.com/bots) 和网上聊天为预配置。 除了通过 Bot Connector 服务提供的标准通道，还可以使用 Direct Line 作为通道，将机器人连接到你自己的客户端应用程序。

可以借助 Bot Framework Service 规范化机器人发送到通道的消息，采用与通道无关的方式开发机器人。 这涉及将其从 Bot Builder 架构转换为通道架构。 但是，如果通道不支持 Bot Builder 架构的所有方面，该服务会尝试将消息转换为通道支持的格式。 例如，如果机器人向短信通道发送的消息中包含一张带有操作按钮的卡，Connector 可能会将该卡作为一个图像发送，并包含这些操作作为消息文本中的链接。



对于大多数通道，必须提供通道配置信息以在通道上运行机器人。 大多数通道要求机器人在通道上具有帐户，而其他通道（如 Facebook Messenger）则要求机器人还需在通道上注册一个应用程序。

若要配置机器人以连接到通道，请完成以下步骤：

1. 登录到 <a href="https://portal.azure.com" target="_blank">Azure 门户</a>。
1. 选择要配置的机器人。
3. 在“机器人服务”边栏选项卡，单击“机器人管理”下的“通道”。
4. 单击要添加到机器人的通道图标。

![连接到通道](./media/channels/connect-to-channels.png)

配置通道后，该通道上的用户便可以开始使用机器人。

## <a name="publish-a-bot"></a>发布机器人

发布过程对于每个通道都是不同的。

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>其他资源
SDK 包含可以用于生成机器人的示例。 访问 [GitHub](https://github.com/Microsoft/BotBuilder-samples) 存储库就可以看到示例列表。
