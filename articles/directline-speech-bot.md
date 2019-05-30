---
title: 开发 DirectLine 语音机器人 | Microsoft Docs
description: 开发 DirectLine 语音机器人
keywords: 开发 driect line 语音机器人, 语音机器人
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 966e1b6e884486ddc3d57bea0a52ee07ac982346
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66214315"
---
## <a name="use-direct-line-speech-in-your-bot"></a>在机器人中使用 Direct Line 语音 

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line 语音使用 Bot Framework 的一项新的基于 WebSocket 的流式处理功能，在 Direct Line 语音通道和机器人之间交换消息。 在 Azure 门户中启用 Direct Line 语音通道以后，需更新机器人，使之侦听并接受这些 WebSocket 连接。 以下说明介绍如何执行该操作。

## <a name="add-the-nuget-package"></a>添加 NuGet 包
就 Direct Line 语音预览版来说，还需将其他 NuGet 包添加到机器人。 这些包托管在 myget.org NuGet 源上：
1.  在 Visual Studio 中打开机器人的项目。

2.  转到机器人项目的属性下的“管理 Nuget 包”。

3.  如果尚未将其设置为源，请在右上角的 NuGet 源设置中将其添加为 `https://botbuilder.myget.org/F/experimental/api/v3/index.json` 源。

4.  选择此 NuGet 源，然后添加一个 `Microsoft.Bot.Protocol.StreamingExtensions.NetCore` 包。

5.  接受任何提示，完成将包添加到项目的操作。

## <a name="option-1-update-your-net-core-bot-code-if-your-bot-has-a-botcontrollercs"></a>选项 1：如果机器人有 BotController.cs，请更新 .NET Core 机器人代码 
在 Azure 门户中使用某个模板（例如 EchoBot）创建新的机器人时，会得到一个包含 ASP.NET MVC 控制器的机器人，该控制器会公开单个 POST 终结点。 以下说明介绍了如何对其进行扩展，使之还公开一个终结点，以便接受属于 GET 终结点的 WebSocket 流式处理终结点。
1.  打开解决方案中 Controllers 文件夹下的 BotController.cs。

2.  在类中找到 PostAsync 方法，将其修饰从 [HttpPost] 更新为 [HttpPost, HttpGet]：
```cs
[HttpPost, HttpGet]
public async Task PostAsync()
{ 
    await _adapter.ProcessAsync(Request, Response, _bot);
}
```

3.  保存并关闭 BotController.cs

4.  在解决方案的根目录中打开 Startup.cs。

5.  添加新命名空间：

```cs
using Microsoft.Bot.Protocol.StreamingExtensions.NetCore;
```

6.  在 ConfigureServices 方法中，不使用 AdapterWithErrorHandler，改用相应 services.AddSingleton 调用中的 WebSocketEnabledHttpAdapter：

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...    

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    services.AddTransient<IBot, EchoBot>();

    ...
}
```

7. 此时仍然在 Startup.cs 中，导航到 ConfigureServices 方法底部。 在调用 app.UseMvc() 之前调用（这很重要，因为 Use 调用的顺序会影响结果），添加 app.UseWebSockets()。 方法末尾应该如下所示：

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

8.  机器人代码的余下部分保持不变！

## <a name="option-2-update-your-net-core-bot-code-if-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>选项 2：如果机器人使用 AddBot 和 UseBotFramework 而不是 BotController，请更新 .NET Core 机器人代码 

如果在使用 Bot Builder SDK 版本 4.3.2 之前使用了 v4 版，则机器人可能不包含 BotController，但会使用 Startup.cs 文件中的 AddBot() 和 UseBotFramework() 方法来公开 POST 终结点（机器人在其中接收消息）。 若要公开新的流式处理终结点，需添加 BotController 并删除 AddBot() 和 UseBotFramework() 方法。 以下说明详细介绍了需要进行的更改。

1.  通过添加名为 BotController.cs 的文件，向机器人项目添加新的 MVC 控制器。 向以下文件添加控制器代码：

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter _adapter;
    private readonly IBot _bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost, HttpGet]
    public async Task ProcessMessageAsync()
    {
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```
2.  在 Startup.cs 文件中，找到 Configure 方法。 删除 UseBotFramework() 行，务必保留以下行：

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

3.  另外，也是在 Startup.cs 文件中，找到 ConfigureServices 方法。 删除 AddBot() 行，务必保留以下行：

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```
4.  机器人代码的余下部分保持不变！

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [将机器人连接到 Direct Line 语音（预览版）](./bot-service-channel-connect-directlinespeech.md)
