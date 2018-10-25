---
title: Bot Builder SDK 中的会话 | Microsoft Docs
description: 介绍 Bot Builder SDK 中的会话内容。
keywords: 聊天流, 识别意向, 单轮次, 多轮次, 机器人聊天
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 19f0b67454a8c0a4bf171579f8e481e630db83ac
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998914"
---
# <a name="conversation-flow"></a>会话流
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

设计机器人的会话流涉及决定机器人在用户向其说出某些内容时如何响应。 机器人首先根据来自用户的消息识别任务或会话主题。 要确定与用户消息相关联的任务或主题（称为“意向”），机器人可以在用户消息的文本中查找单词或模式，或者可以利用[语言理解](bot-builder-concept-luis.md)和 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) 等服务。

一旦机器人识别出用户意向，根据场景，机器人可以通过单个回复完成用户的请求，一个轮次完成会话，或者可能需要一系列轮次才能完成。 对于多个轮次会话流，Bot Builder SDK 提供用于跟踪会话的[状态管理](./bot-builder-howto-v4-state.md)、用于请求信息的[提示](bot-builder-prompts.md)、以及用于封装会话流的[对话框](bot-builder-dialog-manage-conversation-flow.md)。

在具有多个子系统的复杂机器人中，可以使用多个服务来识别意向，每项服务针对机器人的每个子组件。 当将聊天子系统组合到一个机器人中时，[调度工具](bot-builder-tutorial-dispatch.md)可以在一个位置获得多个服务的结果。

<!-- 
A conversation identifies a series of activities sent between a bot and a user on a specific channel and represents an interaction between one or more bots and either a _direct_ conversation with a specific user or a _group_ conversation with multiple users.
A bot communicates with a user on a channel by receiving activities from, and sending activities to the user.

- Each user has an ID that is unique per channel.
- Each conversation has an ID that is unique per channel.
- The channel sets the conversation ID when it starts the conversation.
- The bot cannot start a conversation; however, once it has a conversation ID, it can resume that conversation.
- Not all channels support group conversations.
-->

## <a name="single-turn-conversation"></a>单轮次会话

最简单的会话流是单轮次。 在单轮次流中，机器人在一个轮次中完成其任务，包括来自用户的一条消息和来自机器人的一条回复。

<!-- The following isn't always true, it's a generalization -->

最简单类型的单轮次机器人不需要跟踪聊天状态。 每次收到消息时，它只根据当前传入的消息的上下文进行响应，不知道过去的会话的轮次。

![单轮次天气机器人](./media/concept-conversation/weather-single-turn.png)

天气机器人可能具有单轮次流，它只是向用户提供天气报告，而无需来回询问城市或日期。 显示天气报告的所有逻辑都基于机器人刚收到的消息。 在每个聊天轮次中，机器人会收到一个[轮次上下文](bot-builder-concept-activity-processing.md#turn-context)，机器人可以使用它来确定下一步该做什么以及如何进行聊天。

## <a name="multiple-turns"></a>多轮次

大多数类型的会话无法在单轮次中完成，因此机器人也可以具有多轮次会话流。 某些需要多会话轮次的方案包括：

* 提示用户提供完成任务所需其他信息的机器人。 机器人需要跟踪它是否具有完成任务的所有参数。
* 引导用户完成流程中步骤（例如下订单）的机器人。 机器人需要跟踪用户是否按照步骤序列操作。

例如，如果机器人响应“天气怎么样？”，则天气机器人可能具有多轮次流 。

![多轮次天气机器人](./media/concept-conversation/weather-multi-turn.png)

当用户以城市回复机器人的提示并且机器人收到“西雅图”时，机器人需要具有已保存的某些上下文，以理解当前消息是对先前提示的响应并且是获取天气的请求的一部分。 多轮次机器人跟踪状态以对新消息作出适当响应。

有关详细信息，请参阅[如何管理聊天和用户状态](bot-builder-howto-v4-state.md)。

> [!NOTE]
> 与 REST API 客户端的多轮次会话需要跟踪其自身状态，例如在数据库或表存储中。

## <a name="conversation-topics"></a>会话主题

你可以将机器人设计为处理多种类型的任务。 例如，你可能有一个机器人，它提供不同的聊天流，用于问候用户、下单、取消订单以及获得帮助。 在不同任务或会话主题的会话之间处理此切换的一种方法是从当前消息识别意向（用户想要做什么）。

### <a name="recognize-intent"></a>识别意向

Bot Builder SDK 提供识别器，用于处理消息以确定意向，因此机器人可以启动适当的聊天流。 调用识别器的 _recognize_ 异步方法，以根据其消息内容确定用户的意向。 然后，可以对结果调用_获取最高得分意向_方法，以获取识别器的最高预测。

识别器可以使用正则表达式、语言理解或你开发的其他逻辑。 下面是可能的识别器的示例：

* 一种识别器，它使用 QnA Maker 来检测用户何时询问知识库中包含的问题。
* 一种识别器，它使用语言理解 (LUIS) 来定型服务，其中包含用户可能寻求帮助的方式示例，并将其映射到“`Help`”意向。
* 一种自定义识别器，它使用正则表达式来查找命令。
* 一种自定义识别器，它使用服务来转换输入。

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>考虑如何中断会话流或更改主题

跟踪会话中你所处位置的一种方法是使用[会话状态](bot-builder-howto-v4-state.md)来保存有关当前活动主题或已完成哪些序列步骤的信息。

当机器人变得更加复杂时，还可以想象堆栈中发生的一系列会话流；例如，机器人将调用新的订单流，然后调用产品搜索流。 然后，用户将选择产品并确认，完成产品搜索流，然后完成订单。

然而，聊天很少遵循这种线性的逻辑路径。 用户不会在“堆栈”中进行通信，而是频繁地更改其想法。 下面是一个示例：

![用户讲出一些意外内容](./media/concept-conversation/interruption.png)

虽然机器人可能在逻辑上构建了一堆栈流，但用户可能决定做一些完全不同的事情，或者询问可能与当前主题无关的问题。 在该示例中，用户询问问题而不是提供流期望的是/否响应。 流应如何响应？

* 坚持让用户先回答问题。
* 忽略用户之前完成的所有操作，重置整个流堆栈，并从头开始尝试回答用户的问题。
* 尝试回答用户的问题，然后返回到是/否问题并尝试从该位置继续。

此问题没有正确答案，因为最佳解决方案将取决于你的具体应用场景，以及用户如何合理地期望机器人做出响应。 请参阅如何[使用对话](bot-builder-dialog-manage-conversation-flow.md)和[处理中断](bot-builder-howto-handle-user-interrupt.md)来管理聊天流。

## <a name="conversation-lifetime"></a>会话生存期

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links --> 无论何时将机器人添加到会话，其他成员已添加到会话或从会话中删除，或者会话元数据已更改，机器人都会收到会话更新活动。
你可能希望让机器人通过问候用户或自我介绍来对会话更新活动作出反应。

机器人接收会话结束活动以指示用户已结束会话。 机器人可以发送_聊天结束_活动以指示正在结束聊天。
如果要存储有关会话的信息，则你希望在会话结束时清除该信息。

<!--  Types of conversations -->

机器人可以支持多轮次交互，在交互中它会提示用户输入多条信息。 它可以专注于非常特定的任务或支持多种类型的任务。
Bot Builder SDK 具有对语言理解 (LUIS) 和 QnA Maker 的某种内置支持，可以向机器人添加自然语言“问答”功能。

## <a name="conversations-channels-and-users"></a>会话、通道和用户

会话可以是与特定用户的直接会话，也可以是与多个用户的组合会话。
机器人通过从用户接收活动并向用户发送活动来与通道上的用户通信。

* 每个用户都有特定于每个通道的唯一 ID。
* 每个会话都有特定于每个通道的唯一 ID。
* 通道在启动会话时设置会话 ID。
* 机器人无法启动会话；但是，一旦它有了会话 ID，它就可以恢复该会话。
* 并非所有通道都支持群组会话。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->
