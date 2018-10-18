---
title: 发送消息 | Microsoft Docs
description: 了解如何在 Bot Builder SDK 中发送消息。
keywords: 发送消息, 消息活动, 简单文本消息, 语音, 语音消息
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5b7faaae63bdc084dac570cb33ebbc755ccbcc19
ms.sourcegitcommit: aef7d80ceb9c3ec1cfb40131709a714c42960965
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/17/2018
ms.locfileid: "49383112"
---
# <a name="send-text-and-spoken-messages"></a>发送文本和语音消息

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人与用户通信以及接收通信等的主要方法是通过消息活动。 某些消息可能会只包含纯文本，而其他消息可能包含更丰富的内容，如卡或附件。 机器人的轮次处理程序从用户那里接收消息，然后你可以向用户发送响应。 轮次上下文对象提供用于将消息发送回用户的方法。 有关一般情况下活动处理的更多信息，请参阅[活动处理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)。

本文介绍如何发送简单的文本和语音消息。 若要发送更丰富的内容，请参阅如何[添加丰富的媒体附件](bot-builder-howto-add-media-attachments.md)。 有关如何使用提示对象的信息，请参阅如何[提示用户输入](bot-builder-prompts.md)。

## <a name="send-a-simple-text-message"></a>发送简单文本消息

要发送简单文本消息，请指定要作为活动发送的字符串：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在机器人的 OnTurn 方法中，使用轮次上下文对象的 SendActivity 方法发送单个消息响应。 此外可以使用该对象的 SendActivities 方法一次性发送多个响应。

```cs
await context.SendActivity("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的轮次处理程序中，使用轮次上下文对象的 sendActivity 方法发送单个消息响应。 此外可以使用该对象的 sendActivities 方法一次性发送多个响应。

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>发送语音消息

某些通道支持启用了语音的机器人，使它们能够与用户对话。 消息可以同时有写入和语音内容。

> [!NOTE]
> 对于那些不支持语音的通道，将忽略语音内容。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用可选语音参数来提供作为响应语音部分的文本。

```cs
await context.SendActivity(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要添加语音，需要使用 `Microsoft.Bot.Builder.MessageFactory` 生成消息。 `MessageFactory` 更多地用于[丰富的媒体](bot-builder-howto-add-media-attachments.md)，其中给出了更多解释，但现在我们只需在此处使用它。

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---
