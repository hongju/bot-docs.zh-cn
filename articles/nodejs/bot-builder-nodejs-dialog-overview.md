---
title: 对话概述 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for Node.js 中的对话来为聊天建模和管理聊天流。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c92d68c429dc48722640f29032338528ac96e24c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297649"
---
# <a name="dialogs-in-the-bot-builder-sdk-for-nodejs"></a>Bot Builder SDK for Node.js 中的对话
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-dialogs.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-overview.md)

使用 Bot Builder SDK for Node.js 中的对话可为聊天建模和管理聊天流。 机器人通过聊天来与用户通信。 聊天组织成对话。 对话可以包含瀑布式步骤和提示。 当用户与机器人交互时，机器人会启动、停止和切换不同的对话，以便对用户消息做出响应。 若要成功设计和创建强大的机器人，了解对话的工作原理非常关键。 

本文介绍对话的概念。 阅读本文后，请单击[后续步骤](#next-steps)部分中的链接来深入探讨这些概念。

## <a name="conversations-through-dialogs"></a>通过对话展开的聊天

Bot Builder SDK for Node.js 将聊天定义为通过一个或多个对话在机器人与用户之间进行的通信。 从根本上讲，对话是一个执行操作或者从用户收集信息的可重用模块。 可以在可重用的对话代码中封装机器人的复杂逻辑。

可通过多种方式来构建和更改聊天：

- 它可以源自[默认对话](#default-dialog)。
- 它可以从一个对话重定向到另一个对话。
- 它可以恢复。
- 它可以遵循[瀑布](bot-builder-nodejs-dialog-waterfall.md)模式，引导用户完成一系列的步骤，或者通过一系列的问题来[提示](bot-builder-nodejs-dialog-prompt.md)用户。
- 它可以使用[操作](bot-builder-nodejs-dialog-actions.md)来收听触发不同对话的单词或短语。 

可将聊天视为对话的父级。 因此，聊天包含对话堆栈并维护自身的状态数据集，即 `conversationData` 和 `privateConversationData`。 另一方面，对话维护 `dialogData`。 有关状态数据的详细信息，请参阅[管理状态数据](bot-builder-nodejs-state.md)。

## <a name="dialog-stack"></a>对话堆栈

机器人通过对话堆栈中维护的一系列对话来与用户交互。 在聊天过程中，对话将会推入和弹出堆栈。 堆栈的工作方式类似于普通的 LIFO 堆栈，即，最后一个添加的对话是第一个要完成的对话。 某个对话完成后，控制权将返回到堆栈中的上一个对话。

当机器人首次启动或者某个聊天结束时，对话堆栈为空。 此时，如果某个用户向机器人发送消息，机器人将以默认对话做出响应。

## <a name="default-dialog"></a>默认对话

在 Bot Framework 版本 3.5 之前，需要通过添加名为 `/` 的对话来定义根对话，因此其命名约定类似于 URL。 此命名约定并不适合用于对话的命名。 

> [!NOTE]
> 从 Bot Framework 版本 3.5 开始，默认对话注册为 [`UniversalBot`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html#constructor) 构造函数中的第二个参数。  

以下代码片段演示如何在创建 `UniversalBot` 对象时定义默认对话。

```javascript
var bot = new builder.UniversalBot(connector, [
    //...Default dialog waterfall steps...
    ]);
```

每当对话堆栈为空并且未通过 LUIS 或其他[识别器](bot-builder-nodejs-recognize-intent-messages.md)[触发](bot-builder-nodejs-dialog-actions.md)其他对话时，默认对话将会运行。 由于默认对话是机器人向用户提供的第一个响应，因此，默认对话应向用户提供一些上下文信息，例如可用命令的列表，或机器人的功能概述。

## <a name="dialog-handlers"></a>对话处理程序

对话处理程序管理聊天流。 为了不断推进聊天，对话处理程序会通过启动和结束对话来定向流。 

## <a name="starting-and-ending-dialogs"></a>启动和结束对话

若要启动新对话（将其推送到堆栈），请使用 [`session.beginDialog()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#begindialog)。 若要结束对话（将其从堆栈中删除，并将控制权返回给调用方对话），请使用 [`session.endDialog()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialog) 或 [`session.endDialogWithResult()`](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#enddialogwithresult)。 

## <a name="using-waterfalls-and-prompts"></a>使用瀑布和提示

使用[瀑布](bot-builder-nodejs-dialog-waterfall.md)可以方便地建模和管理聊天流。 瀑布包含一系列步骤。 在每个步骤中，可以代表用户完成某个操作，或者[提示](bot-builder-nodejs-dialog-prompt.md)用户输入信息。

使用由函数集合构成的对话实现瀑布。 每个函数定义瀑布中的一个步骤。 以下代码示例演示了一个简单的聊天，该聊天使用双步骤瀑布来提示用户输入其姓名，然后使用该姓名来问候用户。

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

当机器人到达瀑布终点但未结束对话时，来自用户的下一条消息将在瀑布的第一个步骤处重启该对话。 这可能会让用户感到沮丧，因为他们觉得自己陷在一个循环中。 若要避免这种情况，当聊天或对话结束时，最好是显式调用 `endDialog`、`endDialogWithResult` 或 `endConversation`。

## <a name="next-steps"></a>后续步骤

若要更深入地了解对话，必须理解瀑布模式的工作原理，以及如何使用它来引导用户完成整个过程。

> [!div class="nextstepaction"]
> [使用瀑布定义聊天步骤](bot-builder-nodejs-dialog-waterfall.md)
