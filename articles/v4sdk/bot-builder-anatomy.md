---
title: 机器人剖析 | Microsoft Docs
description: 剖析机器人的各个组成部分及其工作原理
keywords: ''
author: ivorb
ms.author: ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 238be22eff746edff30a5446d20991dfaedc945a
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928285"
---
# <a name="anatomy-of-a-bot"></a>机器人剖析

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

在[最基本的意义](bot-builder-basics.md)上讲，机器人是使用消息来与用户通信的聊天应用。 它遵循 Web 应用的基本结构，采用该应用的相应语言，并通过 Bot Framework SDK 构建在该应用的顶层。

机器人由多个不同的组件构成，可让你[发送消息](./bot-builder-howto-send-messages.md)和跟踪[状态](./bot-builder-storage-concept.md)。 可以通过[仿真器](../bot-service-debug-emulator.md)、[网络聊天](../bot-service-manage-test-webchat.md)或者在生产环境中通过某个[通道](bot-concepts.md)来与机器人通信。 

本文逐步讲解一个基本的聊天机器人，并探讨机器人的每个工作部件。

## <a name="files-created-for-echo-bot"></a>为聊天机器人创建的文件

每种编程语言为聊天机器人（在本示例中名为 **BasicEcho**）创建不同的文件，这些文件划分为三个不同的部分：系统部分、帮助器项和机器人核心。 下表列出了 C# 和 JavaScript 的主要文件。

| C# | JavaScript |
| --- | --- |
| `Properties > launchSettings.json` <br> `wwwroot > default.htm` <br> `appsettings.json` <br> `BasicEcho.bot` <br> `EchoBot.cs` <br> `EchoState.cs` <br> `Program.cs` <br> `readme.md` <br> `Startup.cs` | `.env` <br> `app.js` <br> `package.json` <br> `README.md` <br> `basicEcho.bot` |

下面按部分逐步讲解这些文件中的一些关键特征。 其中的某些文件带有自身的包含内容集，这些内容特定于机器人，并且是常规的编程库。 这些包含内容提供所需的函数集，以及可在应用程序中添加的函数。 本文不会介绍这些包含内容的详细信息，但如果你有兴趣，可以任意使用 Visual Studio 来查看定义及其所属的命名空间。

## <a name="system-section"></a>系统部分

系统部分包含使机器人作为标准应用程序正常运行所需的内容。 其中包括配置文件、适配器和一些 JSON 文件。 可以（并且通常应该）保留这些项，但某些配置文件可能需要特定的信息。

# <a name="ctabcs"></a>[C#](#tab/cs)

机器人是一种 [ASP.NET Core Web](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1) 应用程序框架。 在 [ASP.NET 基础知识](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x)中，可以看到 appsettings.json、Program.cs 和 Startup.cs 等文件包含类似的代码，下面将进行相关的讨论。 这些文件并非特定于机器人，所有 Web 应用都需要它们。 此处未复制其中一些文件中的代码，但运行机器人时会看到这些代码。

### <a name="appsettingsjson"></a>appsettings.json

此文件仅包含机器人的设置信息，主要是应用 ID 和密码，采用基本 JSON 格式。  使用[仿真器](../bot-service-debug-emulator.md)进行测试时不需要这些信息，因此可将其留空，但在生产环境中需要这些信息。

### <a name="programcs"></a>Program.cs

要使 ASP.NET Web 应用正常工作，必须提供此文件，其中指定了要运行的 `Startup` 类。

### <a name="startupcs"></a>Startup.cs

**Startup.cs** 包含稍后将会介绍的其他相关部分，但此处只是介绍其中的系统部分。

`Startup` 的构造函数指定应用设置和环境变量，并为 Web 应用创建配置。

`Configure` 方法通过指定应用需使用 Bot Framework 和其他几个文件来完成应用的配置。 使用 Bot Framework 的所有机器人需要该配置调用。

```cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace BasicEcho
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
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

        // ...
        // Definition of ConfigureServices covered in the next section
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
        }
    }
}

```

### <a name="launchsettingsjson-and-readmemd"></a>launchSettings.json 和 readme.md

**launchSettings.json** 仅包含 Web 应用的某些设置配置，**readme.md** 是有关聊天机器人的单行式简单说明。 这些文件的内容对于了解此机器人并不重要。 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

系统部分主要包含 **package.json**、**.env** 文件、**app.js** 和 **README.md**。 此处未复制某些文件中的代码，但运行机器人时会看到这些代码。

### <a name="packagejson"></a>package.json

**package.json** 指定机器人的依赖项及其关联的版本。 所有这些信息由模板和系统设置。

### <a name="env-file"></a>.env 文件

**.env** 文件指定机器人的配置信息，例如端口号、应用 ID 和密码等。 如果使用特定的技术或者在生产环境中使用此机器人，则需要将特定的密钥或 URL 添加到此配置。 但是，对于此聊天机器人，目前不需要执行任何操作；暂时可将应用 ID 和密码保持未定义状态。 

若要使用 **.env** 配置文件，需在模板中包含一个附加的包。  首先，从 npm 获取 `dotenv` 包：

`npm install dotenv`

然后，将以下行连同其他所需的库添加到机器人：

```javascript
const dotenv = require('dotenv');
```

### <a name="appjs"></a>app.js

`app.js` 文件的顶部包含可让机器人与用户通信和发送响应的服务器与适配器。 服务器将侦听 **.env** 配置中指定的端口，或者故障回复到 3978 以便与仿真器建立连接。 适配器充当机器人的指挥者，可以定向传入和传出的通信、身份验证，等等。 

```javascript
// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});
``` 

### <a name="readmemd"></a>README.md

**README.md** 提供构建机器人时涉及的某些方面的有用信息，例如，机器人的基本结构或“对话”的定义，但对于了解机器人并不重要。

---


## <a name="helper-items"></a>帮助器项

与任何程序一样，使用帮助器方法和其他定义可以简化代码。 在机器人中，有几个组成部分（例如 `.bot` 文件）属于该类别。这些组成部分对于机器人而言非常重要，但不一定要用作核心机器人逻辑的一部分。

### <a name="bot-file"></a>.bot 文件

`.bot` 文件包含可让[通道](bot-concepts.md)连接到机器人的信息，例如终结点、应用 ID 和密码。 此文件是在通过模板构建机器人时创建的，但你可以通过仿真器或其他工具创建自己的文件。

除非机器人的名称不是 *BasicEcho*，否则该文件的内容应与此处所示相匹配。 可能需要根据机器人的使用方式更改和更新此信息，但暂时不需要进行任何更改即可运行此聊天机器人。

```json
{
  "name": "BasicEcho",
  "secretKey": "",
  "services": [
    {
      "appId": "",
      "id": "http://localhost:3978/api/messages",
      "type": "endpoint",
      "appPassword": "",
      "endpoint": "http://localhost:3978/api/messages",
      "name": "BasicEcho"
    }
  ]
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="echostatecs"></a>EchoState.cs

**EchoState.cs** 包含机器人用来保持当前状态的简单类。 其中仅包含一个用于递增轮次计数器的 `int`。 下一部分将会更详细地介绍状态，但现在，我们只需了解 `EchoState` 是包含轮次计数的类。

```cs
namespace BasicEcho
{
    /// <summary>
    /// Class for storing conversation state.
    /// </summary>
    public class EchoState
    {
        public int TurnCount { get; set; } = 0;
    }
}
```

### <a name="defaulthtm"></a>default.htm

**default.htm** 是运行机器人时显示的网页，以 `html` 编写。 其中显示用于连接机器人以及如何与其交互的有用信息，但是，该页的内容不会影响机器人的行为。 运行机器人时，会看到代码弹出。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="required-libraries"></a>所需的库

在 `app.js` 文件的最上面，可以看到所需的一系列模块或库。 通过这些模块可以访问可能要包含在应用程序中的函数集。 

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');
```

---

## <a name="core-of-the-bot"></a>机器人的核心

最后是我们真正感兴趣的部分！ 机器人的核心定义机器人如何与用户交互，包括中间件和机器人逻辑。

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="middleware"></a>中间件

在 **Startup.cs** 中，有一个名为 `ConfigureServices` 的方法。 此方法设置凭据提供程序和添加中间件。

此处添加的[中间件](bot-builder-concept-middleware.md)执行两项操作。 第一项操作是异常处理，可让机器人正常失败。 此操作以 lambda 表达式的内联形式进行定义，它只是将异常列显在终端上，并告知用户出现了问题。

第二个中间件使用紧靠在它前面定义的 `MemoryStorage` 来保留状态。 此状态定义为 `ConversationState`，仅表示它正在保留聊天的状态。 此中间件使用 `EchoState`（在轮次计数器中定义的类）来存储所需的信息。

``` cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot.
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facilitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent.
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // The File data store, shown here, is suitable for bots that run on 
        // a single machine and need durable state across application restarts.

        // IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());

        // For production bots use the Azure Table Store, Azure Blob, or 
        // Azure CosmosDB storage provides, as seen below. To include any of 
        // the Azure based storage providers, add the Microsoft.Bot.Builder.Azure 
        // Nuget package to your solution. That package is found at:
        //      https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/

        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureTableStorage("AzureTablesConnectionString", "TableName");
        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage("AzureBlobConnectionString", "containerName");

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
```

### <a name="bot-logic"></a>机器人逻辑

主要机器人逻辑在机器人的 `OnTurn` 处理程序中定义。 在 `OnTurn` 中以参数形式提供了一个[上下文](./bot-builder-concept-activity-processing.md#turn-context)变量，该变量提供有关传入的[活动](bot-builder-concept-activity-processing.md)、聊天等的信息。

传入的活动可以属于[不同的类型](../bot-service-activities-entities.md#activity-types)，因此，让我们先检查机器人是否收到了消息。 如果收到的内容是消息，则我们创建一个状态变量用于保存机器人聊天的当前信息。 接下来，我们递增用于跟踪轮次计数的 `int`，然后，将计数和发送的消息回显给用户。

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace BasicEcho
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, bumps the 
        /// turn conversation 'Turn' count, and then echoes the users typing
        /// back to them. 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
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
    }    
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="middleware"></a>中间件

在 **app.js** 中，我们使用此机器人的[中间件](bot-builder-concept-middleware.md)，通过在 `botbuilder` 的顶部定义为所需模块之一的 `MemoryStorage` 来保留状态。 此状态定义为 `ConversationState`，仅表示它正在保留聊天的状态。 `ConversationState` 将存储所需的信息，在本例中，该信息只是内存中的一个轮次计数器。

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

### <a name="bot-logic"></a>机器人逻辑

*processActivity* 中的第三个参数是在收到的[活动](bot-builder-concept-activity-processing.md)由适配器预先处理并通过任何中间件路由后，将要调用的以执行机器人逻辑的函数处理程序。 可以使用作为参数传递给函数处理程序的[上下文](./bot-builder-concept-activity-processing.md#turn-context)变量来提供有关传入的活动、发送方和接收方、通道、聊天等的信息。

我们先检查机器人是否收到了消息。 如果机器人未收到消息，则我们回显收到的活动类型。 接下来，创建一个状态变量用于保存机器人聊天的信息。 然后，将计数变量设置为 0（如果未定义，启动机器人时存在这种情况），或者每收到新的消息，就递增计数变量。 最后，将计数和发送的消息回显给用户。

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---
