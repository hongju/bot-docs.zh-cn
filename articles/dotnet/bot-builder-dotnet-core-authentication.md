---
title: 使用 .NET Core 对活动进行身份验证 |Microsoft Docs
description: 了解如何使用 .NET Core 对机器人活动进行身份验证。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 431367cf4afe702fd83feff60b0ee4e260d50f17
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905759"
---
# <a name="authenticating-activities-using-net-core"></a>使用 .NET Core 对活动进行身份验证

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

如果选择使用 [.NET Core](/dotnet/core/index) 开发机器人，可使用 [Bot Framework 连接器](bot-builder-dotnet-connector.md)发送和接收来自机器人的[活动](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html)消息。 要使用连接器服务，需要为目标框架版本设置适当的身份验证模型。

Bot Framework Connector.AspNetCore 支持以下版本的 ASP.NET：
* 对于 .NET Core v1.1/AspNetCore1.x，使用 Microsoft.Bot.Connector.AspNetCore 1.x
* 对于 .NET Core v2.0/AspNetCore2.x，使用 Microsoft.Bot.Connector.AspNetCore 2.x

本文将展示如何为机器人所面向的特定框架设置身份验证模型。

## <a name="prerequisites"></a>先决条件

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows)。 安装要面向的 .NET Core 版本（例如 .NET Core v1.1 或 .NET Core v 2.0）。
* [注册机器人](~/bot-service-quickstart-registration.md)。 注册机器人以获取身份验证进程所需的 AppID 和密码。

## <a name="create-a-net-core-project"></a>创建 .NET Core 项目

要创建 .NET Core 项目，请执行以下操作：

1. 打开 Visual Studio 2017，再依次单击“文件”>“新建”>“项目...”。
2. 展开 Visual C# 节点并单击“.NET Core”。
3. 选择“ASP.NET Core Web 应用程序”项目类型，并填写项目信息（例如名称、位置和解决方案名称字段）。
4. 单击“确定”。
5. 确保项目面向的是所需的 .NET Core 和 ASP.NET Core 版本。 例如，下面的屏幕截图显示项目以 .NET Core 和 ASP.NET Core 2.0 为目标：

![创建 ASP.NET Core v2.0 项目](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > 如果要以 ASP.NET Core 2.0 作为目标，请确保计算机上已安装的版本不低于 2.x。

6. 选择 Web API 项目类型。
7. 单击“确定”以创建该项目  。

## <a name="download-the-nuget-package"></a>下载 NuGet 包

要使用 Bot Framework 连接器，请安装适合 .NET Core 版本的 NuGet 包。 要安装 NuGet 包，请执行以下操作：

1. 在解决方案资源管理器中，右键单击项目名称和“管理 NuGet 包...”
2. 单击“浏览”并搜索 Connector.ASPNetCore。 
3. 选择要作为目标的版本。 例如，下面的屏幕截图显示选择了 2.0.0.3 版本。 要安装 1.1.3.2 版本，请单击“版本”下拉框并选择 1.1.3.2 版本。
![适用于 ASP Net Core 版本 2.0.0.3 的 NuGet 包](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. 单击“安装”。

## <a name="update-the-appsettingsjson"></a>更新 appsettings.json

Bot Framework 连接器需使用 AppID 和密码来验证机器人的身份。 可在 Web 应用项目的 appsettings.json 中设置这些值。

> [!NOTE]
> 要查找机器人的 AppID 和 AppPassword，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="appsettingsjson-for-net-core-v11"></a>适用于 .NET Core v1.1 的 appsettings.json：

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>适用于 .NET Core v2.0 的 appsettings.json：

```json
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>更新 startup.cs 类

更新 startup.cs 类。 根据项目版本，更新相应的代码片段以进行身份验证。

### <a name="startupcs-for-net-core-v11"></a>适用于 .NET Core v1.1 的 startup.cs

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>适用于 .NET Core v2.0 的 startup.cs：

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>创建 MessagesController.cs 类

从解决方案资源管理器中添加一个新的名为 MessagesController.cs 的空类。 在 MessagesController.cs 类中，使用以下代码更新 Post 方法。 这将使机器人能够发送并接收用户的消息。

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
