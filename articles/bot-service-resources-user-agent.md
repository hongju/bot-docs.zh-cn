---
title: Bot Framework User-Agent 请求 | Microsoft Docs
description: 了解从 Bot Framework 到 Web 服务器的请求。
author: JohnD-ms
ms.author: v-jodemp
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: befd46cc13fa10a2d1a0f8cf2beed4cb041ab3bd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998534"
---
# <a name="bot-framework-user-agent-requests"></a>Bot Framework User-Agent 请求

如果你正在阅读此消息，可能已收到来自 Microsoft Bot Framework 服务的请求。 本指南将帮助你了解这些请求的性质，并提供阻止它们的步骤（如果需要）。

如果收到来自我方服务的请求，它可能包含类似于以下字符串格式的 User-Agent 标头：

```User-Agent: BF-DirectLine/3.0 (Microsoft-BotFramework/3.0 +https://botframework.com/ua)```

此字符串中最重要的部分是 Microsoft-BotFramework（它是工具和服务的集合，允许独立软件开发人员创建和操作自己的机器人）使用的 Microsoft Bot Framework 标识符。

如果你是机器人开发人员，可能已经知道为什么这些请求会被定向到你的服务。 如果不是，请继续阅读了解详细信息。

## <a name="why-is-microsoft-contacting-my-service"></a>为什么 Microsoft 要联系我的服务？

Bot Framework 将 Skype 和 Facebook Messenger 等聊天服务上的用户连接到机器人，即 Web 服务器（其中可访问 Internet 的端点上运行有 REST API）。 对机器人进行的 HTTP 调用（也称为 Webhook 调用）仅发送给由已在 Bot Framework 开发人员门户注册的机器人开发人员指定的 URL。

如果收到从 Bot Framework 服务到 Web 服务的请求（未经请求），可能是因为开发人员无意或有意输入了你的 URL 作为其机器人的 Webhook 回调。

## <a name="to-stop-these-requests"></a>阻止这些请求

对于阻止不需要的请求发送到 Web 服务方面的帮助，请联系 [bf-reports@microsoft.com](mailto://bf-reports@microsoft.com)。
