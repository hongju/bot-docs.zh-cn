---
title: 实现顺序聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用对话管理简单的聊天流。
keywords: 简单聊天流, 顺序聊天流, 对话, 提示, 瀑布, 对话集
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2264b6927ccb863f153f2feb829cc0fb99c711f7
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/28/2018
ms.locfileid: "52452079"
---
# <a name="implement-sequential-conversation-flow"></a>实现顺序聊天流

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以使用对话库来管理简单的和复杂的聊天流。 在简单的交互中，机器人将按固定的顺序运行一组步骤，直到聊天完成。 在本文中，我们将使用一个瀑布对话、几个提示和一个对话集来创建简单的交互，向用户提出一系列问题。

## <a name="prerequisites"></a>先决条件
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- 本文中的代码基于 **multi-turn-prompt** 示例。 需要获取 [C# ](https://aka.ms/cs-multi-prompts-sample) 或 [JS](https://aka.ms/js-multi-prompts-sample) 示例的副本。
- 了解[机器人基础知识](bot-builder-basics.md)、[对话库](bot-builder-concept-dialog.md)、[对话状态](bot-builder-dialog-state.md)和 [.bot](bot-file-basics.md) 文件。


以下部分说明了为大多数机器人实现简单对话所要执行的步骤：

## <a name="configure-your-bot"></a>配置机器人

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们将在 **Startup.cs** 文件的配置代码中初始化机器人对话状态的状态属性访问器。

定义一个 `MultiTurnPromptsBotAccessors` 类用于保存机器人的状态管理对象和状态属性访问器。
此处只显示了代码的某些部分。

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

在 `Statup` 类的 `ConfigureServices` 方法中注册访问器类。
同样，此处只显示了代码的某些部分。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

通过依赖项注入，访问器可供机器人的构造函数代码使用。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 文件中定义状态管理对象。
此处只显示了代码的某些部分。

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

机器人的构造函数将创建机器人的状态属性访问器：`this.dialogState` 和 `this.userProfile`。

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>更新机器人轮次处理程序以调用对话

若要运行对话，机器人的轮次处理程序需要为包含机器人对话的对话集创建对话上下文。 机器人可以定义多个对话集，但根据一般经验法则，只应为机器人定义一个对话集。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

对话从机器人的轮次处理程序运行。 处理程序首先创建 `DialogContext`，并继续运行活动的对话，或根据情况开始新对话。 然后，处理程序在轮次结束时保存聊天和用户状态。

在 `MultiTurnPromptsBot` 类中，我们已定义包含对话集的 `_dialogs` 属性，从中可生成对话上下文。 同样，此处只显示了轮次处理程序代码的一部分。

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

机器人代码使用对话库中的几个类。

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

对话从机器人的轮次处理程序运行。 处理程序首先创建 `DialogContext` (`dc`)，并继续运行活动的对话，或根据情况开始新对话。 然后，处理程序在轮次结束时保存聊天和用户状态。

`MultiTurnBot` 类在 **bot.js** 文件中定义。 此类的构造函数添加对话集的 `dialogs` 属性，从中可生成对话上下文。 此机器人使用 `WHO_ARE_YOU` 对话收集用户数据一次。 填充用户配置文件后，机器人使用 `HELLO_USER` 对话做出响应。 同样，此处只显示了轮次处理程序代码的一部分。

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

在机器人的轮次处理程序中，创建对话集的对话上下文。 对话上下文访问机器人的状态缓存，可有效记住上一轮次停止时聊天所处的位置。

如果有活动的对话，对话上下文的 _continue dialog_ 方法将使用触发此轮次的用户输入来递进此对话；否则，机器人将调用对话上下文的 _begin dialog_ 方法来启动对话。

最后，针对状态管理对象调用 _save changes_ 方法来保存此轮次发生的任何更改。

### <a name="about-dialog-and-bot-state"></a>关于对话和机器人状态

在此机器人中，我们定义了两个状态属性访问器：

* 一个访问器是在对话状态属性的聊天状态中创建的。 对话状态跟踪用户在对话集的对话中所处的位置，由对话上下文更新，例如，当我们调用 begin dialog 或 continue dialog 方法时，就会更新对话状态。
* 一个访问器是在用户配置文件属性的用户状态中创建的。 机器人使用此访问器来跟踪有关用户的信息，我们将在机器人代码中显式管理此状态。

状态属性访问器的 _get_ 和 _set_ 方法在状态管理对象的缓存中获取和设置属性值。 首次在某个轮次中请求状态属性的值时，将填充该缓存，但是，必须显式持久保存该值。 为了持久保存对这两个状态属性所做的更改，我们将调用相应状态管理对象的 _save changes_ 方法。

## <a name="initialize-your-bot-and-define-your-dialog"></a>初始化机器人并定义对话

我们的简单聊天建模为一系列向用户提出的问题。 适用于 C# 和 JavaScript 版本的步骤略有不同：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 请求用户输入其姓名。
1. 询问他们是否愿意提供其年龄。
1. 如果他们愿意，则请求提供年龄；否则跳过此步骤。
1. 询问收集的信息是否正确。
1. 发送状态消息并结束。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

对于 `who_are_you` 对话：

1. 请求用户输入其姓名。
1. 询问他们是否愿意提供其年龄。
1. 如果他们愿意，则请求提供年龄；否则跳过此步骤。
1. 发送状态消息并结束。

对于 `hello_user` 对话：

1. 显示机器人收集的用户信息。

---

下面是在定义自己的瀑布步骤时要记住的几个要点。

* 每个机器人轮次会反映用户提供的输入，后接机器人提供的响应。 因此，将在瀑布步骤结束时请求用户输入，并在下一个瀑布步骤中接收用户的回答。
* 每个提示实际上是由两个步骤组成的对话，该对话会循环显示提示，直到收到“有效”输入。 

在此示例中，对话在机器人文件中定义，并在机器人的构造函数中初始化。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

为对话集定义一个实例属性。

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

在机器人的构造函数内创建对话集，向该对话集内添加提示和瀑布对话。

```csharp
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

在此示例中，我们将每个步骤定义为单独的方法。 也可以使用 lambda 表达式在构造函数中定义内联步骤。

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}


private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此示例中，瀑布对话是在 **bot.js** 文件中定义的。

定义要用于状态属性访问器、提示和对话的标识符。

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

在机器人的构造函数中定义并创建对话集，并将提示和瀑布对话添加到该集。
`NumberPrompt` 包含自定义验证，以确保用户输入大于 0 的年龄。

```javascript
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }
        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

由于我们的对话步骤方法引用实例属性，因此我们需要使用 `bind` 方法，以便在每个步骤方法中正确解析 `this` 对象。

在此示例中，我们将每个步骤定义为单独的方法。 也可以使用 lambda 表达式在构造函数中定义内联步骤。

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

---

此示例从对话内部更新用户配置文件状态。 这种做法适用于简单的机器人；如果你想要在多个机器人中重复使用某个对话，则此做法不适用。

有多种选项可将对话步骤与机器人状态相分离。 例如，在对话收集完整信息后，你可以：

* 使用 _end dialog_ 方法将收集的数据作为返回值返回给父上下文。 此上下文可能是机器人的轮次处理程序，或对话堆栈中以前的某个活动对话。 这就是提示类的设计方式。
* 向相应的服务生成请求。 如果机器人充当较大服务的前端，此选项可能很适合。

## <a name="test-your-dialog"></a>测试对话

在本地生成并运行机器人，然后使用仿真器来与机器人交互。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. 机器人发送初始问候消息，以响应将用户添加到聊天的聊天更新活动。
1. 输入 `hi` 或其他内容。 由于在此轮次尚未出现活动的对话，因此机器人会启动 `details` 对话。
   * 机器人发送第一条对话提示，并等待更多输入。
1. 用户回答机器人的提问；对话不断递进。
1. 对话的最后一个步骤根据输入发送 `Thanks` 消息。
   * 对话结束时，将从对话堆栈中删除该对话，机器人不再有活动的对话。
1. 输入 `hi` 或其他内容再次启动对话。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 机器人发送初始问候消息，以响应将用户添加到聊天的聊天更新活动。
1. 输入 `hi` 或其他内容。 由于在此轮次尚未出现活动的对话并且未提供用户配置文件，因此机器人会启动 `who_are_you` 对话。
   * 机器人发送第一条对话提示，并等待更多输入。
1. 用户回答机器人的提问；对话不断递进。
1. 对话的最后一个步骤发送简短的确认消息。
1. 输入 `hi` 或其他内容。
   * 机器人启动单步 `hello_user` 对话，这会显示收集的数据中的信息，并立即结束。

---

## <a name="additional-resources"></a>其他资源
可以依赖适用于每种提示的内置验证（如下所示），也可以将自己的自定义验证添加到提示。 有关详细信息，请参阅[使用对话提示收集用户输入](bot-builder-prompts.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用分支和循环创建高级聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)
