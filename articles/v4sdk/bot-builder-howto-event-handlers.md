---
title: 事件处理程序 | Microsoft Docs
description: 了解如何使用事件处理程序。
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/14/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0c054b8f4209004806e4564be45e83bdf3a8ec25
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298308"
---
# <a name="event-handlers"></a>事件处理程序

事件处理程序是我们可以添加到[轮](bot-builder-basics.md#defining-a-turn)内未来活动事件的函数。 这些活动包括 `SendActivity`、`UpdateActivity` 和 `DeleteActivity`，每个活动都有其自己的处理程序。 当你需要对当前上下文对象的每个该类型的未来活动执行某个操作时，这些处理程序非常有用。

可以为每个活动事件添加多个事件处理程序，并且这些事件处理程序将在添加**之后**为每个事件进行处理，因此在代码中添加它们的位置很重要。

> [!NOTE]
> 本文中的事件处理程序不遵循事件和处理程序的传统语言特定定义， 而是引用了*事件*和*处理程序*的更概念性编程思想。

## <a name="using-the-next-delegate"></a>使用 *next* 委托

通过调用 *next* 委托，每个处理程序按照添加处理程序的顺序将执行传递给以下处理程序。 最后一个 *next* 委托调用适配器来实际发送、更新或删除活动。

除非有其他目的，否则请务必在处理程序中调用 *next()*，如果不这样做会使活动[短路](bot-builder-create-middleware.md#short-circuit-routing)。 使活动短路与中间件之间的区别在于，短路完全取消了活动，而中间件更改了允许运行的代码，但活动的执行仍在继续。

对于发送活动，如果成功，*next* 委托将返回分配给已发送消息的 ID。

## <a name="expected-return-value"></a>预期的返回值

对于返回值，事件处理程序预期它是 next 委托的结果。 如果需要，可以在返回之前查看和存储该结果，也可以直接返回该结果，如下所示。

`SendActivity` 处理程序的返回值是作为相应 next 委托的返回值通过链传递回的返回值。

# <a name="ctabcseventhandler"></a>[C#](#tab/cseventhandler)

提供的三种类型的事件处理程序是 `OnSendActivities()`、`OnUpdateActivity()` 和 `OnDeleteActivity()`。 在这里，我们将在机器人代码中添加了一个 `OnSendActivities()` 处理程序，但也可以在中间件代码中添加处理程序。

处理程序通常以 lambda 表达式的形式添加，如此示例所示。 在这里，我们将在写入 **help** 时侦听用户的输入活动。

```cs
public Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        
        context.OnSendActivities(async (handlerContext, activities, handlerNext) =>
        {
            if (handlerContext.Activity.Text == "help")
            {
                Console.WriteLine("help!");
                // Do whatever logging you want to do for this help message
            }

            return await handlerNext();
        });
        // ...
    }
}
```

# <a name="javascripttabjseventhandler"></a>[JavaScript](#tab/jseventhandler)

提供的三种类型的事件处理程序是 `onSendActivities()`、`onUpdateActivity()` 和 `onDeleteActivity()`。 在这里，我们将在机器人代码中添加了一个 `onSendActivities()` 处理程序，但也可以在中间件代码中添加处理程序。

此示例将在写入 **help** 时侦听用户的输入活动。

```js
adapter.processActivity(req, res, async (context) => {

    if (context.activity.type === 'message') {

        context.onSendActivities(async (handlerContext, activities, handlerNext) => { 
            
            if(handlerContext.activity.text === 'help'){
                console.log('help!')
                // Do whatever logging you want to do for this help message
            }
            // Add handler logic here
        
            await handlerNext(); 
        });
        await context.sendActivity(`you said ${context.activity.text}`);
    }
});
```

---

请务必区分“发送活动”和“更新或删除活动”事件，其中第一个事件会创建一个全新的活动事件，而后一个事件会对过去的活动产生影响。 此外， 并非所有通道都支持“更新”或“删除”活动。 建议围绕对这些活动及其处理程序的调用添加适当的异常处理，以应对这种可能性。

