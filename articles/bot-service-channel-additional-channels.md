---
title: 其他通道 | Microsoft Docs
description: 了解如何为机器人配置其他通道。
keywords: 机器人通道, 环聊, Twilio, facebook, azure 门户
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: d6cf068f3cd4d57ff9cde2be6d41e084cee6524d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693737"
---
# <a name="additional-channels"></a>其他通道

除了这些文档中列出的通道，还有一些作为适配器提供的其他通道，均可通过 Botkit 从[提供的平台](https://botkit.ai/docs/v4/platforms/)获取，或者通过[社区存储库](https://github.com/BotBuilderCommunity/)对其进行访问。 下面是提供的其他通道：

- [Webex Teams](https://botkit.ai/docs/v4/platforms/webex.html)
- [Websocket 和 Webhook](https://botkit.ai/docs/v4/platforms/web.html)
- [Google 环聊和 Google Assistant](https://github.com/BotBuilderCommunity/)（通过社区获取）
- [Amazon Alexa](https://github.com/BotBuilderCommunity/)（通过社区获取）

## <a name="currently-available-adapters"></a>目前可用的适配器

可以[在此处找到](https://botkit.ai/docs/v4/platforms/)可用适配器的的完整列表。 你会注意到，某些通过也可作为适配器使用。你可以自行决定何时使用通道，何时使用适配器。

### <a name="when-to-use-an-adapter"></a>何时使用适配器

1. 此服务不支持你所需的通道
2. 根据部署的安全性与合规性要求，你不能依赖于外部服务
3. 在特定通道中需要的各种功能可能不受支持

### <a name="when-to-use-a-channel"></a>何时使用通道

1. 需要跨通道兼容：机器人应该在多个可用通道上工作。
2. 内置支持：每当第三方进行更新时，Microsoft 都会为你的每个通道进行维护、修补并提供无缝服务
3. 允许访问其他专属的 Microsoft 通道，例如正在快速发展的 Microsoft Teams
4. 如果你需要依赖某个 GUI 界面为机器人启用其他通道
