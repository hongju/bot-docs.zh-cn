---
title: 如何在 SDK V4 中运行 .NET SDK V3 机器人 | Microsoft Docs
description: 了解如何使用经典 NuGet 包将机器人从 3.x 转换为 4.0。
keywords: 迁移, 经典机器人, 转换 v3, v3 到 v4
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/25/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a808076e4865a181802b85cfc24ce342dbf23cba
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297772"
---
# <a name="how-to-run-net-sdk-v3-bots-in-sdk-40"></a>如何在 SDK 4.0 中运行 .NET SDK V3 机器人

**Microsoft.Bot.Builder.Classic** NuGet 包简化了机器人从 Microsoft Bot Framework 3.x 版迁移到 4.0 版的过程。

**注意：** 此过程仅适用于 **ASP.NET Web 应用程序 (.NET Framework)** 机器人。 这不适用于 **ASP.NET Core Web 应用程序**机器人。

## <a name="the-process"></a>此过程

此过程相对简单：

- 将 **Microsoft.Bot.Builder.Classic** NuGet 包添加到项目中。
    - 可能还需要更新 **Autofac** NuGet 包。
- 更新 **Microsoft.Bot.Builder.Classic** 命名空间。
- 使用基于 4.0 **ITurnContext** 的 **Conversation.SendAsync()** 调用对话。

### <a name="add-the-microsoftbotbuilderclassic-nuget-package"></a>添加 Microsoft.Bot.Builder.Classic NuGet 包

若要添加 **Microsoft.Bot.Builder.Classic** NuGet 包，请使用**管理 NuGet 包**来添加该包。

### <a name="update-the-namespaces"></a>更新命名空间

若要更新命名空间，请删除以下找到的任何 `using` 语句：

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.FormFlow;
using Microsoft.Bot.Builder.Scorables;
```

然后在其位置添加以下 `using` 语句：

```csharp
using Microsoft.Bot.Builder.Adapters;
using Microsoft.Bot.Builder.Classic.Dialogs;
using Microsoft.Bot.Builder.Classic.Dialogs.Internals;
using Microsoft.Bot.Builder.Classic.FormFlow;
using Microsoft.Bot.Builder.Classic.Scorables;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;
```

如果有 `[BotAuthentication]` 行，请删除该行或将其注释掉。

### <a name="invoke-your-3x-dialog"></a>调用 3.x 对话

若要调用 3.x 对话，仍然使用 `Conversation.SendAsync`，只是现在它使用 4.0  **ITurnContext** 而不是 **Activity**。

```csharp
// invoke a Classic V3 IDialog 
await Conversation.SendAsync(turnContext, () => new EchoDialog());
```

如果没有 **ITurnContext** 对象，但是有 **Activity** 对象，则可以通过以下方式获取 **ITurnContext** 对象：

```csharp
BotFrameworkAdapter adapter = new BotFrameworkAdapter("", "");

await adapter.ProcessActivity(this.Request.Headers.Authorization?.Parameter,
        activity,
        async (context) =>
        {
            // Do something with context here. For example, the body of your Post() method may go here.
        });
```

## <a name="fix-assembly-conflicts"></a>修复程序集冲突

使用经典 NuGet 包生成的机器人将与其程序集发生冲突。 此冲突将在生成后显示在 Visual Studio 的“错误列表”窗口中。

### <a name="if-you-see-warning-found-conflicts-between-different-versions-of-the-same-dependent-assembly"></a>如果你看到“警告: 发现同一依赖程序集的不同版本之间存在冲突”

如果发现以下列文本开头的警告：**发现同一依赖程序集的不同版本之间存在冲突**：

- 双击警告消息。 将出现一个对话框，询问“是否要通过在应用程序配置文件中添加绑定重定向记录来修复这些冲突?”
- 单击“是”。
- 重新生成项目。

### <a name="if-you-see-error-missing-method-exception-on-startup"></a>如果你看到“错误: 启动时出现缺少方法异常”

当你使用较旧的 .NET 4.6 项目并升级到 4.6.1 然后尝试对其使用 .NET 标准库时，似乎会出现此 .NET Standard 的 bug。 从本质上讲，有两个不同的 System.Net.Http 程序集尝试动态换出。解决方法是将 System.Net.Http 的绑定重定向添加到 Web.config 中。 

如果收到此错误，请将以下项添加到 Web.config 文件：

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="B03F5F7F11D50A3A" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.2.0.0" newVersion="4.2.0.0" />
</dependentAssembly>
```

有关此问题的更多详细信息，请参阅[从 MSBuild 工具复制/加载的 System.Net.Http v4.2.0.0 #25773](https://github.com/dotnet/corefx/issues/25773)。

## <a name="sample-of-a-converted-bot"></a>转换后的机器人示例

有关已转换的机器人，可以查看 [EchoBot-Classic](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/Microsoft.Bot.Samples.EchoBot-Classic) 示例，该示例显示转换为使用 4.0 的 3.x 机器人。

## <a name="limitations"></a>限制
Microsoft.Bot.Builder.Classic 只是一个 .NET 4.61 库，它不适用于 .NET core。
