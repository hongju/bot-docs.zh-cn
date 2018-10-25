---
title: 替换对话 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for Node.js 来替换对话，以便重新提示用户输入并管理聊天流。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 30ac28f5ce700829b8c382c49905883ffa45da29
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000054"
---
# <a name="replace-dialogs"></a>替换对话

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

在聊天过程中需要验证用户输入或重复某个操作时，可以使用替换对话的功能。 使用 Bot Builder SDK for Node.js 时，可以通过 [`session.replaceDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) 方法来替换对话。 此方法可以结束当前对话，在不返回到调用方的情况下将其替换为新的对话。 

## <a name="create-custom-prompts-to-validate-input"></a>创建自定义提示来验证输入

Bot Builder SDK for Node.js 包含的输入验证适用于某些类型的[提示](bot-builder-nodejs-dialog-prompt.md)，例如 `Prompts.time` 和 `Prompts.choice`。 若要验证收到的用于响应 `Prompts.text` 的文本输入，必须创建你自己的验证逻辑和自定义提示。 

如果输入必须符合所定义的特定值、模式、范围或条件，则可能需要对输入进行验证。 如果输入没有通过验证，机器人可以使用 `session.replaceDialog` 方法提示用户再次输入该信息。

以下代码示例演示如何创建自定义提示来验证用户针对某个电话号码提供的输入。

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

在此示例中，系统一开始提示用户提供其电话号码。 验证逻辑使用正则表达式来匹配用户输入中的一系列数字。 如果输入包含 10 个或 11 个数字，则会在响应中返回该电话号码。 否则，系统会执行 `session.replaceDialog` 方法来重复 `phonePrompt` 对话，提示用户再次进行输入。这一次提供更具体的指南，详细说明预期的输入格式。

调用 `session.replaceDialog` 方法时，请指定要重复的对话的名称和一个参数列表。 在此示例中，参数列表包含的 `{ reprompt: true }` 可以让机器人根据这是初始提示还是重复提示来提供不同的提示消息，但是你可以指定机器人可能需要的任意参数。 

## <a name="repeat-an-action"></a>重复操作

在聊天过程中，有时可能需要重复某个对话，让用户多次完成某个特定的操作。 例如，如果机器人提供多种服务，一开始可以显示服务菜单，引导用户完成请求某个服务的过程，然后再次显示服务菜单，让用户请求另一服务。 为此，可以使用 [`session.replaceDialog`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog) 方法来再次显示服务菜单，而不必使用 ['session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) 方法来结束聊天。 

以下示例演示如何使用 `session.replaceDialog` 方法来实现此类方案。 首先，服务菜单定义如下：

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

`mainMenu` 对话是通过默认对话调用的，因此此菜单会在聊天开始时显示给用户。 另外，[`triggerAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction) 是附加到 `mainMenu` 对话的，因此每当用户输入为“主菜单”时，此菜单也会显示。 用户在看到菜单并选择某个选项以后，系统会通过 `session.beginDialog` 方法调用与用户的选择相对应的对话。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

在此示例中，如果用户选择选项 1 来订外卖餐，系统会调用 `orderDinner` 对话并引导用户完成订餐过程。 过程结束时，机器人会确认订单并通过 `session.replaceDialog` 方法再次显示主菜单。

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

`orderDinner` 对话附加了两个触发器，因此用户可以在订餐过程中随时“重填”或“取消”订单。 

第一个触发器是 [`reloadAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction)，用户只需发送输入“重填”即可再次从头开始订餐过程。 当触发器匹配到话语“重填”时，`reloadAction` 会从头开始重启对话。 

第二个触发器是 [`cancelAction`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction)，用户只需发送输入“取消”即可彻底取消订餐过程。 此触发器不会自动地再次显示主菜单，而是发送一条内容为“键入‘主菜单’继续”的消息，告知用户下一步如何操作。

### <a name="dialog-loops"></a>对话循环

在以上示例中，用户每次订餐只能选择一个菜。 也就是说，如果用户要从菜单中订两个菜，则第一个菜需完成整个订餐过程，第二个菜也需再次重复整个订餐过程。 

以下示例演示了如何改进上一个机器人，即重构订餐菜单，使之成为独立的对话。 这样机器人就可以在一个循环中重复订餐菜单，使用户一次订餐可以选择多个菜。

首先，向菜单添加“结账”选项。 此选项会让用户退出选菜过程并结账。

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

接下来，重构订餐提示，使之成为独立的对话，这样机器人就可以不断重复此菜单，让用户在订餐时添加多个菜。

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

在此示例中，订单存储在机器人数据存储中，该存储的范围仅限当前聊天 `session.conversationData.orders`。 每次有新订单时，此变量会使用新的数组重新初始化；每次用户选择一个菜时，机器人会将菜名添加到 `orders` 数组，将价格计入总计并存储在结账的 `Price` 变量中。 用户选完要订的菜以后，说一声“结账”就可以结账并完成订餐的后续过程。

> [!NOTE]
> 有关机器人数据存储的详细信息，请参阅[管理状态数据](bot-builder-nodejs-state.md)。 

最后，在 `orderDinner` 对话中更新这一系列操作的第二步，对订单进行确认。 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>取消对话

虽然可以使用 `session.replaceDialog` 方法将当前对话替换为新对话，但不能将它用于替换其位置在对话堆栈更下方的对话。 若要替换对话堆栈中不属当前对话的对话，请改用 [`session.cancelDialog`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog) 方法。 

`session.cancelDialog` 方法可以用来结束某个对话（不管该对话存在于对话堆栈的何处），并可选择性地就地调用新对话。 若要调用 `session.cancelDialog` 方法，请指定要取消的对话的 ID，并可选择性地指定要就地调用的对话的 ID。 例如，以下代码片段取消 `orderDinner` 对话，将其替换为 `mainMenu` 对话：

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

调用 `session.cancelDialog` 方法时，会对对话堆栈进行后向搜索，并会取消该对话的第一个匹配项，将该对话及其子对话（如果有）从对话堆栈中删除。 然后，系统会将控制返回到调用对话，该对话可以搜索 [`results.resumed`](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed) 代码，而此代码等效于用于检测取消操作的 [`ResumeReason.notCompleted`](http://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted)。

也可在调用 `session.cancelDialog` 方法时不指定要取消的对话的 ID，改为指定要取消的对话的索引（从零开始，表示对话在对话堆栈中的位置）。 例如，以下代码片段终止当前正在进行的对话（索引 = 0），并就地启动 `mainMenu` 对话。 `mainMenu` 对话是在对话堆栈的 0 位置调用的，因此成为新的默认对话。

```javascript
session.cancelDialog(0, 'mainMenu');
```

以上述[对话循环](#dialog-loops)中讨论的示例为例， 当用户访问选菜菜单时，该对话 (`addDinnerItem`) 是对话堆栈 `[default dialog, mainMenu, orderDinner, addDinnerItem]` 中的第四个对话。 如何才能让用户在 `addDinnerItem` 对话中取消其订单？ 如果将 `cancelAction` 触发器附加到 `addDinnerItem` 对话，则只会将用户返回到前一对话 (`orderDinner`)，而该对话又会将用户直接送回 `addDinnerItem` 对话。

这种情况下就可以使用 `session.cancelDialog` 方法。 从[对话循环示例](#dialog-loops)着手，将“取消订单”作为一个显式选项添加到订餐菜单中。

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

然后更新 `addDinnerItem` 对话，检查是否存在“取消订单”请求。 如果检测到“取消”字样，请使用 `session.cancelDialog` 方法取消默认对话（即堆栈中索引编号为 0 的对话），并就地调用 `mainMenu` 对话。 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

通过这种方式使用 `session.cancelDialog` 方法，即可实现机器人所需的任何聊天流。

## <a name="next-steps"></a>后续步骤

可以看到，若要替换堆栈中的对话，可以使用各种类型的**操作**。 操作在管理聊天流时具有很大的灵活性。 让我们更仔细地看看**操作**，了解如何才能更好地处理用户操作。

> [!div class="nextstepaction"]
> [处理用户操作](bot-builder-nodejs-dialog-actions.md)
