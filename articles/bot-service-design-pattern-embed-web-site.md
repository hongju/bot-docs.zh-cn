---
title: 在网站中嵌入机器人 | Microsoft Docs
description: 了解如何设计嵌入到网站中的机器人。
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f9fa2bee156752f1545d201768040b6106558e01
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711831"
---
# <a name="embed-a-bot-in-a-website"></a>在网站中嵌入机器人

虽然机器人通常在网站外，但它们也可以嵌入网站中。 例如，可在网站中嵌入[知识机器人](~/bot-service-design-pattern-knowledge-base.md)，以便用户能在复杂网站结构中快速查找信息；如果未嵌入机器人，可能很难找到。 或者，也可以在支持网站中嵌入机器人，作为传入用户请求的第一响应者。 机器人可以独立解决简单问题，将更复杂的问题[移交](~/bot-service-design-pattern-handoff-human.md)给人工处理。 

本文探讨了如何将机器人与网站集成以及使用反向通道机制促进网页和机器人之间私有通信的过程。 

Microsoft 提供两种不同的方法来在网站中集成机器人：Skype Web 控件和开源 Web 控件。

## <a name="skype-web-control"></a>Skype Web 控件

[Skype Web 控件](https://aka.ms/bot-skype-web-control)本质上是启用了 Web 的控件中的 Skype 客户端。 机器人可通过内置的 Skype 身份验证对用户进行身份验证与识别，而无需开发人员编写任何自定义代码。 Skype 会自动识别在其 Web 客户端中使用的 Microsoft 帐户。 

由于 Skype Web 控件只用作 Skype 的前端，因此用户的 Skype 客户端可自动访问 Web 控件所促成的任何聊天的完整上下文。 即使在 Web 浏览器关闭之后，用户也可以使用 Skype 客户端继续与机器人交互。 

## <a name="open-source-web-control"></a>开源 Web 控件

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">开源网上聊天控件</a>基于 ReactJS，它使用 [Direct Line API][directLineAPI] 与 Bot Framework 进行通信。 网上聊天控件提供一个用于实现网上聊天的空白画布，你可以完全控制其行为及其提供的用户体验。 

借助反向通道机制，托管控件的网页可以采用对用户完全不可见的方式直接与机器人通信。 此功能支持多种实用方案： 

- 网页可将相关数据发送给机器人（如 GPS 位置）。
- 网页可就用户操作向机器人提供建议（如“用户刚从下拉列表中选择了 A 选项”）。
- 网页可向机器人发送登录用户的身份验证令牌。
- 机器人可将相关数据发送给网页（如用户项目组合的当前值）。
- 机器人可向网页发送“命令”（例如更改背景色）。

## <a name="using-the-backchannel-mechanism"></a>使用反向通道机制

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>代码示例

可通过 GitHub 获得<a href="https://aka.ms/BotFramework-WebChat" target="_blank">开源网上聊天控件</a>。 有关如何使用开源网上聊天控件和 Bot Framework SDK for Node.js 实施反向通道机制的详细信息，请参阅[使用反向通道机制](~/nodejs/bot-builder-nodejs-backchannel.md)。

## <a name="additional-resources"></a>其他资源

- [Direct Line API][directLineAPI]
- [开源网上聊天控件](https://github.com/Microsoft/BotFramework-WebChat)
- [使用反向通道机制](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
