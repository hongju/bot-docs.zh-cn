---
title: 使用对话管理简单的聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用对话管理简单的聊天流。
keywords: 简单的聊天流, 对话, 提示, 瀑布, 对话集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c70711d747e9646acf63b6ee206d0b8db25ef202
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389678"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>使用对话管理简单的聊天流

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以使用对话库来管理简单的和复杂的聊天流。 在简单的聊天流中，用户从瀑布的第一个步骤开始一直执行到最后一个步骤，直到聊天最终完成。 [复杂聊天流](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)包括分支和循环。

<!-- TODO: This paragraph belongs in a conceptual topic. -->

对话是机器人中的结构，像函数一样在机器人的程序中运行。 对话生成你的机器人发送的消息，并执行所需的计算任务。 它们设计用来按特定的顺序执行特定的操作。 可以通过不同的方式调用它们 - 有时候是在对用户的响应中调用，有时候是在对某个外部刺激的响应中，或者通过其他对话进行调用。

机器人开发者可以使用对话来引导聊天流。 你可以创建多个对话并将其链接在一起来创建你希望机器人处理的聊天流。 Bot Builder SDK 中的“对话”库包含内置功能（例如提示、瀑布对话、组件对话），可以帮助你管理机器人的聊天流。 可以使用提示来请求用户提供不同类型的信息。 可以使用瀑布将多个步骤合并到一个序列中。 并且可以使用组件对话来创建包含多个子对话的模块化对话系统。

在本文中，我们使用对话集来创建一个包含提示和瀑布的聊天流。 我们将基于**多轮提示** [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)] 示例来编写代码。

有关对话的概述，请参阅[对话库](bot-builder-concept-dialog.md)和[对话状态](bot-builder-dialog-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

通常情况下，若要使用对话，需要为项目或解决方案安装 `Microsoft.Bot.Builder.Dialogs` NuGet 程序包。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

通常情况下，若要使用对话，你需要 `botbuilder-dialogs` 库，可以从 NPM 下载该库。

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>使用对话来指导用户完成各步骤

在此示例中，我们将创建一个多步骤对话，以使用对话集提示用户输入信息。

### <a name="create-a-dialog-with-waterfall-steps"></a>使用瀑布图步骤创建一个对话

**WaterfallDialog** 是对话的特定实现，通常用于从用户那里收集信息或指导用户完成一系列任务。 聊天的每个步骤都是作为函数实现的。 在每个步骤中，机器人会[提示用户输入](bot-builder-prompts.md)，等待响应，然后将结果传递到下一步。 第一个函数的结果作为参数传递给下一个函数，依此类推。

例如，以下代码示例定义了表示**瀑布**步骤的委托数组。 在每个提示后，机器人会确认用户的输入。 有许多方式可用来持久保存你在对话中收集的输入。 请参阅[持久保存用户数据](bot-builder-tutorial-persist-user-inputs.md)来了解其中的一些选项。

此示例在对话中收集信息的同时直接将其写入到用户的配置文件中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在此示例中，瀑布对话是在机器人文件中定义的。

请引用此文件中使用的命名空间。

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

为对话集定义一个实例属性。

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

在机器人的构造函数内创建对话集，向该对话集内添加提示和瀑布对话。

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
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

并且，将每个步骤定义为单独的方法。 还可以使用 lambda 表达式以内联方式定义步骤。

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
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

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
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

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
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

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
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

对话从机器人的每轮次处理程序运行，该处理程序首先创建一个对话上下文，并根据情况继续操作或启动对话，然后在该轮次结束时保存聊天和用户状态。

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在此示例中，瀑布对话是在 **bot.js** 文件中定义的。

请导入你需要用于代码的对象。

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

在机器人的构造函数内定义并创建对话集，向该对话集内添加提示和瀑布对话。

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
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

并且，将每个步骤定义为单独的方法。 还可以使用 lambda 表达式以内联方式定义步骤。

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

对话从机器人的每轮次处理程序运行，该处理程序首先创建一个对话上下文，并根据情况继续操作或启动对话，然后在该轮次结束时保存聊天和用户状态。

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>对话上下文和瀑布步骤上下文对象

使用对话上下文对象从机器人的轮次处理程序内通过对话集进行互动。
使用瀑布步骤上下文对象从瀑布步骤内通过对话集进行互动。

## <a name="to-start-a-dialog"></a>启动对话

若要启动某个对话，请将要启动的 *dialogId* 传入对话上下文的 _beginDialog_、_prompt_ 或 _replaceDialog_ 方法。 _beginDialog_ 方法会将对话推送到堆栈的顶层，而 _replaceDialog_ 方法会将当前对话从堆栈中弹出，并将用于替换的对话推送到堆栈上。

对话上下文的 _prompt_ 方法是一个帮助程序方法，它接受参数并为提示构造相应的选项；然后它启动提示对话。 有关提示的详细信息，请参阅[提示用户输入](bot-builder-prompts.md)。

## <a name="to-end-a-dialog"></a>结束对话：

_end dialog_ 方法通过从堆栈中弹出对话并将可选结果返回到父对话的方式来结束对话。

最佳做法是在对话结束时显式调用 _endDialog_ 方式。

## <a name="to-clear-the-dialog-stack"></a>清除对话堆栈

若要使所有对话从堆栈中弹出，可以调用对话上下文的 _cancel all dialogs_ 方法来清除对话堆栈。

## <a name="to-repeat-a-dialog"></a>重复对话

若要重复某个对话，请使用 _replace dialog_ 方法，该方法会将当前对话从堆栈中弹出，并将用于替换的对话推送到堆栈顶层，然后启动该对话。 这是处理[复杂聊天流](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)和管理菜单的极佳方法。

## <a name="next-steps"></a>后续步骤

了解如何管理简单的聊天流后，让我们学习如何利用 _replace dialog_ 方法来处理复杂的聊天流。

> [!div class="nextstepaction"]
> [管理复杂的聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)
