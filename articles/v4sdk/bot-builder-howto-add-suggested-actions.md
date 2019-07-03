---
title: 使用按钮进行输入 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for JavaScript 在消息中发送建议的操作。
keywords: 建议的操作, 按钮, 额外输入
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43c9b0f062f4db909612f3fb9dc048be65c64c57
ms.sourcegitcommit: d29d3d7ccef401aa1e84e19e623db33b5ff13e63
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2019
ms.locfileid: "67160637"
---
# <a name="use-button-for-input"></a>使用按钮进行输入

[!INCLUDE[applies-to](../includes/applies-to.md)]

你可以使机器人能够显示按钮供用户点击来提供输入。 按钮改进了用户体验，因为用户只需点击按钮便可回答问题或进行选择，而不必使用键盘输入响应。 与资讯卡中显示的按钮（即使在点击后仍然可见且可供用户访问）不同，建议的操作窗格中显示的按钮将在用户进行选择后消失。 这可以防止用户在会话中点击过时按钮并简化机器人开发（因为将不需要说明该场景）。 

## <a name="suggest-action-using-button"></a>使用按钮提供操作建议

*建议的操作*让机器人能够显示按钮。 可以创建一个建议的操作列表（也称为“快速回复”），该列表将作为单轮聊天显示给用户： 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处显示的源代码基于[建议操作示例](https://aka.ms/SuggestedActionsCSharp)。

[!code-csharp[suggested actions](~/../botbuilder-samples/samples/csharp_dotnetcore/08.suggested-actions/Bots/SuggestedActionsBot.cs?range=87-101)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处显示的源代码基于[建议操作示例](https://aka.ms/SuggestActionsJS)。

[!code-javascript[suggested actions](~/../botbuilder-samples/samples/javascript_nodejs/08.suggested-actions/bots/suggestedActionsBot.js?range=61-64)]

---

## <a name="additional-resources"></a>其他资源

可以从 [CSharp 示例](https://aka.ms/SuggestedActionsCSharp)或 [JavaScript 示例](https://aka.ms/SuggestActionsJS)访问此处显示的完整源代码。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [保存用户和聊天数据](./bot-builder-howto-v4-state.md)
