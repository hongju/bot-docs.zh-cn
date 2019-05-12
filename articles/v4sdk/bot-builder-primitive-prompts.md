---
title: 创建自己的提示来收集用户输入 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用原始提示来管理聊天流。
keywords: 聊天流, 提示, 聊天状态, 用户状态, 自定义提示
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/25/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 74c3af688a8f35b4583aa7b195348a6b205292a2
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033163"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>创建自己的提示来收集用户输入

[!INCLUDE[applies-to](../includes/applies-to.md)]

机器人与用户之间的聊天通常涉及到请求（提示）用户输入信息、分析用户的响应，然后对该信息采取措施。 机器人应该跟踪聊天上下文，以便可以管理聊天行为并记住先前问题的回答。 机器人的状态是它为了正确响应传入消息而跟踪的信息。 

> [!TIP]
> 对话库提供内置提示，这些提示提供的功能超出用户能够使用的功能。 有关这些提示的示例，可参阅[实现顺序聊天流](bot-builder-dialog-manage-conversation-flow.md)一文。

## <a name="prerequisites"></a>先决条件

- 本文中的代码基于“提示用户输入”示例。 需要获取 **[C# 示例](https://aka.ms/cs-primitive-prompt-sample)或 [JavaScript 示例](https://aka.ms/js-primitive-prompt-sample)** 的副本。
- 了解[管理状态](bot-builder-concept-state.md)以及如何[保存用户和聊天数据](bot-builder-howto-v4-state.md)。

## <a name="about-the-sample-code"></a>关于示例代码

示例机器人会向用户提出一系列问题，验证他们的某些回答，然后保存其输入。 以下图示显示了机器人、用户配置文件和聊天流类之间的关系。 

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![custom-prompts](media/CustomPromptBotSample-Overview.png)

- 一个 `UserProfile` 类，用于跟踪机器人要收集的用户信息。
- 一个 `ConversationFlow` 类，用于在收集用户信息时控制聊天状态。
- 一个内部 `ConversationFlow.Question` 枚举，用于跟踪我们在聊天中所处的位置。

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- 一个 `userProfile` 类，用于跟踪机器人要收集的用户信息。
- 一个 `conversationFlow` 类，用于在收集用户信息时控制聊天状态。
- 一个内部 `conversationFlow.question` 枚举，用于跟踪我们在聊天中所处的位置。

---

用户状态会跟踪用户的姓名、年龄和所选日期，而聊天状态则会跟踪我们提问用户的内容。
由于我们不打算部署此机器人，因此会将用户和聊天状态配置为使用内存存储。 

我们将使用机器人的消息轮次处理程序以及用户和聊天状态属性来管理聊天流与输入的收集。 在机器人中，我们将记录消息轮次处理程序的每次迭代期间收到的状态属性信息。

## <a name="create-conversation-and-user-objects"></a>创建聊天和用户对象

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

用户和聊天状态对象在启动时创建，依赖项会注入机器人构造函数中。 

**Startup.cs** [!code-csharp[Startup](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=27-34)]

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 中创建状态属性和机器人，然后从 `processActivity` 内部调用 `run` 机器人方法。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/index.js?range=32-35)]

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/index.js?range=55-58)]

---

## <a name="create-property-accessors"></a>创建属性访问器

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们首先创建属性服务器，这些访问器为我们提供 `OnMessageActivityAsync` 方法中的 `BotState` 的句柄。 接下来，我们调用 `GetAsync` 方法以获取已正确设置了范围的密钥：

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

最后，我们使用 `SaveChangesAsync` 方法保存数据。

[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=42-43)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在构造函数中，我们创建状态属性服务器，并设置用于聊天的状态管理对象（已在上面创建）。

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=23-29)]

然后，我们定义第二个处理程序 (`onDialog`)，该程序在主消息处理程序（在下一部分介绍）之后运行。 这第二个处理程序可以确保我们每个轮次都保存状态。

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=41-48)]

---

## <a name="the-bots-message-turn-handler"></a>机器人的消息轮次处理程序

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们使用帮助程序方法 _FillOutUserProfileAsync()_ 来处理消息活动，然后使用 _SaveChangesAsync()_ 来保存状态。 下面是完整的代码。

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

为了处理消息活动，我们先设置聊天和用户数据，然后使用帮助程序方法 `fillOutUserProfile()`。 下面是轮次处理程序的完整代码。

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]
---

## <a name="filling-out-the-user-profile"></a>填写用户个人资料

首先收集信息。 每个应用程序提供类似的界面。

- 返回值将指示输入是否为此问题的有效回答。
- 如果通过了验证，则生成可保存的已分析规范化值。
- 如果未通过验证，则生成一条消息，机器人可凭此再次请求提供信息。

 在下一部分，我们将定义用于分析和验证用户输入的帮助程序方法。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=52-116)]

---

## <a name="parse-and-validate-input"></a>分析和验证输入

我们将使用以下条件来验证输入。

- **姓名**必须为非空字符串。 我们将通过修剪空白字符来规范化此值。
- **年龄**必须介于 18 与 120 之间。 我们将通过返回整数来规范化此值。
- **日期**必须是将来至少一小时后的任何日期或时间。
  我们将通过只返回已分析输入的日期部分来规范化此值。

> [!NOTE]
> 对于年龄和日期输入，我们将使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) 库来执行初始分析。
> 尽管我们提供了示例代码，但不会解释文本识别器库的工作原理，它只是分析输入的一种方式。
> 有关这些库的详细信息，请参阅存储库的**自述文件**。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

将以下验证方法添加到机器人。

**Bots/CustomPromptBot.cs** [!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs** [!code-javascript[custom prompt bot](~/../botbuilder-samples/samples//javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=118-189)]

---

## <a name="test-the-bot-locally"></a>在本地测试机器人
下载并安装用于在本地测试机器人的 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)。

1. 在计算机本地运行示例。 如需说明，请参阅 [C# 示例](https://aka.ms/cs-primitive-prompt-sample)或 [JS 示例](https://aka.ms/js-primitive-prompt-sample)的自述文件。
1. 按如下所示使用仿真器测试机器人。

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>其他资源

[对话库](bot-builder-concept-dialog.md)提供的某些类可以从多个方面将聊天管理自动化。 

## <a name="next-step"></a>后续步骤

> [!div class="nextstepaction"]
> [实现顺序聊天流](bot-builder-dialog-manage-conversation-flow.md)
