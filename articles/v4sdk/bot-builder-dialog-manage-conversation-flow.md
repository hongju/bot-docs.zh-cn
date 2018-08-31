---
title: 使用对话管理简单的聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用对话管理简单的聊天流。
keywords: 简单的聊天流, 对话, 提示, 瀑布, 对话集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 8/2/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77162601f542e6faa8908bc71abc971eb99fcc93
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756400"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>使用对话管理简单的聊天流

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以使用对话库管理来简单和复杂的聊天流。 在简单的聊天流中，用户从瀑布的第一个步骤开始一直执行到最后一个步骤，直到聊天交换最终完成。 对话也可以处理[复杂的聊天流](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)，其中的对话部分可以分支和循环。

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

<!-- TODO: This paragraph belongs in a conceptual topic. -->对话类似于程序中的函数。 对话通常用于按特定的顺序执行特定的操作，并可根据需要随时调用。 机器人开发员可以使用对话来引导聊天流。 就像希望机器人处理的任何会话流一样，可以将多个对话框串联在一起处理。 Bot Builder SDK 中的“对话”库包含一些内置功能（例如提示和瀑布对话）用于帮助管理聊天流。 可以使用提示来请求用户提供不同类型的信息。 可以使用瀑布将多个步骤合并到一个序列中。

本文使用对话集来创建一个包含提示和瀑布步骤的聊天流。 我们有两个示例对话。 第一个对话是单步对话，执行无需用户输入的操作。 第二个对话是多步对话，提示用户提供一些信息。

## <a name="install-the-dialogs-library"></a>安装对话框库

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们从一个基本的 EchoBot 模板着手。 有关说明，请参阅[适用于 .NET 的快速入门](~/dotnet/bot-builder-dotnet-quickstart.md)。

要使用对话框，请为项目或解决方案安装 `Microsoft.Bot.Builder.Dialogs` NuGet 包。
然后，根据需要在代码文件的 using 语句中引用对话库。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

可以从 NPM 下载 `botbuilder-dialogs` 库。 要安装 `botbuilder-dialogs` 库，请运行以下 NPM 命令：

```cmd
npm install --save botbuilder-dialogs@preview
```

若要在机器人中使用**对话**，请将对话包含在 **app.js** 文件中。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

## <a name="create-a-dialog-stack"></a>创建对话框堆栈

第一个示例创建一个单步对话，该对话可将两个数字相加并显示结果。

要使用对话框，首先必须创建对话框集。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` 库提供 `DialogSet` 类。
创建 **AdditionDialog** 类，并添加稍后需要用到的 using 语句。
可将命名的对话和对话集添加到一个对话集，然后可按名称访问它们。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

从 **DialogSet** 派生类，并定义用于标识此对话集的对话和输入信息的 ID 与键。

```csharp
/// <summary>Defines a simple dialog for adding two numbers together.</summary>
public class AdditionDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Main = "additionDialog";

    /// <summary>Defines the IDs of the input arguments.</summary>
    public struct Inputs
    {
        public const string First = "first";
        public const string Second = "second";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`botbuilder-dialogs` 库提供 `DialogSet` 类。
DialogSet 类定义对话框堆栈并提供简单的接口来管理堆栈。
Bot Builder SDK 将堆栈实现为数组。

要创建 DialogSet，请执行以下操作：

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

上述调用将使用名为 `dialogStack` 的默认对话框堆栈来创建 DialogSet。
如果要为堆栈命名，可以将其作为参数传递到 DialogSet()。 例如：

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```

---

创建对话框只是将对话框定义添加到集合中。 在通过调用 begin 或 replace 方法将对话框推送到对话框堆栈之前，该对话框不会运行。

对话框名称（例如，`addTwoNumbers`）在每个对话框集内必须是唯一的。 可以根据需要在每个集内定义任意数量的对话框。 若要创建多个对话集并使它们无缝协同工作，请参阅[创建模块化机器人逻辑](bot-builder-compositcontrol.md)。

对话框库定义以下对话框：

* 一个提示对话框，其中对话框至少使用两个函数，一个用于提示用户输入，另一个用于处理输入。 可以使用瀑布图模型将这些函数串联起来。
* 一个瀑布图对话框，用于定义按顺序运行的一系列瀑布图步骤。 瀑布图对话框可以只有一个步骤，在这种情况下，它可以被视为一个简单的一步对话框。

## <a name="create-a-single-step-dialog"></a>创建单步对话框

单步对话框也可用于捕获单轮次会话流。 本示例创建一个机器人，可用于检测用户是否说出类似“1 + 2”的内容，并启动一个 `addTwoNumbers` 对话框回复“1 + 2 = 3”。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

值作为 `IDictionary<string,object>` 属性包传递到对话框，并从对话框返回。

要在对话框集内创建一个简单对话框，请使用 `Add` 方法。 以下操作添加一个名为 `addTwoNumbers` 的一步瀑布图。

此步骤假设传入的对话框参数包含表示要添加的数字的 `first` 和 `second` 属性。

将以下构造函数添加到 **AdditionDialog** 类。

```csharp
/// <summary>Defines the steps of the dialog.</summary>
public AdditionDialog()
{
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Get the input from the arguments to the dialog and add them.
            var x =(double)args[Inputs.First];
            var y =(double)args[Inputs.Second];
            var sum = x + y;

            // Display the result to the user.
            await dc.Context.SendActivity($"{x} + {y} = {sum}");

            // End the dialog.
            await dc.End();
        }
    });
}
```

### <a name="pass-arguments-to-the-dialog"></a>将参数传递到对话框

在机器人代码中更新 using 语句。

```cs
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
```

将静态属性添加到加法对话的类。

```cs
private static AdditionDialog AddTwoNumbers { get; } = new AdditionDialog();
```

要从机器人的 `OnTurn` 方法内调用对话框，请修改 `OnTurn` 以包含以下内容：

```cs
public async Task OnTurn(ITurnContext context)
{
    // Handle any message activity from the user.
    if (context.Activity.Type is ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var conversationState = context.GetConversationState<ConversationData>();

        // Generate a dialog context for the addition dialog.
        var dc = AddTwoNumbers.CreateContext(context, conversationState.DialogState);

        // Call a helper function that identifies if the user says something
        // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add.
        if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
        {
            // Start the dialog, passing in the numbers to add.
            var args = new Dictionary<string, object>
            {
                [AdditionDialog.Inputs.First] = first,
                [AdditionDialog.Inputs.Second] = second
            };
            await dc.Begin(AdditionDialog.Main, args);
        }
        else
        {
            // Echo back to the user whatever they typed.
            await context.SendActivity($"You said '{context.Activity.Text}'");
        }
    }
}
```

将 **TryParseAddingTwoNumbers** 帮助器函数添加到机器人类。 帮助程序函数只使用一个简单的正则表达式来检测用户的消息是否请求添加 2 个数字。

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number,
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7.
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public static bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";

    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";

    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;

    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);

    first = 0;
    second = 0;
    if (matches.Count > 0)
    {
        var matched = matches[0];
        if (double.TryParse(matched.Groups[1].Value, out first)
            && double.TryParse(matched.Groups[2].Value, out second))
        {
            return true;
        }
    }
    return false;
}
```

如果使用 EchoBot 模板，请将 **EchoState** 类的名称更改为 **ConversationData**，并对其进行修改以包含以下内容。

```cs
using System.Collections.Generic;

/// <summary>
/// Class for storing conversation state.
/// </summary>
public class ConversationData
{
    /// <summary>Property for storing dialog state.</summary>
    public Dictionary<string, object> DialogState { get; set; } = new Dictionary<string, object>();
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

开始使用[使用 Bot Builder SDK v4 创建机器人](../javascript/bot-builder-javascript-quickstart.md)中所述的 JS 模板。 在 app.js 中，添加一条要求使用 `botbuilder-dialogs` 的语句。

```js
const {DialogSet} = require('botbuilder-dialogs');
```

在 app.js 中添加以下代码，用于定义作为 `dialogs` 集一部分的名为 `addTwoNumbers` 的简单对话框：

```javascript
const dialogs = new DialogSet("myDialogStack");

// Show the sum of two numbers.
dialogs.add('addTwoNumbers', [async function (dc, numbers){
        var sum = Number.parseFloat(numbers[0]) + Number.parseFloat(numbers[1]);
        await dc.context.sendActivity(`${numbers[0]} + ${numbers[1]} = ${sum}`);
        await dc.end();
    }]
);
```

替换 app.js 中的代码，使用以下内容处理传入活动。 机器人调用帮助程序函数来检查传入消息是否类似于添加两个数字的请求。 如果是，则将参数中的数字传递到 `DialogContext.begin`。

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        // State will store all of your information
        const convoState = conversationState.get(context);
        const dc = dialogs.createContext(context, convoState);

        if (isMessage) {
            // TryParseAddingTwoNumbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await TryParseAddingTwoNumbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                const count = (convoState.count === undefined ? convoState.count = 0 : ++convoState.count);
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`);
            }
        }
        else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }

        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "What's 2+3?"`);
            }
        }
    });
});

```

向 app.js 添加帮助程序函数。 帮助程序函数只使用一个简单的正则表达式来检测用户的消息是否请求添加 2 个数字。 如果正则表达式匹配，则返回一个数组，其中包含要添加的数字。

```javascript
async function TryParseAddingTwoNumbers(message) {
    const ADD_NUMBERS_REGEXP = /([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))(?:\s*)\+(?:\s*)([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))/i;
    let matched = ADD_NUMBERS_REGEXP.exec(message);
    if (!matched) {
        // message wasn't a request to add 2 numbers
        return null;
    }
    else {
        var numbers = [matched[1], matched[2]];
        return numbers;
    }
}
```

---

### <a name="run-the-bot"></a>运行机器人

尝试在 Bot Framework Emulator 中运行机器人，并对它说出类似于“1+1 的结果是什么？” 的内容。

![运行机器人](./media/how-to-dialogs/bot-output-add-numbers.png)

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>使用对话框来指导用户完成各步骤

下一个示例创建一个多步对话，以提示用户输入信息。

### <a name="create-a-dialog-with-waterfall-steps"></a>使用瀑布图步骤创建一个对话框

瀑布图是特定的对话框实现，最常用于从用户那里收集信息或指导用户完成一系列任务。 任务作为一组函数实现，其中第一个函数的结果作为参数传递到下一个函数，依次类推。 每个函数通常表示整个过程中的一步。 在每个步骤中，机器人会[提示用户输入](bot-builder-prompts.md)，等待响应，然后将结果传递到下一步。

例如，以下代码示例在表示三个**瀑布**步骤的数组中定义三个函数。 在每个提示的后面，机器人确认了用户输入，但未保存输入。 若要保存用户输入，请参阅[保存用户数据](bot-builder-tutorial-persist-user-inputs.md)了解更多详细信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此示例显示了某个问候对话的构造函数，其中，**GreetingDialog** 派生自 **DialogSet**，**Inputs.Text** 包含用于 **TextPrompt**对象的 ID，**Main** 包含问候对话自身的 ID。

```csharp
public GreetingDialog()
{
    // Include a text prompt.
    Add(Inputs.Text, new TextPrompt());

    // Define the dialog logic for greeting the user.
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Ask for their name.
            await dc.Prompt(Inputs.Text, "What is your name?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var name = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"Hi, {name}!");

            // Ask where they work.
            await dc.Prompt(Inputs.Text, "Where do you work?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var work = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"{work} is a fun place.");

            // End the dialog.
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, results){
        var workPlace = results;
        await dc.context.sendActivity(`${workPlace} is a fun place.`);
        await dc.end(); // Ends the dialog
    }
]);

dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```

---

瀑布图步骤的签名如下：

| 参数 | Description |
| :---- | :----- |
| `dc` | 对话框上下文。 |
| `args` | （可选）包含传入该步骤的参数。 |
| `next` | （可选）用于在不发出提示的情况下转到下一瀑布步骤的方法。 调用此方法时，可以提供 *args* 参数，以便将参数传递给瀑布中的下一个步骤。 |

在返回之前，每个步骤必须调用以下方法之一：*next()* 委托，或对话上下文方法 *begin*、*end*、*prompt* 和 *replace* 之一；否则，机器人将陷在该步骤中。 也就是说，如果函数未使用其中一个方法完成，则每次用户向机器人发送消息时，所有用户输入都会导致此步骤重新执行。

到达瀑布的末尾时，最好是使用 _end_ 方法返回，使对话可从堆栈中弹出。 有关详细信息，请参阅下面的[结束对话](#end-a-dialog)部分。 同理，若要从一个步骤转到下一个步骤，瀑布步骤必须以提示结束，或显式调用 _next_ 委托来推进瀑布。

## <a name="start-a-dialog"></a>启动对话框

若要启动某个对话，请将要启动的 *dialogId* 传入对话上下文的_begin_、_prompt_ 或 _replace_ 方法。 _begin_ 方法会将对话推送到堆栈的顶层，而 _replace_ 方法会将当前对话从堆栈中弹出，并将用于替换的对话推送到堆栈上。

在不使用参数的情况下启动对话框：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog.
await dc.Begin("greetings");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

---

使用参数启动对话框：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog, passing in a property bag.
await dc.Begin("greetings", args);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog with the 'userName' passed in.
await dc.begin('greetings', userName);
```

---

启动提示对话框：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处，**Inputs.Text** 包含同一对话集中 **TextPrompt** 的 ID。

```csharp
// Ask a user for their name.
await dc.Prompt(Inputs.Text, "What is your name?");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Ask a user for their name.
await dc.prompt('textPrompt', "What is your name?");
```

---

根据启动的不同提示类型，提示的参数签名可能不同。 DialogSet.prompt  方法是一个帮助程序方法。 此方法接受参数并为提示构造相应的选项；然后，它调用 begin 方法来启动提示对话框。 有关提示的详细信息，请参阅[提示用户输入](bot-builder-prompts.md)。

## <a name="end-a-dialog"></a>结束对话框

_end_ 方法通过从堆栈中弹出对话并将可选结果返回到父对话的方式来结束对话。

最好是在对话结束时显式调用 _end_ 方法；但这一定要这样做，因为在达到瀑布的末尾时，对话将自动从堆栈中弹出。

结束对话框：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog by popping it off the stack.
await dc.End();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

---

若要结束对话并向父对话返回信息，请包含一个属性包参数。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and return information to the parent dialog.
await dc.end(new Dictionary<string, object>
    {
        ["property1"] = value1,
        ["property2"] = value2
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end({
    "property1": value1,
    "property2": value2
});
```

---

## <a name="clear-the-dialog-stack"></a>清除对话框堆栈

若要使所有对话从堆栈中弹出，可以调用 _end all_ 方法来清除对话堆栈。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Pop all dialogs from the current stack.
await dc.EndAll();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

---

## <a name="repeat-a-dialog"></a>重复对话框

若要重复对话，请使用 _replace_ 方法。 对话上下文的 *replace* 方法会将当前对话从堆栈中弹出，并将用于替换的对话推送到堆栈顶层，然后开始该对话。 这是处理[复杂聊天流](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md)和管理菜单的极佳方法。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and start the main menu dialog.
await dc.Replace("mainMenu");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu');
```

---

## <a name="next-steps"></a>后续步骤

了解如何管理简单的聊天流后，让我们学习如何利用 _replace_ 方法来处理复杂的聊天流。

> [!div class="nextstepaction"]
> [管理复杂的聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)
