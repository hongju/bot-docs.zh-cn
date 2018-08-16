---
title: 截获消息 | Microsoft Docs
description: 了解如何通过使用 Bot Builder SDK for Node.js 来截获和处理信息交换，以创建日志或其他记录。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9160e81f9086f88f808b41e8eb01745776e0fc9f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297962"
---
# <a name="intercept-messages"></a>截获消息
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>示例

以下代码示例演示了如何使用 Bot Builder SDK for Node.js 中的中间件的概念来截获用户和机器人之间交换的消息。 

首先，为传入消息 (`botbuilder`) 和传出消息 (`send`) 配置处理程序。

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

然后，实施每个处理程序，为要截获的每条消息定义要执行的操作。

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

现在，每条入站消息（从用户到机器人）都将触发 `logIncomingMessage`，每条出站消息（从机器人到用户）都将触发 `logOutgoingMessage`。
在此示例中，机器人只是打印了有关每条消息的一些信息，但你可以根据需要更新 `logIncomingMessage` 和 `logOutgoingMessage`，以定义想要为每条消息执行的操作。 

## <a name="sample-code"></a>代码示例

有关演示如何使用 Bot Builder SDK for Node.j 来截获和记录消息的完整示例，请参阅 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/capability-middlewareLogging" target="_blank">中间件和日志记录示例</a>。
