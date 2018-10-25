---
title: 创建自己的中间件 | Microsoft Docs
description: 了解如何编写自己的中间件。
keywords: 中间件, 自定义中间件, 短路, 回退, 活动处理程序
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78c0afd6095cf9a10e04e1b0131d66ce6964f369
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000194"
---
# <a name="create-your-own-middleware"></a>创建自己的中间件

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

中间件允许你为机器人编写丰富的插件，然后其他人也可以使用这些插件。 这里我们将演示如何添加和实现基本中间件，并展示其工作原理。 v4 SDK 提供了一些中间件，例如状态管理、LUIS、QnAMaker 和转换等。 有关更多信息，请参阅 Bot Builder SDK for [.NET](https://github.com/Microsoft/botbuilder-dotnet) 或 Bot Builder SDK for [JavaScript](https://github.com/Microsoft/botbuilder-js)。

## <a name="adding-middleware"></a>添加中间件

以下示例基于通过[入门](~/bot-service-quickstart.md)体验创建的基本机器人示例，将两个不同的中间件添加到服务中，其中的每个类包含一个新实例。

> [!IMPORTANT]
> 请记住，它们添加到选项的顺序决定了它们的执行顺序。 如果使用多个中间件，请务必考虑这将如何生效。

**Startup.cs**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

对于要添加的每个中间件，将 `options.Middleware.Add(new MyMiddleware());` 方法调用添加到机器人服务选项。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

对于要添加的每个中间件，将 `adapter.use(MyMiddleware());` 添加到适配器。

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>实现中间件

每一个中间件都继承自中间件接口，并始终实现它的处理程序，该处理程序在发送给机器人的每个活动上运行。 对于添加的每个中间件，处理程序可以修改上下文对象或执行任务（例如记录），然后才允许其他中间件或机器人逻辑在它向下继续处理管道时与上下文对象进行交互。

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

每个中间件都继承自 `IMiddleware` 并始终实现 `OnTurn()`。

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

每个中间件都继承自 `MiddlewareSet` 并始终实现 `onTurn()`。

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

调用 `next()` 导致继续执行到下一个中间件。 因为能够选择何时传递执行，所以可以编写在中间件堆栈的其余部分运行了之后运行的代码。 机器人逻辑和其他中间件运行之后，我们可以对处理程序的“后缘”执行操作。  在上面的示例中，实现的第一个中间件只是做了：如果我们响应了这个上下文对象，那么在沿着管道向上传递回执行之前进行报告。

## <a name="short-circuit-routing"></a>短路路由

在某些情况下，建议停止对所接收活动进行任何进一步处理，我们称之为短路处理。 如果中间件完全处理请求并且为特定命令提供简单响应，或者如果中间件能够处理传入请求而没有需要查看它的机器人逻辑，这将非常有用。

我们来创建一个中间件，每当用户说“ping”时，它都会发送回复并阻止请求的任何进一步路由：

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>回退处理

可能需要做的另一件事是对尚未得到响应的请求做出响应。 通过检查 `context.Responded` 属性，可以使用处理程序的后缘轻松完成此操作。 我们来创建一个简单的中间件，如果机器人无法处理请求，它会自动声明“我不明白”：

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> 这可能不会在所有情况下都起作用，例如当其他中间件或许能够响应用户或机器人正确收到消息但没有回复时。 以“我不明白”来响应将误导我们的用户。


