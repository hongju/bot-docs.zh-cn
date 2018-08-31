---
title: Bot Builder SDK 中的会话 | Microsoft Docs
description: 介绍 Bot Builder SDK 中的会话内容。
keywords: 会话流, 识别意向, 单轮次, 多轮次
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/11/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 32486cb024dfe852a7478ccba4a0eedc476431b0
ms.sourcegitcommit: ee63d9dc1944a6843368bdabf5878950229f61d0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/23/2018
ms.locfileid: "42795026"
---
# <a name="conversation-flow"></a>会话流
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

由于可将机器人视作会话用户界面，因此会话流是我们与用户交互的方式，可以采用不同的形式。 拥有正确的会话流有助于改善用户的交互和机器人性能。

设计机器人的会话流涉及决定机器人在用户向其说出某些内容时如何响应。 机器人首先根据来自用户的消息识别任务或会话主题。 要确定与用户消息相关联的任务或主题（称为“意向”），机器人可以在用户消息的文本中查找单词或模式，或者可以利用[语言理解 (LUIS)](bot-builder-concept-luis.md)和 QnA Maker 等服务。 

一旦机器人识别出用户意向，根据场景，机器人可以通过单个回复完成用户的请求，一个轮次完成会话，或者可能需要一系列轮次才能完成。 对于多个轮次会话流，Bot Builder SDK 提供用于跟踪会话的[状态管理](./bot-builder-howto-v4-state.md)、用于请求信息的[提示](bot-builder-prompts.md)、以及用于封装会话流的[对话框](bot-builder-dialog-manage-conversation-flow.md)。 

在具有多个子系统的复杂机器人中，可以使用多个服务来识别意向，每项服务针对机器人的每个子组件。 当将会话子系统组合到一个机器人中时，[调度工具](bot-builder-tutorial-dispatch.md)可以在一个位置获得多个服务的结果。 
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



<!-- 
The EchoBot sample in the BotBuilder SDK is a single-turn bot. Here are other examples of single turn conversation flow:
* A bot for getting the weather report, that just tells the user what the weather is, when they say "What's the weather?".
* An IoT bot that responds to "turn on the lights" by calling an IoT service. -->

<!-- The following isn't always true, it's a generalization --> 最简单的单轮次机器人不需要跟踪会话状态。 每次收到消息时，它只根据当前传入的消息的上下文进行响应，不知道过去的会话的轮次。

![单轮次天气机器人](./media/concept-conversation/weather-single-turn.png)

天气机器人具有单轮次流，它只是向用户提供天气报告，而无需来回询问城市或日期。 显示天气报告的所有逻辑都基于机器人刚收到的消息。 在每个聊天轮次中，机器人会收到一个[轮次上下文](bot-builder-concept-activity-processing.md#turn-context)，机器人可以使用它来确定下一步该做什么以及如何进行聊天。 

## <a name="multiple-turns"></a>多轮次

大多数类型的会话无法在单轮次中完成，因此机器人也可以具有多轮次会话流。 某些需要多会话轮次的方案包括：

 * 提示用户提供完成任务所需其他信息的机器人。 机器人需要跟踪它是否具有完成任务的所有参数。
 * 引导用户完成流程中步骤（例如下订单）的机器人。 机器人需要跟踪用户是否按照步骤序列操作。

例如，如果机器人通过询问城市响应“天气怎么样？”，则天气机器人具有多轮次流 。

![多轮次天气机器人](./media/concept-conversation/weather-multi-turn.png)

当用户以城市回复机器人的提示并且机器人收到“西雅图”时，机器人需要具有已保存的某些上下文，以理解当前消息是对先前提示的响应并且是获取天气的请求的一部分。 多轮次机器人跟踪状态以对新消息作出适当响应。

<!--
```
// TBD: snippet showing receiving message and using ConversationProperties
```
-->

有关管理状态的概述，请参阅[管理状态](bot-builder-storage-concept.md)，有关示例，请参阅[如何使用用户和会话属性](bot-builder-howto-v4-state.md)。

> [!NOTE]
> 与 REST API 客户端的多轮次会话需要跟踪其自身状态，例如在数据库或表存储中。 

## <a name="conversation-topics"></a>会话主题

你可以将机器人设计为处理多种类型的任务。 例如，你可能有一个机器人，它提供不同的会话流，用于问候用户、下单、取消以及获得帮助。 在不同任务或会话主题的会话之间处理此切换的一种方法是从当前消息识别意向（用户想要做什么）。 

### <a name="recognize-intent"></a>识别意向

Bot Builder SDK 提供识别器，用于处理每个传入消息以确定意向，因此机器人可以启动适当的会话流。 在接收回调之前，识别器查看来自用户的消息内容以确定意向，然后使用接收回调中的轮次上下文对象将意向返回到机器人，存储为[轮次上下文](bot-builder-concept-activity-processing.md#turn-context)对象中的“顶级意向”。 

确定顶级意向的识别器可以简单地使用正则表达式、语言理解 (LUIS) 或作为中间件开发的其他逻辑。 以下是识别器示例：
   
* 你可以使用正则表达式设置识别器，以便在用户每次说出帮助一词时进行检测。
* 可以使用语言理解 (LUIS) 来定型服务，其中包含用户可能寻求帮助的方式示例，并将其映射到“帮助”意向。
* 可以创建自己的识别器中间件来检查传入的活动，并在每次检测到其他语言的消息时返回“翻译”意向。

有关详细信息，请参阅[具有 LUIS 的语言理解](bot-builder-concept-luis.md)。 <!-- TODO: ADD THIS TOPIC OR SNIPPET-->

### <a name="consider-how-to-interrupt-conversation-flow-or-change-topics"></a>考虑如何中断会话流或更改主题

跟踪会话中你所处位置的一种方法是使用[会话状态](bot-builder-howto-v4-state.md)来保存有关当前活动主题或已完成哪些序列步骤的信息。

当机器人变得更加复杂时，还可以想象堆栈中发生的一系列会话流；例如，机器人将调用新的订单流，然后调用产品搜索流。 然后，用户将选择产品并确认，完成产品搜索流，然后完成订单。

然而，会话很少遵循这种线性的逻辑路径。 用户不会在“堆栈”中进行通信，而是频繁地更改其想法。 下面是一个示例：

![用户讲出一些意外内容](./media/concept-conversation/interruption.png)

虽然机器人可能在逻辑上构建了一堆栈流，但用户可能决定做一些完全不同的事情，或者询问可能与当前主题无关的问题。 在该示例中，用户询问问题而不是提供流期望的是/否响应。 流应如何响应？

* 坚持让用户先回答问题。
* 忽略用户之前完成的所有操作，重置整个流堆栈，并从头开始尝试回答用户的问题。
* 尝试回答用户的问题，然后返回到是/否问题并尝试从该位置继续。

此问题没有正确答案，因为最佳解决方案将取决于你的具体应用场景，以及用户如何合理地期望机器人做出响应。 有关详细信息，请参阅[处理用户中断](bot-builder-howto-handle-user-interrupt.md)。

> [!TIP]
> 如果正在使用 Node.Js 适用的 Bot Builder SDK，则可以使用[对话框](bot-builder-dialog-manage-conversation-flow.md)来管理会话流。

## <a name="conversation-lifetime"></a>会话生存期

<!-- Note: these activities are dependent on whether the channel actually sends them. Also, we should add links --> 无论何时将机器人添加到会话，其他成员已添加到会话或从会话中删除，或者会话元数据已更改，机器人都会收到会话更新活动。
你可能希望让机器人通过问候用户或自我介绍来对会话更新活动作出反应。

机器人接收会话结束活动以指示用户已结束会话。 机器人可以发送会话结束活动以向用户指示正在结束会话。 如果要存储有关会话的信息，则你希望在会话结束时清除该信息。

<!--  Types of conversations

Your bot can support multi-turn interactions where it prompts users for multiple peices of information. It can be focused on a very specific task or support multiple types of tasks. 
The Bot Builder SDK has some built-in support for Language Understatnding (LUIS) and QnA Maker for adding natural language "question and answer" features to your bot.

<!--TODO: Add with links when these topics are available:
[Conversation flow] and other design articles.
[Using recognizers] [Using state and storage] and other how tos.
-->
## <a name="conversations-channels-and-users"></a>会话、通道和用户

会话可以是与特定用户的直接会话，也可以是与多个用户的组合会话。
机器人通过从用户接收活动并向用户发送活动来与通道上的用户通信。

- 每个用户都有特定于每个通道的唯一 ID。
- 每个会话都有特定于每个通道的唯一 ID。
- 通道在启动会话时设置会话 ID。
- 机器人无法启动会话；但是，一旦它有了会话 ID，它就可以恢复该会话。
- 并非所有通道都支持群组会话。

## <a name="next-steps"></a>后续步骤

对于复杂的会话，例如上面强调的一些会话，我们需要能够将保存信息的时间长于一个轮次。 我们接下来将了解状态和存储。

> [!div class="nextstepaction"]
> [状态和存储](bot-builder-storage-concept.md)

<!-- In addition, your bot can send activities back to the user, either _proactively_, in response to internal logic, or _reactively_, in response to an activity from the user or channel.-->
<!--TODO: Link to messaging how tos.-->

<!--  TODO: Change to next steps, one for each of LUIS and State
## See also

- Activities
- Adapter
- Context
- Proactive messaging
- State
-->

[QnAMaker]:(bot-builder-luis-and-qna.md#using-qna-maker)

<!-- TODO: Update when the Dispatch concept is pushed -->
[Dispatch]:(bot-builder-concept-luis.md)
