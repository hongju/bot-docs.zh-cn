---
title: 使用 Web 控件交换信息 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 在机器人和网页之间交换信息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 06fcdd26980b719e7f7db76bc585650176c1d84f
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225672"
---
# <a name="use-the-backchannel-mechanism"></a>使用反向通道机制

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to backchannel mechanism](../includes/snippet-backchannel.md)]

## <a name="walk-through"></a>演练

开源网上聊天控件使用名为 <a href="https://github.com/microsoft/botframework-DirectLinejs" target="_blank">DirectLineJS</a> 的 JavaScript 类访问 Direct Line API。 该控件可以创建自己的 Direct Line 实例，也可以与托管页面共享一个。 如果该控件与托管页面共享 Direct Line 实例，则该控件和该页都将能够发送和接收活动。 下图显示了使用开源网上（聊天）控件和 Direct Line API 支持机器人功能的网站的高级体系结构。 

![反向通道](../media/designing-bots/patterns/back-channel.png)

### <a name="sample-code"></a>代码示例 

在此示例中，机器人和网页将使用反向通道机制来交换对用户不可见的信息。 机器人将请求网页更改其背景颜色，并且当用户单击页面上的按钮时，网页将通知机器人。 

> [!NOTE]
> 本文中的代码片段源自<a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">反向通道示例</a>和<a href="https://github.com/ryanvolum/backChannelBot" target="_blank">反向通道机器人</a>。 

#### <a name="client-side-code"></a>客户端代码

首先，网页创建一个 **DirectLine** 对象。

```javascript
var botConnection = new BotChat.DirectLine(...);
```

然后，它在创建 WebChat 实例时共享 **DirectLine** 对象。

```javascript
BotChat.App({
    botConnection: botConnection,
    user: user,
    bot: bot
}, document.getElementById("BotChatGoesHere"));
```

当用户单击网页上的按钮时，该网页将发布“事件”类型的活动以通知机器人该按钮被单击。

```javascript
const postButtonMessage = () => {
    botConnection
        .postActivity({type: "event", value: "", from: {id: "me" }, name: "buttonClicked"})
        .subscribe(id => console.log("success"));
    }
```

> [!TIP]
> 使用属性 `name` 和 `value` 来传达机器人正确解释和/或响应事件可能需要的任何信息。 

最后，该网页还会侦听来自机器人的特定事件。
在此示例中，网页侦听 type="event" 且 name="changeBackground" 的活动。 当它收到此类活动时，会将网页的背景颜色更改为活动指定的 `value`。 

```javascript
botConnection.activity$
    .filter(activity => activity.type === "event" && activity.name === "changeBackground")
    .subscribe(activity => changeBackgroundColor(activity.value))
```

#### <a name="server-side-code"></a>服务器端代码

<a href="https://github.com/ryanvolum/backChannelBot" target="_blank">反向通道机器人</a>使用帮助程序函数创建事件。

```javascript
var bot = new builder.UniversalBot(connector, 
    function (session) {
        var reply = createEvent("changeBackground", session.message.text, session.message.address);
        session.endDialog(reply);
    }
);

const createEvent = (eventName, value, address) => {
    var msg = new builder.Message().address(address);
    msg.data.type = "event";
    msg.data.name = eventName;
    msg.data.value = value;
    return msg;
}
```

同样，机器人还会侦听来自客户端的事件。 在此示例中，如果机器人收到 `name="buttonClicked"` 的事件，它会向用户发送一条消息，说“我看到你单击了一个按钮”。

```javascript
bot.on("event", function (event) {
    var msg = new builder.Message().address(event.address);
    msg.data.textLocale = "en-us";
    if (event.name === "buttonClicked") {
        msg.data.text = "I see that you clicked a button.";
    }
    bot.send(msg);
})
```

## <a name="additional-resources"></a>其他资源

- [Direct Line API][directLineAPI]
- <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework 网上聊天控件</a>
- <a href="https://aka.ms/v3-js-backchannel-sample" target="_blank">反向通道示例</a>
- <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">反向通道机器人</a>

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
