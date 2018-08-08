---
title: 上下文对象的 API 参考 | Microsoft Docs
description: 了解如何在聊天设计器机器人中引用上下文对象。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 5ba70189c16815539524d3c9046da03ed0344b9b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297667"
---
# <a name="api-reference"></a>API 参考
> [!IMPORTANT]
> 聊天设计器尚未可供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

使用聊天设计器可以向机器人添加自定义业务逻辑。 这些脚本函数在“脚本”编辑器中实现。 这些函数可以在各种位置关联，如条件响应模板、任务的*触发器*或*操作*或对话状态，并且它们都接收其函数参数中 **[IConversationContext]** 类型的 `context` 对象。

下面的代码示例显示了带有传入的**上下文**对象的条件响应函数的签名。

```javascript
/**
* @param {IConversationContext} context
*/
module.exports.NewConditionalResponse_onRun = function(context) {
    // Business logic here
    return true; // Returns a boolean
}
```

通过 `context` 对象，你可以访问有关用户和机器人之间聊天的信息。

## <a name="script-callback-functions"></a>脚本回调函数

你创建的自定义脚本回调函数可能采用多种形式。 虽然你可以在功能上给它们不同的名称，但它们采用以下形式之一。

| 形式 | 参数 | 返回类型 | Description |
| ---- | ---- | ---- | ---- |
| 在响应函数之前 | 上下文 | **void** 或 **promise** | 在给出响应之前执行的函数。 |
| 处理函数 | 上下文 | **void** 或 **promise** | 执行业务逻辑的函数。 |
| 决策函数 | 上下文 | **string** 或 **promise** | 基于业务逻辑做出决策的函数。 返回字符串应与[决策](conversation-designer-dialogues.md#decision-state)块中的条件匹配。 |
| 代码识别器函数 | 上下文 | **boolean** 或 **promise** | **脚本触发**发生时运行的自定义业务逻辑。 返回 `true` 表示匹配。 否则，返回 `false` 取消匹配。 |
| onRecognize 函数 | 上下文 | **boolean** 或 **promise** | 仅在 LUIS 识别器中存在匹配项时才执行的函数。 使用此回调函数处理 LUIS 实体并返回适当的 **boolean** 值。 返回 `true` 表示匹配。 否则，返回 `false` 取消匹配。 |

## <a name="iconversationcontext-interface"></a>IConversationContext 接口

`IConversationContext` 接口跟踪用户和机器人之间的聊天信息。 通过聊天设计器关联的所有自定义函数都将接收 `context` 对象作为参数自变量。

## <a name="context-properties"></a>上下文属性
`context` 对象公开以下属性。

| 名称 |  代码 | Description |
| ---- | ---- | ---- |
| `request` | `context.request` | 获取包含机器人活动的请求对象。  |
| | `context.request.attachment` | 可能包含自适应卡的附件活动。 |
| | `context.request.text` | 包含来自客户端的传入文本消息的文本活动。 |
| | `context.request.speak` | 包含来自客户端的语音文本（如果可用）的讲述活动。 |
| | `context.request.type` | 指定活动类型（默认值：“消息”）。 |
| `responses` | `context.responses` | 维护一系列活动，这些活动将在当前状态或执行后代码结束时发送回客户端。 |
| | `context.responses.push` | 在响应中添加活动。 |
| `global` | `context.global` | 包含你定义的聊天数据的 JavaScript 对象。 该对象在整个聊天中持续存在。 |
| `local` | `context.local` | 包含你定义的任务数据的 JavaScript 对象。 此对象在特定任务的持续时间内持续存在。 LUIS 意向始终返回给本地上下文。 如果要持久保留 LUIS 结果，请考虑将其复制到 `context.global` 上下文。 |
| | `context.local['@description']` | 返回从 LUIS 收到的原始实体。 |
| `sticky` | `context.sticky` | 指示当前任务名称 |
| `currentTemplate` | `context.currentTemplate` | 为显示和讲述评估调用的[条件响应模板](conversation-designer-response-templates.md#conditional-response-templates)。 此对象包含三个属性： <br/>1. **name**：当前模板的名称。 <br/>2. **modalityDisplay**：一个布尔值，指示模态与显示评估相关联。 <br/>3. **modalitySpeak**：一个布尔值，指示模态与讲述评估相关联。 |

## <a name="context-methods"></a>上下文方法
`context` 对象公开以下方法。

| 名称 | 返回类型 | 代码 | Description |
| ---- | ---- | ---- | ---- |
| `getCurrentTurn` | **数字** | `context.getCurrentTurn();` | 如果正在执行重新提示，则从堆栈顶部的帧获取该轮次。 |

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [创建机器人](conversation-designer-create-bot.md)
