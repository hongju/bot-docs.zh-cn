---
title: 发送主动消息 | Microsoft Docs
description: 了解如何通过 Bot Builder SDK for Node.js 使用主动消息中断当前聊天流
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4ca33d59c967bc4eebc2f88fa4ddd67a9a6af6d7
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997146"
---
# <a name="send-proactive-messages"></a>发送主动消息
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>主动消息的类型

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>发送临时主动消息

下面的代码示例演示如何通过使用 Bot Builder SDK for Node.js 发送临时主动消息。

为了能够向用户发送临时消息，机器人必须先从当前聊天中收集并保存有关用户的信息。 消息的地址属性包括机器人稍后向用户发送临时消息所需的所有信息。 

```javascript
bot.dialog('adhocDialog', function(session, args) {
    var savedAddress = session.message.address;

    // (Save this information somewhere that it can be accessed later, such as in a database, or session.userData)
    session.userData.savedAddress = savedAddress;

    var message = 'Hello user, good to meet you! I now know your address and can send you notifications in the future.';
    session.send(message);
})
```

> [!NOTE]
> 只要机器人稍后能够访问用户数据，便可以任何方式将其存储。

机器人收集有关用户的信息后，它可以随时向用户发送临时主动消息。 要执行此操作，只需检索先前存储的用户数据、构造消息并发出消息。

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

var bot = new builder.UniversalBot(connector)
                .set('storage', inMemoryStorage); // Register in-memory storage 

function sendProactiveMessage(address) {
    var msg = new builder.Message().address(address);
    msg.text('Hello, this is a notification');
    msg.textLocale('en-US');
    bot.send(msg);
}
```

> [!TIP]
> 可以从异步触发器发起临时主动消息，如 http 请求、计时器、队列或开发者选择的任何其他位置。

## <a name="send-a-dialog-based-proactive-message"></a>发送基于对话的主动消息

下面的代码示例演示如何通过使用 Bot Builder SDK for Node.js 发送基于对话的主动消息。 可以在 [Microsoft/BotBuilder-Samples/Node/core-proactiveMessages/startNewDialog](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog) 文件夹中找到完整的有效示例。

为了能够向用户发送基于对话的消息，机器人必须先从当前聊天中收集（并保存）信息。 `session.message.address` 对象包含机器人向用户发送基于对话的主动消息所需的所有信息。 

```javascript
// proactiveDialog dialog
bot.dialog('proactiveDialog', function (session, args) {

    savedAddress = session.message.address;

    var message = 'Hey there, I\'m going to interrupt our conversation and start a survey in five seconds...';
    session.send(message);

    message = `You can also make me send a message by accessing: http://localhost:${server.address().port}/api/CustomWebApi`;
    session.send(message);

    setTimeout(() => {
        startProactiveDialog(savedAddress);
    }, 5000);
});
```

需要发送消息时，机器人将创建一个新的对话，并将其添加到对话堆栈的顶部。 新的对话将成为主角，传递主动消息，然后关闭，再将控制权返回给堆栈中的上一对话。 

```javascript
// initiate a dialog proactively 
function startProactiveDialog(address) {
    bot.beginDialog(address, "*:survey");
}
```

> [!NOTE]
> 上面的代码示例需要自定义文件  **botadapter.js**，可[从 GitHub 下载](https://github.com/Microsoft/BotBuilder-Samples/blob/master/Node/core-proactiveMessages/startNewDialog/botadapter.js)该文件。

调查对话会控制聊天，直至结束。 然后，它将关闭（通过调用 `session.endDialog()`），从而将控制权返回给上一对话。 


```javascript
// handle the proactive initiated dialog
bot.dialog('survey', function (session, args, next) {
  if (session.message.text === "done") {
    session.send("Great, back to the original conversation");
    session.endDialog();
  } else {
    session.send('Hello, I\'m the survey dialog. I\'m interrupting your conversation to ask you a question. Type "done" to resume');
  }
});
```

## <a name="sample-code"></a>代码示例

有关演示如何通过使用 Bot Builder SDK for Node.js 发送主动消息的完整示例，请参阅 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages" target="_blank">主动消息示例</a>。 在主动消息示例中，<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> 演示了如何发送临时主动消息，而 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog" target="_blank">startNewDialog</a> 演示了如何发送基于对话的主动消息。

## <a name="additional-resources"></a>其他资源

- [设计聊天流](../bot-service-design-conversation-flow.md)
