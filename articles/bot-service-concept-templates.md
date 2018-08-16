---
title: 机器人服务模板 | Microsoft Docs
description: 了解使用机器人服务创建机器人时可以使用的不同模板。
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 286d7057afb28983964ef992de2c11cebd74e0da
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298102"
---
# <a name="bot-service-templates"></a>机器人服务模板
机器人服务包括五个模板来帮助你开始构建机器人。 这些模板提供了一个功能齐全、立即可用的机器人，可以帮助你快速开始。 [创建机器人](bot-service-quickstart.md)时，为机器人选择模板和 SDK 语言。

每个模板都提供了一个基于机器人核心功能的起点。 

## <a name="basic-bot"></a>基础机器人
若要创建使用对话框来响应用户输入的机器人，请选择基本模板。 “基本”模板可以创建一个具有最小文件和代码集以开始使用的机器人。 无论用户输入什么内容，机器人都会回复。 可以使用此模板开始在机器人中构建会话流。

## <a name="form-bot"></a>表单机器人
若要创建通过引导式会话收集用户输入的机器人，请选择“表单”模板。 表单机器人旨在收集用户的一组特定信息。 例如，设计用来获取用户的三明治订单的机器人需要收集面包种类、浇头选择、三明治大小等信息。

使用 C# 语言通过表单模板创建的机器人使用 [FormFlow](dotnet/bot-builder-dotnet-formflow.md) 来管理表单，使用 Node.js 语言通过表单模板创建的机器人使用[瀑布图](nodejs/bot-builder-nodejs-dialog-waterfall.md)管理表单。

## <a name="language-understanding-bot"></a>语言理解机器人
若要创建使用自然语言模型来理解用户意向的机器人，请选择“语言理解”模板。 该模板利用<a href="https://www.luis.ai" target="_blank">语言理解 (LUIS)</a> 提供自然语言理解。

如果用户发送“获取有关虚拟现实公司的新闻”之类的消息，机器人可以使用 LUIS 来解释这条消息的含义。 你可以使用 LUIS 快速部署 HTTP 终结点，该终结点将根据其传达的意向（查找新闻）和存在的关键实体（虚拟现实公司）来解释用户输入。 LUIS 让你能够指定与应用程序相关的意向和实体集，然后指导你完成构建语言理解应用程序的过程。

在使用语言理解模板创建机器人时，机器人服务会创建一个相应的空（即，始终返回 `None`）的 LUIS 应用程序。 若要更新 LUIS 应用程序模型以便能够解释用户输入，必须登录 <a href="https://www.luis.ai" target="_blank">LUIS</a>，单击“我的应用程序”，选择服务为你创建的应用程序，然后创建意向，指定实体并定型应用程序。

## <a name="question-and-answer-bot"></a>问答机器人
若要创建一个从问题和答案对等半结构化数据中提取出不同且有用的答案的机器人，请选择“问答”模板。 此模板利用 <a href="https://qnamaker.ai">QnA Maker</a> 服务分析问题并提供答案。 

使用“问答”模板创建机器人时，必须登录 <a href="https://qnamaker.ai">QnA Maker</a> 并创建新的 QnA 服务。 此 QnA 服务将为你提供知识库 ID 和订阅密钥，你可以用它们来更新 QnAKnowldegebasedId 和 QnASubscriptionKey的 [应用设置](bot-service-manage-settings.md)值。 设置这些值后，机器人可以回答 QnA 服务存储在知识库中的任何问题。

## <a name="proactive-bot"></a>主动机器人
若要创建可向用户发送主动消息的机器人，请选择“主动模板”。 通常，机器人向用户发送的每条消息与用户先前的输入直接相关。 但在某些情况下，机器人可能需要向用户发送与用户最近的消息不直接相关的信息。 这些类型的消息称为主动消息。 主动消息在各种场景中都可以发挥作用。 例如，如果机器人设置了计时器或提醒，它可能需要在时间到来时通知用户。 或者，如果机器人收到关于外部事件的通知，则可能需要将该信息传达给用户。 

使用主动模板创建机器人时，会自动创建几个 Azure 资源，而且这些资源会被添加到资源组。 默认情况下，这些 Azure 资源已配置为启用非常简单的主动式消息收发方案。 

| 资源 | Description |
|----|----|
| Azure 存储 | 用于创建队列。 |
| Azure 函数应用 | 一个 `queueTrigger` Azure 函数，每当队列中有消息时就会触发此函数。 它通过使用 [Direct Line](https://docs.microsoft.com/bot-framework/rest-api/bot-framework-rest-direct-line-3-0-concepts) 与机器人服务通信。 此函数使用机器人绑定将消息作为触发器有效负载的一部分发送。 我们的示例函数从队列中按原样转发用户的消息。
| Bot 服务 | 你的机器人。 包含从用户接收消息的逻辑，将消息添加到 Azure 队列，从 Azure 函数接收触发器，并通过触发器的有效负载发回它收到的消息。 |

下图显示了使用主动模板创建机器人时已触发事件的工作原理。

![示例主动机器人的概述](~/media/bot-proactive-diagram.png)

当用户通过 Bot Framework 服务器向机器人发送消息时，此过程开始（步骤 1）。

## <a name="next-steps"></a>后续步骤
现在你了解了不同的模板，了解了在机器人中使用的认知服务。

> [!div class="nextstepaction"]
> [适用于机器人的认知服务](bot-service-concept-intelligence.md)
