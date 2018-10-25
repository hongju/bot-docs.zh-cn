---
title: 处理用户操作 | Microsoft Docs
description: 了解如何通过让机器人能够使用 Bot Builder SDK for Node.js 侦听和处理包含特定关键字的用户输入来处理用户操作。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 26f6e9520fe5d2ebb83ceb4e6a497a35e9d2611f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999254"
---
# <a name="handle-user-actions"></a>处理用户操作

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-actions.md)

用户通常尝试通过使用“帮助”、“取消”或“重新开始”等关键字来访问机器人中的某些功能。 用户会在聊天中途，在机器人期望得到不同的答复时这样操作。 通过实现“操作”，可调整机器人，使其更优雅地处理此类请求。 处理程序将检查用户针对所指定的关键字（例如“帮助”、“取消”或“重新开始”）输入的内容并进行适当响应。 

![用户的说话方式](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>操作类型

下表列出了可附加到对话的操作类型。 每个操作名称的链接均会转到一个提供该操作详细信息的部分。

| 操作 | 范围 | Description |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | 全局 | 如果将操作绑定到对话，这将清除对话堆栈并将其自身推送到堆栈的底部。 使用 `onSelectAction` 选项替代此默认行为。 |
| [customAction](#bind-a-customaction) | 全局 | 将自定义操作绑定到可处理信息或执行操作但不影响对话堆栈的机器人。 使用 `onSelectAction` 选项自定义此操作的功能。 |
[beginDialogAction](#bind-a-begindialogaction) | 与上下文相关 | 将操作绑定到触发时启动另一对话的对话。 起始对话将推送到堆栈中并在结束时消失。 |
[reloadAction](#bind-a-reloadaction) | 与上下文相关 | 将操作绑定到触发时导致另一对话重载的对话。 可使用 `reloadAction` 来处理诸如“重新开始”之类的用户话语。 |
[cancelAction](#bind-a-cancelaction) | 与上下文相关 | 将操作绑定到触发时取消另一对话的对话。 可使用 `cancelAction` 来处理诸如“取消”或“没关系”之类的用户话语。 |
[endConversationAction](#bind-an-endconversationaction) | 与上下文相关 | 将操作绑定到触发时结束与用户聊天的对话。 可使用 `endConversationAction` 来处理诸如“再见”之类的用户话语。 |

## <a name="action-precedence"></a>操作优先顺序 

当机器人接收到用户的话语时，它会根据对话堆栈上所有已注册的操作来检查该话语。 从堆栈顶部开始匹配，再一直匹配到堆栈的底部。 如果没有匹配，则将根据所有全局操作的 `matches` 选项来检查该话语。

如果在不同的上下文中使用同一个命令，则操作优先顺序非常重要。 例如，可将机器人的“Help”命令作为常规帮助进行使用。 还可对每项任务使用“Help”，但这些帮助命令与每项任务的上下文相关。 要了解详细阐述此要点的作业示例，请参阅[答复用户输入的内容](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input)。

## <a name="bind-actions-to-dialog"></a>将操作绑定到对话

用户话语或按钮单击都可触发与[对话](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html)相关的操作。
如果指定了匹配，则操作将侦听用户，检查其是否说出触发操作的字词或短语。  `matches` 选项可采用正则表达式或[识别器][RecognizeIntent]的名称。
要将操作绑定到“按钮单击”，请使用 [CardAction.dialogAction()][CardAction] 触发该操作。

操作是可链接的，因此可根据需要将任意数量的操作绑定到对话中。

### <a name="bind-a-triggeraction"></a>绑定 triggerAction

要将 [triggerAction][triggerAction] 绑定到对话，请执行以下操作：

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

将 `triggerAction` 绑定到对话时会将其注册到机器人。 触发后，`triggerAction` 将清除对话堆栈并将已触发的对话推送到堆栈。 此操作最适用于切换[聊天主题](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation)，用户也可使用它来请求任意独立任务。 如果要替代此操作清除对话堆栈的行为，请向 `triggerAction` 添加 `onSelectAction` 选项。

下面的代码片段展示了如何在不清除对话堆栈的情况下从全局上下文中提供常规帮助。

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

在此情况下，`triggerAction` 附加到 `help` 对话本身（这与 `orderDinner` 对话不同）。 用户可使用 `onSelectAction` 选项在不清除对话堆栈的情况下启动此对话。 这使用户能够处理“帮助”、“相关内容”和“支持”等全局请求。请注意，`onSelectAction` 选项显式调用 `session.beginDialog` 方法来启动已触发的对话。 通过 `args.action` 提供已触发对话的 ID。 不要将对话 ID（例如“帮助”）手动编码到此方法中，否则可能收到运行时错误。 如果希望触发 `orderDinner` 任务本身的上下文帮助消息，请考虑将 `beginDialogAction` 附加到 `orderDinner` 对话中。

### <a name="bind-a-customaction"></a>绑定 customAction

与其他操作类型不同，`customAction` 未定义任何默认操作。 由用户定义此操作的用途。 使用 `customAction` 的优势在于可选择在不操作对话堆栈的情况下处理用户请求。 触发 `customAction` 时，`onSelectAction` 选项可处理请求，而不将新的对话推送到堆栈上。 操作完成后，控制将传递回堆栈顶部的对话，并且机器人可以继续操作。

可以使用 `customAction` 提供常规和快速操作请求，例如“现在外面的温度是多少？”、“现在巴黎是几点？”、“提醒我今天下午 5 点去买牛奶”等等。这些是机器人可在不操纵堆栈的情况下执行的常规操作。

与 `customAction` 的另外一个主要区别是它绑定到机器人而不是对话。

以下代码示例展示如何将 `customAction` 绑定到侦听“设置提醒”请求的 `bot`。

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>绑定 beginDialogAction

将 `beginDialogAction` 绑定到对话时会向对话注册操作。 触发时，此方法将启动另一个对话。 此操作的行为与调用 [beginDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog) 方法的行为类似。 新的对话已推送到对话堆栈顶部，因此它不自动结束当前任务。 新对话结束后，当前任务仍然继续。 

以下代码片段展示了如何将 [beginDialogAction][beginDialogAction] 绑定到对话。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

如果需要将其他参数传递到新的对话中，可向操作添加 [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) 选项。

通过上述示例，可以修改为接受通过 `dialogArgs` 传入的参数。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>绑定 reloadAction

将 `reloadAction` 绑定到对话时会将其注册到对话中。 如果将此操作绑定到对话，这会在触发操作时让对话从头开始。 触发此操作与调用 [replaceDialog](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog) 方法类似。 这对于实现处理用户话语（如“重新开始”）或创建[循环](bot-builder-nodejs-dialog-replace.md#repeat-an-action)的逻辑非常有帮助。

以下代码片段展示了如何将 [reloadAction][reloadAction] 绑定到对话。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

如果需要将其他参数传递到重载的对话中，可向操作添加 [`dialogArgs`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs) 选项。 此选项已传递到 `args` 参数中。 重写上述示例代码以接收重载操作的相关参数；此行为将如下所示：

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>绑定 cancelAction

绑定 `cancelAction` 会将其注册到对话。 触发后，此操作将突然结束对话。 对话结束后，父级对话将恢复，并有一个恢复的代码显示它处于 `canceled` 状态。 用户可使用此操作处理“没关系”或“取消”等话语。 如果需要以编程方式取消对话，请参阅[取消对话](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog)。 要详细了解已恢复的代码，请参阅[提示结果](bot-builder-nodejs-dialog-prompt.md#prompt-results)。 

以下代码片段展示了如何将 [cancelAction][cancelAction] 绑定到对话。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>绑定 endConversationAction

绑定 `endConversationAction` 会将其注册到对话。 触发后，此操作将结束与用户的聊天。 触发此操作与调用 [endConversation](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation) 方法类似。 聊天结束后，Bot Builder SDK for Node.js 将清除对话堆栈和持久化的状态数据。 要详细了解持久化的状态数据，请参阅[管理状态数据](bot-builder-nodejs-state.md)。

以下代码片段展示了如何将 [endConversationAction][endConversationAction] 绑定到对话。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>确认中断

其中大部分操作（若不是全部）中断的是常规的聊天流。 很多操作都可能中断，因此要谨慎处理。 例如，`triggerAction`、`cancelAction` 或 `endConversationAction` 将清除对话堆栈。 如果用户错误地触发了这些操作中的任何一个，他们将不得不重新开始任务。 要确保用户真的想触发这些操作，可向这些操作添加 `confirmPrompt` 选项。 `confirmPrompt` 将询问用户是否确定取消或结束当前任务。 它允许用户改变主意并继续操作。

以下代码片段显示了一个 [cancelAction][cancelAction]，它带有 [confirmPrompt](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt)，目的是确保用户真的想取消订单流程。

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

触发此操作后，它将询问用户“是否确定?” 要继续此操作，则用户将必须回答“是”；要取消此操作并继续原本的进程，则必须回答“否”。

## <a name="next-steps"></a>后续步骤

“操作”让你能够预测用户请求，同时让机器人能够优雅地处理这些请求。 其中很多操作都会中断当前聊天。 如果要使用户能够关闭和恢复聊天，需要在关闭之前保存用户状态。让我们更深入地了解如何保存用户状态和管理状态数据。

> [!div class="nextstepaction"]
> [管理状态数据](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction
