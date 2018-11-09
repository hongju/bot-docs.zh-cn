---
title: 管理聊天和用户状态 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 来保存和检索状态数据。
keywords: 聊天状态, 用户状态, 聊天流
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/18/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2a3709111b048730805b5578306c669591122dda
ms.sourcegitcommit: 633008f8db06f1bb5be7bacdb7dd8de6f8165328
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/01/2018
ms.locfileid: "50753605"
---
# <a name="manage-conversation-and-user-state"></a>管理聊天和用户状态

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

对于出色的机器人设计，其关键是跟踪聊天上下文，使机器人能够记住上一问题答案等内容。 根据机器人的用途，甚至可能需要跟踪状态或长时间存储信息，使信息保留时间超过聊天生存期。 机器人的状态是它为了正确响应传入消息而记住的信息。 Bot Builder SDK 提供两种类，用于存储和检索作为与用户或聊天关联的对象的状态数据。

- **聊天状态**可帮助机器人跟踪机器人与用户之间的当前聊天。 如果机器人需要完成一系列步骤或在聊天主题之间进行切换，可使用聊天属性来管理按顺序执行的步骤或跟踪当前主题。 

- **用户状态**可用于多种目的，例如确定用户之前聊天的中断位置或仅按名称来问候返回的用户。 如果存储了用户的首选项，则可在下次聊天时使用该信息自定义聊天。 例如，可提醒用户注意有关其感兴趣话题的新闻文章，或者在可进行预约时提醒用户。 

`ConversationState` 和 `UserState` 是状态类，是 `BotState` 类的特化，其中包含的策略可以控制存储在其中的对象的生存期和作用域。 需存储状态的组件会创建一个属性并将其注册到某个类型，然后使用属性访问器来访问状态。 机器人状态管理器可以使用内存存储、CosmosDB 和 Blob 存储。 

> [!NOTE] 
> 请使用机器人状态管理器来存储首选项、用户名或订购的最后一个项目，但不要用它来存储关键的业务数据。 对于关键数据，请创建你自己的存储组件或将数据直接写入[存储](bot-builder-howto-v4-storage.md)。
> 内存中数据存储仅用于测试。 此存储是易失性的临时存储。 每次重启机器人时都会清除数据。

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>使用聊天状态和用户状态来指示聊天流
在设计聊天流时，通过定义状态标记来指示聊天流非常有用。 该标记可以是简单的布尔类型，也可以是包含当前主题名称的类型。 该标记可帮助跟踪在聊天中所处的位置。 例如，布尔类型标记可指出你是否参与了聊天，而主题名称属性则可以指出你目前在进行哪个聊天。



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>聊天和用户状态
对于此操作方法，一开始可以使用[带计数器的回显机器人示例](https://aka.ms/EchoBot-With-Counter)。 首先在 `TopicState.cs` 中创建 `TopicState` 类，管理当前的聊天主题，如下所示：

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
然后在 `UserProfile.cs` 中创建 `UserProfile` 类，管理用户状态。
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
`TopicState` 类有一个用于跟踪我们在聊天中的位置的标记，并使用聊天状态来存储它。 提示会初始化为“askName”，以便启动聊天。 机器人收到用户的响应以后，提示会被重新定义为“askNumber”，以便启动下一轮聊天。 `UserProfile` 类跟踪用户名称和电话号码，并将其存储在用户状态中。

### <a name="property-accessors"></a>属性访问器
示例中的 `EchoBotAccessors` 类作为单一实例创建，通过依赖项注入传递到 `class EchoWithCounterBot : IBot` 构造函数中。 `EchoBotAccessors` 类包含 `ConversationState`、`UserState` 和关联的 `IStatePropertyAccessor`。 `conversationState` 对象存储主题状态，`userState` 对象存储用户配置文件信息。 `ConversationState` 和 `UserState` 对象稍后会在 Startup.cs 文件中创建。 可以在聊天和用户状态对象中持久保存聊天和用户范围内的任何内容。 

更新了构造函数，使之包含 `UserState`，如下所示：
```csharp
public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
为 `TopicState` 和 `UserProfile` 访问器创建唯一名称。
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
接下来，定义两个访问器。 第一个存储聊天的主题，第二个存储用户的名称和电话号码。
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

用于获取 ConversationState 的属性已定义，但需添加 `UserState`，如下所示：
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
进行更改后，请保存文件。 接下来，我们将更新 Startup 类，以便创建 `UserState` 对象来持久保存用户范围内的任何内容。 `ConversationState` 已存在。 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
`options.State.Add(ConversationState);` 和 `options.State.Add(userState);` 行分别添加聊天状态和用户状态。 接下来，创建并注册状态访问器。 在此处创建的访问器每一轮都传递到 IBot 派生的类中。 修改代码，使之包含用户状态，如下所示：
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

接下来，使用 `TopicState` 和 `UserProfile` 创建这两个访问器，并通过依赖项注入将其传递到 `class EchoWithCounterBot : IBot` 类中。
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new EchoBotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>(EchoBotAccessors.TopicStateName),
        UserProfile = userState.CreateProperty<UserProfile>(EchoBotAccessors.UserProfileName),
     };

     return accessors;
 });
```

聊天和用户状态通过 `services.AddSingleton` 代码块链接到单一实例，并通过代码中以 `var accessors = new EchoBotAccessor(conversationState, userState)` 开头的状态存储访问器进行保存。

### <a name="use-conversation-and-user-state-properties"></a>使用聊天和用户状态属性 
在 `EchoWithCounterBot : IBot` 类的 `OnTurnAsync` 处理程序中修改代码，提示用户先输入用户名称，然后输入电话号码。 若要跟踪自己在聊天中的位置，请使用在 TopicState 中定义的 Prompt 属性。 该属性已初始化为“askName”。 获得用户名称以后，请将该属性设置为“askNumber”，然后将 UserName 设置为用户键入的名称。 电话号码收到以后，请发送一条确认消息，并将 Prompt 设置为 'confirmation'，因为你位于聊天的末尾。

```csharp
if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>聊天和用户状态

对于此操作方法，一开始可以使用[带计数器的回显机器人示例](https://aka.ms/EchoBot-With-Counter-JS)。 此示例已使用 `ConversationState` 来存储消息计数。 需添加 `TopicStates` 对象来跟踪聊天状态，添加 `UserState` 来跟踪 `userProfile` 对象中的用户信息。 

在主机器人的 `index.js` 文件中，向需求列表添加 `UserState`：

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

接下来，使用 `MemoryStorage` 作为存储提供程序来创建 `UserState`，然后将其作为第二个参数传递给 `MainDialog` 类。

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

在 `bot.js` 文件中更新构造函数，使之接受 `userState` 作为第二个参数。 然后从 `conversationState` 创建 `topicState` 属性，从 `userState` 创建 `userProfile` 属性。

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>使用聊天和用户状态属性

在 `MainDialog` 类的 `onTurn` 处理程序中修改代码，提示用户先输入用户名称，然后输入电话号码。 若要跟踪自己在聊天中的位置，请使用在 `topicState` 中定义的 `prompt` 属性。 该属性初始化为“askName”。 获得用户名称以后，请将该属性设置为“askNumber”，然后将 UserName 设置为用户键入的名称。 电话号码收到以后，请发送一条确认消息，并将 Prompt 设置为 `undefined`，因为你位于聊天的末尾。

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。 
2. 选择创建 Visual Studio 解决方案时所在目录中的 .bot 文件。

### <a name="interact-with-your-bot"></a>与机器人交互

向机器人发送消息“Hi”，机器人会要求你提供姓名和电话号码。 在你提供该信息后，机器人会发送确认消息。 如果你此后继续聊天，机器人会再次执行相同的循环。

![正在运行的模拟器](../media/emulator-v4/emulator-running-manage-state.png)

如果决定自行管理状态，请参阅[使用自己的提示管理聊天流](bot-builder-primitive-prompts.md)。 还可使用瀑布对话框。 该对话持续跟踪聊天状态，因此你无需创建标记进行跟踪。 有关详细信息，请参阅[使用对话框管理简单的聊天](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="next-steps"></a>后续步骤
你已了解如何使用状态从存储中读取机器人数据或将其写入到存储中，接下来让我们了解如何直接从存储中读取或直接写入存储。

> [!div class="nextstepaction"]
> [如何直接写入存储](bot-builder-howto-v4-storage.md)。
