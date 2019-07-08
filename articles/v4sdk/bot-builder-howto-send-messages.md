---
title: 发送和接收文本消息 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中发送和接收文本消息。
keywords: 发送消息, 消息活动, 简单的文本消息, 消息, 文本消息, 接收消息
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a72c103204384188d509777639c2c90e63431dd8
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496694"
---
# <a name="send-and-receive-text-message"></a>发送和接收文本消息

[!INCLUDE[applies-to](../includes/applies-to.md)]

机器人与用户通信以及接收通信等的主要方法是通过消息  活动。 某些消息可能会只包含纯文本，而其他消息可能包含更丰富的内容，如卡或附件。 机器人的轮次处理程序从用户那里接收消息，然后你可以向用户发送响应。 轮次上下文对象提供用于将消息发送回用户的方法。 本文介绍了如何发送简单的文本消息。

大多数文本字段支持 Markdown，但支持可能因通道而异。

有关正在运行的机器人如何发送和接收消息，请遵循目录顶部的快速入门，或查看[机器人工作原理](bot-builder-basics.md#bot-structure)一文，该文章所提供的简单示例链接，可帮助你自行运行。

## <a name="send-a-text-message"></a>发送文本消息

要发送简单文本消息，请指定要作为活动发送的字符串：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在机器人的活动处理程序中，使用轮次上下文对象的 `SendActivityAsync` 方法发送单个消息响应。 还可以使用该对象的 `SendActivitiesAsync` 方法一次性发送多个响应。

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的活动处理程序中，使用轮次上下文对象的 `sendActivity` 方法发送单个消息响应。 还可以使用该对象的 `sendActivities` 方法一次性发送多个响应。

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>接收短信

若要接收简单的文本消息，请使用 *activity* 对象的 *text* 属性。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在机器人的活动处理程序中，使用以下代码来接收消息。 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的活动处理程序中，使用以下代码来接收消息。

```javascript
let text = turnContext.activity.text;
```

---

## <a name="additional-resources"></a>其他资源

- 有关一般情况下的活动处理的详细信息，请参阅[活动处理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)。
- 有关格式设置的详细信息，请参阅“Bot Framework 活动架构”的[“消息活动”部分](https://aka.ms/botSpecs-activitySchema#message-activity)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [向消息添加媒体](./bot-builder-howto-add-media-attachments.md)
