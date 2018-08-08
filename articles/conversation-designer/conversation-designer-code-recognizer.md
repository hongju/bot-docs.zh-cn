---
title: 将代码识别器定义为任务触发器 | Microsoft Docs
description: 了解如何使用自定义代码识别器作为任务触发器。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 6d44cd299e46971b2218bb78d16e454d5df3c1d9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297668"
---
# <a name="define-code-recognizer-as-task-trigger"></a>将代码识别器定义为任务触发器
> [!IMPORTANT]
> 聊天设计器尚未可供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

使用代码识别器可以编写自定义脚本来帮助执行任务。 基于正则表达式的表达式评估或调用其他服务可用于帮助确定用户的意向。 自定义脚本将指示聊天运行时触发任务。 

## <a name="create-a-script-function"></a>创建脚本函数
若要将脚本用作识别器，请在任务编辑器中选择“脚本函数”作为识别器，指定函数名称，然后单击“创建/查看函数”开始编辑脚本。 你也可以单击“创建/查看函数”而不指定脚本名称，此时将使用默认名称为你创建一个空函数。 

## <a name="script-trigger-function-parameter"></a>脚本触发函数参数

始终将使用 [`context`](conversation-designer-context-object.md) 对象调用指定的脚本回调函数。

`context` 对象包括 `taskEntities` 和 `contextEntities`。 任务实体是为此任务定义或生成的实体，上下文实体是可以在与用户的聊天中保留实体的属性包。

## <a name="return-value"></a>返回值

**脚本触发**函数应返回一个布尔值。

## <a name="sample-regex-based-recognizer"></a>基于正则表达式的识别器示例
以下代码示例使用正则表达式处理请求并返回布尔值，以便聊天运行时可以确定要执行的脚本触发器。

```javascript
module.exports.fnFindPhoneTrigger = function(context) {
    if (context.request.text.includes("call") || context.request.text.includes("ring")) {
        return true;
    } else {
        return false;
    }
} 
```

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [回复操作](conversation-designer-reply.md)
