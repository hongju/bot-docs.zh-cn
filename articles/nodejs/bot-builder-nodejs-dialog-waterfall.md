---
title: 使用瀑布图定义会话步骤 | Microsoft Docs
description: 了解如何使用瀑布图来定义使用 Bot Framework SDK for Node.js 的聊天的步骤。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 526091d61f10ac0c241b994aa3ea99c1d2a70074
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225322"
---
# <a name="define-conversation-steps-with-waterfalls"></a>使用瀑布图定义会话步骤

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

会话是用户和机器人之间交换的一系列消息。 当机器人的目的是引导用户完成一系列步骤时，可以使用瀑布图来定义会话步骤。

瀑布图是特定[对话框](bot-builder-nodejs-dialog-overview.md)实现，最常用于从用户那里收集信息或指导用户完成一系列任务。 任务作为一组函数实现，其中第一个函数的结果作为输入传递到下一个函数，依次类推。 每个函数通常表示整个过程中的一步。 在每个步骤中，机器人会提示用户输入，等待响应，然后将结果传递到下一步。

本文将帮助你了解瀑布图的工作原理以及如何使用它来[管理会话流](bot-builder-nodejs-dialog-manage-conversation.md)。

## <a name="conversation-steps"></a>会话步骤

会话通常涉及用到户与机器人之间的多个提示/响应交换。 每个提示/响应交换表示会话迈进了一步。 可以使用只有两个步骤的瀑布图来创建会话。

例如，考虑以下 `greetings` 对话框。 瀑布图的第一步是提示用户名，第二步使用响应按名称问候用户。

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

可以实现此做法的是使用提示。 Bot Framework SDK for Node.js 提供多种不同类型的内置[提示](bot-builder-nodejs-dialog-prompt.md)，可用于要求用户提供不同类型的信息。

下面的示例代码显示了一个对话框，该对话框通过 4 步骤瀑布图使用提示从用户那里收集各种信息。

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
        builder.Prompts.text(session, "How many people are in your party?");
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

在此示例中，默认对话框具有四个函数，每个函数代表瀑布图中的一个步骤。 每个步骤会提示用户输入，并将结果发送到下一个要处理的步骤。 此过程将继续，直到执行最后一个步骤，从而确认预订和结束对话框。

以下屏幕截图显示在 [Bot Framework Emulator](~/bot-service-debug-emulator.md) 中运行的机器人的结果。

![使用瀑布图管理会话流](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>使用多个瀑布图创建会话

可以在会话中使用多个瀑布图来定义机器人所需的任何会话结构。 例如，可以使用一个瀑布图来管理会话流，并使用其他瀑布图从用户那里收集信息。 每个瀑布图均封装在一个对话框中，可以通过调用 `session.beginDialog` 函数进行调用。

下面的代码示例演示如何在一个会话中使用多个瀑布图。 `greetings` 对话框中的瀑布图管理会话流，而 `askName` 对话框中的瀑布图从用户那里收集信息。

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>推进瀑布图

瀑布图按照函数在数组中定义的顺序逐步推进。 瀑布图中的第一个函数可以接收传递给对话框的参数。 瀑布图内的任何函数都可以使用 `next` 函数前进到下一步，而无需提示用户输入。

下面的代码示例演示如何使用对话框中的 `next` 函数，指导用户完成提供其用户配置文件信息的过程。 在瀑布图的每个步骤中，机器人提示用户输入一条信息（如有必要），并由瀑布图的后续步骤处理用户响应（如果有的话）。 `ensureProfile` 对话框中瀑布图的最后一步结束该对话框，并将已完成的配置文件信息返回到调用对话框，后者随后向用户发送个性化问候。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> 此示例使用两个不同的数据包来存储数据：`dialogData` 和 `userData`。 如果机器人分布在多个计算节点上，瀑布图的每个步骤可以由另一个节点处理。 有关存储机器人数据的详细信息，请参阅[管理状态数据](bot-builder-nodejs-state.md)。

## <a name="end-a-waterfall"></a>结束瀑布图

使用瀑布图创建的对话框必须显式结束，否则机器人将无限期地重复该瀑布图。 可使用下述方法之一来结束瀑布图：

* `session.endDialog`：如果没有数据返回到调用对话框，则使用此方法来结束瀑布图。

* `session.endDialogWithResult`：如果有数据返回到调用对话框，则使用此方法来结束瀑布图。 返回的 `response` 参数可以是 JSON 对象或任何 JavaScript 基元数据类型。 例如：
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation`：如果瀑布图结束表示聊天结束，则使用此方法结束瀑布图。

作为使用这三种方法之一来结束瀑布图的替代方法，可以将 `endConversationAction` 触发器附加到对话框。 例如：

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>后续步骤

通过瀑布图，可以使用提示从用户那里收集信息。 让我们深入了解如何提示用户输入。

> [!div class="nextstepaction"]
> [提示用户输入](bot-builder-nodejs-dialog-prompt.md)
