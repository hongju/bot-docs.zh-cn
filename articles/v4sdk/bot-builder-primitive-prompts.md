---
title: 使用原始提示管理聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK 中使用原始提示来管理聊天流。
keywords: 聊天流, 提示
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b563197c111a37cf2f0f14fef183d52f38cca66
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999156"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>使用自己的提示来提示用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人与用户之间的聊天通常涉及到请求（提示）用户输入信息、分析用户的响应，然后对该信息采取措施。 有关[使用“对话”库提示用户](bot-builder-prompts.md)的主题详细介绍了如何使用“对话”库来提示用户输入。 除此之外，“对话”库还会负责跟踪当前聊天，以及当前提出的问题。 它还提供方法用于请求和验证不同类型的信息，例如文本、数字、日期和时间，等等。

在某些情况下，内置的“对话”可能不是适合机器人的解决方案；“对话”可能会给简单的机器人带来过多的开销、过于严格，或者无法让机器人达到你自己的目的。 在这种情况下，可以跳过该库，并实现自己的提示逻辑。 本主题介绍如何设置基本的聊天机器人，以便可以使用自己的提示来管理聊天。

## <a name="track-prompt-states"></a>跟踪提示状态

在提示驱动的聊天中，需要跟踪你目前处于聊天的哪个位置，以及当前提出的问题。 在代码中，这相当于管理多个标志。

例如，可以创建多个想要跟踪的属性。

这些状态会跟踪我们当前所处的主题和提示。 为了确保在部署到云后这些标志可按预期方式运行，我们将其存储在[聊天状态](bot-builder-howto-v4-state.md)中。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们将创建两个类来跟踪状态。 **TopicState** 用于跟踪会话提示的进度，**UserProfile** 用于跟踪用户提供的信息。 我们将在稍后的步骤中将此信息存储在机器人[状态](bot-builder-howto-v4-state.md)中。

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
const storage = new MemoryStorage(); // Volatile memory
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);
const dialogState = conversationState.createProperty('dialogState');
const userProfile = userState.createProperty('userProfile');
```

请将此代码放在主机器人逻辑中。

```javascript
// Pull the state of the dialog out of the conversation state manager.
const convo = await dialogState.get(context, {
    prompt: undefined,
    topic: 'profile'
});

// Pull the user profile out of the user state manager.
const userProfile = await userProfile.get(context, {  // Define the user's profile object
        "userName": undefined,
        "age": undefined,
        "workPlace": undefined
    }
);
```

---

## <a name="manage-a-topic-of-conversation"></a>管理聊天主题

在有序聊天（例如，从用户收集信息的聊天）中，需要知道用户何时进入了聊天，以及用户在聊天中所处的位置。 可以通过在主机器人轮次处理程序中设置并检查前面定义的提示标志，然后采取相应的措施，来跟踪此信息。 以下示例演示如何使用这些标志通过聊天来收集用户的个人资料信息。

机器人代码在此处提供，并在下一部分中讨论。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

对于 ASP.NET Core，我们需要首先设置机器人和依赖项注入。

将文件 **EchoWithCounterBot.cs** 重命名为 **PrimitivePromptsBot.cs**，并更新类名。 此类存储我们的机器人逻辑，我们很快就会回来更新它。

将文件 **EchoBotAccessors.cs** 重命名为 **BotAccessors.cs**，并更新类名。 此类包含机器人的状态管理对象和状态属性访问器。 将定义更新为以下内容。

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

从设置 `IStorage` 对象的位置开始，更新 **Startup.cs** 文件的 `ConfigureServices` 方法。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

最后，更新 **PrimitivePromptsBot.cs** 文件中的机器人逻辑。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === ActivityTypes.Message);

        // Set up a list of fields that we need to collect from the user.
        const fields = {
            userName: "What is your name?",
            age: "How old are you?",
            workPlace: "Where do you work?"
        }

        // Pull the state of the dialog out of the conversation state manager.
        const convo = await dialogState.get(context, {
            topic: 'profile',
            prompt: undefined
        });

        // Pull the user profile out of the user state manager.
        const userProfile = await userProfile.get(context, {  // Define the user's profile object
                "userName": undefined,
                "age": undefined,
                "workPlace": undefined
            }
        );

        if (isMessage) {

            if (convo.topic === 'profile') {
                // If a prompt flag is set in the conversation state, use it to capture the incoming value
                // into the appropriate field of the user profile.
                if (convo.prompt) {
                    userProfile[convo.prompt] = context.activity.text;
                }

                 // Determine which fields are not yet set.
                 const empty_fields = [];
                 Object.keys(fields).forEach(function(key) {
                    if (!userProfile[key]) {
                        empty_fields.push({
                           key: key,
                           prompt: fields[key]
                        });
                     }
                 });

                 if (empty_fields.length) {

                    // If all the fields are empty, send a welcome message.
                    if (empty_fields.length == Object.keys(fields).length) {
                        await context.sendActivity('Welcome new user, please fill out your profile information.');
                    }

                    // We have at least one empty field. Prompt for the next empty field.
                    await context.sendActivity(empty_fields[0].prompt);

                    // update the flag to indicate which prompt we just sent
                    // so that the response can be captured at the beginning of the next turn.
                    convo.prompt = empty_fields[0].key;

                 } else {
                    // Our user profile is complete!
                    await context.sendActivity('Thank you. Your profile is complete.');
                    convo.prompt = null;
                    convo.topic = null;

                 }
            } else if (context.activity.text && context.activity.text.match(/hi/ig)) {
                // Check to see if the user said "hi" and respond with a greeting
                await context.sendActivity(`Hi ${ userProfile.userName }.`);
            } else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

        // End the turn by writing state changes back to storage
        await conversationState.saveChanges(context);
        await userState.saveChanges(context);
    });
});

```

---

上面的示例代码将_主题_标志初始化为 `profile`，以便启动配置文件收集聊天。 机器人通过将_提示_标志更新为表示当前所问问题的值，从而在聊天中向前移动。 将此标志设置为正确值后，机器人将知道如何处理从用户收到的下一条消息并相应地处理它。

最后，当机器人询问完信息时，将重置标志。 否则，机器人将陷入循环，永远不会从最后的问题继续前进。

可以扩展此模式来管理更复杂的聊天流：只需定义其他标志，或者根据用户输入将聊天分支即可。

## <a name="manage-multiple-topics-of-conversations"></a>管理多个聊天主题

能够处理一个聊天主题之后，可以扩展机器人逻辑来处理多个聊天主题。 可以通过检查其他条件，然后选择适当的路径，来处理多个主题。

可以扩展上面的示例，以实现其他功能和聊天主题，例如预订餐位或下订单。 根据需要向主题状态添加其他标志以跟踪聊天。

另一种可能有帮助的做法是将代码拆分为独立的函数或方法，以便更轻松地跟踪聊天流。 常见的模式是让机器人评估消息和状态，然后将控制委托给实现该功能细节的函数。

为了帮助你的用户更好地浏览多个聊天主题，请考虑提供主菜单。 例如，使用[建议的操作](bot-builder-howto-add-suggested-actions.md)，可以让用户选择他们想要参与的主题，而不是猜测机器人可以执行的操作。

## <a name="validate-user-input"></a>验证用户输入

“对话”库提供内置的方法用于验证用户的输入，但我们也可以使用自己的提示来实现此目的。 例如，如果我们询问用户的年龄，我们希望确保获得一个数字，而不是像“Bob”这样的答复。

分析数字或日期和时间是一项复杂的任务，不在本主题的讨论范畴之内。 幸运的是，我们可以利用库。 若要分析此信息，可以使用 [Microsoft 文本识别器](https://github.com/Microsoft/Recognizers-Text)库。 此包通过 NuGet 提供，也可以从存储库中下载。 （它也包含在**对话**库中，即使我们在这里没有使用它，也值得注意。）

此库对于解析日期、时间或电话号码等复杂输入特别有用。 此示例要验证聚会订餐人数，但可以延用相同的思路来执行更复杂的验证操作。

以下示例只演示 `RecognizeNumber` 的用法。 有关如何使用该库中其他识别器方法的详细信息，请参阅该[存储库的文档](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用 **Microsoft.Recognizers.Text.Number** 库，请包含 NuGet 包并将其添加到 using 语句中。

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq;
```

然后，定义实际执行验证的函数。

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(value, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        int.TryParse(result.First().Text, out int partySize);

        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivityAsync("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用**识别器**库，在 **app.js** 中需要它：

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

然后，定义实际执行验证的函数。

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if (value < 6) {
            throw new Error(`Party size too small.`);
        } else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    } catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

在处理用户对提示的响应时，请先调用验证函数，然后再转到下一个提示。 如果验证失败，则再次提问。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (topicState.Prompt == "partySize")
{
    if (await ValidatePartySize(turnContext, turnContext.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which
        // is a new class we've added to our state
        // UserFieldInfo partySize;
        partySize.SetValue(userProfile, turnContext.Activity.Text);

        // Ask next question.
        topicState.Prompt = "reserveName";
        await turnContext.SendActivityAsync("Who's name will this be under?");
    }
    else
    {
        // Ask again.
        await turnContext.SendActivityAsync("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if (convo.prompt == "partySize") {
    if (await validatePartySize(context, context.activity.text)) {
        // Save user's response
        reservationInfo.partySize = context.activity.text;

        // Ask next question
        convo.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    } else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>后续步骤

掌握如何自行管理提示状态后，接下来让我们了解如何利用“对话”库来提示用户输入。

> [!div class="nextstepaction"]
> [使用“对话”提示用户输入](bot-builder-prompts.md)
