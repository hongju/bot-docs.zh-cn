---
title: 使用可评分项的全局消息处理程序
description: 在 Bot Builder SDK for .NET 中使用可评分项创建更灵活的对话。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8ba63ad99c772c7cf5884180a62244e0dfe11db2
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574903"
---
# <a name="global-message-handlers-using-scorables"></a>使用可评分项的全局消息处理程序

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

当机器人期待不同的响应时，用户尝试通过在聊天中使用“帮助”、“取消”或“重新开始”等单词来访问机器人内的某些功能。 可以将机器人设计为使用可评分对话恰当地处理此类请求。

可评分对话监视所有传入消息，并确定消息是否以某种方式可操作。 可评分的消息由每个可评分对话分配一个 [0 - 1] 之间的分数。 确定最高分的可评分对话将添加到对话堆栈的顶部，然后将响应交付给用户。 在可评分对话完成执行后，聊天将从中断处继续。

可评分项通过允许用户“中断”你在常规对话中找到的正常聊天流，使你能够创建更灵活的聊天。

## <a name="create-a-scorable-dialog"></a>创建可评分对话

首先，定义一个新[对话](bot-builder-dotnet-dialogs.md)。 以下代码使用从 `IDialog` 接口派生的对话。

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
若要创建可评分对话，请创建一个继承自 `ScorableBase` 抽象类的类。 以下代码演示 `SampleScorable` 类。

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
`ScorableBase` 抽象类继承自 `IScorable` 接口。 需要在类中实现以下 `IScorable` 方法：

- `PrepareAsync` 是在可评分实例中调用的第一个方法。 它接受传入的消息活动，分析并设置对话的状态，该状态将传递给 `IScorable` 接口的所有其他方法。

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- `HasScore` 方法检查状态属性，以确定可评分对话是否应为消息提供分数。 如果它返回 false，则可评分对话将忽略该消息。

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- `GetScore` 仅在 `HasScore` 返回 true 时触发。 你将在此方法中预配逻辑，以确定 0 到 1 之间的消息分数。

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- 在 `PostAsync` 方法中，定义要为可评分类执行的核心操作。 所有可评分对话将监视传入消息，并根据可评分项的 GetScore 方法为有效消息分配分数。 然后，确定最高分（在 0 - 1.0 之间）的可评分类将触发 scorable 的 `PostAsync` 方法。

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- 评分过程完成后调用 `DoneAsync`。 使用此方法释放任何作用域资源。

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>创建一个模块以注册 IScorable 服务

接下来，定义一个 `Module`，它将 `SampleScorable` 类注册为组件。 这将预配 `IScorable` 服务。

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>注册模块  

此过程的最后一步是将 `SampleScorable` 应用于机器人的聊天容器。 这将在 Bot Framework 的消息处理管道中注册可评分服务。 以下代码显示如何在 **Global.asax.cs** 中更新机器人应用初始化中的 `Conversation.Container`：

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>其他资源
* [全局消息处理程序示例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers)
* [简单的可评分机器人示例](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample)
* [对话概述](bot-builder-dotnet-dialogs.md)
* [AutoFac](https://autofac.org/)
