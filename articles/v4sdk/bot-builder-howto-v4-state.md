---
title: 保存用户和聊天数据 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK 来保存和检索状态数据。
keywords: 聊天状态, 用户状态, 聊天, 保存状态, 管理机器人状态
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b198408a800feaedff3c13dbab965ae63307eeb0
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215340"
---
# <a name="save-user-and-conversation-data"></a>保存用户和聊天数据

[!INCLUDE[applies-to](../includes/applies-to.md)]

机器人在本质上是无状态的。 部署机器人后，根据轮次的不同，它不一定会在相同的进程或计算机中运行。 但是，机器人可能需要跟踪聊天上下文，以便可以管理聊天行为并记住先前问题的回答。 使用 Bot Framework SDK 的状态和存储功能可将状态添加到机器人。 机器人使用状态管理和存储对象来管理并持久保存状态。 状态管理器提供一个抽象层，让你可以使用属性访问器来访问状态属性（不考虑基础存储的类型）。

## <a name="prerequisites"></a>先决条件
- 需要了解[机器人基础知识](bot-builder-basics.md)以及机器人如何[管理状态](bot-builder-concept-state.md)。
- 本文中的代码基于**状态管理机器人示例**。 需要 [CSharp](https://aka.ms/statebot-sample-cs) 或 [JavaScript](https://aka.ms/statebot-sample-js) 示例的副本。

## <a name="about-this-sample"></a>关于此示例
收到用户输入以后，此示例会检查存储的聊天状态，看系统以前是否已提示该用户提供其名称。 如果否，则系统会请求用户的名称，并将该输入存储在用户状态中。 如果是这样，系统会使用用户状态中存储的名称与用户聊天，并将用户的输入数据以及接收时间、输入通道 ID 返回给用户。 将会从用户聊天数据中检索时间和通道 ID，然后将其保存到聊天状态中。 以下图示显示了机器人、用户配置文件和聊天数据类之间的关系。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![状态机器人示例](media/StateBotSample-Overview.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![状态机器人示例](media/StateBotSample-JS-Overview.png)

---

## <a name="define-classes"></a>定义类

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

设置状态管理时，第一步是定义类，这些类将包含我们需要在用户和聊天状态中管理的所有信息。 就此示例来说，我们已定义以下内容：

- 在 **UserProfile.cs** 中，我们为机器人将要收集的用户信息定义 `UserProfile` 类。 
- 在 **ConversationData.cs** 中，我们定义一个 `ConversationData` 类，用于在收集用户信息时控制聊天状态。

以下代码示例演示如何为 UserProfile 类创建定义。

**UserProfile.cs** [!code-csharp[UserProfile](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/UserProfile.cs?range=7-11)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

第一步将要求使用 botbuilder 服务，该服务包含 `UserState` 和 `ConversationState` 的定义。

**index.js** [!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=7-9)]

---

## <a name="create-conversation-and-user-state-objects"></a>创建聊天和用户状态对象

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

接下来，我们注册用于创建 `UserState` 和 `ConversationState` 对象的 `MemoryStorage`。 用户和聊天状态对象在`Startup`时创建，依赖项会注入机器人构造函数中。 机器人的其他已注册服务：凭据提供程序、适配器和机器人实现。

**Startup.cs** [!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=16-38)]

**StateManagementBot.cs** [!code-csharp[StateManagement](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=15-22)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

接下来，我们注册随后用于创建 `UserState` 和 `ConversationState` 对象的 `MemoryStorage`。

**index.js** [!code-javascript[DefineMemoryStore](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=32-38)]

---

## <a name="add-state-property-accessors"></a>添加状态属性访问器

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

现在，我们使用 `CreateProperty` 方法来创建属性访问器，该方法提供 `BotState` 对象的句柄。 每个状态属性访问器允许获取或设置关联状态属性的值。 在使用状态属性之前，我们将使用每个访问器从存储加载属性，并从状态缓存获取该属性。 为了将范围设置适当的密钥与状态属性相关联，我们调用 `GetAsync` 方法。

**StateManagementBot.cs** [!code-csharp[StateAccessors](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-46)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

现在，我们为 `UserState` 和 `ConversationState` 创建属性访问器。 每个状态属性访问器允许获取或设置关联状态属性的值。 我们使用每个访问器从存储加载关联的属性，并从缓存中检索其当前状态。

**StateManagementBot.js**。
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=6-19)]

---

## <a name="access-state-from-your-bot"></a>从机器人访问状态
前面几个部分介绍了将状态属性访问器添加到机器人的初始化时步骤。 现在，我们可以在运行时使用这些访问器来读取和写入状态信息。 下面的代码示例使用以下逻辑流：
- 如果 userProfile.Name 为空且 conversationData.PromptedUserForName 为 _true_，我们会检索提供的用户名并将其存储在用户状态中。
- 如果 userProfile.Name 为空且 conversationData.PromptedUserForName 为 _false_，我们会请求用户提供其名称。
- 如果 userProfile.Name 以前已存储，我们会从用户输入中检索消息时间和通道 ID，将所有数据回显给用户，然后将检索的数据存储在聊天状态中。

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**StateManagementBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-85)]

在退出轮次处理程序之前，我们会使用状态管理对象的 _SaveChangesAsync()_ 方法将所有状态更改写回到存储中。

**StateManagementBot.cs** [!code-csharp[OnTurnAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=24-31)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**StateManagementBot.js** [!code-javascript[OnMessage](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=21-54)]

在退出每个对话轮次之前，我们会使用状态管理对象的 _saveChanges()_ 方法来持久保存所有更改，方法是将状态写回到存储中。

**StateManagementBot.js** [!code-javascript[OnDialog](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=60-67)]

---

## <a name="test-the-bot"></a>测试机器人

下载并安装最新的 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

1. 在计算机本地运行示例。 如需说明，请参阅 [C# 示例](https://aka.ms/statebot-sample-cs)或 [JS 示例](https://aka.ms/statebot-sample-js)的自述文件。
1. 按如下所示使用仿真器测试机器人。

![测试状态机器人示例](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>其他资源

**隐私：** 如果你想要存储用户的个人数据，应确保遵守[一般数据保护条例](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)。

**状态管理：** 所有状态管理调用都是异步的，默认采用“上次写入优先”。 在实践中，应在机器人中获取、设置和保存尽量邻近的状态。

**关键业务数据：** 请使用机器人状态来存储首选项、用户名或订购的最后一个项目，但不要用它来存储关键的业务数据。 对于关键数据，请[创建自己的存储组件](bot-builder-custom-storage.md)或将数据直接写入[存储](bot-builder-howto-v4-storage.md)。

**Recognizer-Text：** 该示例使用 Microsoft/Recognizers-Text 库来分析和验证用户输入。 有关详细信息，请参阅[概述](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview)页。

## <a name="next-steps"></a>后续步骤

了解如何配置状态以帮助自己在存储中读取和写入机器人数据后，接下来让我们了解如何向用户提出一系列问题、验证其回答，然后保存其输入。

> [!div class="nextstepaction"]
> [创建自己的提示来收集用户输入](bot-builder-primitive-prompts.md)。
