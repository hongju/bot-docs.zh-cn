---
title: 使用对话管理复杂的聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用对话管理复杂的聊天流。
keywords: 复杂的聊天流, 重复, 循环, 菜单, 对话, 提示, 瀑布, 对话集
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 304de6783a268140bf74f95d96cd9c24e12c4c05
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/21/2018
ms.locfileid: "40236355"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>使用对话管理复杂的聊天流

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

可以使用对话库管理来简单和复杂的聊天流。 在[简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)中，用户从瀑布的第一个步骤开始一直执行到最后一个步骤，聊天交换最终将会完成。 在本文中，我们将使用对话来管理包含可分支和循环部分的较复杂聊天。 为此，我们将使用对话上下文的 _replace_ 方法，并在对话的不同部分之间传递参数。

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

为了让你更好地控制对话堆栈，“对话”库提供了 _replace_ 方法。 使用此方法可将当前对话从堆栈中弹出，将新对话推送到堆栈顶层，然后启动新的对话。 借助此功能可以提供更复杂的聊天。 可以使用这些方法来创建具有任意复杂性的聊天。 如果聊天复杂性不断增大，以致对话变得难以管理，则你可以创建自己的控制逻辑流来跟踪用户的聊天。

<!-- TODO: This is probably a good place to add a link to the modular/composite dialogs topic. -->

在本文中，我们将为某个酒店机器人创建对话，宾客可以使用这些对话来预订餐桌，或者订购餐饮并将其递送到客房。 顶层对话为宾客提供以下两个选项。 如果宾客想要预订餐桌，则顶层对话将启动餐桌预订对话。 如果宾客想要订餐，则顶层对话将启动订餐对话。 订餐对话首先要求宾客在菜单中选择食品，然后要求提供房号。 食品的选择也是一个对话，使我们能够在处理宾客的餐饮订单之前，让宾客选择多个项目。

此图演示了我们要在本文中创建的对话及其相互关系。

![本文中使用的对话插图](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>如何分支

对话上下文维护对话堆栈，对于堆栈中的每个对话，将跟踪下一个步骤是什么。 它的 _begin_ 方法将对话推送到堆栈的顶层，_end_ 方法从堆栈中弹出顶层对话。

一个对话可以通过调用对话上下文的 _begin_ 方法并提供新对话的 ID，来启动同一对话集中的新对话（也称为分支）。 原始对话仍保留在堆栈中，但对对话上下文的 _continue_ 方法的调用只会发送到位于堆栈顶层的对话，即“活动对话”。 将某个对话从堆栈中弹出后，在执行到堆栈中瀑布的下一步骤时，对话上下文将其弹出位置处恢复。

因此，可以通过在一个对话中包含一个步骤（该步骤可按条件选择一个对话来启动一组潜在对话），在聊天流中创建分支。

## <a name="how-to-loop"></a>如何循环

使用对话上下文的 _replace_ 方法可以替换位于堆栈顶层的对话。 旧对话的状态将被丢弃，新对话将从头开始启动。 可以使用此方法并将某个对话替换为其自身，来创建循环。 但是，若要[保存状态](bot-builder-howto-v4-state.md)，需在 _replace_ 方法的调用中，将信息传递给该对话的新实例。 在以下示例中可以看到，订餐购物车已按对话状态排序，重复 `orderPrompt` 对话时，会传入当前对话状态，使新对话的状态可以继续在购物车中添加项目。 如果想要将这些信息存储在其他状态中，请参阅[保存用户数据](bot-builder-tutorial-persist-user-inputs.md)。

## <a name="create-the-dialogs-for-the-hotel-bot"></a>为酒店机器人创建对话

在本部分，我们将创建对话来管理所述酒店机器人的聊天。

### <a name="install-the-dialogs-library"></a>安装对话框库

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
npm install --save botbuilder-dialogs
```

若要在机器人中使用对话框，请将其包含在机器人代码中。 例如，将此代码添加到 **app.js** 文件：

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>创建对话集

创建一个对话集，以便在其中添加本示例所述的所有对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

创建 **HotelDialog** 类，并添加稍后需要用到的 using 语句。

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
```

从 **DialogSet** 派生类，并定义用于标识此对话集的对话、提示和状态信息的 ID 与键。

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialog : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

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

若要在机器人中使用对话框，请将其包含在机器人代码中。

从 **app.js** 文件引用该库。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

然后创建对话集。

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

---

### <a name="add-the-prompts-to-the-set"></a>将提示添加到该集

我们将使用 **ChoicePrompt** 来询问宾客是否要订餐或预订餐桌，以及要在餐饮菜单中选择哪个选项。 此外，当宾客订餐时，我们将使用 **NumberPrompt** 来询问其房号。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelDialog** 构造函数中添加两个提示。

```csharp
// Add the prompts.
this.Add(Inputs.Choice, new ChoicePrompt(Culture.English));
this.Add(Inputs.Number, new NumberPrompt<int>(Culture.English));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

将这两个提示添加到对话集。

```javascript
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
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
    /// <summary>The text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>The ID of the associated dialog for this option.</summary>
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

    /// <summary>The name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>The price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>The text to show the guest for this option.</summary>
    public string Description => (double.IsNaN(Price)) ? Name : $"{Name} - ${Price:0.00}";
}

/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>The options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static List<string> WelcomeList { get; } = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the top-level dialog.</summary>
    public static List<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(WelcomeList);

    /// <summary>The reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(WelcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>The options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static List<string> MenuList { get; } = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the food-selection dialog.</summary>
    public static List<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(MenuList);

    /// <summary>The reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(MenuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

创建 **dinnerMenu** 常量用于包含此信息。

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

此对话框使用 `ChoicePrompt` 显示菜单并等待用户选择一个选项。 当用户选择 `Order dinner` 或 `Reserve a table` 时，会启动相应选项的对话；该对话完成后，`mainMenu` 对话会通过再次启动 `mainMenu` 对话来重复自身，而不仅仅是结束最后一个步骤中的对话，使用户不知道发生了什么情况。 使用这种简单的结构，机器人始终会显示菜单，而用户也始终知道有哪些选项。

**MainMenu** 对话包含以下瀑布步骤。

* 在瀑布的第一个步骤中，我们将初始化或清除对话状态、欢迎宾客，并要求他们从可用选项中进行选择：`Order dinner` 或 `Reserve a table`。
* 在第二个步骤中，我们检索宾客的选项，然后启动与该选项关联的子对话。 子对话结束后，此对话将在后一个步骤中恢复。
* 在最后一个步骤中，我们使用 **DialogContext.Replace** 方法将此对话替换为其自身的新实例，这样就可以有效地将欢迎对话转换为循环菜单。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the main welcome dialog.
this.Add(MainMenu, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Greet the guest and ask them to choose an option.
        await dc.Context.SendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.Prompt(Inputs.Choice, "How may we serve you today?", new ChoicePromptOptions
        {
            Choices = Lists.WelcomeChoices,
            RetryPromptActivity = Lists.WelcomeReprompt,
        });
    },
    async (dc, args, next) =>
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)args["Value"];
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        await dc.Begin(dialogId, dc.ActiveDialog.State);
    },
    async (dc, args, next) =>
    {
        // Start this dialog over again.
        await dc.Replace(MainMenu);
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
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
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);
```

---

### <a name="create-the-order-dinner-dialog"></a>创建订餐对话

在订餐对话中，我们欢迎宾客使用“订餐服务”，并立即启动下一部分将要介绍的食品选择对话。 重要的是，如果宾客请求该服务处理其订单，则食品选择对话将返回订单中的项目列表。 为了完成整个过程，此对话会请求提供要将食品递送到的房号，然后在结束之前发送确认消息。 如果宾客取消了顺序，则此对话不会请求房号，而是立即结束。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **HotelDialog** 类中添加一个数据结构，可以使用它来跟踪宾客的餐饮订单。

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice> { }
```

在 **HotelDialog** 构造函数中添加订餐对话。

* 在第一个步骤中，我们发送欢迎消息，并启动食品选择对话。
* 在第二个步骤中，我们检查食品选择对话是否返回了购物车。
  * 如果已返回，则提示宾客提供房号，并保存购物车信息。
  * 如果未返回，则认为宾客取消了订单，此时会调用 **DialogContext.End** 来结束此对话。
* 最后一个步骤记录宾客的房号，并向其发送一条确认消息，然后结束。 机器人在此步骤中将订单转发到处理服务。

```csharp
// Add the order-dinner dialog.
this.Add(Dialogs.OrderDinner, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        await dc.Context.SendActivity("Welcome to our Dinner order service.");

        // Start the food selection dialog.
        await dc.Begin(Dialogs.OrderPrompt);
    },
    async (dc, args, next) =>
    {
        if (args.TryGetValue(Outputs.OrderCart, out object arg) && arg is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            dc.ActiveDialog.State[Outputs.OrderCart] = cart;
            await dc.Prompt(Inputs.Number, "What is your room number?", new PromptOptions
            {
                RetryPromptString = "Please enter your room number."
            });
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            await dc.End();
        }
    },
    async (dc, args, next) =>
    {
        // Get and save the guest's answer.
        var roomNumber = args["Text"] as string;
        dc.ActiveDialog.State[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await dc.Context.SendActivity($"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.");
        await dc.End();
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Order dinner:
// Help user order dinner from a menu
dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        
        await dc.begin('orderPrompt', dc.activeDialog.state.orderCart); // Prompt for orders
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

在 **HotelDialog** 构造函数中添加食品选择对话。

* 在第一个步骤中，我们初始化对话状态。 如果对话的输入参数包含购物车信息，则我们会将该信息保存到对话状态；否则，会创建并添加一个空购物车。 然后，我们提示宾客从餐饮菜单中做出选择。
* 在下一个步骤中，我们可以查看宾客选择的选项：
  * 如果所做的选择是处理订单的请求，则检查购物车是否包含任何项目。
    * 如果包含，则结束对话并返回购物车内容。
    * 否则，向宾客发送错误消息，并从对话的开头重新开始。
  * 如果所做的选择是取消订单的请求，则结束对话并返回空字典。
  * 如果所做的选择是一个餐饮项目，则将其添加到购物车、发送状态消息，从头开始启动对话，并传入当前订单状态。

```csharp
// Add the food-selection dialog.
this.Add(Dialogs.OrderPrompt, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            if (args is null || !args.ContainsKey(Outputs.OrderCart))
            {
                // First time through, initialize the order state.
                dc.ActiveDialog.State[Outputs.OrderCart] = new OrderCart();
                dc.ActiveDialog.State[Outputs.OrderTotal] = 0.0;
            }
            else
            {
                // Otherwise, set the order state to that of the arguments.
                dc.ActiveDialog.State = new Dictionary<string, object>(args);
            }

            await dc.Prompt(Inputs.Choice, "What would you like?", new ChoicePromptOptions
            {
                Choices = Lists.MenuChoices,
                RetryPromptActivity = Lists.MenuReprompt,
            });
        },
        async (dc, args, next) =>
        {
            // Get the guest's choice.
            var choice = (FoundChoice)args["Value"];
            var option = Lists.MenuOptions[choice.Index];

            // Get the current order from dialog state.
            var cart = (OrderCart)dc.ActiveDialog.State[Outputs.OrderCart];

            if (option.Name is MenuChoice.Process)
            {
                if (cart.Count > 0)
                {
                    // If there are any items in the order, then exit this dialog,
                    // and return the list of selected food items.
                    await dc.End(new Dictionary<string, object>
                    {
                        [Outputs.OrderCart] = cart
                    });
                }
                else
                {
                    // Otherwise, send an error message and restart from
                    // the beginning of this dialog.
                    await dc.Context.SendActivity(
                        "Your cart is empty. Please add at least one item to the cart.");
                    await dc.Replace(Dialogs.OrderPrompt);
                }
            }
            else if (option.Name is MenuChoice.Cancel)
            {
                await dc.Context.SendActivity("Your order has been cancelled.");

                // Exit this dialog, returning an empty property bag.
                dc.ActiveDialog.State.Clear();
                await dc.End(new Dictionary<string, object>());
            }
            else
            {
                // Add the selected food item to the order and update the order total.
                cart.Add(option);
                var total = (double)dc.ActiveDialog.State[Outputs.OrderTotal] + option.Price;
                dc.ActiveDialog.State[Outputs.OrderTotal] = total;

                await dc.Context.SendActivity($"Added {option.Name} (${option.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.");

                // Present the order options again, passing in the current order state.
                await dc.Replace(Dialogs.OrderPrompt, dc.ActiveDialog.State);
            }
        },
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc, orderCart){
        // Define a new cart of one does not exists
        if(!orderCart){
            // Initialize a new cart
            // convoState = conversationState.get(dc.context);
            dc.activeDialog.state.orderCart = {
                orders: [],
                total: 0
            };
        }
        else {
            dc.activeDialog.state.orderCart = orderCart;
        }
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        // Get state object
        // convoState = conversationState.get(dc.context);

        if(choice.value.match(/process order/ig)){
            if(dc.activeDialog.state.orderCart.orders.length > 0) {
                // Process the order
                // ...
                dc.activeDialog.state.orderCart = undefined; // Reset cart
                await dc.context.sendActivity("Processing your order.");
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            //dc.activeDialog.state.orderCart = undefined; // Reset cart
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!item){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                dc.activeDialog.state.orderCart.orders.push(item);
                dc.activeDialog.state.orderCart.total += item.Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${dc.activeDialog.state.orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt', dc.activeDialog.state.orderCart);
            }
        }
    }
]);
```

上面的示例代码显示了主 `orderDinner` 对话框使用名为 `orderPrompt` 的帮助程序对话框处理用户选择。 `orderPrompt` 对话显示食品项目的菜单，请求用户选择项目、将项目添加到购物车，并在循环中再次提示。 这样，用户可以向订单添加多个项。 对话框循环，直到用户选择 `Process order` 或 `Cancel`。 此时，执行已交接到父对话（`orderDinner` 对话）。 如果用户想要处理订单，`orderDinner` 对话框会进行最后一分钟的保管；否则，它会结束并将执行返回到其父对话框（例如：`mainMenu`）。 然后，`mainMenu` 对话框继续执行最后一步，即简单地重新显示主菜单选项。

---
### <a name="create-the-reserve-table-dialog"></a>创建预订餐桌对话

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

为了精简本示例，此处只显示了 `reserveTable` 对话的最简短实现。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the table-reservation dialog.
this.Add(Dialogs.ReserveTable, new WaterfallStep[]
    {
        // Replace this waterfall with your reservation steps.
        async (dc, args, next) =>
        {
            await dc.Context.SendActivity("Your table has been reserved.");
            await dc.End();
        }
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(dc, args, next){
        await dc.context.sendActivity("Your table has been reserved");
        await dc.end();
    }
]);
```

---

## <a name="next-steps"></a>后续步骤

可以通过在选项菜单中提供“更多信息”或“帮助”等其他选项来增强此机器人。 有关实现此类中断的详细信息，请参阅[处理用户中断](bot-builder-howto-handle-user-interrupt.md)。

了解如何使用对话、提示和瀑布来管理复杂的聊天流之后，让我们学习如何将对话（例如 `orderDinner` 和 `reserveTable` 对话）分解成独立的对象，而不是将它们全部混杂在一个大型对话集中。

> [!div class="nextstepaction"]
> [使用复合控件创建机器人模块化逻辑](bot-builder-compositcontrol.md)
