---
title: 使用分支和循环创建高级聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用对话管理复杂的聊天流。
keywords: 复杂的聊天流, 重复, 循环, 菜单, 对话, 提示, 瀑布, 对话集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9605a2f078be753023e6d178247a211ace107873
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/09/2018
ms.locfileid: "51333021"
---
# <a name="create-advance-conversation-flow-using-branches-and-loops"></a>使用分支和循环创建高级聊天流

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

在上一篇文章中，我们演示了如何使用对话库来管理简单聊天。 在[顺序聊天流](bot-builder-dialog-manage-conversation-flow.md)中，用户从瀑布的第一个步骤开始一直执行到最后一个步骤，聊天交换最终将会完成。 在本文中，我们将使用对话来管理包含可分支和循环部分的较复杂聊天。 为此，我们将使用在对话上下文和瀑布步骤上下文中定义的各种方法，并在对话的不同部分之间传递参数。

有关对话的更多背景信息，请参阅[对话库](bot-builder-concept-dialog.md)。

为了让你更好地控制对话堆栈，“对话”库提供了“替换对话”方法。 使用此方法可将当前处于活动状态的对话交换为另一个对话，同时保持聊天的状态和流。 使用“开始对话”和“替换对话”方法可按需执行分支和循环语句，以创建更复杂的交互。 如果聊天复杂性增大，以致瀑布对话变得难以管理，请考虑[重复使用对话](bot-builder-compositcontrol.md)，或者基于 `Dialog` 基类构建一个自定义对话管理类。

在本文中，我们将为某个酒店接待机器人创建示例对话，宾客可以使用这些对话来访问常用的服务：在酒店的餐厅预订餐桌，以及通过客房服务订餐。  其中的每项功能及其连接的菜单将创建为对话集中的对话。

机器人的顶层对话为宾客提供以下两个选项。 如果宾客想要预订餐桌，则顶层对话将使用“开始对话”异步方法来启动餐桌预订对话。 如果宾客想要预订客房服务，则顶层对话将启动订餐对话。

订餐对话首先要求宾客在菜单中选择食品，然后要求提供房号。 食品的选择也是一个对话 - 它在提交餐饮订单之前宾客选择菜单项时会多次发挥作用。

此图演示了我们要在本文中创建的对话及其相互关系。

![本文中使用的对话插图](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>如何分支

对话上下文维护对话堆栈，对于堆栈中的每个对话，将跟踪下一个步骤是什么。 它的 _begin dialog_ 方法将对话框推送到堆栈的顶层，_end dialog_ 方法从堆栈中弹出顶层对话框。

若要执行分支，请从一组子对话中选择一个对话。 有关此概念的详细信息，请参阅[将聊天分支](bot-builder-concept-dialog.md#branch-a-conversation)。

## <a name="how-to-loop"></a>如何循环

使用对话上下文的“替换对话”方法可以替换位于堆栈顶层的对话。 旧对话的状态将被丢弃，新对话将从头开始启动。 可以使用此方法并将某个对话替换为其自身，来创建循环。 但是，若要[保存机器人已收集的任何信息](bot-builder-howto-v4-state.md)，需在“替换对话”方法的调用中，将该信息传递给该对话的新实例。

在以下示例中可以看到，客房服务订单存储在对话状态中，重复 `orderPrompt` 对话时，会传入当前对话状态，使新对话的状态可以继续在购物车中添加项。 若要在对话外部的机器人状态中存储此信息，请参阅如何[保存用户数据](bot-builder-tutorial-persist-user-inputs.md)。

## <a name="create-the-dialogs-for-the-hotel-bot"></a>为酒店机器人创建对话

在本部分，我们将创建对话来管理所述酒店机器人的聊天。

### <a name="install-the-dialogs-library"></a>安装对话框库

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们从一个基本的 EchoBot 模板着手。 有关说明，请参阅[适用于 .NET 的快速入门](../dotnet/bot-builder-dotnet-sdk-quickstart.md)。

要使用对话框，请为项目或解决方案安装 `Microsoft.Bot.Builder.Dialogs` NuGet 包。
然后，根据需要在代码文件的 using 语句中引用对话库。

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我们将从 EchoBot 模板着手。 有关说明，请参阅[适用于 JavaScript 的快速入门](../javascript/bot-builder-javascript-quickstart.md)。



可以从 npm 下载 `botbuilder-dialogs` 库。 若要安装 `botbuilder-dialogs` 库，请运行以下 npm 命令：

```cmd
npm install --save botbuilder-dialogs
```

若要在机器人中使用对话框，请将其包含在机器人代码中。 例如，将此代码添加到 **index.js** 文件：

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

并将此代码添加到 **bot.js** 文件：

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>创建对话集

创建一个对话集，以便在其中添加本示例所述的所有对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

创建 **HotelDialogs** 类，并添加稍后需要用到的 using 语句。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

从 **DialogSet** 派生该类。 包含一个采用 `IStatePropertyAccessor<DialogState>` 参数的构造函数，用于管理对话集实例的内部状态。 另外，定义用于标识此对话集的对话、提示和状态信息的 ID 与键。

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **index.js** 文件中，添加用于创建状态属性访问器的代码来管理对话状态，并使用该访问器来创建要对机器人使用的对话集。

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

然后更新活动处理调用，以使用机器人对象。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>将提示添加到该集

我们将使用 **ChoicePrompt** 来询问宾客是否要订餐或预订餐桌，以及要在餐饮菜单中选择哪个选项。 此外，当宾客订餐时，我们将使用 **NumberPrompt** 来询问其房号。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelDialogs** 构造函数中添加两个提示。

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

更新机器人构造函数，并将两个提示添加到对话集。

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
```

---

### <a name="define-some-of-the-supporting-information"></a>定义一些支持信息

由于我们需要有关餐饮菜单中每个选项的信息，因此，现在让我们进行相关的设置。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

创建一个内部静态 **Lists** 类来包含此信息。 我们还要创建内部 **WelcomeChoice** 和 **MenuChoice** 类来包含有关每个选项的信息。

接下来，我们在顶层欢迎对话中添加选项列表的信息，并创建支持列表，以便稍后在提示宾客确认此信息时使用。 前期的工作量稍有增加，但会使对话代码变得更简单。

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** 文件中，创建 **dinnerMenu** 常量用于包含此信息。

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel"],
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
```

---

### <a name="create-the-welcome-dialog"></a>创建欢迎对话

此对话使用 `ChoicePrompt` 来显示菜单，然后等待用户选择一个选项。 当用户选择 `Order dinner` 或 `Reserve a table` 时，会启动相应的对话。 子对话完成后，`mainMenu` 对话会通过再次启动 `mainMenu` 对话来进行循环，而不仅仅是结束最后一个步骤中的对话，使用户不知道发生了什么情况。 使用这种简单的结构，机器人始终会显示菜单，而用户也始终知道有哪些选项。

`mainMenu` 对话包含以下瀑布步骤：

* 在瀑布的第一个步骤中，我们将初始化或清除对话状态、欢迎宾客，并要求他们从可用选项中进行选择：`Order dinner` 或 `Reserve a table`。
* 在第二个步骤中，我们检索宾客的选项，然后启动与该选项关联的子对话。 子对话结束后，此对话将在后一个步骤中恢复。
* 在最后一个步骤中，我们使用“替换对话”方法将此对话替换为其自身的新实例，这样就可以有效地将欢迎对话转换为循环菜单。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在构造函数中添加瀑布对话。 然后定义瀑布步骤。 此处，我们在一个嵌套类中定义这些步骤。

在 **HotelDialogs** 构造函数中。

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

在 **HotelDialogs** 类中添加步骤定义。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人构造函数中添加 `mainMenu` 瀑布对话。

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>创建订餐对话

在订餐对话中，我们欢迎宾客使用“订餐服务”，并立即启动下一部分将要介绍的食品选择对话。 重要的是，如果宾客请求该服务处理其订单，则食品选择对话将返回订单中的项目列表。 为了完成该过程，此对话会请求提供要将食品递送到的房号，然后发送确认消息。 如果宾客取消了顺序，则此对话不会请求房号，而是立即结束。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelDialog** 类中添加一个数据结构，可以使用它来跟踪宾客的餐饮订单。 由于此信息将保存在对话状态中，因此该类需要使用默认构造函数来完成序列化。

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

在构造函数中添加瀑布对话。 然后定义瀑布步骤。 此处，我们在一个嵌套类中定义这些步骤。

在 **HotelDialogs** 构造函数中。

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

在 **HotelDialogs** 类中添加步骤定义。

* 在第一个步骤中，我们发送欢迎消息，并启动食品选择对话。
* 在第二个步骤中，我们检查食品选择对话是否返回了购物车。
  * 如果已返回，则提示宾客提供房号，并保存购物车信息。
  * 如果未返回，则认为宾客取消了订单，此时会调用“结束对话”方法来结束此对话。
* 最后一个步骤记录宾客的房号，并向其发送一条确认消息，然后结束。 机器人在此步骤中将订单转发到处理服务。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人构造函数中添加 `orderDinner` 瀑布对话。

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
```

---

### <a name="create-the-order-prompt-dialog"></a>创建订单提示对话

在食品选择对话中，我们向宾客提供一个选项列表，其中包含可订购的餐饮项目，以及两个订单处理请求。 此对话会一直循环到宾客选择让机器人处理或取消其订单为止。

* 当宾客选择某个餐饮项目时，我们会将其添加到购物车。
* 如果宾客选择处理其订单，则我们先检查购物车是否为空。
  * 如果为空，则发送错误消息并继续循环。
  * 否则，我们会结束此对话，并将购物车信息返回到父对话。
* 如果宾客选择取消其订单，则我们会结束对话，且不返回购物车信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在构造函数中添加瀑布对话。 然后定义瀑布步骤。 此处，我们在一个嵌套类中定义这些步骤。

在 **HotelDialogs** 构造函数中。

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* 在第一个步骤中，我们初始化对话状态。 如果对话的输入参数包含购物车信息，则我们会将该信息保存到对话状态；否则，会创建并添加一个空购物车。 然后，我们提示宾客从餐饮菜单中做出选择。
* 在下一个步骤中，我们可以查看宾客选择的选项：
  * 如果所做的选择是处理订单的请求，则检查购物车是否包含任何项目。
    * 如果包含，则结束对话并返回购物车内容。
    * 否则，向宾客发送错误消息，并从对话的开头重新开始。
  * 如果所做的选择是取消订单的请求，则结束对话并返回空字典。
  * 如果所做的选择是一个餐饮项目，则将其添加到购物车、发送状态消息，从头开始启动对话，并传入当前订单状态。

在 **HotelDialogs** 类中添加步骤定义。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人构造函数中添加 `orderPrompt` 瀑布对话。

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

上面的示例代码显示了主 `orderDinner` 对话框使用名为 `orderPrompt` 的帮助程序对话框处理用户选择。 `orderPrompt` 对话显示食品项目的菜单，请求用户选择项目、将项目添加到购物车，并在循环中再次提示。 这样，用户可以向订单添加多个项。 对话框循环，直到用户选择 `Process order` 或 `Cancel`。 此时，执行已交接到父对话（`orderDinner` 对话）。 如果用户想要处理订单，`orderDinner` 对话框会进行最后一分钟的保管；否则，它会结束并将执行返回到其父对话框（例如：`mainMenu`）。 然后，`mainMenu` 对话框继续执行最后一步，即简单地重新显示主菜单选项。

---

### <a name="create-the-reserve-table-dialog"></a>创建预订餐桌对话

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

为了精简本示例，此处只显示了 `reserveTable` 对话的最简短实现。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在构造函数中添加瀑布对话。 然后定义瀑布步骤。 此处，我们在一个嵌套类中定义这些步骤。

在 **HotelDialogs** 构造函数中。

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

在 **HotelDialogs** 类中添加步骤定义。

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人构造函数中添加占位符 `reserveTable` 瀑布对话。

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>更新机器人代码以调用对话

更新机器人的轮次处理程序代码以调用对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

将 **EchoBotAccessors.cs** 重命名为 **BotAccessors.cs**，并将类从 `EchoBotAccessors` 重命名为 `BotAccessors`。 更新 using 语句和类定义，以提供需要对此机器人使用的状态属性访问器。

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

更新 **Startup.cs** 文件以配置 `BotAccessors` 单一实例。

1. 更新 using 语句。

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. 更新 `ConfigureServices` 方法中用于注册机器人状态属性访问器的部分。

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

将 EchoWithCounterBot.cs 文件重命名为 HotelBot.cs，并将类从 EchoWithCounterBot 重命名为 HotelBot。

1. 更新机器人的初始化代码。

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. 更新机器人的轮次处理程序，使其运行对话。

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>后续步骤

可以通过在选项菜单中提供“更多信息”或“帮助”等其他选项来增强此机器人。 有关实现此类中断的详细信息，请参阅[处理用户中断](bot-builder-howto-handle-user-interrupt.md)。

了解如何使用对话、提示和瀑布来管理复杂的聊天流之后，让我们学习如何将对话（例如 `orderDinner` 和 `reserveTable` 对话）分解成独立的对象，而不是将它们全部混杂在一个大型对话集中。

> [!div class="nextstepaction"]
> [使用复合控件创建机器人模块化逻辑](bot-builder-compositcontrol.md)
