---
title: 创建 Direct Line 机器人和客户端 | Microsoft Docs
description: 了解如何使用用于 .NET 的 Bot Builder SDK V4 版创建 Direct Line 机器人和客户端。
keywords: direct line 机器人, direct line 客户端, 自定义通道, 基于控制台, 发布
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c13733af0f6b26654952a0aab190f6d8cb06d059
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999974"
---
# <a name="create-a-direct-line-bot-and-client"></a>创建 Direct Line 机器人和客户端

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Microsoft Bot Framework Direct Line 机器人是可以通过自行设计的自定义客户端运行的机器人。 Direct Line 机器人非常类似于普通机器人， 只是不需要使用提供的通道。

可以随意编写 Direct Line 客户端。 可以编写 Android 客户端、iOS 客户端甚至基于控制台的客户端应用程序。

本主题介绍如何创建并部署 Direct Line 机器人，以及如何创建并运行基于控制台的 Direct Line 客户端应用。

## <a name="create-your-bot"></a>创建机器人

每种语言按不同的方法来创建机器人。 Javascript 在 Azure 中创建机器人，然后修改代码，而 C# 则在本地创建机器人，然后将其发布到 Azure，但两种方法都是有效的，可以用于任一语言。 若要详细了解如何将机器人发布到 Azure，请参阅[在 Azure 中部署机器人](../bot-builder-howto-deploy-azure.md)。

# <a name="ctabcscreatebot"></a>[C#](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>在 Visual Studio 中创建解决方案

若要在 Visual Studio 2015 或更高版本中为 Direct Line 机器人创建解决方案，请执行以下操作：

1. 在 **Visual C#** > **.NET Core** 中创建新的 **ASP.NET Core Web 应用程序**。

1. 输入 **DirectLineBotSample** 作为名称，然后单击“确定”。

1. 确保选择“.NET Core”和“ASP.NET Core 2.0”，并选择“空”项目模板，然后单击“确定”。

#### <a name="add-dependencies"></a>添加依赖项

1. 在“解决方案资源管理器”中右键单击“依赖项”，然后选择“管理 NuGet 包”。

1. 单击“浏览”，然后确保“包括预发行版”复选框已选中。

1. 搜索并安装以下 NuGet 包：
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Microsoft.Rest.ClientRuntime
    - Newtonsoft.Json

### <a name="create-the-appsettingsjson-file"></a>创建 appsettings.json 文件

appsettings.json 文件会包含 Microsoft 应用 ID、应用密码和数据连接字符串。 由于此机器人不会存储状态信息，因此 Data Connection 字符串保留为空。 如果只使用 Bot Framework Emulator，则所有这些字符串都可以保留为空。

若要创建 **appsettings.json** 文件，请执行以下操作：

1. 右键单击“DirectLineBotSample”项目，选择“添加” > “新建项”。

1. 在 **ASP.NET Core** 中，单击“ASP.NET 配置文件”。

1. 键入“appsettings.json”作为名称，然后单击“添加”。

将 appsettings.json 文件的内容替换为以下代码：

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

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>编辑 Startup.cs

将 Startup.cs 文件的内容替换为以下代码：

**注意**：**DirectBot** 会在下一步定义，因此可以忽略它的红色下划线。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>创建 DirectBot 类

DirectBot 类包含此机器人的大部分逻辑。

1. 右键单击 **DirectLineBotSample** > **，然后选择“添加”** > **“新建项”**。

1. 在 **ASP.NET Core** 中单击“类”。

1. 键入“DirectBot.cs”作为名称，然后单击“添加”。

将 DirectBot.cs 文件的内容替换为以下代码：

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**注意：** 现有的 Program.cs 文件不需更改。

若要验证一切是否正常，请按“F6”生成此项目。 应该没有警告或错误。

### <a name="publish-the-bot-from-visual-studio"></a>从 Visual Studio 发布机器人

1. 在 Visual Studio 的解决方案资源管理器中，右键单击“DirectLineBotSample”项目，然后单击“设为启动项目”。

    ![Visual Studio 的“发布”页](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. 再次右键单击“DirectLineBotSample”，然后单击“发布”。 此时会显示 Visual Studio 的“发布”页。

1. 如果已经有此机器人的发布配置文件，请将其选中，单击“发布”按钮，然后转到下一部分。

1. 若要创建新的发布配置文件，请选择“Microsoft Azure 应用服务”图标。

1. 选择“新建”。

1. 单击“上传”按钮 发布。 此时会显示“创建应用服务”对话框。

    ![“创建应用服务”对话框](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - 对于“应用名称”，请为其提供一个可供以后查找的名称。 例如，可将电子邮件名称添加到应用名称的开头。

    - 验证是否使用了正确的订阅。

    - 对于“资源组”，请验证是否使用了正确的资源组。 资源组可在机器人的“概览”边栏选项卡上找到。 **注意：** 不正确的资源组很难更正。

    - 验证是否使用了正确的应用服务计划。
    
1. 单击“创建”  按钮。 Visual Studio 就会开始部署机器人。

发布机器人以后，会显示一个浏览器，其中有机器人的 URL 终结点。

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>在 Microsoft Azure 上创建“机器人通道注册”机器人

Direct Line 机器人可以托管在任何平台上。 在此示例中，该机器人将托管在 Microsoft Azure 上。 

若要在 Microsoft Azure 上创建机器人，请执行以下操作：

1. 在 Microsoft Azure 门户中，单击“+ 创建资源”，然后搜索“机器人通道注册”。

1. 单击“创建”。 此时会显示“机器人通道注册”边栏选项卡。

    ![“机器人通道注册”边栏选项卡，包含“机器人名称”、“订阅”、“资源组”、“位置”、“定价层”、“消息传送终结点”等字段。](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. 在“机器人通道注册”边栏选项卡中，输入“机器人名称”、“订阅”、“资源组”、“位置”和“定价层”。

1. 将“消息传送终结点”保留为空。 此值会稍后进行填充。

1. 单击“Microsoft 应用 ID 和密码”，然后单击“自动创建应用 ID 和密码”。

1. 勾选“固定到仪表板”复选框。

1. 单击“创建”  按钮。

部署需数分钟，但完成后就会显示在仪表板中。

### <a name="update-the-appsettingsjson-file"></a>更新 appsettings.json 文件

1. 单击仪表板中的机器人，或者转到新的资源，方法是先单击“所有资源”，然后搜索机器人通道注册的名称。

1. 单击“概览”。

1. 将资源组的名称复制到 **DirectLineClientSample** 项目的 **Program.cs** 中的 **botId** 字符串。

1. 单击“设置”，然后单击靠近“Microsoft 应用 ID”字段的“管理”。

1. 复制应用程序 ID 并将其粘贴到 **appsettings.json** 文件的“MicrosoftAppId”字段中。

1. 单击“生成新密码”按钮。

1. 复制新密码并将其粘贴到 **appsettings.json** 文件的“MicrosoftAppPassword”字段中。

### <a name="set-the-messaging-endpoint"></a>设置消息传送终结点

若要向机器人添加终结点，请执行以下操作：

1. 从发布机器人后弹出的浏览器中复制 URL 地址。

    ![机器人通道注册的设置边栏选项卡，突出显示了消息传送终结点](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. 打开机器人的“设置”边栏选项卡。

1. 将地址粘贴到“消息传送终结点”中。

1. 编辑该地址，使它以“https://”开头，以“/api/messages”结尾。 例如，如果从浏览器复制的地址为“http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net”，则将其编辑为“https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages”。

1. 单击“设置”边栏选项卡上的“保存”按钮。

**注意：** 如果发布失败，则可能需要停止 Azure 上的机器人，然后才能发布对机器人所做的更改。

1. 找到应用服务名称（位于“https://”和“.azurewebsites.net”之间）。

1. 在 Azure 门户的“所有资源”中搜索该资源。

1. 单击应用服务资源。 此时会显示“应用服务”边栏选项卡。

1. 单击“应用服务”边栏选项卡上的“停止”按钮。

1. 在 Visual Studio 中再次发布机器人。

1. 单击“应用服务”边栏选项卡上的“启动”按钮。

### <a name="test-your-bot-in-webchat"></a>在网上聊天中测试机器人

若要验证机器人是否工作，请在网上聊天中检查机器人：

1. 在机器人的“机器人通道注册”边栏选项卡中单击“设置”，然后单击“在网上聊天中测试”。

1. 键入“你好”。 机器人会使用一条欢迎消息响应。

1. 键入“为我显示一张英雄卡”。 机器人会显示一张英雄卡。

1. 键入“为我发送一张 Botframework 图像”。 机器人会显示 Bot Framework 文档中的一张图像。

1. 键入任何其他内容，机器人会回复“你说”以及引号引起来的消息。

### <a name="troubleshooting"></a>故障排除

如果机器人不工作，请验证或重新输入 Microsoft 应用 ID 和密码。 即使以前已复制过，但仍请将机器人通道注册的设置边栏选项卡上的 Microsoft 应用 ID 与 appsettings.json 文件中“MicrosoftAppId”字段的值进行对照检查。

验证密码或者创建并使用新密码： 

1. 在“机器人通道注册”边栏选项卡上单击“Microsoft 应用 ID”字段旁边的“管理”。

1. 登录到“应用程序注册门户”。

1. 验证密码的头三个字母是否与 **appsettings.json** 文件中的“MicrosoftAppPassword”字段匹配。 

1. 如果值不匹配，请生成新密码并将其值存储在 **appsettings.json** 文件的“MicrosoftAppPassword”字段中。

# <a name="javascripttabjscreatebot"></a>[JavaScript](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>在 Microsoft Azure 上创建 Web 应用机器人

若要在 Microsoft Azure 上创建机器人，请执行以下操作： 

1. 在 Microsoft Azure 门户中，单击“创建资源”，然后搜索“Web 应用机器人”。

1. 单击“创建”。 此时会显示“Web 应用机器人”边栏选项卡。

![Web 应用机器人注册](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. 在“Web 应用机器人”边栏选项卡中，输入“机器人名称”、“订阅”、“资源组”、“位置”、“定价层”、“应用名称”。

1. 选择要使用的机器人模板。 “SDK 版本: SDK v4”和“SDK 语言: Node.js” 

1. 选择基本 v4 预览模板。 

1. 选择“应用服务计划/位置”、“Azure 存储”和“Application Insights 位置”。

1. 勾选“固定到仪表板”复选框。

1. 单击“创建”按钮。

1. 等待机器人部署。 由于勾选了“固定到仪表板”复选框，因此机器人会显示在仪表板上。

### <a name="edit-your-direct-line-bot"></a>编辑 Direct Line 机器人

现在，请向回显机器人添加更多的功能。

1. 在“Web 应用机器人”边栏选项卡中，单击“生成”。 

1. 单击“下载 zip 文件”。 

1. 将 zip 文件解压缩到本地目录。

1. 导航到解压缩的文件夹，并在偏好的 IDE 中打开源文件。

1. 在 app.js 文件中，复制并粘贴以下代码。

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

准备就绪后，可以将源发布回 Azure。 执行这些步骤，了解如何将机器人发布回 [Azure](../bot-service-build-download-source-code.md)

---


## <a name="create-the-console-client-app"></a>创建控制台客户端应用

# <a name="ctabcsclientapp"></a>[C#](#tab/csclientapp)

控制台客户端应用程序在两个线程中运行。 主线程接受用户输入并向机器人发送消息。 第二个线程每秒轮询机器人一次，目的是接收机器人的消息，然后显示接收的消息。

若要创建控制台项目，请执行以下操作：

1. 在解决方案资源管理器中右键单击“解决方案‘DirectLineBotSample’”。

1. 选择“添加” > “新建项目”。

1. 在“Visual C#”>“Windows 经典桌面”*中，选择“控制台应用(.NET Framework)”。
 
    **注意：** 请勿选择“控制台应用(.NET Core)”。

1. 输入 **DirectLineClientSample** 作为名称。

1. 单击“确定”。

### <a name="add-the-nuget-packages-to-the-console-app"></a>向控制台应用添加 NuGet 包

1. 右键单击“引用”。

1. 单击“管理 NuGet 包”。

1. 单击“浏览”。 确保“包括预发行版”复选框已选中。

1. 搜索并安装以下 NuGet 包：
    - Microsoft.Bot.Connector.DirectLine (v3.0.2)
    - Newtonsoft.Json

### <a name="edit-the-programcs-file"></a>编辑 Program.cs 文件

将 DirectLineClientSample **Program.cs** 文件的内容替换为以下代码：

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

若要验证一切是否正常，请按“F6”生成此项目。 应该没有警告或错误。

# <a name="javascripttabjsclientapp"></a>[JavaScript](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>创建 Direct Line 客户端 

部署 Web 应用机器人后即可创建 Direct Line 客户端。

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

接下来，请安装 npm 中的这些包

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

创建 **app.js** 文件。 将以下代码复制并粘贴到该文件中。

我们会在下一步获取 directLineSecret。
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>配置 Direct Line 通道

添加 Direct Line 通道会使此机器人成为 Direct Line 机器人。

若要配置 Direct Line 通道，请执行以下操作：

![“连接到通道”边栏选项卡，突出显示了“Direct Line 通道”按钮。](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. 在“机器人通道注册”边栏选项卡上，单击“通道”。 此时会显示“连接到通道”边栏选项卡。

    ![“配置 Direct Line”边栏选项卡，显示了两个密钥。](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. 在“连接到通道”边栏选项卡上，单击“配置 Direct Line 通道”按钮。 

1. 如果未选中 **3.0**，请将复选标记置于复选框中。

1. 针对至少一个密钥单击“显示”。

1. 复制其中一个密钥，然后将其粘贴到客户端应用的“directLineSecret”字符串中。

1. 单击“完成”。

## <a name="run-the-client-app"></a>运行客户端应用

# <a name="ctabcsrunclient"></a>[C#](#tab/csrunclient)

机器人现在已做好与 Direct Line 控制台客户端应用程序通信的准备。 若要运行控制台应用，请执行以下操作：

1. 在 Visual Studio 中右键单击“DirectLineClientSample”项目，然后选择“设为启动项目”。

1. 检查 **DirectLineClientSample** 项目中的 **Program.cs** 文件。

    - 验证 Direct Line 密码是否在 **directLineSecret** 字符串中。

1. 如果 **directLineSecret** 不正确，请转到 Azure 门户，依次单击 Direct Line 机器人、“通道”、“配置 Direct Line 通道”（或“编辑”），显示一个密钥，然后将该密钥复制到 **directLineSecret** 字符串中。

    - 验证资源组是否在 **botId** 字符串中。

1. 如果不在其中，请通过“概览”边栏选项卡将**资源组**复制并粘贴到 **botId** 字符串中。

1. 按“F5”或“开始调试”。

控制台客户端应用此时会启动。 若要测试此应用，请执行以下操作： 

1. 输入“你好”。 机器人会显示“你说‘你好’”。

1. 输入“为我显示一张英雄卡”。 机器人会显示一张英雄卡。

1. 输入“为我发送一张 Botframework 图像”。 机器人会启动一个浏览器，用于显示 Bot Framework 文档中的一张图像。

1. 输入任何其他内容，机器人会回复“你说”以及引号引起来的消息。

# <a name="javascripttabjsrunclient"></a>[JavaScript](#tab/jsrunclient)

若要运行此示例，需运行 DirectLineClient 应用。

1. 打开 CMD 控制台，以 CD 方式转到 DirectLineClient 目录

1. 运行 `node app.js`

若要测试自定义消息，请在客户端的控制台中键入“为我显示一张英雄卡”或“为我发送一张 Botframework 图像”，然后就会看到以下结果。

![结果](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>后续步骤
