---
title: 通过 Bot Builder SDK for .NET 创建机器人 | Microsoft Docs
description: 通过 Bot Builder SDK for .NET 创建机器人，这是一个功能强大的机器人构建框架。
keywords: Bot Builder SDK, 创建机器人, 快速入门, 入门
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fcb21fa38750c09f110a3c71f763a941c4437979
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298416"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>通过 Bot Builder SDK for .NET 创建机器人
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [机器人服务](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

<a href="https://github.com/Microsoft/BotBuilder" target="_blank">Bot Builder SDK for .NET</a> 是一个易于使用的框架，用于使用 Visual Studio 和 Windows 开发机器人。 SDK 利用 C# 为 .NET 开发者提供熟悉的方法来创建功能强大的机器人。


本教程将指导你使用 Bot Application 模板和 Bot Builder SDK for .NET 生成机器人，然后使用 Bot Framework Emulator 对其进行测试。

## <a name="prerequisites"></a>先决条件
1. Visual Studio [2017](https://www.visualstudio.com/)。

2. 在 Visual Studio 中，将所有[扩展](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-update-a-visual-studio-extension)更新为其最新版本。

3. [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3) 的机器人模板。

> [!TIP]
> Visual Studio 2017 项目模板目录通常位于 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\`，项模板目录位于 `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#\`

## <a name="create-your-bot"></a>创建机器人

打开 Visual Studio 并创建一个新的 C# 项目。 为新项目选择“简单的回响机器人应用程序”模板。

![Visual Studio 创建项目](../media/connector-getstarted-create-project.png)

> [!NOTE]
> Visual Studio 可能会显示需要下载并安装 [IIS Express](https://www.microsoft.com/en-us/download/details.aspx?id=48264)。 

得益于“机器人应用程序”模板，项目包含本教程中创建机器人所需的所有代码。 实际上不需要编写任何其他代码。 但是，在我们继续测试机器人之前，请快速查看“机器人应用程序”模板提供的一些代码。

> [!TIP] 
> 如果需要，请更新 [NuGet 包](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

## <a name="explore-the-code"></a>浏览代码

首先，Controllers\MessagesController.cs 中的 `Post` 方法接收来自用户的消息并调用根对话。

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}

```

根对话处理消息并生成响应。 Dialogs\RootDialog.cs 中的 `MessageReceivedAsync` 方法发送回复消息，回响用户的消息，前缀为“你已发送”文本，并以“共 ## 个字符”文本结束，其中 ## 表示用户消息中的字符数。

```csharp
public class RootDialog : IDialog<object>
{
    public Task StartAsync(IDialogContext context)
    {
        context.Wait(MessageReceivedAsync);

        return Task.CompletedTask;
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
    {
        var activity = await result as Activity;

        // Calculate something for us to return
        int length = (activity.Text ?? string.Empty).Length;

        // Return our reply to the user
        await context.PostAsync($"You sent {activity.Text} which was {length} characters");

        context.Wait(MessageReceivedAsync);
    }
}
```

## <a name="test-your-bot"></a>测试机器人

[!INCLUDE [Get started test your bot](../includes/snippet-getstarted-test-bot.md)]

### <a name="start-your-bot"></a>启动机器人

安装模拟器后，使用浏览器作为应用程序主机在 Visual Studio 中启动机器人。
此 Visual Studio 屏幕截图显示，单击“运行”按钮时，机器人将在 Microsoft Edge 中启动。

![Visual Studio 运行项目](../media/connector-getstarted-start-bot-locally.png)

单击运行按钮时，Visual Studio 将生成应用程序，将其部署到 localhost，然后启动 Web 浏览器以显示应用程序的 default.htm 页面。
例如，下面是 Microsoft Edge 中显示的应用程序的 default.htm 页面：

![运行 localhost 的 Visual Studio 机器人](../media/connector-getstarted-bot-running-localhost.png)

> [!NOTE]
> 可修改项目中的 default.htm 文件，以指定机器人应用程序的名称和说明。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

此时，机器人在本地运行。
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 创建新的机器人配置。 在地址栏中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。

2. 单击“保存并连接”。 无需指定“Microsoft 应用 ID”和“Microsoft 应用密码”。 可以暂时将这些字段留空。 稍后[注册机器人](~/bot-service-quickstart-registration.md)时将获取此信息。

### <a name="test-your-bot"></a>测试机器人

现在机器人在本地运行并连接到模拟器，通过在模拟器中键入一些消息来测试机器人。
应该看到机器人通过使用前缀为“你已发送”文本、结尾为“共 ## 个字符”文本（其中 ## 是已发送消息中的总字符数）来回响消息以回复你发送的每条消息。


> [!TIP]
> 在模拟器中，单击聊天中的任何聊天气泡。 有关该消息的详细信息将以 JSON 格式显示在“详细信息”窗格中。

已使用“机器人应用程序”模板和 Bot Builder SDK for .NET 成功创建了一个机器人！

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已使用“机器人应用程序”模板和 Bot Builder SDK for .NET 创建了一个简单的机器人，并使用 Bot Framework Emulator 验证了机器人的功能。

接下来，了解 Bot Builder SDK for .NET 的重要概念。

> [!div class="nextstepaction"]
> [Bot Builder SDK for .NET 中的重要概念](bot-builder-dotnet-concepts.md)
