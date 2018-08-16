---
title: 使用对话框管理会话流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用对话框管理会话流。
keywords: 会话流, 对话框, 提示, 瀑布图, 对话框集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99184ba71072c159c598c7f68289c42a51926795
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297949"
---
# <a name="manage-conversation-flow-with-dialogs"></a>使用对话框管理会话流
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


管理会话流是构建机器人过程中不可或缺的任务。 借助 Bot Builder SDK，可以使用对话框来管理会话流。

对话框就像程序中的函数。 这通常是为了执行特定操作，并可根据需要随时调用。 就像希望机器人处理的任何会话流一样，可以将多个对话框串联在一起处理。 Bot Builder SDK 中的对话框库包含一些内置功能，如提示和瀑布图，有助于使用对话框管理会话流。 提示库提供了各种提示，可以使用这些提示向用户询问不同类型的信息。 瀑布图提供了一种将序列中的多个步骤组合在一起的方法。

本文介绍如何创建对话框对象，并将提示和瀑布图步骤添加到对话框集中，用于管理简单会话流和复杂会话流。 

## <a name="install-the-dialogs-library"></a>安装对话框库

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
要使用对话框，请为项目或解决方案安装 `Microsoft.Bot.Builder.Dialogs` NuGet 包。
然后，使用代码文件中的语句引用对话框库。 例如：

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
可以从 NPM 下载 `botbuilder-dialogs` 库。 要安装 `botbuilder-dialogs` 库，请运行以下 NPM 命令：

```cmd
npm install --save botbuilder-dialogs
```

若要在机器人中使用对话框，请将其包含在机器人代码中。 例如：

**app.js**

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```
---

## <a name="create-a-dialog-stack"></a>创建对话框堆栈

要使用对话框，首先必须创建对话框集。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`Microsoft.Bot.Builder.Dialogs` 库提供 `DialogSet` 类。
可以向对话框集添加命名的对话框和对话框集，稍后可以通过名称进行访问。

```csharp
IDialog dialog = null;
// Initialize dialog.

DialogSet dialogs = new DialogSet();
dialogs.Add("dialog name", dialog);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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

对话框名称（例如，`addTwoNumbers`）在每个对话框集内必须是唯一的。 可以根据需要在每个集内定义任意数量的对话框。

对话框库定义以下对话框：
-   一个提示对话框，其中对话框至少使用两个函数，一个用于提示用户输入，另一个用于处理输入。
    可以使用瀑布图模型将这些函数串联起来。
-   一个瀑布图对话框，用于定义按顺序运行的一系列瀑布图步骤。
    瀑布图对话框可以只有一个步骤，在这种情况下，它可以被视为一个简单的一步对话框。

## <a name="create-a-single-step-dialog"></a>创建单步对话框

单步对话框也可用于捕获单轮次会话流。 本示例创建一个机器人，可用于检测用户是否说出类似“1 + 2”的内容，并启动一个 `addTwoNumbers` 对话框回复“1 + 2 = 3”。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

值作为 `IDictionary<string,object>` 属性包传递到对话框，并从对话框返回。

要在对话框集内创建一个简单对话框，请使用 `Add` 方法。 以下操作添加一个名为 `addTwoNumbers` 的一步瀑布图。

此步骤假设传入的对话框参数包含表示要添加的数字的 `first` 和 `second` 属性。

开始使用 EchoBot 模板。 然后在机器人类中添加代码，用于在构造函数中添加对话框。
```csharp
public class EchoBot : IBot
{
    private DialogSet _dialogs;

    public EchoBot()
    {
        _dialogs = new DialogSet();
        _dialogs.Add("addTwoNumbers", new WaterfallStep[]
        {              
            async (dc, args, next) =>
            {
                double sum = (double)args["first"] + (double)args["second"];
                await dc.Context.SendActivity($"{args["first"]} + {args["second"]} = {sum}");
                await dc.End();
            }
        });
    }

    // The rest of the class definition is omitted here but would include OnTurn()
}

```

### <a name="pass-arguments-to-the-dialog"></a>将参数传递到对话框

要从机器人的 `OnTurn` 方法内调用对话框，请修改 `OnTurn` 以包含以下内容：
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // create a dialog context
        var dialogCtx = _dialogs.CreateContext(context, state);

        // Bump the turn count. 
        state.TurnCount++;

        await dialogCtx.Continue();
        if (!context.Responded)
        {
            // Call a helper function that identifies if the user says something 
            // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add            
            if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
            { 
                var dialogArgs = new Dictionary<string, object>
                {
                    ["first"] = first,
                    ["second"] = second
                };                        
                await dialogCtx.Begin("addTwoNumbers", dialogArgs);
            }
            else
            {
                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn: {state.TurnCount}. You said '{context.Activity.Text}'");
            }
        }
    }
}
```

将帮助程序函数添加到机器人类。 帮助程序函数只使用一个简单的正则表达式来检测用户的消息是否请求添加 2 个数字。

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number, 
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7. 
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";
    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";
    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;
    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);
    var succeeded = false;
    first = 0;
    second = 0;
    if (matches.Count == 0)
    {
        succeeded = false;
    }
    else
    {
        var matched = matches[0];
        if ( System.Double.TryParse(matched.Groups[1].Value, out first) 
            && System.Double.TryParse(matched.Groups[2].Value, out second))
        {
            succeeded = true;
        } 
    }
    return succeeded;
}
```

如果你使用的是 EchoBot 模板，请在 EchoState.cs 中修改 `EchoState` 类，如下所示：

```cs
/// <summary>
/// Class for storing conversation state.
/// This bot only stores the turn count in order to echo it to the user
/// </summary>
public class EchoState: Dictionary<string, object>
{
    private const string TurnCountKey = "TurnCount";
    public EchoState()
    {
        this[TurnCountKey] = 0;            
    }

    public int TurnCount
    {
        get { return (int)this[TurnCountKey]; }
        set { this[TurnCountKey] = value; }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

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
        if (isMessage) {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // create a dialog context
            const dc = dialogs.createContext(context, state);

            // MatchesAdd2Numbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await MatchesAdd2Numbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {    
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`); 
            }           
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "what's 1+2?"`);
            }
        }
    });
});
```

向 app.js 添加帮助程序函数。 帮助程序函数只使用一个简单的正则表达式来检测用户的消息是否请求添加 2 个数字。 如果正则表达式匹配，则返回一个数组，其中包含要添加的数字。

```javascript
async function MatchesAdd2Numbers(message) {
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="create-a-composite-dialog"></a>创建一个复合对话框

以下代码片段摘自 botbuilder-dotnet 存储库中的 [Microsoft.Bot.Samples.Dialog.Prompts](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/MIcrosoft.Bot.Samples.Dialog.Prompts) 示例代码。

在 Startup.cs 中：
1.  将机器人重命名为 `DialogContainerBot`。
1.  使用简单字典作为机器人会话状态的属性包。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<DialogContainerBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(new MemoryStorage()));
    });
}
```

将 `EchoBot` 重命名为 `DialogContainerBot`。

在 `DialogContainerBot.cs` 中，为配置文件对话框定义一个类。

```csharp
public class ProfileControl : DialogContainer
{
    public ProfileControl()
        : base("fillProfile")
    {
        Dialogs.Add("fillProfile", 
            new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State = new Dictionary<string, object>();
                    await dc.Prompt("textPrompt", "What's your name?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["name"] = args["Value"];
                    await dc.Prompt("textPrompt", "What's your phone number?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["phone"] = args["Value"];
                    await dc.End(dc.ActiveDialog.State);
                }
            }
        );
        Dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
    }
}
```

然后，在机器人定义中，为机器人的主对话框声明一个字段，并在机器人的构造函数中对其初始化。
机器人的主对话框包括配置文件对话框。

```csharp
private DialogSet _dialogs;

public DialogContainerBot()
{
    _dialogs = new DialogSet();

    _dialogs.Add("getProfile", new ProfileControl());
    _dialogs.Add("firstRun",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                    await dc.Context.SendActivity("Welcome! We need to ask a few questions to get started.");
                    await dc.Begin("getProfile");
            },
            async (dc, args, next) =>
            {
                await dc.Context.SendActivity($"Thanks {args["name"]} I have your phone number as {args["phone"]}!");
                await dc.End();
            }
        }
    );
}
```

在机器人的 `OnTurn` 方法中：
-   在会话启动时问候用户。
-   每次收到用户的消息时，都初始化并继续主对话框。

    如果对话框尚未生成响应，则假定它已提前完成或尚未启动，指定要开始使用的集合中对话框的名称。

```csharp
public async Task OnTurn(ITurnContext turnContext)
{
    try
    {
        switch (turnContext.Activity.Type)
        {
            case ActivityTypes.ConversationUpdate:
                foreach (var newMember in turnContext.Activity.MembersAdded)
                {
                    if (newMember.Id != turnContext.Activity.Recipient.Id)
                    {
                        await turnContext.SendActivity("Hello and welcome to the Composite Control bot.");
                    }
                }
                break;

            case ActivityTypes.Message:
                var state = ConversationState<Dictionary<string, object>>.Get(turnContext);
                var dc = _dialogs.CreateContext(turnContext, state);

                await dc.Continue();

                if (!turnContext.Responded)
                {
                    await dc.Begin("firstRun");
                }

                break;
        }
    }
    catch (Exception e)
    {
        await turnContext.SendActivity($"Exception: {e.Message}");
    }
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="create-a-dialog-with-waterfall-steps"></a>使用瀑布图步骤创建一个对话框

会话由用户和机器人之间交换的一系列消息组成。 如果机器人的目的是引导用户完成一系列步骤，可以使用瀑布图模型来定义会话步骤。

瀑布图是特定的对话框实现，最常用于从用户那里收集信息或指导用户完成一系列任务。 任务作为一组函数实现，其中第一个函数的结果作为参数传递到下一个函数，依次类推。 每个函数通常表示整个过程中的一步。 在每个步骤中，机器人会[提示用户输入](bot-builder-prompts.md)，等待响应，然后将结果传递到下一步。

例如，下面的代码示例在表示瀑布图三个步骤的一个数组中定义三个函数:

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

瀑布图步骤的签名如下：

| 参数 | Description |
| ---- | ----- |
| `context` | 对话框上下文。 |
| `args` | （可选）包含传递到该步骤的参数。 |
| `next` | （可选）这是一种可允许继续执行瀑布图下一步的方法。 在调用此方法时，可以提供一个 args 参数，用于将（一个或多个）参数传递到瀑布图中的下一步。 |

在返回之前，每一步必须调用以下任一方法：next()、dialogs.prompt()、dialogs.end()、dialogs.begin() 或 Promise.resolve()；否则机器人在该步骤中将停止运行。 也就是说，如果函数没有返回这些方法之一，那么每次用户向机器人发送消息时，所有用户输入都将导致重新执行此步骤。

到达瀑布图的末尾时，最好使用 `end()` 方法返回，以便可以从堆栈弹出对话框。 有关详细信息，请参阅[结束对话框](#end-a-dialog)部分。 同样，要从一个步骤继续下一个步骤，瀑布图步骤必须通过提示结束或显式调用 `next()` 函数以推进瀑布图。 

### <a name="start-a-dialog"></a>启动对话框

要启动一个对话框，请将要启动的 dialogId 传递到 `begin()`、`prompt()` 或 `replace()` 方法。 begin  方法会将对话框推送到堆栈顶部，而 replace 方法会将当前对话框从堆栈中弹出，并将替换后的对话框推送到堆栈上。

在不使用参数的情况下启动对话框：

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

使用参数启动对话框：

```javascript
// Start the 'greetings' dialog with the 'userName' passed in. 
await dc.begin('greetings', userName);
```

启动提示对话框：

```javascript
// Start a 'choicePrompt' dialog with choices passed in as an array of colors to choose from.
await dc.prompt('choicePrompt', `choice: select a color`, ['red', 'green', 'blue']);
```

根据启动的不同提示类型，提示的参数签名可能不同。 DialogSet.prompt  方法是一个帮助程序方法。 此方法接受参数并为提示构造相应的选项；然后，它调用 begin 方法来启动提示对话框。

更换堆栈上的对话框：

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); // Can optionally passed in an 'args' as the second argument.
```

下面的[重复对话框](#repeat-a-dialog)和[对话框循环](#dialog-loops)部分介绍了如何使用 replace() 方法的详细信息。

## <a name="end-a-dialog"></a>结束对话框

通过从堆栈弹出对话框并将可选结果返回到父对话框的方式来结束对话框。 父对话框将使用任何返回的结果调用其 Dialog.resume() 方法。

最好在对话框结束时显式调用 `end()` 方法；但这不是必需的，因为当你到达瀑布图的末端时，对话框将自动从堆栈中弹出。

结束对话框：

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

使用传递到父对话框的可选参数结束对话框：

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end(result);
```

或者，还可以通过返回解析的约定来结束对话框：

```javascript
await Promise.resolve();
```

对 `Promise.resolve()` 的调用将导致对话框结束并从堆栈弹出。 然而，此方法不会调用父对话框来继续执行。 在调用 `Promise.resolve()` 后，执行停止，并且当用户向机器人发送消息时，机器人将在父对话框停止的地方继续。 这可能不是结束对话框的理想用户体验。 考虑使用 `end()` 或 `replace()` 结束对话框，以便你的机器人可以继续与用户进行交互。

### <a name="clear-the-dialog-stack"></a>清除对话框堆栈

如果要使所有对话框从堆栈弹出，可以通过调用 `dc.endAll()` 方法清除对话框堆栈。

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

### <a name="repeat-a-dialog"></a>重复对话框

要重复对话框，请使用 `dialogs.replace()` 方法。

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); 
```

如果要默认显示主菜单，可以使用以下步骤创建 `mainMenu` 对话框：

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
        else {
            // Repeat the menu
            await dc.replace('mainMenu');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);

dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
```

此对话框使用 `ChoicePrompt` 显示菜单并等待用户选择一个选项。 用户选择 `Order Dinner` 或 `Reserve a table` 时，它会启用相应选择的对话框，任务完成后，此对话框不是在最后一步中结束对话框，而是自行重复。

### <a name="dialog-loops"></a>对话框循环

使用 `replace()` 方法的另一个方法是模拟循环。 以此应用场景为例。 如果要允许用户将多个菜单项添加到购物车，可以循环菜单选项，直到用户完成订购。

```javascript
// Order dinner:
// Help user order dinner from a menu

var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "More info", "Process order", "Cancel", "Help"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }

}

// The order cart
var orderCart = {
    orders: [],
    total: 0,
    clear: function(dc) {
        this.orders = [];
        this.total = 0;
        dc.context.activity.conversation.orderCart = null;
    }
};

dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        orderCart.clear(dc); // Clears the cart.

        await dc.begin('orderPrompt'); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);

// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                // ...
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);

// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());

```

上面的示例代码显示了主 `orderDinner` 对话框使用名为 `orderPrompt` 的帮助程序对话框处理用户选择。 `orderPrompt` 对话框显示该菜单，让用户选择项，将项添加到购物车并再次提示。 这样，用户可以向订单添加多个项。 对话框循环，直到用户选择 `Process order` 或 `Cancel`。 此时，执行交还给父对话框（例如：`orderDinner`）。 如果用户想要处理订单，`orderDinner` 对话框会进行最后一分钟的保管；否则，它会结束并将执行返回到其父对话框（例如：`mainMenu`）。 然后，`mainMenu` 对话框继续执行最后一步，即简单地重新显示主菜单选项。

---

## <a name="next-steps"></a>后续步骤

现在你已了解如何使用对话框、提示和瀑布图管理会话流，让我们看看如何将对话框分解为模块化任务，而不是将它们全部集中在主机器人逻辑的 `dialogs` 对象中。

> [!div class="nextstepaction"]
> [使用复合控件创建机器人模块化逻辑](bot-builder-compositcontrol.md)