---
title: 使用按钮进行输入 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for JavaScript 在消息中发送建议的操作。
keywords: 建议的操作, 按钮, 额外输入
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 10b9fa9664e8c18cdc5dcd2fcf3ae400296a4abb
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451979"
---
# <a name="use-button-for-input"></a>使用按钮进行输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

你可以使机器人能够显示按钮供用户点击来提供输入。 按钮改进了用户体验，因为用户只需点击按钮便可回答问题或进行选择，而不必使用键盘输入响应。 与资讯卡中显示的按钮（即使在点击后仍然可见且可供用户访问）不同，建议的操作窗格中显示的按钮将在用户进行选择后消失。 这可以防止用户在会话中点击过时按钮并简化机器人开发（因为将不需要说明该场景）。 

## <a name="suggest-action-using-button"></a>使用按钮提供操作建议

*建议的操作*让机器人能够显示按钮。 可以创建一个建议的操作列表（也称为“快速回复”），该列表将作为单轮聊天显示给用户： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

可以从 [GitHub](https://aka.ms/SuggestedActionsCSharp) 访问此处使用的源代码

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply("What is your favorite color?");

reply.SuggestedActions = new SuggestedActions()
{
    Actions = new List<CardAction>()
    {
        new CardAction() { Title = "Red", Type = ActionTypes.ImBack, Value = "Red" },
        new CardAction() { Title = "Yellow", Type = ActionTypes.ImBack, Value = "Yellow" },
        new CardAction() { Title = "Blue", Type = ActionTypes.ImBack, Value = "Blue" },
    },

};
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
可以从 [GitHub](https://aka.ms/SuggestActionsJS) 访问此处使用的源代码。

```javascript
const { ActivityTypes, MessageFactory, TurnContext } = require('botbuilder');

async sendSuggestedActions(turnContext) {
    var reply = MessageFactory.suggestedActions(['Red', 'Yellow', 'Blue'], 'What is the best color?');
    await turnContext.sendActivity(reply);
}
```

---

## <a name="additional-resources"></a>其他资源

可以从 GitHub [[C#](https://aka.ms/SuggestedActionsCSharp) | [JS](https://aka.ms/SuggestActionsJS)] 访问此处显示的完整源代码。
