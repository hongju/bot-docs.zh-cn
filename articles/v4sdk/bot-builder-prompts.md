---
title: 使用对话框库提示用户输入 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用“对话”库提示用户输入。
keywords: 提示, 对话框, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 验证
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bd1fe8516cddaf2b75d3c11b469e372265b59be3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997624"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>使用“对话”库提示用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

通过发布问题来收集信息是机器人与用户交互的主要方式之一。 可以使用 [turn context](~/v4sdk/bot-builder-basics.md#defining-a-turn) 对象的 _send activity_ 方法直接这样操作，然后将下一个传入消息作为响应处理。 不过，Bot Builder SDK 提供一个**对话框**库，该库提供的方法旨在方便提问，并确保响应符合特定的数据类型或满足自定义验证规则。 本主题详细介绍如何使用**提示**来请求用户输入，以便达到此目的。

本文介绍如何使用对话框中的提示。 有关使用对话的一般信息，请参阅[使用对话管理聊天流](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="prompt-types"></a>提示类型

对话框库提供多种不同类型的提示，每种提示用于收集不同类型的响应。

| Prompt | Description |
|:----|:----|
| **AttachmentPrompt** | 提示用户提供附件，例如文档或图像。 |
| **ChoicePrompt** | 提示用户从一组选项中进行选择。 |
| **ConfirmPrompt** | 提示用户确认其操作。 |
| **DatetimePrompt** | 提示用户输入日期时间。 用户可以使用自然语言响应，如“明天晚上 8 点”或“星期五上午 10 点”。 Bot Framework SDK 使用 LUIS `builtin.datetimeV2` 预生成实体。 有关详细信息，请参阅 [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)。 |
| **NumberPrompt** | 提示用户输入数字。 用户可以使用“10”或“十”进行响应。 如果响应为“10”，例如，提示会将响应转换为一个数字并返回 `10` 作为结果。 |
| **TextPrompt** | 提示用户输入文本字符串。 |

## <a name="add-references-to-prompt-library"></a>添加对提示库的引用

可以通过向机器人添加 **botbuilder-dialogs** 包获取**对话框**库。 [使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)中介绍了对话，本文将在提示中使用对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

从 NuGet 安装 **Microsoft.Bot.Builder.Dialogs** 包。

然后，包括对机器人代码中库的引用。

```cs
using Microsoft.Bot.Builder.Dialogs;
```

需通过访问器设置聊天对话框状态。 我们不会在这里对此代码进行过多讨论，但你可以在[状态](bot-builder-howto-v4-state.md)一文中找到此方面的详细信息。

在 **Startup.cs** 的机器人选项中，首先定义状态对象，然后添加单一实例，为机器人构造函数提供访问器类。 `BotAccessor` 的这个类直接存储聊天和用户状态，以及适用于每个此类项的访问器。 本文末尾处在链接的示例中提供了完整的类定义。 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

接下来，请在机器人代码中定义对话框集的以下对象。

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

使用 Echo 模板创建 JavaScript 机器人。 有关详细信息，请参阅 [JavaScript 快速入门](../javascript/bot-builder-javascript-quickstart.md)。

通过 npm 安装对话框包：

```cmd
npm install --save botbuilder-dialogs
```

若要在机器人中使用对话框，请将其包含在机器人代码中。

1. 在 **bot.js** 文件中，添加以下内容。

    ```javascript
    // Import components from the dialogs library.
    const { DialogSet, TextPrompt, WaterfallDialog } = require("botbuilder-dialogs");

    // Name for the dialog state property accessor.
    const DIALOG_STATE_PROPERTY = 'dialogState';

    // Define the names for the prompts and dialogs for the dialog set.
    const TEXT_PROMPT = 'textPrompt';
    const MAIN_DIALOG = 'mainDialog';
    ```

    对话框集将包含此机器人的对话框，我们将使用文本提示来要求用户提供输入。 我们还需要一个对话框状态属性访问器，供对话框集用来跟踪其状态。

1. 更新机器人的构造函数代码。 我们很快会添加此方面的更多内容。

    ```javascript
      constructor(conversationState) {
        // Track the conversation state object.
        this.conversationState = conversationState;

        // Create a state property accessor for the dialog set.
        this.dialogState = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    }
    ```

---

## <a name="prompt-the-user"></a>提示用户

若要提示用户进行输入，请使用某个内置的类（例如 **TextPrompt**）定义一个提示，然后将其添加到对话框集并为其分配一个对话框 ID。

添加提示以后，请在双步瀑布对话框中使用它。瀑布对话框可以用来定义一系列步骤。 多个提示可以链接到一起，创建多步聊天。 有关详细信息，请参阅[使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)中的[使用对话](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps)部分。

例如，以下对话框提示用户输入其名称，然后使用响应来问候他们。 在第一轮，对话框会提示用户输入其名称。 用户的响应作为参数传递给第二个步骤函数，后者在处理输入后发送个性化问候。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此外，还为对话框中使用的每个提示指定了一个名称，由对话框或机器人用于访问提示。 在所有这些示例中，我们将提示 ID 公开为常量。

在机器人构造函数中，添加供双步瀑布框使用的定义以及供对话框使用的提示。 在这里，我们将它们作为独立函数添加，但也可以根据偏好将它们定义为内联 lambda。

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

然后，在机器人中定义两个瀑布步骤。 对于文本提示，请指定上面定义的 `TextPrompt` 的 *name* ID。 请注意，这些方法名称与上述 `WaterfallStep[]` 的方法名称匹配。 此处将来的示例不会包含该代码，但必须知道的是，对于后续步骤，需在该 `WaterfallStep[]` 中按正确顺序添加方法名称。

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 在机器人的构造函数中，创建对话框集并向其添加一个文本提示和一个瀑布对话框。

    ```javascript
    // Create the dialog set, and add the prompt and the waterfall dialog.
    this.dialogs = new DialogSet(this.dialogState)
        .add(new TextPrompt(TEXT_PROMPT))
        .add(new WaterfallDialog(MAIN_DIALOG, [
            async (step) => {
                // The results of this prompt will be passed to the next step.
                return await step.prompt(TEXT_PROMPT, 'What is your name?');
            },
            async (step) => {
                // The result property contains the result from the previous step.
                const userName = step.result;
                await step.context.sendActivity(`Hi ${userName}!`);
                return await step.endDialog();
            }
        ]));
    ```

1. 更新机器人的轮次处理程序以运行此对话框。

    ```javascript
    async onTurn(turnContext) {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Create a dialog context for the dialog set.
            const dc = await this.dialogs.createContext(turnContext);
            // Continue the dialog if it's active.
            await dc.continueDialog();
            if (!turnContext.responded) {
                // Otherwise, start the dialog.
                await dc.beginDialog(MAIN_DIALOG);
            }
        } else {
            // Send a default message for activity types that we don't handle.
            await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        }
    }
    ```

---

> [!NOTE]
> 若要启动对话框，请获取对话框上下文，并使用其 _begin dialog_ 方法。 有关详细信息，请参阅[使用对话管理简单的聊天流](./bot-builder-dialog-manage-conversation-flow.md)。

## <a name="reusable-prompts"></a>可重用的提示

可以重复使用提示来提问不同的问题，只要回答是同一类型的即可。 例如，上面的示例代码定义文本提示，并使用它向用户询问其名称。 还可以使用相同的提示来请求用户提供另一个文本字符串，例如，“您在哪里工作？”。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在示例中，文本提示的 ID *name* 并没有提高代码的清晰度。 不过，这也说明了提示 ID 可以随意选择。

现在，我们的方法包含第三个步骤：询问用户的工作地点。

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的构造函数中，修改用于提问第二个问题的瀑布对话框。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new TextPrompt(TEXT_PROMPT))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their name.
        return await step.prompt(TEXT_PROMPT, 'What is your name?');
    },
    async (step) => {
        // Acknowledge their response and ask for their place of work.
        const userName = step.result;
        return await step.prompt(TEXT_PROMPT, `Hi ${userName}; where do you work?`);
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const workPlace = step.result;
        await step.context.sendActivity(`${workPlace} is a cool place!`);
        return await step.endDialog();
    }
    ]));
```

---

如需使用多个不同的提示，请为每个提示提供唯一的 *dialogId*。 添加到对话框集的每个对话框或提示都需要唯一 ID。 还可以创建多个相同类型的**提示**对话框。 例如，可以为上面的示例创建两个 TextPrompt 对话框：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

例如，可以将此项替换为

```javascript
.add(new TextPrompt(TEXT_PROMPT))
```

以下内容，

```javascript
.add(new TextPrompt('namePrompt'))
.add(new TextPrompt('workPlacePrompt'))
```

然后更新相应的瀑布步骤，以便按照相应的名称使用这些提示。

---

为了提高代码的可重用性，所有这些提示只需定义一个 `TextPrompt`，因为它们都预期用户会使用文本作为响应。 在需要对提示的输入应用不同的验证规则时，为对话框命名这一功能就会给你带来方便。 让我们看看如何使用 `NumberPrompt` 来验证提示响应。

## <a name="specify-prompt-options"></a>指定提示选项

在对话框步骤内使用提示时，还可以提供提示选项，例如重新提示字符串。

当用户输入无法满足提示时，指定重新提示字符串很有用，因为它采用的是提示无法解析的格式（例如，用于数字提示的“明天”），或者是因为输入使验证条件失效。 数字提示可以解释各种输入，例如“十二”或“一个季度”以及“12”和“0.25”。

在某些提示（例如 **NumberPrompt**）中，locale 是可选参数。 它有助于提示更准确地分析输入，但不是必需参数。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

下面的代码将向现有对话框集 **_dialogs** 添加数字提示。

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

在对话框步骤中，下面的代码会提示用户输入，并在他们的输入不能解释为数字的情况下提供要使用的重新提示字符串。

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

从对话框库中导入 `NumberPrompt` 类。

```javascript
const { NumberPrompt } = require("botbuilder-dialogs");
```

在瀑布对话框中使用数字提示，指定初始的和重试的提示字符串。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySize'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('partySize', {
            prompt: 'How many people in your party?',
            retryPrompt: 'Sorry, please specify the number of people in your party.'
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const partySize = step.result;
        await step.context.sendActivity(`That's a party of ${partySize}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

选择提示有另一必需参数：向用户提供的选择列表。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

通过 **ChoicePrompt** 要求用户在一组选项之间进行选择时，必须提供包含该组选项的提示（在 **PromptOptions** 对象中提供）。 在这里，我们使用 ChoiceFactory 将选项列表转换为相应的格式。

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

从对话框库中导入 `NumberPrompt` 类。

```javascript
const { ChoicePrompt } = require("botbuilder-dialogs");
```

在瀑布对话框中使用选项提示，指定可用的选项。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
const list = ['green', 'blue', 'red', 'yellow'];
this.dialogs = new DialogSet(this.dialogState)
    .add(new ChoicePrompt('choicePrompt'))
    .add(new WaterfallDialog(MAIN_DIALOG, [
    async (step) => {
        // Ask the user for their party size.
        return await step.prompt('choicePrompt', {
            prompt: 'Please choose a color:',
            retryPrompt: 'Sorry, please choose a color from the list.',
            choices: list
        });
    },
    async (step) => {
        // Acknowledge their response and exit the dialog.
        const choice = step.result;
        await step.context.sendActivity(`That's ${choice.value}, thanks.`);
        return await step.endDialog();
    }
]));
```

---

## <a name="validate-a-prompt-response"></a>验证提示响应

可以在将值返回到**瀑布**对话框的下一个步骤之前验证提示响应。 例如，若要验证 **NumberPrompt** 是否在 6 到 20 这个数字范围内，可以包括一个如下所示的验证函数：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在将提示添加到对话框集以包括验证程序函数时更改

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

然后将验证定义为它自己的方法，根据是否通过验证来指示 true 或 false。 如果返回 false，则会再次提示用户。

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在创建提示时添加验证方法。

```javascript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new NumberPrompt('partySizePrompt', async (promptContext) =>                                                 {
        // Check to make sure a value was recognized.
        if (promptContext.recognized.succeeded) {
            const value = promptContext.recognized.value;
            try {
                if (value < 6) {
                    throw new Error('Party size too small.');
                } else if (value > 20) {
                    throw new Error('Party size too big.')
                } else {
                    return true; // Indicate that this is a valid value.
                }
            } catch (err) {
                await promptContext.context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
                return false; // Indicate that this is invalid.
            }
        } else {
            return false;
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('partySizePrompt', {
                prompt: 'How large is your party?',
                retryPrompt: 'Sorry, please specify a size between 6 and 20.'
            });
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const size = step.result;
            await step.context.sendActivity(`That's a party of ${size}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

---

同样，如果想要验证未来某个日期和时间的 DatetimePrompt 响应，可以使用与以下类似的验证逻辑：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

有关更多示例，请参阅[示例存储库](https://aka.ms/bot-samples-readme)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { DateTimePrompt } = require("botbuilder-dialogs");
```

```JavaScript
// Create the dialog set, and add the prompt and the waterfall dialog.
this.dialogs = new DialogSet(this.dialogState)
    .add(new DateTimePrompt('dateTimePrompt', async (promptContext) => {
        try {
            if (!promptContext.recognized.succeeded) { throw new Error('Value not recognized.') }
            const values = promptContext.recognized.value;
            if (!Array.isArray(values) || values.length < 0) { throw new Error('Value missing.'); }
            if ((values[0].type !== 'datetime') && (values[0].type !== 'date')) { throw new Error('Unsupported type.'); }
            const now = new Date();
            const value = new Date(values[0].value);
            if (value.getTime() < now.getTime()) { throw new Error('Value in the past.') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = [value];
            return true; // indicate valid
        } catch (err) {
            await promptContext.context.sendActivity(`${err} Please specify a date or a date and time in the future, like tomorrow at 9am.`);
            return false; // indicate invalid
        }
    }))
    .add(new WaterfallDialog(MAIN_DIALOG, [
        async (step) => {
            // Ask the user for their party size.
            return await step.prompt('dateTimePrompt', 'When would you like to schedule that for?');
        },
        async (step) => {
            // Acknowledge their response and exit the dialog.
            const time = step.result;
            await step.context.sendActivity(`That's ${time}, thanks.`);
            return await step.endDialog();
        }
    ]));
```

有关更多示例，请参阅[示例存储库](https://aka.ms/bot-samples-readme)。

---

> [!TIP]
> 如果用户提供的答案不明确，日期时间提示可以解析为几个不同的日期。 根据使用意图，你可能想要检查提示结果提供的所有解析，而不仅仅是第一个。

可以使用类似技术来验证任何提示类型的提示响应。

## <a name="save-user-data"></a>保存用户数据

提示用户输入时，可以通过几个选项来处理此输入。 例如，可以使用和放弃输入、可以将其保存到全局变量、可以将其保存到一个临时的或内存中存储容器、可以将其保存到文件中，或者可以将其保存到外部数据库。 有关如何保存用户数据的详细信息，请参阅[管理用户数据](bot-builder-howto-v4-state.md)。

## <a name="additional-resources"></a>其他资源

如需通过完整的示例来了解如何使用这其中的一些提示，请参阅“适用于 [C#](https://aka.ms/cs-multi-prompts-sample) 或 [JavaScript](https://aka.ms/js-multi-prompts-sample) 的多轮提示机器人”。

## <a name="next-steps"></a>后续步骤

现已了解如何提示用户进行输入，我们可以通过对话框管理各种会话流来增强机器人代码和用户体验。

> [!div class="nextstepaction"]
> [使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)
