---
title: 进行音频通话 | Microsoft Docs
description: 了解如何在使用 Node.js 的机器人中通过 Skype 进行音频通话
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fbfe65526335b7a8797ab229871472d540735e20
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297713"
---
# <a name="support-audio-calls-with-skype"></a>支持使用 Skype 进行音频通话

Skype 支持名为“呼叫机器人”的丰富功能。  启用时，用户可以向机器人发出语音呼叫，并使用交互式语音应答 (IVR) 与之交互。  Bot Builder for Node.js SDK 包括一个特殊的[呼叫 SDK][calling_sdk]，可供开发人员用于将呼叫功能添加到聊天机器人。   

呼叫 SDK 与[聊天 SDK][chat_sdk] 非常相似。 它们具有相似的类，共享常见构造，甚至可以使用聊天 SDK 向与之通话的用户发送消息。  这两个 SDK 旨在并行运行，但它们又很相似，因此尽管有一些重要差异，通常应避免混合使用这两个库中的类。  

## <a name="create-a-calling-bot"></a>创建呼叫机器人
下面的代码示例显示呼叫机器人的“Hello World”与常规聊天机器人的相似性。 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> 若要查找机器人的“AppID”和“AppPassword”，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

模拟器目前不支持测试呼叫机器人。 若要测试机器人，需要完成发布机器人所需的大部分步骤。  此外需要使用 Skype 客户端与机器人进行交互。 

### <a name="enable-the-skype-channel"></a>启用 Skype 通道
[注册机器人](../bot-service-quickstart-registration.md)并启用 Skype 通道。 注册机器人时，需提供消息终结点。 建议将呼叫机器人与聊天机器人配对，以便聊天机器人的终结点就是通常置于该字段中的终结点。  如果仅注册呼叫机器人，只需将呼叫终结点粘贴到该字段。  

若要启用实际呼叫功能，需要进入机器人的 Skype 通道，并打开呼叫功能。 然后，将为你提供一个字段，以将呼叫终结点复制到其中。 确保将 https ngrok 链接用于呼叫终结点的主机部分。

在机器人注册过程中，将向你分配一个应用 ID 和密码，应将其粘贴到 hello world 机器人的连接器设置中。 还需要使用完整的呼叫链接，并将其粘贴到 callbackUrl 设置中。

### <a name="add-bot-to-contacts"></a>将机器人添加到联系人
在开发人员门户的机器人注册页，可以看到机器人 Skype 通道旁边的“添加到 Skype”按钮。 单击该按钮以将机器人添加到 Skype 中的联系人列表。  这样做之后，你（以及为其提供联接链接的任何人）将能够与机器人进行通信。

### <a name="test-your-bot"></a>测试机器人
可以使用 Skype 客户端测试机器人。 单击机器人联系人条目时（可能需要搜索机器人才能看到），会注意到高亮显示的呼叫图标。如果你已向现有机器人添加呼叫，呼叫图标亮起可能需要几分钟的时间。  

如果按呼叫按钮，它应拨号给机器人，你应会听到“Watson... come here!” 然后挂断。

## <a name="calling-basics"></a>呼叫基础知识
[UniversalCallBot](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) 和 [CallConnector](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) 类让你可以像创建聊天机器人一样创建呼叫机器人。 将实质上与[聊天对话框](bot-builder-nodejs-manage-conversation-flow.md)相同的对话框添加到机器人。 可以将[瀑布图](bot-builder-nodejs-prompts.md)添加到机器人。 会话对象 [CallSession](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession) 类包含添加的 [answer()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer)、[hangup()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup) 和 [reject()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) 方法，用于管理当前呼叫。 一般情况下，无需担心这些呼叫控制方法，因为 CallSession 具有自动管理呼叫的逻辑。 如果采取了发送消息或调用内置提示的操作，会话将自动响应呼叫。 如果你调用 [endConversation()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation) 或者它检测到你已停止询问呼叫方问题（未调用内置提示），它也会自动挂断/拒绝呼叫。

呼叫机器人和聊天机器人之间的另一个区别是，虽然聊天机器人通常向用户发送消息、卡和键盘，但由呼叫机器人处理操作和结果。 需要使用 Skype 呼叫机器人来创建由一个或多个[操作](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction)组成的[工作流](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow)。  这是另一件在实践中不必太过担心的事，因为 Bot Builder 呼叫 SDK 将为你管理大部分内容。 可使用 [CallSession.send()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) 方法传递操作或字符串，它会将其转变为 [PlayPromptActions](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction)。  此会话包含自动批处理逻辑，将多个操作合并到提交给呼叫服务的单个工作流，以便可以多次安全地调用 send()。  在处理所有结果时，应依赖于 SDK 的内置[提示](bot-builder-nodejs-prompts.md)来收集用户输入。  

[calling_sdk]: http://docs.botframework.com/en-us/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_