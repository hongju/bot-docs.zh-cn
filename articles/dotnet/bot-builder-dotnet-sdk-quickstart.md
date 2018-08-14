---
title: 通过 Bot Builder SDK for .NET 创建机器人 | Microsoft Docs
description: 通过功能强大的机器人构造框架 Bot Builder SDK for .NET 创建机器人。
keywords: Bot Builder SDK, 创建机器人, 快速入门, .NET, 入门
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 98c6103f0cd38bf6d3f477735dafcf01952304d5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298435"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-for-net"></a>通过 Bot Builder SDK v4 for .NET 创建机器人
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入门介绍如何使用 Bot Application 模板和 Bot Builder SDK for .NET 生成机器人，然后使用 Bot Framework Emulator 对其进行测试。 这基于 [Microsoft Bot Builder SDK v4](https://github.com/Microsoft/botbuilder-dotnet)。

## <a name="prerequisites"></a>先决条件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- 面向 [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) 的 Bot Builder SDK v4 模板
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) 中的异步编程

## <a name="create-a-bot"></a>创建机器人
安装在先决条件部分中下载的 BotBuilderVSIX.vsix 模板。 

在 Visual Studio 中创建一个新的机器人项目。

![Visual Studio 项目](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 如果需要，请更新 [NuGet 包](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

## <a name="explore-code"></a>浏览代码
打开 Startup.cs 文件以查看 `ConfigureServices(IServiceCollection services)` 方法中的代码。 系统会将 `CatchExceptionMiddleware` 中间件添加到消息传送管道中。 该中间件处理由其他中间件或 OnTurn 方法引发的任何异常。 

```cs
options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
{
    await context.TraceActivity("EchoBot Exception", exception);
    await context.SendActivity("Sorry, it looks like something went wrong!");
}));
```

聊天状态中间件使用内存中存储。 它读取 EchoState 并将其写入存储。  EchoState 类中的回合计数会跟踪发送给机器人的消息数量。 可以使用类似的技术来维护回合之间的状态。

```cs
 IStorage dataStore = new MemoryStorage();
 options.Middleware.Add(new ConversationState<EchoState>(dataStore));
```

`Configure(IApplicationBuilder app, IHostingEnvironment env)` 方法调用 `.UseBotFramework`，以将传入活动路由到机器人适配器。 

EchoBot.cs 文件中的 `OnTurn(ITurnContext context)` 方法用于检查传入活动类型并向用户发送回复。 

```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```
## <a name="start-your-bot"></a>启动机器人

- `default.html` 页面将显示在浏览器中。
- 记下该页面的 localhost 端口号。 与机器人交互时将需要此信息。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

此时，机器人在本地运行。
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎使用”选项卡中的“新建机器人配置”链接。 

2. 输入**机器人名称**并输入机器人代码的目录路径。 机器人配置文件将保存到此路径。

3. 在“终结点 URL”字段中键入 `http://localhost:port-number/api/messages`，其中 *port-number* 与运行应用程序的浏览器中显示的端口号相匹配。

4. 单击“连接”以连接到机器人。 无需指定 **Microsoft 应用 ID** 和 **Microsoft 应用密码**。 可以暂时将这些字段留空。 以后当你注册机器人时，将获得这些信息。

## <a name="interact-with-your-bot"></a>与机器人交互

向机器人发送“Hi”，机器人将使用“Turn 1: You sent Hi”来响应该消息。

## <a name="next-steps"></a>后续步骤

接下来，[将机器人部署到 Azure](../bot-builder-howto-deploy-azure.md) 或者跳转到解释机器人及其工作原理的概念。

> [!div class="nextstepaction"]
> [基本机器人概念](../v4sdk/bot-builder-basics.md)
