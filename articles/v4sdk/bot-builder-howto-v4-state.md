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
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8c3696d0642e1b1ce9c3d3e23118a7bd9ab0023b
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224582"
---
# <a name="save-user-and-conversation-data"></a>保存用户和聊天数据

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人在本质上是无状态的。 部署机器人后，根据轮次的不同，它不一定会在相同的进程或计算机中运行。 但是，机器人可能需要跟踪聊天上下文，以便可以管理聊天行为并记住先前问题的回答。 使用 SDK 的状态和存储功能可将状态添加到机器人。

## <a name="prerequisites"></a>先决条件

- 需要了解[机器人基础知识](bot-builder-basics.md)以及机器人如何[管理状态](bot-builder-concept-state.md)。
- 本文中的代码基于 **StateBot** 示例。 需要获取 [C# ](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) 或 [JS]() 示例的副本。
- 用于在本地测试机器人的 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。

## <a name="about-the-sample-code"></a>关于示例代码

本文介绍管理机器人状态的配置方面。 为了添加状态，我们将配置状态属性、状态管理和存储，然后在机器人中使用它们。

- 每个状态属性包含机器人的状态信息。
- 每个状态属性访问器允许获取或设置关联状态属性的值。
- 每个状态管理对象可用于在存储中自动读取和写入关联的状态信息。
- 存储层连接到状态的后备存储（例如，用于测试的内存中存储）或 Azure Cosmos DB 存储（用于生产）。

需要使用状态属性访问器配置机器人。当机器人在运行时处理活动时，可以使用访问器获取和设置状态。 状态属性访问器是使用状态管理对象创建的，而状态管理对象是使用存储层创建的。 因此，我们先从存储级别着手，然后不断提高操作级别。

## <a name="configure-storage"></a>配置存储

由于我们不打算部署此机器人，因此将使用内存存储。在下一步骤中，我们将使用内存存储来配置用户状态和聊天状态。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中配置存储层。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 文件中配置存储层。

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>创建状态管理对象

我们将跟踪用户状态和聊天状态，并在下一步骤中使用它们来创建状态属性访问器。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

创建状态管理对象时，请在 **Startup.cs** 中引用存储层。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 文件中，将 `UserState` 添加到 require 语句。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

然后，在创建聊天和用户状态管理对象时引用存储层。

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>创建状态属性访问器

若要声明状态属性，请先使用某个状态管理对象创建状态属性访问器。 将机器人配置为跟踪以下信息：

- 用户的姓名：将在用户状态中定义。
- 我们是否刚刚向用户询问过其姓名，以及有关用户刚刚发送的消息的其他信息。

机器人使用访问器从轮次上下文获取状态属性。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

首先定义类，用于包含我们要在每个状态类型中管理的所有信息。

- 一个 `UserProfile` 类，用于跟踪机器人要收集的用户信息。
- 一个 `ConversationData` 类，用于跟踪有关消息抵达时间以及消息发送者的信息。

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

接下来，定义一个类用于包含配置机器人实例所需的状态管理信息。

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

将状态管理对象直接传递给机器人的构造函数，并让机器人自行创建状态属性访问器。

创建机器人时，请在 **index.js** 中提供状态管理对象。

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

在 **bot.js** 中，定义管理和跟踪状态时所需的标识符。

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>配置机器人

现在，我们可以定义状态属性访问器并配置机器人。
将为聊天流状态属性访问器使用聊天状态管理对象。
将为用户个人资料状态属性访问器使用用户状态管理对象。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，将 ASP.NET 配置为提供捆绑的状态属性和管理对象。 将通过 ASP.NET Core 中的依赖项注入框架从机器人的构造函数检索此信息。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

当 ASP.NET 创建机器人时，会在机器人的构造函数中提供 `CustomPromptBotAccessors` 对象。

```csharp
// Defines a bot for filling a user profile.
public class CustomPromptBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的构造函数中（在 **bot.js** 文件中），创建状态属性访问器并将其添加到机器人。 此外，添加对状态管理对象的引用，因为需要使用这些引用来保存所做的任何状态更改。

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>从机器人访问状态

前面几个部分介绍了将状态属性访问器添加到机器人的初始化时步骤。
现在，我们可以在运行时使用这些访问器来读取和写入状态信息。

1. 在使用状态属性之前，我们将使用每个访问器从存储加载属性，并从状态缓存获取该属性。
   - 每当通过状态属性的访问器获取该属性时，都应该提供默认值。 否则，可能会收到 null 值错误。
1. 在退出轮次处理程序之前：
   1. 使用访问器的 _set_ 方法将更改推送到机器人状态。
   1. 使用状态管理对象的 _save changes_ 方法将这些更改写入存储。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>测试机器人

1. 在计算机本地运行示例。 如需说明，请参阅 [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) 或 [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot) 示例的自述文件。
1. 按如下所示使用仿真器测试机器人。

![测试状态机器人示例](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>其他资源

**隐私：** 如果你想要存储用户的个人数据，应确保遵守[一般数据保护条例](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr)。

**状态管理：** 所有状态管理调用都是异步的，默认采用“上次写入优先”。 在实践中，应在机器人中获取、设置和保存尽量邻近的状态。

**关键业务数据：** 请使用机器人状态来存储首选项、用户名或订购的最后一个项目，但不要用它来存储关键的业务数据。 对于关键数据，请[创建自己的存储组件](bot-builder-custom-storage.md)或将数据直接写入[存储](bot-builder-howto-v4-storage.md)。

**Recognizer-Text：** 该示例使用 Microsoft/Recognizers-Text 库来分析和验证用户输入。 有关详细信息，请参阅[概述](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview)页。

## <a name="next-step"></a>后续步骤

了解如何配置状态以帮助自己在存储中读取和写入机器人数据后，接下来让我们了解如何向用户提出一系列问题、验证其回答，然后保存其输入。

> [!div class="nextstepaction"]
> [创建自己的提示来收集用户输入](bot-builder-primitive-prompts.md)。
