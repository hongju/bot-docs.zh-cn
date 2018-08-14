---
title: 为 Telegram 创建机器人 |Microsoft Docs
description: 了解如何配置到 Telegram 的机器人连接。
keywords: 配置机器人, Telegram, 机器人通道, Telegram 机器人, 访问令牌
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ccd6bbbf30a9af83d6c687ee68f94d2f31c1c9cd
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297870"
---
# <a name="connect-a-bot-to-telegram"></a>将机器人连接到 Telegram

可配置机器人，使其与使用 Telegram 消息传递应用的用户进行通信。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>访问 Bot Father 来新建一个 Telegram 机器人

使用 Bot Father <a href="https://telegram.me/botfather" target="_blank">创建新的 Telegram 机器人</a>。

![访问 Bot Father](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>创建新的 Telegram 机器人
要创建新的 Telegram 机器人，请发送命令 `/newbot`。

![创建新的机器人](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>指定易记名称

为 Telegram 机器人提供易记名称。

![为机器人提供易记名称](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>指定用户名

为 Telegram 机器人提供唯一用户名。

![为机器人提供唯一名称](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>复制访问令牌

复制 Telegram 机器人的访问令牌。

![复制访问令牌](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>输入 Telegram 机器人的访问令牌

将之前复制的令牌粘贴到“访问令牌”字段并单击“提交”。

## <a name="enable-the-bot"></a>启用机器人
勾选“在 Telegram 上启用此机器人”。 然后单击“我已完成 Telegram 配置”。

完成上述步骤后，机器人便配置成功，可与使用 Telegram 的用户通信。
