---
title: 排查机器人配置问题 | Microsoft Docs
description: 如何排查已部署机器人中的配置问题。
keywords: 排查, 配置, 网上聊天, 问题。
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/30/2019
ms.openlocfilehash: c208cef52d1850a00b62828ae0ea622a2606ec5b
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033420"
---
# <a name="troubleshoot-bot-configuration-issues"></a>排查机器人配置问题

排查机器人问题时，第一步是在网上聊天中对其进行测试。 这样就可以确定问题是因为机器人（机器人在任何通道中都不工作），还是因为特定的通道（机器人在某些通道中工作，但在其他通道中不工作）。

## <a name="test-in-web-chat"></a>通过网页聊天执行测试

1. 在 [Azure 门户](http://portal.azure.com/)中打开机器人资源。
1. 打开“通过网上聊天执行测试”窗格。
1. 向机器人发送一条消息。

![通过网上聊天执行测试](./media/test-in-webchat.png)

如果机器人没有使用预期的输出进行响应，请转到[机器人在网上聊天中不工作](#bot-does-not-work-in-web-chat)。 否则，请转到[机器人在网上聊天中工作，但在其他通道中不工作](#bot-works-in-web-chat-but-not-in-other-channels)。

## <a name="bot-does-not-work-in-web-chat"></a>机器人在网上聊天中不工作

机器人不工作可能有许多原因。 最可能的情况是，机器人应用程序出了问题，无法接收消息，或者机器人可以接收消息，但无法响应。

若要查看机器人是否正在运行，请执行以下操作：

1. 打开“概览”窗格。
1. 复制**消息传送终结点**，将其粘贴到浏览器中。

如果终结点返回“HTTP 错误 405”，则意味着可以访问机器人，且机器人能够响应消息。 应该调查是机器人[超时](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md)，还是机器人[发生故障并出现 HTTP 5xx 错误](bot-service-troubleshoot-500-errors.md)。

如果终结点返回错误“此站点无法访问”或“无法访问此页面”，则表明机器人出了问题，需重新部署。

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>机器人在网上聊天中工作，但在其他通道中不工作

如果机器人在网上聊天中按预期工作，但在某个其他的通道中不工作，则可能原因为：

- [通道配置问题](#channel-configuration-issues)
- [特定于通道的行为](#channel-specific-behavior)
- [通道中断](#channel-outage)

### <a name="channel-configuration-issues"></a>通道配置问题

可能是通道配置参数设置不正确，或者已通过外部方式更改。 例如，机器人为特定页面配置了 Facebook 通道，该页面随后被删除。 最简单的解决方案是删除该通道，然后重新进行通道配置。

下面的链接说明了如何配置 Bot Framework 支持的通道：

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [电子邮件](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [Skype for Business](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>特定于通道的行为

某些功能的实现可能因通道而异。 例如，并非所有通道都支持自适应卡。 大多数通道支持按钮，但其呈现方式特定于通道。 如果发现某些消息类型在不同通道中的工作方式存在差异，请查看[通道参考](bot-service-channels-reference.md)。

下面是一些其他的链接，提供单个通道的帮助信息：

- [Add bots to Microsoft Teams apps](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)（向 Microsoft Teams 应用添加机器人）
- [Facebook：Messenger 平台简介](https://developers.facebook.com/docs/messenger-platform/introduction)
- [Principles of Cortana Skills design](https://docs.microsoft.com/cortana/skills/design-principles)（Cortana 技能设计原则）
- [Skype for Developers](https://dev.skype.com/bots)（开发人员版 Skype）
- [Slack:Enabling interactions with bots](https://api.slack.com/bot-users)（Slack：启用与机器人的交互）

### <a name="channel-outage"></a>通道中断

有时候，某些通道可能会出现服务中断的情况。 通常情况下，此类中断不会持续很长时间。 不过，如果怀疑发生了中断，请查看通道网站或社交媒体。

若要确定某个通道是否已中断，另一种方法是创建一个测试机器人（例如简单的 Echo 机器人）并添加一个通道。 如果测试机器人在某些通道中工作，但在其他通道中不工作，则表明问题不在生产机器人中。

## <a name="additional-resources"></a>其他资源

请参阅如何[调试机器人](bot-service-debug-bot.md)和该部分中的其他调试文章。