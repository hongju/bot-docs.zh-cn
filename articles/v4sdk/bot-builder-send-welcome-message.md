---
title: 向用户发送欢迎消息 | Microsoft Docs
description: 了解如何开发提供欢迎用户体验的机器人。
keywords: 概述, 开发, 用户体验, 欢迎, 个性化体验, C#, JS, 欢迎消息, 机器人, 问候, 欢迎
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/13/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0858cf73c1a744100c2f7106013fd963cff84454
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032520"
---
# <a name="send-welcome-message-to-users"></a>向用户发送欢迎消息

[!INCLUDE[applies-to](../includes/applies-to.md)]

创建任何机器人的主要目标都是让用户参与富有意义的聊天。 实现此目标的最佳方法之一是确保从用户首次连接的那一刻起，他们就了解你的机器人主要用途和功能，以及创建它的原因。 本文提供了帮助你欢迎用户使用机器人的代码示例。

## <a name="prerequisites"></a>先决条件
- 了解[机器人基础知识](bot-builder-basics.md)。 
- [C# 示例](https://aka.ms/welcome-user-mvc)或 [JS 示例](https://aka.ms/bot-welcome-sample-js)中**欢迎用户示例**的副本。 示例中的代码用于解释如何发送欢迎消息。

## <a name="about-this-sample-code"></a>关于此示例代码
此示例代码演示如何在新用户开始连接到机器人时检测到他们并发送欢迎消息。 下图显示了此机器人的逻辑流。 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
机器人遇到的两个主要事件为：
- `OnMembersAddedAsync`：在新用户连接到机器人时调用
- `OnMessageActivityAsync`：在收到新用户输入时调用。

![欢迎用户逻辑流](media/welcome-user-flow.png)

只要有新用户进行连接，机器人就会向其提供 `WelcomeMessage`、`InfoMessage` 和 `PatternMessage`。 收到新用户输入以后，系统会检查 WelcomeUserState，看是否已将 `DidBotWelcomeUser` 设置为 _true_。 如果否，则会将初始的欢迎用户消息返回给用户。

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
机器人遇到的两个主要事件为：
- `onMembersAdded`：在新用户连接到机器人时调用
- `onMessage`：在收到新用户输入时调用。

![欢迎用户逻辑流](media/welcome-user-flow-js.png)

只要有新用户进行连接，机器人就会向其提供 `welcomeMessage`、`infoMessage` 和 `patternMessage`。 收到新用户输入以后，系统会检查 `welcomedUserProperty`，看是否已将 `didBotWelcomeUser` 设置为 _true_。 如果否，则会将初始的欢迎用户消息返回给用户。

---

 如果 DidBotWelcomeUser 为 _true_，则会评估用户的输入。 此机器人会根据用户的输入内容执行下述操作之一：
- 回显从用户收到的问候语。
- 显示一张英雄卡，其中包含有关机器人的其他信息。
- 重新发送 `WelcomeMessage`，说明此机器人的预期输入。

## <a name="create-user-object"></a>创建用户对象
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
用户状态对象在启动时创建，依赖项会注入机器人构造函数中。

**Startup.cs** [!code-csharp[ConfigureServices](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=29-33)]

**WelcomeUserBot.cs** [!code-csharp[Class](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
启动时，内存存储和用户状态都在 index.js 中定义。

**Index.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/Index.js?range=8-10,33-41)]

---

## <a name="create-property-accessors"></a>创建属性访问器
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
我们现在创建一个属性访问器，该访问器在 OnMessageActivityAsync 方法中为我们提供 WelcomeUserState 的句柄。
接下来，调用 GetAsync 方法以获取已正确设置了范围的密钥。 然后，使用 `SaveChangesAsync` 方法在每次用户输入迭代以后保存用户状态数据。

**WelcomeUserBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71, 102-105)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
我们现在创建一个属性访问器，该访问器为我们提供在 UserState 中持久保存的 WelcomedUserProperty 的句柄。

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=7-10,16-22)]

---

## <a name="detect-and-greet-newly-connected-users"></a>检测并问候新连接的用户

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 **WelcomeUserBot** 中使用 `OnMembersAddedAsync()` 检查活动更新即可了解是否已将新用户添加到聊天中，然后向该用户发送一组（共三条）初始的欢迎消息：`WelcomeMessage`、`InfoMessage`、`PatternMessage`。 此交互的完整代码显示在下面。

**WelcomeUserBot.cs** [!code-csharp[WelcomeMessages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-40, 55-66)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
添加用户时，此 JavaScript 代码会发送初始的欢迎消息。 这是通过检查聊天活动并验证是否已将新成员添加到聊天中来完成的。

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=65-87)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>欢迎新用户并放弃初始输入

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
此外还必须考虑在实际情况下用户的输入何时可能包含有用信息，这可能因通道而异。 为确保用户在所有可能的通道上获得良好的体验，我们将检查状态标志 _didBotWelcomeUser_。如果其值为“false”，我们将不处理初始用户输入， 而是向用户提供初始欢迎消息。 然后，布尔型 _welcomedUserProperty_ 会被设置为“true”并存储在 UserState 中。现在，代码将处理此用户在所有其他消息活动中的输入。

**WelcomeUserBot.cs** [!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-82)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
此外还必须考虑在实际情况下用户的输入何时可能包含有用信息，这可能因通道而异。 为确保用户在所有可能的通道上获得良好的体验，我们将检查 didBotWelcomedUser 属性。如果该属性不存在，我们会将其设置为“false”，并且不处理初始用户输入， 而是向用户提供初始欢迎消息。 然后，布尔值 _didBotWelcomeUser_ 将设置为“true”，代码将处理所有其他消息活动中的用户输入。

**WelcomeBot.js** [!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-38,57-59,63)]

---

## <a name="process-additional-input"></a>处理其他输入

欢迎新用户以后，将会针对每个消息轮次评估用户输入信息，机器人会根据该用户输入的上下文提供响应。 以下代码显示用于生成该响应的决策逻辑。 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
输入为“简介”或“帮助”时，会调用函数 `SendIntroCardAsync`，为用户呈现一张说明性的英雄卡。 我们会在本文下一部分详细分析该代码。

**WelcomeUserBot.cs** [!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
输入为“简介”或“帮助”时，会通过 CardFactory 为用户呈现一张简介性的自适应卡片。 我们会在本文下一部分详细分析该代码。

**WelcomeBot.js** [!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

---

## <a name="using-hero-card-greeting"></a>使用英雄卡问候语

如上所述，某些用户输入会导致系统生成一张英雄卡来响应用户请求。 可以在此处详细了解英雄卡问候语：[发送简介卡](./bot-builder-howto-add-media-attachments.md)。 下面是创建此机器人的英雄卡响应所需的代码。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**WelcomeUserBot.cs** [!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-127)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**WelcomeBot.js** [!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=91-116)]

---

## <a name="test-the-bot"></a>测试机器人

下载并安装最新的 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

1. 在计算机本地运行示例。 如需说明，请参阅 [C# 示例](https://aka.ms/welcome-user-mvc)或 [JS 示例](https://aka.ms/bot-welcome-sample-js)的自述文件。
1. 按如下所示使用仿真器测试机器人。

![测试欢迎机器人示例](media/welcome-user-emulator-1.png)

测试英雄卡问候语。

![测试欢迎机器人卡片](media/welcome-user-emulator-2.png)

## <a name="additional-resources"></a>其他资源

在[向消息添加媒体](./bot-builder-howto-add-media-attachments.md)一文中详细了解各种媒体响应。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [收集用户输入](bot-builder-prompts.md)
