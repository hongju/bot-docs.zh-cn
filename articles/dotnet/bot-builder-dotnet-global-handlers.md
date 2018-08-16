---
title: 实现全局消息处理程序 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 让机器人侦听并处理包含某些关键字的用户输入。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 00ffc96672ea34cb2702b3b38fad5b98cf124042
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298026"
---
# <a name="implement-global-message-handlers"></a>实现全局消息处理程序

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>侦听用户输入中的关键字

以下演练将逐步介绍如何使用 Bot Builder SDK for .NET 来实现全局消息处理程序。

首先，`Global.asax.cs` 注册 `GlobalMessageHandlersBotModule`，其实现方法如下所示。 在此示例中，该模块会注册两个 scorable：一个用于管理要更改设置的请求 (`SettingsScorable`)，另一个用于管理要取消的请求 (`CancelScoreable`)。

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

`CancelScorable` 包含定义触发器的 `PrepareAsync` 方法：如果消息文本为“取消”，则将触发此 scorable。

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

收到“取消”请求时，`CancelScoreable` 中的 `PostAsync` 方法将重置对话框堆栈。 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

收到“更改设置”请求时，`SettingsScorable` 中的 `PostAsync` 方法调用 `SettingsDialog`（将请求传递给该对话框），从而将 `SettingsDialog` 添加到对话框堆栈的顶部并由会话对其进行控制。

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>代码示例

有关演示如何使用 Bot Builder SDK for .NET 实现全局消息处理程序的完整示例，请参阅 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">全局消息处理程序示例</a>。

## <a name="additional-resources"></a>其他资源

- [设计和控制会话流](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">Bot Builder SDK for .NET 参考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">全局消息处理程序示例 (GitHub)</a>
