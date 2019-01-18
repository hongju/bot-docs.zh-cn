---
title: 使用对话框管理会话流 | Microsoft Docs
description: 了解如何在 Bot Framework SDK for Node.js 中使用对话框管理机器人与用户之间的聊天。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 96c28101c3ea72c70c6ad53b06306f4ea00b2929
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225602"
---
# <a name="manage-conversation-flow-with-dialogs"></a>使用对话框管理会话流

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

管理聊天流是构建机器人不可或缺的任务。 机器人需要能够漂亮地执行核心任务，并优雅地处理中断情况。 使用 Bot Framework SDK for Node.js 时，可通过对话框来管理聊天流。

对话框就像程序中的函数。 这通常是为了执行特定操作，并可根据需要随时调用。 就像希望机器人处理的任何会话流一样，可以将多个对话框串联在一起处理。 可使用 Bot Framework SDK for Node.js 中的内置功能（如[提示](bot-builder-nodejs-dialog-prompt.md)和[瀑布图](bot-builder-nodejs-dialog-waterfall.md)）来管理聊天流。

本文列举了一系列示例，说明如何同时管理简单的聊天流和复杂的聊天流，其中机器人可使用对话完美地处理中断并继续聊天流。 这些示例基于以下场景： 

1. 机器人想要预订晚餐。
2. 机器人可在预订期间随时处理“帮助”请求。
3. 机器人可处理当前预订步骤中上下文相关的“帮助”。
4. 机器人可处理聊天的多个主题。

## <a name="manage-conversation-flow-with-a-waterfall"></a>使用瀑布管理聊天流

[瀑布](bot-builder-nodejs-dialog-waterfall.md)是指一种对话，使机器人能够轻松引导用户完成一系列任务。 在此示例中，预订机器人向用户询问一系列问题，使机器人可以处理预订请求。 机器人将提示用户输入以下信息：

1. 预订日期和时间
2. 用餐人数
3. 预订人的姓名

以下代码示例演示如何使用瀑布来指导用户完成一系列提示。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.number(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;
        
        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

此机器人的核心功能将出现在默认对话中。 默认对话在创建机器人时进行定义： 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

此外，在此创建过程中，还可设置希望使用的[数据存储](bot-builder-nodejs-state.md)类型。 例如，要使用内存中存储，可以进行如下设置：

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

默认对话作为定义瀑布的步骤的函数数组进行创建。 在示例中，有四个函数，因此瀑布包含四个步骤。 每一步都执行一个任务，并在下一步中处理相关结果。 该过程一直持续到最后一步，最后一步确认预订并结束对话。

以下屏幕截图显示在 [Bot Framework Emulator](../bot-service-debug-emulator.md) 中运行的机器人的结果：

![使用瀑布管理聊天流](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>提示用户输入

此示例的每个步骤都将使用提示要求用户进行输入。 提示是特殊类型的对话，用于提醒用户输入、等待响应以及向瀑布的下一步返回响应。 有关可在机器人中使用的各种类型的提示信息，请参阅[提示用户输入](bot-builder-nodejs-dialog-prompt.md)。

在此示例中，机器人使用 `Prompts.text()` 以文本格式形式向用户请求自由格式的响应。 用户可使用任何文本进行响应，并且机器人必须决定处理响应的方式。 `Prompts.time()` 使用 [Chrono](https://github.com/wanasit/chrono) 库分析字符串中的日期和时间信息。 这样，机器人便可理解更多用于指定日期和时间的自然语言。 例如：“2017 年 6 月 6 日晚上 9 点”、“今天晚上 7:30”、“下星期一下午 6 点”，等等。

> [!TIP] 
> 根据托管机器人的服务器的时区，将用户输入的时间转换为 UTC 时间。 服务器与用户所在的时区可能不同，因此请务必考虑到时区。 若要将日期和时间转换为用户的本地时间，请考虑请求用户指定他们所在的时区。

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>使用多个对话管理聊天流

管理聊天流的另一种方法是使用瀑布和多个对话的组合。 通过瀑布，可将对话中的函数串联在一起，而通过的对话，可将聊天拆分成较小的功能并可随时重复使用这些功能。

例如，请考虑预订晚餐的机器人。 下面的代码示例演示重写以使用瀑布和多个对话的上一个示例。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

执行此机器人的结果与上一个机器人（仅使用瀑布）完全相同。 但在编程方面，存在两个主要区别：

1. 默认对话专用于管理聊天流。
2. 聊天的每一步的任务由单独的对话进行管理。 在这种情况下，机器人需要三条信息，因此它会提示用户三次。 现在，每个提示都包含在其自己的对话中。

通过此方法，可将聊天流与任务逻辑分开。 因此，如有必要，此对话可被不同的聊天流重复使用。 

## <a name="respond-to-user-input"></a>响应用户输入

在引导用户完成一系列任务的过程中，如果用户有问题或想要在回答前请求其他信息，如何处理这些请求？ 例如，不考虑用户在聊天中的所在位置，如果用户输入“help”、“Support”或“Cancel”，机器人会如何响应？ 如果用户需要有关某个步骤的其他信息，将会怎么样？ 如果用户改变主意，想要放弃当前任务，而启动一个完全不同的任务，会发生什么情况？

Bot Framework SDK for Node.js 允许机器人侦听全局上下文中的某些输入，或侦听当前对话框范围内本地上下文中的输入。 这些输入被称为[操作](bot-builder-nodejs-dialog-actions.md)，使机器人能够基于 `matches` 子句侦听用户输入。 由机器人来决定如何对特定用户输入作出响应。

### <a name="handle-global-action"></a>处理全局操作

如果希望机器人能够处理聊天任意时刻的操作，请使用 `triggerAction`。 触发器使机器人能够在输入匹配指定术语时调用特定对话。 举个例子，如果想要支持全局“Help”选项，可创建一个帮助对话并附加 `triggerAction`，用于侦听输入匹配的“Help”。

下面的代码示例演示如何将 `triggerAction` 附加到对话，以指定用户输入“help”时，应调用该对话。

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

默认情况下，执行 `triggerAction` 时，将清除对话堆栈，并且已触发的对话将成为新的默认对话。 在此示例中，执行 `triggerAction` 时，将清除对话堆栈，然后 `help` 对话将添加到堆栈作为新的默认对话。 如果这不是期望的行为，可将 `onSelectAction` 选项添加到 `triggerAction`。 `onSelectAction` 选项使机器人能够启动新的对话，而无需清除对话堆栈，这使聊天能够暂时重定向，并在稍后从中断的位置恢复。

以下代码示例演示如何通过 `triggerAction` 使用 `onSelectAction` 选项，以便将 `help` 对话添加到现有对话堆栈上（并且不会清除该对话堆栈）。

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

在此示例中，`help` 对话在用户输入“help”时控制聊天。 由于 `triggerAction` 包含 `onSelectAction` 选项，因此 `help` 对话将被推到现有对话堆栈上并且不会清除该堆栈。 `help` 对话结束时，将从对话堆栈中删除，并且聊天将从使用 `help` 命令中断的位置恢复。

### <a name="handle-contextual-action"></a>处理与上下文相关的操作

在上一示例中，用户在聊天中的任意位置输入“help”时都将调用 `help` 对话。 在这种情况下，对话中只提供一般帮助指南，因为没有任何针对用户帮助请求的特定上下文。 但是，如果用户想要请求有关聊天中的特定位置的帮助该怎么办？ 在这种情况下，必须在当前对话的上下文中触发 `help` 对话。

例如，请考虑预订晚餐的机器人。 当系统询问其聚会的成员数时，如果用户想要了解聚会的最大规模该怎么办？ 若要处理此情况，可向 `askForPartySize` 对话附加 `beginDialogAction`，用于侦听用户输入“help”。

下面的代码示例演示如何使用 `beginDialogAction` 向对话附加上下文。

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

在此示例中，每当用户输入“help”，机器人将向堆栈推送 `partySizeHelp` 对话。 该对话向用户发送帮助消息后结束，并且控制将返回到提示用户聚会规模的 `askForPartySize` 对话。

请务必注意，此上下文相关的帮助仅在用户位于 `askForPartySize` 对话中时执行。 否则，将改为执行 `triggerAction` 中的一般帮助消息。 换言之，本地 `matches` 子句始终优先于全局 `matches` 子句。 例如，如果 `beginDialogAction` 与 help 匹配，则不会执行 `triggerAction` 中的 help 匹配。 有关详细信息，请参阅[操作优先级](bot-builder-nodejs-dialog-actions.md#action-precedence)。

### <a name="change-the-topic-of-conversation"></a>更改聊天主题

默认情况下，执行 `triggerAction` 将清除对话堆栈并重置聊天（从指定的对话开始）。 如果机器人需要从一个聊天主题切换到另一个，例如用户在预订晚餐过程中改为点外卖送往他们的酒店房间，通常首选这种行为。 

下面的示例基于上一个示例，机器人允许用户进行晚餐预订或叫外卖。 在此机器人中，默认对话是一个问候语对话，向用户显示两个选项：`Dinner Reservation` 和 `Order Dinner`。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

前面的示例的默认对话中的晚餐预订逻辑现在位于其名为 `dinnerReservation` 的对话中。 `dinnerReservation` 的流程与前面所述的多个对话版本保持相同。 唯一的区别在于该对话附加了 `triggerAction`。 请注意，在此版本中，`confirmPrompt` 要求用户先确认他们希望更改聊天主题，然后再调用新对话。 在这样的场景中添加 `confirmPrompt` 是种很好的做法，因为清除对话堆栈后，应用将被定向到新的聊天主题，从而放弃正在进行的主题。

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

使用瀑布在 `orderDinner` 对话中定义第二个聊天主题。 此对话只显示晚餐菜单，并在指定订单后提示用户输入房间号。 将 `triggerAction` 附加到对话可指定在用户输入“order dinner”时进行调用，并确保如果用户指出想要更改聊天主题时，提示用户确认选择。

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

在用户启动聊天并选择 `Dinner Reservation` 或 `Order Dinner` 后，他们可能随时改变主意。 例如，如果用户正在预订晚餐并输入“点餐”，机器人将通过提示“这将取消您的当前请求。 是否确定？”进行确认。 如果用户键入“否”，将取消请求，且用户可继续晚餐预订过程。 如果用户键入“yes”，机器人将清除对话堆栈，并将聊天控制转移到 `orderDinner` 对话。

## <a name="end-conversation"></a>结束聊天

在上面的示例中，通过使用 `session.endDialog` 或 `session.endDialogWithResult` 结束对话，这两种都能结束对话，将其从堆栈中删除并将控制返回到调用的对话。 在用户已到达聊天末尾的情况下，应使用 `session.endConversation` 指示聊天已完成。

[`session.endConversation` ](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) 方法可结束聊天并根据需要向用户发送消息。 例如，上例中的 `orderDinner` 对话可使用 `session.endConversation` 结束聊天，如下面的示例代码中所示。

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

调用 `session.endConversation` 将通过清除对话堆栈并重置 [`session.conversationData`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata) 存储来结束聊天。 有关数据存储的详细信息，请参阅[管理状态数据](bot-builder-nodejs-state.md)。

用户完成为机器人设计的聊天流后，应调用 `session.endConversation`。 此外，还可使用 `session.endConversation` 在用户处于聊天中时输入“cancel”或“goodbye”的情况下结束聊天。 为此，只需将 `endConversationAction` 附加到此对话并使此触发器侦听匹配“取消”或“再见”的输入即可。

下面的代码示例演示当用户输入“cancel”或“goodbye”时，如何将 `endConversationAction` 附加到对话来结束聊天。

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

由于使用 `session.endConversation` 或 `endConversationAction` 结束聊天将清除对话堆栈并强制用户重新开始，因此应添加 `confirmPrompt` 以确保用户确实想要那么做。

## <a name="next-steps"></a>后续步骤

在本文中，你了解了如何管理本质上是连续的聊天。 如果想要重复某个对话，或在聊天中使用循环模式该怎么办？ 我们来通过替换堆栈上的对话了解如何实现该操作。

> [!div class="nextstepaction"]
> [替换对话](bot-builder-nodejs-dialog-replace.md)

