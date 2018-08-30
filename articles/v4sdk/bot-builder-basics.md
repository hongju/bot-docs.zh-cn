---
title: Bot Builder 基础知识 | Microsoft Docs
description: Bot Builder SDK 基本结构。
keywords: 轮次上下文, 机器人结构, 接收输入
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 34564b411f911ae82197d5a34cb954a103abe70b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905411"
---
# <a name="basic-bot-structure"></a>机器人基本结构

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Azure 机器人服务和 Bot Builder SDK 提供库、示例和工具来帮助生成和调试机器人。 然而，在我们深入了解任何这些详细信息之前，务必先了解机器人的基本结构以及一切要素的工作原理。 无论选择哪种编程语言，这些概念都适用。 在整个过程中提供了更多深度内容的链接；本文将提供关于机器人如何操作的初始心理框架。

让我们从零开始探讨机器人的基本结构。

## <a name="creation-of-your-bot"></a>创建机器人

机器人可以通过多种方式创建，例如在 [Azure 门户](~/bot-service-quickstart.md)、在 [Visual Studio](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) 中，或通过面向 [JavaScript](~/javascript/bot-builder-javascript-quickstart.md)、[Java](~/java/bot-builder-java-quickstart.md) 或 [Python](~/python/bot-builder-python-quickstart.md) 的命令行工具。 创建后，机器人可以在本地计算机、Azure 或在另一个云服务上运行。 它们的功能都非常相似，无论运行位置及构建方式如何。

## <a name="interaction-with-your-bot"></a>与机器人交互

就其本质而言，机器人不像网站或应用那样具有任何可见 UI，用户通过[会话](~/v4sdk/bot-concepts.md#activities-and-conversations)与其交互。 根据用于连接到机器人的应用程序（我们称之为[通道](~/v4sdk/bot-concepts.md)，但不会在此详细介绍），会在用户和机器人之间来回发送某些信息。

在机器人中，每个信息单元称为“活动”，而且一个活动可以有多种形式。 活动包括来自用户的通信，称为“消息”，或封装在其他少量[活动类型](~/bot-service-activities-entities.md)中的附加信息。 在新参与方加入或离开会话、会话结束等情形下，可以包含此附加信息。来自用户连接的此类通信通过基础系统发送，而用户无需执行任何操作。

机器人接收通信并将其封装在正确类型的活动对象中，以将其提供给机器人代码。 所有其他活动类型都提供有用信息，但最有趣和最常见的活动是来自用户的消息活动。

机器人接收的每个活动都会开始一个轮次，我们将稍后详细介绍。

## <a name="receiving-user-input"></a>接收用户输入

当我们接收来自用户的消息活动时，我们需要理解他们向我们发送的信息。 最简单的方法是，只需将传入消息文本与字符串匹配。 基于它的字符串，我们可以选择执行某些操作，具体取决于机器人想要达到的目标。 这可能包括对用户的响应、更新一些变量或资源、将其保存到[存储](~/v4sdk/bot-builder-storage-concept.md)或类似的处理。

有更复杂的方式来识别用户输入，例如，使用 [LUIS](~/v4sdk/bot-builder-concept-luis.md) 或 [QnA Maker](~/v4sdk/bot-builder-howto-qna.md)，但字符串匹配是最简单的。

## <a name="defining-a-turn"></a>定义轮次

[!INCLUDE [Define a turn](~/includes/snippet-definition-turn.md)]

活动处理由适配器托管，[活动处理](~/v4sdk/bot-builder-concept-activity-processing.md)对此进行了详述。 当适配器接收活动，它会创建轮次上下文以提供有关活动的信息，并提供我们正在处理的当前轮次的上下文。 该轮次上下文在轮次过程中一直存在，然后会被处理掉，表示轮次结束。

[轮次上下文](~/v4sdk/bot-builder-concept-activity-processing.md#turn-context)包含少量信息，并通过机器人的所有层使用相同对象。 这很有用，因为此轮次上下文对象可以并且应该用来存储可能会在后续轮次中用到的信息。

> [!IMPORTANT]
> 所有轮次都彼此独立，自行执行，并且可能会重叠。 机器人可能会在不同通道上的不同用户中一次处理多个轮次。 每个轮次都将有其自己的轮次上下文，但值得考虑的是在某些情况下引入的复杂性。

## <a name="where-to-go-from-here"></a>下一步

本文省略了许多详细信息，例如，如何[处理活动](~/v4sdk/bot-builder-concept-activity-processing.md)、不同[类型](~/v4sdk/bot-builder-conversations.md)、跟踪[状态](~/v4sdk/bot-builder-storage-concept.md)以了解更多智能会话等等。 概念主题的其余部分建立在基础知识之上，并涵盖了了解机器人和 Azure 机器人服务所需的其他想法。 可以按照下面的步骤部分构建知识，也可以跳转到你所感兴趣的部分，尝试[快速入门](~/bot-service-quickstart.md)来生成你的第一个机器人，或深入了解如何[开发](~/v4sdk/bot-builder-howto-send-messages.md)机器人。

## <a name="next-steps"></a>后续步骤

接下来，机器人连接器服务允许机器人与不同平台上的用户进行通信。

> [!div class="nextstepaction"]
> [通道和机器人连接器服务](~/v4sdk/bot-concepts.md)
