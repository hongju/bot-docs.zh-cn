---
title: 向用户发送欢迎消息 | Microsoft Docs
description: 了解如何开发提供欢迎用户体验的机器人。
keywords: 概述, 开发, 用户体验, 欢迎, 个性化体验, C#, JS, 欢迎消息, 机器人, 问候, 欢迎
author: dashel
ms.author: dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eb62df9bd1f74ab6de9b67fe352b1af4620a6bc6
ms.sourcegitcommit: d92fd6233295856052305e0d9e3cba29c9ef496e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2018
ms.locfileid: "51715101"
---
# <a name="send-welcome-message-to-users"></a>向用户发送欢迎消息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

创建任何机器人的主要目标都是让用户参与富有意义的聊天。 实现此目标的最佳方法之一是确保从用户首次连接的那一刻起，他们就了解你的机器人主要用途和功能，以及创建它的原因。 本文提供了帮助你欢迎用户使用机器人的代码示例。

## <a name="prerequisites"></a>先决条件
- 了解[机器人基础知识](bot-builder-basics.md)。 
- [C#](https://aka.ms/proactive-sample-cs) 或 [JS](https://aka.ms/proactive-sample-js) **欢迎用户示例**的副本。 示例中的代码用于解释如何发送欢迎消息。

## <a name="same-welcome-for-different-channels"></a>不同通道的相同欢迎方式
每当用户首次与你的机器人进行互动时，都应当生成欢迎消息。 要实现此目的，可以监视机器人的活动类型并关注新连接。 每个新连接可以生成最多两个聊天更新活动，具体取决于通道。

- 当用户的机器人连接到聊天时生成一个。
- 当用户加入聊天时生成一个。

每当检测到新的聊天更新时简单地生成欢迎消息很容易，但当跨各种通道访问机器人时，这可能会导致意外结果。

有些通道在用户最初连接到该通道时创建一个聊天更新，并且只有在从用户那里收到首个输入消息后才会创建另一个聊天更新。 另一些通道会在用户最初连接到通道时同时生成这两个活动。 在具有两个聊天更新活动的通道上，如果你只是简单地关注聊天更新事件并显示欢迎消息，则用户可能会收到以下内容：

![双重欢迎消息](./media/double_welcome_message.png)

可以通过仅为第二个聊天更新事件生成初始欢迎消息来避免此重复的消息。 当同时满足以下两个条件时可以检测到第二个事件：
- 已发生了一个聊天更新事件。
- 已将新成员（用户）添加到聊天。

以下示例监视新的*聊天更新*活动，仅基于加入聊天的用户发送一条欢迎消息，并设置提示状态标志以忽略用户的初始聊天输入。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们需要为聊天中的给定用户创建一个状态对象及其访问器。

```csharp
/// The state object is used to keep track of various state related to a user in a conversation.
/// In this example, we are tracking if the bot has replied to customer first interaction.
public class WelcomeUserState
{
    public bool DidBotWelcomeUser { get; set; } = false;
}

/// Initializes a new instance of the <see cref="WelcomeUserStateAccessors"/> class.
public class WelcomeUserStateAccessors
{
    public WelcomeUserStateAccessors(UserState userState)
    {
        this.UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<bool> DidBotWelcomeUser { get; set; }

    public UserState UserState { get; }
}
```

然后，我们只需检查活动更新，即可了解是否已将新用户添加到聊天中，然后向他们发送欢迎消息。

```csharp
public class WelcomeUserBot : IBot
{
// Generic message to be sent to user
private const string _genericMessage = @"This is a simple Welcome Bot sample. You can say 'intro' to 
                                         see the introduction card. If you are running this bot in the Bot 
                                         Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel";

// The bot state accessor object. Use this to access specific state properties.
private readonly WelcomeUserStateAccessors _welcomeUserStateAccessors;

// Initializes a new instance of the <see cref="WelcomeUserBot"/> class.
public WelcomeUserBot(WelcomeUserStateAccessors statePropertyAccessor)
{
    _welcomeUserStateAccessors = statePropertyAccessor ?? throw new System.ArgumentNullException("state accessor can't be null");
}

// Every conversation turn for our WelcomeUser Bot will call this method, including
// any type of activities such as ConversationUpdate or ContactRelationUpdate which
// are sent when a user joins a conversation.
// This bot doesn't use any dialogs; it's "single turn" processing, meaning a single
// request and response.
// This bot uses UserState to keep track of first message a user sends.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // Use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.DidBotWelcomeUser.GetAsync(turnContext, () => false);

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser == false)
        {
            // Update user state flag to reflect bot handled first user interaction.
            await _welcomeUserStateAccessors.DidBotWelcomeUser.SetAsync(turnContext, true);
            await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);

            // the channel should sends the user name in the 'From' object
            var userName = turnContext.Activity.From.Name;

            await turnContext.SendActivityAsync($"You are seeing this message because this was your first message ever to this bot.", cancellationToken: cancellationToken);
            await turnContext.SendActivityAsync($"It is a good practice to welcome the user and provide a personal greeting. For example, welcome {userName}.", cancellationToken: cancellationToken);
        }
    }
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate) // Greet when users are added to the conversation.
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Greet anyone that was not the target (recipient) of this message
                // the 'bot' is the recipient for events from the channel,
                // turnContext.Activity.MembersAdded == turnContext.Activity.Recipient.Id indicates the
                // bot was added to the conversation.
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync($"Hi there - {member.Name}. Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync($"You are seeing this message because the bot recieved atleast one 'ConversationUpdate' event,indicating you (and possibly others) joined the conversation. If you are using the emulator, pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' event depends on the channel. You can read more information at https://aka.ms/about-botframewor-welcome-user", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync($"It is a good pattern to use this event to send general greeting to user, explaning what your bot can do. In this example, the bot handles 'hello', 'hi', 'help' and 'intro. Try it now, type 'hi'", cancellationToken: cancellationToken);
                }
            }
        }
    }
    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

添加用户时，此 JavaScript 代码会发送欢迎消息。 这是通过检查聊天活动并验证是否已将新成员添加到聊天来完成的。

``` javascript
// Import required Bot Framework classes.
const { ActivityTypes } = require('botbuilder');
const { CardFactory } = require('botbuilder');

// Welcomed User property name
const WELCOMED_USER = 'DidBotWelcomeUser';

class MainDialog {
    constructor (userState) {
        // Creates a new user property accessor.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
        this.welcomedUserPropery = userState.createProperty(WELCOMED_USER);
    }

    async onTurn(turnContext) 
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) 
        {
            // Read UserState. If the 'DidBotWelcomeUser' does not exist (first time ever for a user)
            // set the default to false.
            let didBotWelcomeUser = await this.welcomedUserPropery.get(turnContext, false);

            // Your bot should proactively send a welcome message to a personal chat the first time
            // (and only the first time) a user initiates a personal chat with your bot.
            if (didBotWelcomeUser === false) 
            {
                // The channel should send the user name in the 'From' object
                let userName = turnContext.activity.from.name;
                await turnContext.sendActivity("You are seeing this message because this was your first message ever sent to this bot.");
                await turnContext.sendActivity(`It is a good practice to welcome the user and provdie personal greeting. For example, welcome ${userName}.`);
                
                // Set the flag indicating the bot handled the user's first message.
                await this.welcomedUserPropery.set(turnContext, true);
            }
            . . .
            
            // Save state changes
            await this.userState.saveChanges(turnContext);
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
            // Send greeting when users are added to the conversation.
            await this.sendWelcomeMessage(turnContext);
        } else {
            // Generic message for all other activities
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    
    // Sends welcome messages to conversation members when they join the conversation.
    // Messages are only sent to conversation members who aren't the bot.
    async sendWelcomeMessage(turnContext) {
        // If any new membmers added to the conversation
        if (turnContext.activity && turnContext.activity.membersAdded) {
            // Define a promise that will welcome the user
            async function welcomeUserFunc(conversationMember) {
                // Greet anyone that was not the target (recipient) of this message.
                // The bot is the recipient of all events from the channel, including all ConversationUpdate-type activities
                // turnContext.activity.membersAdded !== turnContext.activity.aecipient.id indicates 
                // a user was added to the conversation 
                if (conversationMember.id !== this.activity.recipient.id) {
                    // Because the TurnContext was bound to this function, the bot can call
                    // `TurnContext.sendActivity` via `this.sendActivity`;
                    await this.sendActivity(`Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.`);
                    await this.sendActivity("You are seeing this message because the bot recieved atleast one 'ConversationUpdate'" + 
                                            "event,indicating you (and possibly others) joined the conversation. If you are using the emulator, "+ 
                                            "pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' "+
                                            "event depends on the channel. You can read more information at https://aka.ms/about-botframewor-welcome-user");
                    await this.sendActivity(`It is a good pattern to use this event to send general greeting to user, explaining what your bot can do. `+ 
                                            `In this example, the bot handles 'hello', 'hi', 'help' and 'intro. ` +
                                            `Try it now, type 'hi'`);
                }
            }
    
            // Prepare Promises to greet the  user.
            // The current TurnContext is bound so `replyForReceivedAttachments` can also send replies.
            const replyPromises = turnContext.activity.membersAdded.map(welcomeUserFunc.bind(turnContext));
            await Promise.all(replyPromises);
        }
    }
}

module.exports = MainDialog;
```

---

## <a name="discard-initial-user-input"></a>放弃初始用户输入
考虑用户的输入何时可能实际包含有用的信息也很重要，并且这可能也因通道而异。 为了确保用户在所有可能的通道上都获得良好的体验，我们通过提供初始提示并设置要在用户的回复中查找的关键字来避免处理无效的响应数据。

## <a name="ctabcsharpmulti"></a>[C#](#tab/csharpmulti)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // Use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.DidBotWelcomeUser.GetAsync(turnContext, () => false);

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser == false)
        {
            // Previous Code Sample
        }
        else
        {
            // This example hardcodes specific uterances. You should use LUIS or QnA for more advance language understanding.
            var text = turnContext.Activity.Text.ToLowerInvariant();
            switch (text)
            {
                case "hello":
                case "hi":
                    await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
                    break;
                case "intro":
                case "help":
                default:
                    await turnContext.SendActivityAsync(_genericMessage, cancellationToken: cancellationToken);
                    break;
            }
        }
    }
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate) // Greet when users are added to the conversation.
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Previous Code Sample
            }
        }
    }
    else
    {
        // Default behaivor for all other type of events.
        var ev = turnContext.Activity.AsEventActivity();
        await turnContext.SendActivityAsync($"Received event: {ev.Name}");
    }
    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}

```

## <a name="javascripttabjsmulti"></a>[JavaScript](#tab/jsmulti)

``` javascript
class MainDialog 
{ 
    // Previous Code Sample
    
    async onTurn(turnContext) 
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            
            // Previous Code Sample
            
            if (didBotWelcomeUser === false) 
            {
                // Previous Code Sample
            }
            else 
            {
                // This example hardcodes specific uterances. You should use LUIS or QnA for more advance language understanding.
                let text = turnContext.activity.text.toLowerCase();
                switch (text) 
                {
                    case "hello":
                    case "hi":
                        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
                        break;
                    case "intro":
                    case "help":
                    default :
                        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                                        see the introduction card. If you are running this bot in the Bot 
                                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
                }
            }
        }
       
       // Previous Sample Code
    } 
}
```

---

## <a name="using-adaptive-card-greeting"></a>使用自适应卡问候语

问候用户的另一种方法是使用自适应卡问候语。 可以在此处的[发送自适应卡](./bot-builder-howto-add-media-attachments.md)了解有关自适应卡问候语的更多信息。

## <a name="ctabcsharpwelcomeback"></a>[C#](#tab/csharpwelcomeback)

```csharp
// Sends an adaptive card greeting.
private static async Task SendIntroCardAsync(ITurnContext turnContext, CancellationToken cancellationToken)
{
    var response = turnContext.Activity.CreateReply();

    var introCard = File.ReadAllText(@".\Resources\IntroCard.json");

    response.Attachments = new List<Attachment>
    {
        new Attachment()
        {
            ContentType = "application/vnd.microsoft.card.adaptive",
            Content = JsonConvert.DeserializeObject(introCard),
        },
    };

    await turnContext.SendActivityAsync(response, cancellationToken);
}
```

接下来，我们可以使用以下 await 命令发送卡。 让我们将此项放入机器人 _switch (text) case "help"_。

```csharp
switch (text)
{
    case "hello":"
    case "hi":
        await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
        break;
    case "intro":
    case "help":
        await SendIntroCardAsync(turnContext, cancellationToken);
        break;
    default:
        await turnContext.SendActivityAsync(_genericMessage, cancellationToken: cancellationToken);
        break;
}
```


## <a name="javascripttabjswelcomeback"></a>[JavaScript](#tab/jswelcomeback)

首先，我们要将我们的自适应卡添加到 _index.js_ 顶部 Imports 正下方的机器人。

``` javascript
// Adaptive Card content
const IntroCard = require('./Resources/IntroCard.json');
```

接下来我们只需使用机器人的 _switch (text)_ _case "help"_ 部分中的以下代码通过自适应卡响应用户提示。

``` javascript
switch (text) 
{
    case "hello":
    case "hi":
        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
        break;
    case "intro":
    case "help":
        await turnContext.sendActivity({
            text: 'Intro Adaptive Card',
            attachments: [CardFactory.adaptiveCard(IntroCard)]
        });
        break;
    default :
        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                        see the introduction card. If you are running this bot in the Bot 
                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
}
```
---
## <a name="test-the-bot"></a>测试机器人
参阅[自述文件](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/03.welcome-user/readme.md)，获取有关运行和测试机器人的说明。 

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [收集用户输入](bot-builder-prompts.md)
