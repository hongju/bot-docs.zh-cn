---
title: 处理用户中断 | Microsoft Docs
description: 了解如何处理用户中断和直接聊天流。
keywords: 中断, 切换主题, 中断
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/17/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fff4f8e2a4d2d86cf440bee7ab40216e93a8c8c5
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756660"
---
# <a name="handle-user-interruptions"></a>处理用户中断

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

处理用户中断是机器人是否可靠的一个重要方面。 虽然你可能认为用户会按照定义的聊天流逐步操作，但他们可能会改变主意或在流程中提出问题而不是回答问题。 在这些情况下，机器人将如何处理用户的输入呢？ 用户体验会是什么样的？ 如何保留用户状态数据？

这些问题没有正确的答案，因为对于机器人要处理的场景来说，每种情况都是唯一的。 在本主题中，我们将探讨一些用于处理用户中断的常见方法，并提供在机器人中实现它们的一些建议方式。

## <a name="handle-expected-interruptions"></a>处理预期中断

过程化聊天流具有一组要引导用户完成的核心步骤，不同于这些步骤的任何用户操作都是潜在中断。 在正常流中，可以预料到某些中断。

**餐位预订**在餐位预订机器人中，核心步骤是要求用户提供日期和时间、聚会人数和订座人姓名。 在此流程中，可以预料到的某些预期中断包括：

* `cancel`：退出流程。
* `help`：提供有关此流程的其他指导。
* `more info`：提供提示和建议，或提供预订餐位的其他方式（例如：用于联系的电子邮件地址或电话号码）。
* `show list of available tables`：（如果有该选项）显示用户所需日期和时间可提供的餐位列表。

**点餐**：在点餐机器人中，核心步骤是提供菜单项，并允许用户将菜品项添加到购物车。 在此流程中，可以预料到的某些预期中断包括：

* `cancel`：退出点餐流程。
* `more info`：提供有关每个菜单项的食材详情。
* `help`：提供系统使用帮助。
* `process order`：处理订单。

你可以将这些作为“建议操作”列表或作为提示提供给用户，让他们至少知道可以发送哪些机器人能够理解的命令。

例如，在点餐流中，可以将预期中断与菜单项一起提供。 在这种情况下，菜单项将作为 `choices` 的数组发送。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
public class dinnerItem
{
    public string Description;
    public double Price;
}

public class dinnerMenu
{
    static public Dictionary<string, dinnerItem> dinnerChoices = new Dictionary<string, dinnerItem>
    {
        { "potato salad", new dinnerItem { Description="Potato Salad", Price=5.99 } },
        { "tuna sandwich", new dinnerItem { Description="Tuna Sandwich", Price=6.89 } },
        { "clam chowder", new dinnerItem { Description="Clam Chowder", Price=4.50 } }
    };

    static public string[] choices = new string[] {"Potato Salad", "Tuna Sandwich", "Clam Chowder", "more info", "Process order", "help", "Cancel"};
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
            "more info", "Process order", "Cancel"],
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

在点餐逻辑中，可使用字符串匹配或正则表达式来进行检查。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

首先，需要定义一个帮助程序来跟踪订单

```cs
// Helper class for storing the order in the dictionary
public class Orders
{
    public double total;
    public string order;
    public bool processOrder;

    // Initialize order values
    public Orders()
    {
        total = 0;
        order = "";
        processOrder = false;
    }
}
```

然后，将对话添加到机器人。

```cs
dialogs.Add("orderPrompt", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt the user
        await dc.Prompt("choicePrompt",
            "What would you like for dinner?",
            new ChoicePromptOptions
            {
                Choices = dinnerMenu.choices.Select( s => new Choice { Value = s }).ToList(),
                RetryPromptString = "I'm sorry, I didn't understand that. What would you " +
                    "like for dinner?"
            });
    },
    async(dc, args, next) =>
    {
        var convo = ConversationState<Dictionary<string,object>>.Get(dc.Context);

        // Get the user's choice from the previous prompt
        var response = (args["Value"] as FoundChoice).Value.ToLower();

        if(response == "process order")
        {
            try
            {
                var order = convo["order"];

                await dc.Context.SendActivity("Order is on it's way!");

                // In production, you may want to store something more helpful,
                // such as send order off to be made
                (order as Orders).processOrder = true;

                // Once it's submitted, clear the current order
                convo.Remove("order");
                await dc.End();
            }
            catch
            {
                await dc.Context.SendActivity("Your order is empty, please add your order choice");
                // Ask again
                await dc.Replace("orderPrompt");
            }
        }
        else if(response == "cancel" )
        {
            // Get rid of current order
            convo.Remove("order");
            await dc.Context.SendActivity("Your order has been canceled");
            await dc.End();
        }
        else if(response == "more info")
        {
            // Send more information about the options
            var msg = "More info: <br/>" +
                "Potato Salad: contains 330 calaries per serving. Cost: 5.99 <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. Cost: 6.89 <br/>"
                + "Clam Chowder: contains 650 calaries per serving. Cost: 4.50";
            await dc.Context.SendActivity(msg);

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else if(response == "help")
        {
            // Provide help information
            await dc.Context.SendActivity("To make an order, add as many items to your cart as " +
                "you like then choose the \"Process order\" option to check out.");

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else
        {
            // Unlikely to get past the prompt verification, but this will catch
            // anything that isn't a valid menu choice
            if(!dinnerMenu.dinnerChoices.ContainsKey(response))
            {
                await dc.Context.SendActivity("Sorry, that is not a valid item. " +
                    "Please pick one from the menu.");

                // Ask again
                await dc.Replace("orderPrompt");
            }
            else {
                // Add the item to cart
                Orders currentOrder;

                // If there is a current order going, add to it. If not, start a new one
                try
                {
                    currentOrder = convo["order"] as Orders;
                }
                catch
                {
                    convo["order"] = new Orders();
                    currentOrder = convo["order"] as Orders;
                }

                // Add to the current order
                currentOrder.order += (dinnerMenu.dinnerChoices[$"{response}"].Description) + ", ";
                currentOrder.total += (double)dinnerMenu.dinnerChoices[$"{response}"].Price;

                // Save back to the conversation state
                convo["order"] = currentOrder;

                await dc.Context.SendActivity($"Added to cart. Current order: " +
                    $"{currentOrder.order} " +
                    $"<br/>Current total: ${currentOrder.total}");

                // Ask again to allow user to add more items or process order
                await dc.Replace("orderPrompt");
            }
        }
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                dc.context.activity.conversation.dinnerOrder = orderCart;

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
```

---

## <a name="handle-unexpected-interruptions"></a>处理意外中断

某些中断超出了机器人的执行范围。
虽然无法预料到所有中断，但可以设置机器人来处理中断模式。

### <a name="switching-topic-of-conversations"></a>切换聊天主题

如果用户正在聊天中，并希望切换到另一个聊天，该怎么办？ 例如，机器人可以预订餐位和点餐。
用户在预订餐位流中时没有回答“聚会有多少人？”这一问题，而是发送“点餐”消息。 在这种情况下，用户改变了主意，希望改为进行点餐聊天。 应如何处理此中断？

可将主题切换到点餐流，或者让问题变得复杂一点，告诉用户他们需要提供号码，然后重新提示他们。 如果允许他们切换主题，则必须决定是否保存该进程，以便用户能从其中断位置继续，或者可以删除收集的所有信息，这样当他们下次想要预订餐位时，就必须重新启动该流程。 有关管理用户状态数据的详细信息，请参阅[使用聊天和用户属性保存状态](bot-builder-howto-v4-state.md)。

### <a name="apply-artificial-intelligence"></a>应用人工智能

对于范围外的中断，可以尝试猜测用户的意向。 可使用 QnAMaker、LUIS 或自定义逻辑等 AI 服务来执行此操作，然后为机器人认为的用户意向提供建议。

例如，在预订餐位流中，用户说“我想要一个汉堡”。 在此聊天流中，机器人不知道如何处理这种情况。 由于当前流与点餐无关，而机器人的其他聊天命令是“点餐”，因此机器人不知道如何处理此输入。 例如，如果应用 LUIS，则可以训练模型，从而识别用户想要点餐（例如：LUIS 可以返回“orderFood”意向）。 因此，机器人可能提供以下响应：“看来你是想点餐。 要切换到我们的点餐流程吗？” 有关训练 LUIS 和检测用户意向的详细信息，请参阅[针对语言理解的用户 LUIS](bot-builder-howto-v4-luis.md)。

### <a name="default-response"></a>默认响应

如果所有方法都失败了，可以发送通用的默认响应，而非不采取任何措施，导致用户对当前情况感到困惑。 默认响应告诉用户机器人理解哪些命令，以便用户可以重新进行对话。

可以检查机器人逻辑末尾的上下文响应标记，查看机器人在整个流程期间是否向用户发送了任何内容。 如果机器人处理了用户输入却没有响应，很可能是因为机器人不知道如何处理输入。 在这种情况下，可将其捕获并向用户发送默认消息。

此机器人的默认设置是为用户提供 `mainMenu` 对话。 这种情况下机器人为用户提供何种体验完全由你决定。

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
if(!context.Responded)
{
    await dc.EndAll().Begin("mainMenu");
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.endAll().begin('mainMenu');
}
```

---
