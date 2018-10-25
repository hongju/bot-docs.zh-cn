---
title: 创建消息 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for Node.js 创建消息。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/7/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8011611aa11e81cf322ba841f616fa2797038e84
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998164"
---
# <a name="create-messages"></a>创建消息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

机器人和用户之间通过消息进行通信。 机器人将发送消息活动向用户传递信息，同样，也将收到来自用户的消息活动。 某些消息可能只包含纯文本，而其他消息可能包含更丰富的内容，例如语音文本、建议的操作、媒体附件、富卡和特定于通道的数据。

本文介绍了一些常用的消息方法，可用于增强用户体验。

## <a name="default-message-handler"></a>默认消息处理程序

Bot Builder SDK for Node.js 附带内置于 [`session`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html) 对象的默认消息处理程序。 此消息处理程序允许在机器人和用户之间发送和接收文本消息。

### <a name="send-a-text-message"></a>发送文本消息

使用默认消息处理程序发送文本消息很简单，只需调用 `session.send` 并 传入字符串。

此示例演示如何发送短信来问候用户。
```javascript
session.send("Good morning.");
```

此示例演示如何使用 JavaScript 字符串模板发送短信。
```javascript
var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
session.send(msg); //msg: "You ordered: Potato Salad for a total of $5.99."
```

### <a name="receive-a-text-message"></a>接收短信

当用户向机器人发送消息时，机器人会通过 `session.message` 属性接收消息。

此示例演示如何访问用户消息。
```javascript
var userMessage = session.message.text;
```

## <a name="customizing-a-message"></a>自定义消息

若要更好地控制消息的文本格式，可以创建自定义 [`message`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html) 对象并设置必要属性，然后将其发送给用户。

此示例演示如何创建自定义 `message` 对象并设置 `text`、`textFormat` 和 `textLocale` 属性。

```javascript
var customMessage = new builder.Message(session)
    .text("Hello!")
    .textFormat("plain")
    .textLocale("en-us");
session.send(customMessage);
```

如果在范围内没有 `session` 对象，可以使用 `bot.send` 方法向用户发送带格式的消息。

消息的 `textFormat` 属性可用于指定文本格式。 `textFormat` 属性可以设置为“纯本文”、“markdown”或“xml”。 `textFormat` 的默认值为“markdown”。 

## <a name="message-property"></a>消息属性

[`Message`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html) 对象内部有一个 data 属性，可用于管理要发送的消息。 设置的其他属性都是通过此对象向你公开的不同方法。 

## <a name="message-methods"></a>消息方法

可通过对象方法设置和检索消息属性。 下表提供可以调用的方法列表，以设置/获取不同的消息属性。

| 方法 | Description |
| ---- | ---- | 
| [`addAttachment(attachment:AttachmentType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addattachment) | 向消息添加附件|
| [`addEntity(obj:Object)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addentity) | 将实体添加到消息。 |
| [`address(adr:IAddress)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#address) | 消息的地址路由信息。 若要向用户发送主动消息，请在 userData 包中保存消息地址。 |
| [`attachmentLayout(style:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachmentlayout) | 提示客户端应如何布局多个附件。 默认值为“list”。 |
| [`attachments(list:AttachmentType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) | 发送给用户的卡或图像列表。 |
| [`compose(prompts:string[], ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#compose) | 对用户进行复杂随机回复。 |
| [`entities(list:Object[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#entities) | 传递给机器人或用户的结构化对象。 |
| [`inputHint(hint:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#inputhint) | 发送给用户让他们知道机器人是否期望进一步输入的提示。 内置提示会自动为传出消息填充此值。 |
| [`localTimeStamp((optional)time:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#localtimestamp) | 发送消息时的本地时间（由客户端或机器人设置，例如：2016-09-23T13:07:49.4714686-07:00。） |
| [`originalEvent(event:any)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#originalevent) | 传入消息的通道的原始/本机格式消息。 |
| [`sourceEvent(map:ISourceEventMap)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#sourceevent) | 对于传出消息，可用来传递特定于源的事件数据，如自定义附件。 |
| [`speak(ssml:TextType, ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#speak) | 将消息的“语音”字段设置为“语音合成标记语言 (SSML)”。 将在支持设备上向用户陈述。 |
| [`suggestedActions(suggestions:ISuggestedActions `&#124;` IIsSuggestedActions)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#suggestedactions) | 向用户发送的可选建议操作。 将仅在支持建议操作的通道上显示建议操作。 |
| [`summary(text:TextType, ...argus:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#summary) | 显示为回退或消息内容简短说明的文本（例如：最新会话列表中的文本。） |
| [`text(text:TextType, ...args:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#text) | 设置消息文本。 |
| [`textFormat(style:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textformat) | 设置文本格式。 默认格式为“markdown”。 |
| [`textLocale(locale:string)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textlocale) | 设置消息的目标语言。 |
| [`toMessage()`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#tomessage) | 获取消息的 JSON。 |
| [`composePrompt(session:Session, prompts:string[], args?:any[])`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#composeprompt-1) | 将一组提示合并成单个本地化提示，然后根据需要使用传入的参数填充提示模板槽。 |
| [`randomPrompt(prompts:TextType)`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message.html#randomprompt) | 从传入的一组*提示中获取随机提示。 |

## <a name="next-step"></a>后续步骤

> [!div class="nextstepaction"]
> [发送和接收附件](bot-builder-nodejs-send-receive-attachments.md)

