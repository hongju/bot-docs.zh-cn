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
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9cfe077c8d8573145625b211c3c1ca05a6a21e19
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224812"
---
# <a name="send-and-receive-text-message"></a>发送和接收文本消息 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人与用户通信以及接收通信等的主要方法是通过消息活动。 某些消息可能会只包含纯文本，而其他消息可能包含更丰富的内容，如卡或附件。 机器人的轮次处理程序从用户那里接收消息，然后你可以向用户发送响应。 轮次上下文对象提供用于将消息发送回用户的方法。 本文介绍了如何发送简单的文本消息。

## <a name="send-a-text-message"></a>发送文本消息

要发送简单文本消息，请指定要作为活动发送的字符串：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在机器人的 `OnTurnAsync` 方法中，使用轮次上下文对象的 `SendActivityAsync` 方法发送单个消息响应。 还可以使用该对象的 `SendActivitiesAsync` 方法一次性发送多个响应。

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的 `onTurn` 处理程序中，使用轮次上下文对象的 `sendActivity` 方法发送单个消息响应。 还可以使用该对象的 `sendActivities` 方法一次性发送多个响应。

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>接收短信

若要接收简单的文本消息，请使用 *activity* 对象的 *text* 属性。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在机器人的 `OnTurnAsync` 方法中，使用以下代码来接收消息。 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在机器人的 `OnTurnAsync` 方法中，使用以下代码来接收消息。 
```javascript
let text = turnContext.activity.text;
```
---


## <a name="additional-resources"></a>其他资源
有关一般情况下的活动处理的详细信息，请参阅[活动处理](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack)。 若要发送更丰富的内容，请参阅如何添加[富媒体](bot-builder-howto-add-media-attachments.md)附件。
