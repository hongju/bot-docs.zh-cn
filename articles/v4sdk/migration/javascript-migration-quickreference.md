---
title: JavaScript 迁移（v3 到 v4）快速参考 | Microsoft Docs
description: v3 和 v4 JavaScript Bot Framework SDK 主要差异概述。
keywords: JavaScript, 机器人迁移, 对话, v3 机器人
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 82bc8ae5ca0e6787a4c730edebdb7ca193aa663d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693633"
---
# <a name="javascript-migration-quick-reference"></a>JavaScript 迁移快速参考

BotBuilder Javascript SDK v4 引入了多项基本变更，这些变更影响我们创作机器人的方式。 本指南的目的是为用户提供快速参考，让他们了解在 v3 和 v4 SDK 中完成某个任务有哪些常见差异。

- 在机器人和通道之间传递信息的方式已改变。
    在 v3 中，我们使用“连接器”  和“会话”  对象。
    在 v4 中，这些对象被“适配器”  和“轮次上下文”  对象替换。

- 另外，我们已将对话和机器人实例进一步分离。
    在 v3 中，对话直接注册到机器人的构造函数。
    在 v4 中，现在可以将对话作为参数传递到机器人实例中，以便更灵活地进行组合。

- 另外，v4 还提供了一个 `ActivityHandler` 类，用于自动处理不同类型的活动，例如消息、聊天更新和事件活动。   

这些改进导致在 Javascript 中开发机器人的语法发生了变化，尤其是在创建机器人对象、定义对话和编码事件处理逻辑方面发生了变化。

本主题的其余部分将 JavaScript Bot Framework SDK v3 中的构造与 v4 中的相应构造进行了对比。

## <a name="to-listen-for-incoming-requests"></a>侦听传入请求

### <a name="v3"></a>v3

```javascript
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', connector.listen());
```

### <a name="v4"></a>v4

```javascript
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        await bot.run(context);
    });
});
```

## <a name="to-create-a-bot-instance"></a>创建机器人实例

### <a name="v3"></a>v3

```javascript
var bot = new builder.UniversalBot(connector, [ ...DIALOG_STEPS ]);
```

### <a name="v4"></a>v4

```javascript
// Define the bot class as extending ActivityHandler.
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    // ...
}

// Instantiate a new bot instance.
const bot = new MyBot(conversationState, dialog);
```

## <a name="to-send-a-message-to-a-user"></a>向用户发送消息

### <a name="v3"></a>v3

```javascript
session.send('Hello and welcome to the help desk bot.');
```

### <a name="v4"></a>v4

```javascript
await context.sendActivity('Hello and welcome to the help desk bot.');
```

## <a name="to-define-a-default-dialog"></a>定义默认对话

### <a name="v3"></a>v3

```javascript
var bot = new builder.UniversalBot(connector, [
    // Add default dialog waterfall steps...
]);
```

### <a name="v4"></a>v4

```javascript
// In the main dialog class, define a run method.
async run(turnContext, accessor) {
    const dialogSet = new DialogSet(accessor);
    dialogSet.add(this);

    const dialogContext = await dialogSet.createContext(turnContext);
    const results = await dialogContext.continueDialog();
    if (results.status === DialogTurnStatus.empty) {
        await dialogContext.beginDialog(this.id);
    }
}

// Pass conversation state management and a main dialog objects to the bot (in index.js).
const bot = new MyBot(conversationState, dialog);

// Inside the bot's constructor, add the dialog as a member property and define a DialogState property accessor.
this.dialog = dialog;
this.dialogState = this.conversationState.createProperty('DialogState');

// Inside the bot's message handler, invoke the dialog's run method, passing in the turn context and DialogState accessor.
this.onMessage(async (context, next) => {
    // Run the Dialog with the new message Activity.
    await this.dialog.run(context, this.dialogState);
    await next();
});
```

## <a name="to-start-a-child-dialog"></a>启动子对话

子对话结束后，父对话会继续。

### <a name="v3"></a>v3

```javascript
session.beginDialog(DIALOG_ID);
```

### <a name="v4"></a>v4

```javascript
await context.beginDialog(DIALOG_ID);
```

## <a name="to-replace-a-dialog"></a>替换对话

新对话替换堆栈上的当前对话。

### <a name="v3"></a>v3

```javascript
session.replaceDialog(DIALOG_ID);
```

### <a name="v4"></a>v4

```javascript
await context.replaceDialog(DIALOG_ID);
```

## <a name="to-end-a-dialog"></a>结束对话：

### <a name="v3"></a>v3

```javascript
session.endDialog();
```

### <a name="v4"></a>v4

可以添加一个可选的返回值。

```javascript
await context.endDialog(returnValue);
```

## <a name="to-register-a-dialog-with-a-bot-instance"></a>将对话注册到机器人实例

### <a name="v3"></a>v3

```javascript
// Create the bot.
var bot = new builder.UniversalBot(connector, [
    // ...
]};

// Add the dialog to the bot.
bot.dialog('myDialog', require('./mydialog'));
```

### <a name="v4"></a>v4

```javascript
// In the main dialog class constructor.
this.addDialog(new MyDialog(DIALOG_ID));

// In the app entry point file (index.js)
const { MainDialog } = require('./dialogs/main');

// Create the base dialog and bot, passing the main dialog as an argument.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-prompt-a-user-for-input"></a>提示用户输入

### <a name="v3"></a>v3

```javascript
var builder = require('botbuilder');
builder.Prompts.text(session, 'Please enter your destination');
```

### <a name="v4"></a>v4

```javascript
const { TextPrompt } = require('botbuilder-dialogs');

// In the dialog's constructor, register the prompt.
this.addDialog(new TextPrompt('initialPrompt'));

// In the dialog step, invoke the prompt.
return await stepContext.prompt(
    'initialPrompt', {
        prompt: 'Please enter your destination'
    }
);
```

## <a name="to-retrieve-the-users-response-to-a-prompt"></a>检索用户对提示的响应

### <a name="v3"></a>v3

```javascript
function (session, result) {
    var destination = result.response;
    // ...
}
```

### <a name="v4"></a>v4

```javascript
// In the dialog step after the prompt step, retrieve the result from the waterfall step context.
async nextStep(stepContext) {
    const destination = stepContext.result;
    // ...
}
```

## <a name="to-save-information-to-dialog-state"></a>将信息保存到对话状态

### <a name="v3"></a>v3

```javascript
session.dialogData.destination = results.response;
```

### <a name="v4"></a>v4

```javascript
// In a dialog, set the value in the waterfall step context.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>将状态更改写入持久层

### <a name="v3"></a>v3

```javascript
// State data is saved by default at the end of the turn.
// Do this to save it manually.
session.save();
```

### <a name="v4"></a>v4

```javascript
// State changes are not saved automatically at the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>创建并注册状态存储

### <a name="v3"></a>v3

```javascript
var builder = require('botbuilder');

// Create conversation state with in-memory storage provider.
var inMemoryStorage = new builder.MemoryBotStorage();

// Create the base dialog and bot
var bot = new builder.UniversalBot(connector, [ /*...*/ ]).set('storage', inMemoryStorage);
```

### <a name="v4"></a>v4

```javascript
const { MemoryStorage } = require('botbuilder');

// Create the memory layer object.
const memoryStorage = new MemoryStorage();

// Create the conversation state management object, using the storage provider.
const conversationState = new ConversationState(memoryStorage);

// Create the base dialog and bot.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>捕获对话引发的错误

### <a name="v3"></a>v3

```javascript
// Set up the error handler.
session.on('error', function (err) {
    session.send('Failed with message:', err.message);
    session.endDialog();
});

// Throw an error.
session.error('An error has occurred.');
```

### <a name="v4"></a>v4

```javascript
// Set up the error handler, using the adapter.
adapter.onTurnError = async (context, error) => {
    const errorMsg = error.message

    // Clear out conversation state to avoid keeping the conversation in a corrupted bot state.
    await conversationState.delete(context);

    // Send a message to the user.
    await context.sendActivity(errorMsg);
};

// Throw an error.
async () => {
    throw new Error('An error has occurred.');
}
```

## <a name="to-register-activity-handlers"></a>注册活动处理程序

### <a name="v3"></a>v3

```javascript
bot.on('conversationUpdate', function (message) {
    // ...
}

bot.on('contactRelationUpdate', function (message) {
    // ...
}
```

### <a name="v4"></a>v4

```javascript
// In the bot's constructor, add the handlers.
this.onMessage(async (context, next) => {
    // ...
}
  
this.onDialog(async (context, next) => {
    // ...
}

this.onMembersAdded(async (context, next) => {
    // ...
}
```

## <a name="to-send-attachments"></a>发送附件

### <a name="v3"></a>v3

```javascript
var message = new builder.Message()
    .attachments(hotelHeroCards
    .attachmentLayout(builder.AttachmentLayout.carousel));
```

### <a name="v4"></a>v4

```javascript
await context.sendActivity({
    attachments: hotelHeroCards,
    attachmentLayout: AttachmentLayoutTypes.Carousel
});
```
