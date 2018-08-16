---
title: 使用 Bot Builder SDK for JavaScript 创建机器人 | Microsoft Docs
description: 使用 Bot Builder SDK for JavaScript 快速创建机器人。
keywords: 快速入门, bot builder sdk, 入门
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 337b6a0b8739b5e5de4d2d1b2b87dcad55f2c854
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352926"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-preview-for-javascript"></a>使用 Bot Builder SDK v4 for JavaScript（预览版）创建机器人
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入门将介绍如何使用 Yeoman Bot Builder 生成器和 Bot Builder SDK for JavaScript 构建机器人，然后通过 Bot Framework 模拟器进行测试。 这基于 [Microsoft Bot Builder SDK v4](https://github.com/Microsoft/botbuilder-js)。

## <a name="pre-requisites"></a>先决条件
- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/en/)
- [Yeoman](http://yeoman.io/)，可以使用生成器为你创建机器人
- [Bot Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- 了解 [restify](http://restify.com/) 和 Java 中的异步编程

> [!NOTE]
> 对某些安装，restify 安装步骤是提供与 node-gyp 相关的错误。
> 如果是这种情况，则尝试运行 `npm install -g windows-build-tools`。


Bot Builder SDK for JavaScript 包含一系列[包](https://github.com/Microsoft/botbuilder-js/tree/master/libraries)，可使用特殊的 `@preview` 标记从 NPM 安装。

# <a name="create-a-bot"></a>创建机器人

打开提升的命令提示符，创建一个目录，并初始化机器人相应的包。

```bash
md myJsBots
cd myJsBots
```

接下来，安装 Yeoman 和 JavaScript 生成器。

```bash
npm install -g yo
npm install -g generator-botbuilder@preview
```

然后，使用生成器创建 echo 机器人。

```bash
yo botbuilder
```

Yeoman 会提示你输入用来创建机器人的一些信息。
-   为机器人输入名称。
-   输入说明。
-   选择机器人语言，`JavaScript` 或 `TypeScript`。
-   选择要使用的模板。 目前，`Echo` 是唯一模板，但将很快添加其他模板。

Yeoman 在新文件夹中创建机器人。

## <a name="explore-code"></a>浏览代码

当打开新创建的机器人文件夹时，将看到 `app.js` 文件。 此 `app.js` 文件将包含运行机器人应用所需的所有代码。 此文件包含一个回显机器人，它会返回所输入的任何内容，并使计数器递增。 

在下面的代码中，对话状态中间件使用内存中存储。 它读取和写入要存储的状态对象。 计数变量会一直跟踪发送到机器人的消息数。 可以使用类似的技术来维护启用之间的状态。 

**app.js**
```javascript
// Packages are installed for you
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

以下代码侦听传入请求并检查传入活动类型，然后向用户发送回复。

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {
        
            // Get the conversation state
            const state = conversationState.get(context);
            
            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            
            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

## <a name="start-your-bot"></a>启动机器人

将目录更改为为机器人创建的目录，并启动它。

```bash
cd <bot directory>
node app.js
```

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
此时，机器人在本地运行。 接下来，启动模拟器，然后在模拟器中连接到机器人：
1. 单击模拟器“欢迎使用”选项卡中的“新建机器人配置”链接。 

2. 输入机器人名称并输入机器人代码的目录路径。 机器人配置文件将保存到此路径。

3. 在“终结点 URL”字段中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。

4. 单击“连接”以连接到机器人。 无需指定 **Microsoft 应用 ID** 和 **Microsoft 应用密码**。 可以暂时将这些字段留空。 以后当你注册机器人时，将获得此信息。

向机器人发送“Hi”，机器人将使用“0：您说了’Hi’”来响应消息。

## <a name="next-steps"></a>后续步骤

接下来，跳转到解释机器人及其工作原理的概念。

> [!div class="nextstepaction"]
> [基本机器人概念](../v4sdk/bot-builder-basics.md)
