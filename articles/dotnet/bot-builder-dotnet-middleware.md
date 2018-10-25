---
title: 截获消息 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 截获用户和机器人之间的消息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21607dae5c2a8d08ed4b7bf1b6e6983cd9bf1196
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999724"
---
# <a name="intercept-messages"></a>截获消息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>截获并记录消息

以下代码示例演示如何使用 Bot Builder SDK for .NET 中的中间件概念来截获用户和机器人之间交换的消息。 

首先，创建 `DebugActivityLogger` 类并定义 `LogAsync` 方法，以指定针对每条截获的消息要采取的操作。 此示例仅输出每条消息的部分信息。

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

然后，将以下代码添加到 `Global.asax.cs`。  在用户和机器人之间（任意方向）交换的每条消息都将触发 `DebugActivityLogger` 类中的 `LogAsync` 方法。 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

尽管此示例只是输出了每条消息的部分信息，但你可以更新 `LogAsync` 方法，以指定想要对每条消息执行的操作。 

## <a name="sample-code"></a>代码示例 

有关演示如何使用 Bot Builder SDK for .NET 来截获并记录消息的完整示例，请参阅 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">中间件示例</a>。 

## <a name="additional-resources"></a>其他资源

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">中间件示例 (GitHub)</a>
