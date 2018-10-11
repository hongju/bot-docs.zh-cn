---
title: 语言理解 | Microsoft Docs
description: 了解如何使用 Microsoft 认知服务向机器人添加人工智能，使其更有用且更具吸引力。
keywords: LUIS, 意向, 识别器, 调度工具, qna, qna maker
author: ivorb
ms.author: v-ivorb
manager: rstand
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: af79bb40e3d24557fd898fa0a0ca2ef7b0286af4
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707543"
---
# <a name="language-understanding"></a>语言理解

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人可以使用不同的会话方式，从结构化和引导式到自由格式和开放式。 机器人需要根据用户所说的内容决定接下来要在其会话流中执行的操作，而在开放式会话中，用户回复的范围更广。

| 引导式 | 打开 |
|------|------|
| 我是差旅智能机器人。 请选择以下选项之一：查找航班、查找酒店、查找租赁汽车。 | 我可以帮助您预订行程。 您希望做什么？ |
| 还有其他需要吗？ 单击“是”或“否”。 | 还有其他需要吗？ |

用户与机器人之间的交互通常是自由格式，机器人需要自然地根据上下文理解语言。 本文介绍语言理解 (LUIS) 如何帮助确定用户需求，标识句子中的概念和实体，最终让机器人以相应操作做出响应。

## <a name="recognize-intent"></a>识别意向

[LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/home) 通过确定用户的意向提供帮助，即通过用户的话语了解他们的需求，从而让机器人可以做出适当的响应。 当他们对机器人所说的内容不遵循可预测的结构或特定模式时，LUIS 特别有帮助。 如果机器人具有会话式用户界面（用户可在其中说出或键入响应），则由用户口头陈述或通过文本输入的表达可以有无限变体。

例如，考虑差旅智能机器人用户请求预订航班的多种方式。

![预定航班的各种不同格式的表达](media/cognitive-services-add-bot-language/cognitive-services-luis-utterances.png)

这些表达可以有不同结构，并包含你未想到的有关“航班”的各种同义词。 在机器人中，编写匹配所有话语并且仍将它们与包含相同字词的其他意向区分开来的逻辑可能会有难度。 此外，机器人需要提取实体，这是其他重要字词，如位置和时间。 LUIS 通过根据上下文识别意向和实体简化了此过程。

在针对自然语言输入设计机器人时，确定机器人需要识别哪些意向和实体，并考虑它们将如何与机器人所执行的操作联系起来。 在 [luis.ai](https://www.luis.ai) 中，定义自定义意向和实体，并通过为每个意向提供示例和在其中标记实体来指定它们的行为。

机器人使用 LUIS 识别的意向来确定会话主题或开始会话流。 例如，当用户说“我想要预订航班”，机器人会检测 BookFlight 意向，并调用会话流开始搜索航班。 在触发意向的原始表达和之后的会话流中，LUIS 将检测目标城市和出发日期之类的实体。 一旦机器人拥有其所需的所有信息，它便可以满足用户的意向。

![与机器人的会话由 BookFlight 意向触发](media/cognitive-services-add-bot-language/cognitive-services-luis-conversation-high-level.png)

### <a name="recognize-intent-in-common-scenarios"></a>在常见场景中识别意向

为了节省开发时间，LUIS 为常见类别机器人提供识别常见表达的预先定型的语言模型。 

**预生成域**是预先训练、随时可用的意向和实体集合，适用于约会、提醒、管理、健身、娱乐、通信、预订等常见场景。 实用程序预生成域可帮助机器人处理常见任务，如取消、确认、帮助、重复和停止。 查看 LUIS 提供的[预生成域](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-use-prebuilt-domains)。

预生成实体帮助机器人识别常见类型的信息，如日期、时间、数字、温度、货币、地理和年龄。 有关 LUIS 可以识别的类型的背景，请参阅[使用预生成实体](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/pre-builtentities)。

## <a name="how-your-bot-gets-messages-from-luis"></a>机器人如何通过 LUIS 获取消息

设置并连接 LUIS 后，机器人便可以将消息发送到 LUIS 应用，该应用将返回包含意向和实体的 JSON 响应。 然后，可以使用机器人_轮次处理程序_中的[轮次上下文](bot-builder-concept-activity-processing.md#turn-context)，根据 LUIS 响应中的意向路由聊天流。 

![如何将意向和实体传递给机器人](./media/cognitive-services-add-bot-language/cognitive-services-luis-message-flow-bot-code.png)

若要开始在机器人中使用 LUIS 应用，请查看[将 LUIS 用于语言理解][luis-v4-how-to]。

## <a name="best-practices-for-language-understanding"></a>语言理解的最佳做法

为机器人设计语言模型时，请考虑以下做法。

### <a name="consider-the-number-of-intents"></a>考虑意向数

LUIS 应用通过将表达分类到多个类别中的一个来识别意向。 一个自然结果是，从大量意向中确定正确类别会减弱 LUIS 应用区分它们的能力。

减少意向数的一种方法是使用层次结构设计。 假设一个个人助理机器人事例，它有三个与天气有关的意向、三个与家庭自动化相关的意向，以及另外三个实用程序意向（分别为帮助、取消和问候）。 如果将所有意向都放在同一 LUIS 应用中，则已经有 9 个了，随着向机器人添加功能，可能最终会导致数十个意向。 你可以转为使用调度程序 LUIS 应用来确定用户的请求是关于天气、家庭自动化还是实用程序，然后为调度程序确定的类别调用 LUIS 应用。 在这种情况下，每个 LUIS 应用都只从 3 个意向开始。

### <a name="use-a-none-intent"></a>使用 None 意向

这通常出现在机器人用户说了一些令人意想不到的内容，或者与当前会话流无关的内容的情况。 _None_ 意向用于处理这些消息。

如果不对处理回退、默认或“以上都不是”事例的意向进行定型，LUIS 应用只能将消息分类到已定义的意向中。 因此，例如，假设你有包含两个意向的 LUIS 应用：`HomeAutomation.TurnOn` 和 `HomeAutomation.TurnOff`。 如果这些都是唯一的意向，且输入内容不相关，比如“安排在周五约会”，LUIS 应用将别无选择，而只能将该消息分类为 HomeAutomation.TurnOn 或 HomeAutomation.TurnOff。 如果 LUIS 应用有一个包含几个示例的 `None` 意向，则你可以在机器人中提供一些回退逻辑来处理意外表达。

`None` 意向对于改善识别结果非常有用。 此家庭自动化场景中，“安排星期五约会”可能会生成置信度较低的 `HomeAutomation.TurnOn` 意向，机器人应拒绝它。 可将此类短语添加到模型中的 `None` 意向下，以便将其正确解析为 `None`。

### <a name="review-the-utterances-that-luis-app-receives"></a>查看 LUIS 应用接收的表达

LUIS 应用提供了一项功能来改进应用性能，方法是查看用户向其发送的消息。 有关分步演练，请参阅[建议的话语](https://docs.microsoft.com/azure/cognitive-services/LUIS/label-suggested-utterances)。


## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>将多个 LUIS 应用和 QnA 服务与 Dispatch 工具集成

在构建能够理解多个聊天主题的多用途机器人时，可以开始针对每个功能单独开发服务，然后将它们集成在一起。 这些服务可以包括语言理解 (LUIS) 应用和 QnAMaker 服务。 以下是几个示例方案，机器人可以在其中合并多个 LUIS 应用、多个 QnAMaker 服务或将两者结合：

* 个人助理机器人让用户可以调用各种命令。 每个类别的命令构成可以单独开发的“技能”，且每个技能都有一个 LUIS 应用。
* 机器人搜索多个知识库来查找常见问题 (FAQ) 的解答。
* 业务机器人具有用于创建客户帐户和下单的 LUIS 应用，并且还有有关其常见问题解答的 QnAMaker 服务。  

### <a name="the-dispatch-tool"></a>调度工具

调度工具可帮助你将多个 LUIS 应用和 QnA Maker 服务与机器人集成，方法是创建调度应用，这是一个新的 LUIS 应用，该应用将消息路由到相应的 LUIS 和 QnAMaker 服务。 有关将多个 LUIS 应用和 QnA Maker 合并到一个机器人的分步教程，请参阅[调度教程](./bot-builder-tutorial-dispatch.md)。

## <a name="use-luis-to-improve-speech-recognition"></a>使用 LUIS 改进语音识别

对于用户会话的机器人，将其与 LUIS 集成可帮助识别将语音转换为文本时可能被人误解的字词。  例如，在国际象棋场景中，用户可能会说：“Move knight to A 7”。 如果没有用户意向的上下文，该表达可能会被识别为：“Move night 287”。 通过创建代表棋子的实体并以表达进行标记，可以提供语音识别上下文进行标识。 可以使用与必应语音集成的 Bot Framework 通道（比如网上聊天、Bot Framework 模拟器和 Cortana）来[启用语音识别启动][speechrecognitionpriming]。  

## <a name="additional-resources"></a>其他资源
有关详细信息，请参阅[认知服务](https://docs.microsoft.com/en-us/azure/cognitive-services/)文档。
