---
title: 机器人的工作原理 | Microsoft Docs
description: 介绍 Bot Framework SDK 中的活动和 http 工作原理。
keywords: 聊天流, 轮次, 机器人聊天, 对话, 提示, 瀑布, 对话集
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 94a3459760c8f0f14886a068d082dafeb9530b19
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215541"
---
# <a name="how-bots-work"></a>机器人的工作原理

[!INCLUDE[applies-to](../includes/applies-to.md)]

机器人是用户使用文本、图形（例如卡片或图像）或语音通过聊天的方式与之进行交互的应用。 用户与机器人之间的每次交互会生成一个活动。  Bot Framework Service 是 Azure 机器人服务的组件，可在连接机器人的用户应用（例如 Facebook、Skype、Slack 等，称为“通道”）与机器人之间发送信息。  每个通道可以在发送的活动中包含其他信息。 在创建机器人之前，必须了解机器人如何使用活动对象来与其用户通信。 首先，让我们了解在运行简单的聊天机器人时交换的活动。 

![活动示意图](media/bot-builder-activity.png)

此处演示了两种活动类型：聊天更新和消息。  

当某一方参与聊天时，Bot Framework Service 可以发送聊天更新。 例如，使用 Bot Framework Emulator 开始聊天时，会看到两个聊天更新活动（一个活动指出用户正在参与聊天，另一个活动指出机器人正在参与聊天）。 若要区分这些聊天更新活动，请检查“已添加成员”属性是否包含除机器人以外的成员。  

消息活动承载参与方之间的聊天信息。 在聊天机器人示例中，消息活动承载简单的文本，通道呈现这些文本。 消息活动也可以承载要讲述的文本、建议的操作或要显示的卡片。

在此示例中，机器人创建并发送了一个消息活动，以响应它收到的入站消息活动。 但是，机器人可通过其他方式响应收到的消息活动；机器人经常通过在消息活动中发送一些欢迎文本来响应聊天更新活动。 在[欢迎用户](bot-builder-welcome-user.md)中可以找到更多信息。

### <a name="http-details"></a>HTTP 详细信息

活动通过 HTTP POST 请求从 Bot Framework Service 抵达机器人。 机器人使用 200 HTTP 状态代码响应入站 POST 请求。 从机器人发送到通道的活动通过单独的 HTTP POST 发送到 Bot Framework Service。 而此请求也会通过 200 HTTP 状态代码得到确认。

协议不会指定这些 POST 请求及其确认的发送顺序。 但是，为了适应常见的 HTTP 服务框架，这些请求通常会嵌套，这意味着，出站 HTTP 请求将在入站 HTTP 请求的范围内从机器人发出。 上图演示了此模式。 由于两个不同的 HTTP 连接相继出现，安全模型必须能够应对这种情况。

### <a name="defining-a-turn"></a>定义轮次

在聊天时，人们通常一次说一句话，并且会轮流说话。 使用机器人时，通常是由机器人对用户输入进行响应。 在 Bot Framework SDK 中，一轮通话既包含用户传给机器人的活动，又包含机器人发回用户的作为即时响应的活动。  可以将一个轮次视为给定活动抵达时的相关处理。

轮次上下文对象提供有关活动的信息，例如发送方和接收方、通道，以及处理该活动所需的其他数据。  使用该对象还能在机器人的不同层中处理轮次期间添加信息。

轮次上下文是 SDK 中最重要的抽象之一。 轮次上下文不仅将入站活动传递到所有中间件组件将和应用程序逻辑，而且还提供所需的机制让中间件组件和应用程序逻辑发送出站活动。

## <a name="the-activity-processing-stack"></a>活动处理堆栈

让我们深入查看上图，并重点关注消息活动的抵达。

![活动处理堆栈](media/bot-builder-activity-processing-stack.png)

在上述示例中，机器人使用包含相同文本消息的另一个消息活动回复了原始消息活动。 处理工作从 HTTP POST 请求（包含以 JSON 有效负载形式传递的活动信息）抵达 Web 服务器时开始。 在 C# 中，这通常是一个 ASP.NET 项目；在 JavaScript Node.js 项目中，这可能是 Express 或 Restify 等常用框架之一。

适配器（SDK 的集成组件）是 SDK 运行时的核心。  活动以 JSON 形式承载在 HTTP POST 正文中。 将反序列化此 JSON 以创建 Activity 对象，然后通过调用 *process activity* 方法将此对象传递给适配器。 收到活动时，适配器会创建轮次上下文并调用中间件。  

如前所述，轮次上下文提供一个机制来让机器人发送出站活动（主要是为了响应入站活动）。 为此，轮次上下文提供 _send、update 和 delete activity_ 响应方法。 每个响应方法都在异步进程中运行。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="activity-handlers"></a>活动处理程序

当机器人收到某个活动时，它会将该活动传递给其活动处理程序。  幕后还有一个称为“轮次处理程序”的基本处理程序。  所有活动都通过轮次处理程序路由。 然后，对于收到的任何类型的活动，该轮次处理程序会调用各个活动处理程序。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

例如，如果机器人收到了某个消息活动，轮次处理程序将会看到该传入的活动，并将其发送到 `OnMessageActivityAsync` 活动处理程序。 

生成机器人时，用于处理和响应消息的机器人逻辑将进入此 `OnMessageActivityAsync` 处理程序。 同样，用于处理正在添加到聊天中的成员的逻辑将进入 `OnMembersAddedAsync` 处理程序，每次将成员添加到聊天时，都会调用该处理程序。

若要实现这些处理程序的逻辑，需要根据下面的[机器人逻辑](#bot-logic)部分所述，在机器人中重写这些方法。 其中的每个处理程序没有基实现，因此，只需在重写中添加所需的逻辑。

在某些情况下，需要在轮次结束时重写基轮次处理程序，例如[保存状态](bot-builder-concept-state.md)。 执行此操作时，请务必先调用 `await base.OnTurnAsync(turnContext, cancellationToken);`，以确保 `OnTurnAsync` 的基实现在其他代码之前运行。 除此之外，该基实现还负责调用剩余的活动处理程序，例如 `OnMessageActivityAsync`。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

例如，如果机器人收到了某个消息活动，轮次处理程序将会看到该传入的活动，并将其发送到 `onMessage` 活动处理程序。

生成机器人时，用于处理和响应消息的机器人逻辑将进入此 `onMessage` 处理程序。 同样，用于处理正在添加到聊天中的成员的逻辑将进入 `onMembersAdded` 处理程序，每次将成员添加到聊天时，都会调用该处理程序。

若要实现这些处理程序的逻辑，需要根据下面的[机器人逻辑](#bot-logic)部分所述，在机器人中重写这些方法。 对于其中的每个处理程序，请定义机器人逻辑，**并务必在最后调用 `next()`** 。 调用 `next()` 可确保运行下一个处理程序。

只有在非常规的情况下才需要重写基轮次处理程序，因此，在尝试重写时请保持谨慎。 对于在轮次结束时要执行的[保存状态](bot-builder-concept-state.md)等操作，可以使用一个名为 `onDialog` 的特殊处理程序。 `onDialog` 处理程序在剩余的处理程序运行之后才最后运行，与特定的活动类型无关。 与使用上述所有处理程序时一样，请务必调用 `next()` 来确保完成剩余的过程。

---

## <a name="middleware"></a>中间件

中间件非常类似于其他任何消息传送中间件，由一组线性组件构成，其中每个组件按顺序执行，并有机会对活动运行。 中间件管道的最后一个阶段是回调已由应用程序注册到适配器 *process activity* 方法的机器人类中的轮次处理程序。 该轮次处理程序通常是 C# 中的 `OnTurnAsync` 和 JavaScript 中的 `onTurn`。

轮次处理程序采用轮次上下文作为参数。通常，在轮次处理程序函数内部运行的应用程序逻辑将处理入站活动的内容，在响应中生成一个或多个活动，并使用轮次上下文中的 *send activity* 函数发出这些活动。 调用轮次上下文中的 *send activity* 会导致针对出站活动调用中间件组件。 中间件组件在机器人的轮次处理程序函数之前和之后执行。 执行在本质上是嵌套的，因此，有时称作“俄罗斯套娃”或类似叫法。 有关中间件的更深入信息，请参阅[中间件主题](~/v4sdk/bot-builder-concept-middleware.md)。

## <a name="bot-structure"></a>机器人结构

以下部分介绍 EchoBot 的关键片段，可以使用针对 [**CSharp**](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [**JavaScript**](../javascript/bot-builder-javascript-quickstart.md) 提供的模板轻松创建这些片段。 

<!--Need to add section calling out the controller in code, and explaining it further-->

机器人是一个 Web 应用程序，我们提供了适用于每种语言的模板。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

VSIX 模板生成 [ASP.NET MVC Core](https://dotnet.microsoft.com/apps/aspnet/mvc) Web 应用。 在 [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) 基础知识中，可以看到 **Program.cs** 和 **Startup.cs** 等文件包含类似的代码。 这些文件并非特定于机器人，所有 Web 应用都需要它们。

### <a name="appsettingsjson-file"></a>appsettings.json 文件

**appsettings.json** 文件指定机器人的配置信息，例如应用 ID 、密码等。 如果使用特定的技术或者在生产环境中使用此机器人，则需要将特定的密钥或 URL 添加到此配置。 但是，对于此聊天机器人，目前不需要执行任何操作；暂时可将应用 ID 和密码保持未定义状态。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- TODO: Update this aka link to point to samples/javascript_nodejs/02.echobot (instead of samples/javascript_nodejs/02.a.echobot) once work-in-progress is merged into master. -->

Yeoman 生成器创建 [restify](http://restify.com/) 类型的 Web 应用程序。 如果在相应文档中查看 restify 快速入门，会看到类似于生成的 **index.js** 文件的应用。 我们将介绍模板生成的一些关键文件。 此处未复制某些文件中的代码，但在运行机器人时会看到这些文件，或者可以参考 [Node.js echobot](https://aka.ms/js-echobot-sample) 示例。

### <a name="packagejson"></a>package.json

**package.json** 指定机器人的依赖项及其关联的版本。 所有这些信息由模板和系统设置。

### <a name="env-file"></a>.env 文件

**.env** 文件指定机器人的配置信息，例如端口号、应用 ID 和密码等。 如果使用特定的技术或者在生产环境中使用此机器人，则需要将特定的密钥或 URL 添加到此配置。 但是，对于此聊天机器人，目前不需要执行任何操作；暂时可将应用 ID 和密码保持未定义状态。

若要使用 **.env** 配置文件，需在模板中包含一个附加的包。  首先，从 npm 获取 `dotenv` 包：

`npm install dotenv`

---

### <a name="bot-logic"></a>机器人逻辑

机器人逻辑处理来自一个或多个通道的传入活动，并在响应中生成传出活动。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

主要机器人逻辑在机器人代码（此处名为 `Bots/EchoBot.cs`）中定义。 `EchoBot` 派生自 `AcitivityHandler`，后者又派生自 `IBot` 接口。 `ActivityHandler` 为不同类型的活动定义各种处理程序，例如，此处定义了两个处理程序：`OnMessageActivityAsync` 和 `OnMembersAddedAsync`。 这些方法受保护，但可将其覆盖，因为它们派生自 `ActivityHandler`。

`ActivityHandler` 中定义的处理程序为：

| 事件 | Handler | 说明 |
| :-- | :-- | :-- |
| 已收到任一活动类型 | `OnTurnAsync` | 根据收到的活动类型调用其他处理程序之一。 |
| 已收到消息活动 | `OnMessageActivityAsync` | 重写此方法可以处理 `Message` 活动。 |
| 已收到聊天更新活动 | `OnConversationUpdateActivityAsync` | 收到 `ConversationUpdate` 活动时，如果除机器人以外的成员加入或退出聊天，则调用某个处理程序。 |
| 非机器人成员加入了聊天 | `OnMembersAddedAsync` | 重写此方法可以处理加入聊天的成员。 |
| 非机器人成员退出了聊天 | `OnMembersRemovedAsync` | 重写此方法可以处理退出聊天的成员。 |
| 已收到事件活动 | `OnEventActivityAsync` | 收到 `Event` 活动时，调用特定于事件类型的处理程序。 |
| 已收到令牌响应事件活动 | `OnTokenResponseEventAsync` | 重写此方法可以处理令牌响应事件。 |
| 已收到非令牌响应事件活动 | `OnEventAsync` | 重写此方法可以处理其他类型的事件。 |
| 已收到其他活动类型 | `OnUnrecognizedActivityTypeAsync` | 重写此方法可以处理未经处理的任何活动类型。 |

这些不同的处理程序具有一个 `turnContext`，用于提供有关对应于入站 HTTP 请求的传入活动的信息。 活动可以是各种类型，因此，每个处理程序在其轮次上下文参数中提供一个强类型化的活动，在大多数情况下，始终会处理 `OnMessageActivityAsync`。

与在此框架的 4.x 旧版中一样，还有一个选项可以实现公共方法 `OnTurnAsync`。 目前，此方法的基实现会处理错误检查，然后根据传入活动的类型调用每个特定的处理程序（例如本示例中定义的两个处理程序）。 在大多数情况下，可以不用理会该方法并使用单个处理程序，但如果你的情况要求使用 `OnTurnAsync` 的自定义实现，则仍可以考虑该方法。

> [!IMPORTANT]
> 如果重写 `OnTurnAsync` 方法，则需要调用 `base.OnTurnAsync` 以获取用于调用其他所有 `On<activity>Async` 处理程序的基实现，或自行调用这些处理程序。 否则不会调用这些处理程序，并且不会运行该代码。

在本示例中，我们将欢迎新用户，或者回显用户使用 `SendActivityAsync` 调用发送的消息。 出站活动对应于出站 HTTP POST 请求。

```cs
public class MyBot : ActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);
    }

    protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        foreach (var member in membersAdded)
        {
            await turnContext.SendActivityAsync(MessageFactory.Text($"welcome {member.Name}"), cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

主要机器人逻辑在机器人代码（此处名为 `bots\echoBot.js`）中定义。 `EchoBot` 派生自 `AcitivityHandler`。 `ActivityHandler` 为不同类型的活动定义各种处理程序，你可以通过提供其他逻辑（例如，此处使用了 `onMessage` 和 `onConversationUpdate`）来修改机器人的行为。

`ActivityHandler` 中定义的处理程序为：

| 事件 | Handler | 说明 |
| :-- | :-- | :-- |
| 已收到任一活动类型 | `onTurn` | 根据收到的活动类型调用其他处理程序之一。 |
| 已收到消息活动 | `onMessage` | 提供一个相关的函数用于处理 `Message` 活动。 |
| 已收到聊天更新活动 | `onConversationUpdate` | 收到 `ConversationUpdate` 活动时，如果除机器人以外的成员加入或退出聊天，则调用某个处理程序。 |
| 非机器人成员加入了聊天 | `onMembersAdded` | 提供一个相关的函数用于处理加入聊天的成员。 |
| 非机器人成员退出了聊天 | `onMembersRemoved` | 提供一个相关的函数用于处理退出聊天的成员。 |
| 已收到事件活动 | `onEvent` | 收到 `Event` 活动时，调用特定于事件类型的处理程序。 |
| 已收到令牌响应事件活动 | `onTokenResponseEvent` | 提供一个相关的函数用于处理令牌响应事件。 |
| 已收到其他活动类型 | `onUnrecognizedActivityType` | 提供一个相关的函数用于处理未经处理的任何活动类型。 |
| 活动处理程序已完成 | `onDialog` | 提供一个相关的函数，用于处理在剩余活动处理程序都已完成之后，应在轮次结束时完成的工作。 |

在每个轮次，我们先检查机器人是否收到了消息。 当我们收到用户的消息时，会回显他们发送的消息。

```javascript
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async (context, next) => {
            await context.sendActivity(`You said '${ context.activity.text }'`);
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
        this.onConversationUpdate(async (context, next) => {
            await context.sendActivity('[conversationUpdate event detected]');
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
    }
}

module.exports.MyBot = MyBot;
```

---

### <a name="access-the-bot-from-your-app"></a>从应用访问机器人

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="set-up-services"></a>设置服务

`Startup.cs` 文件中的 `ConfigureServices` 方法加载连接的服务、从 `appsettings.json` 或 Azure Key Vault 加载这些服务的密钥（如果有）、连接状态，等等。 此处，我们将在服务中添加 MVC 并设置兼容版本，然后设置可以通过依赖项注入在机器人控制器中使用的适配器和机器人。

<!-- want to explain the singleton vs transient here?-->

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the credential provider to be used with the Bot Framework Adapter.
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

`Configure` 方法通过指定应用需使用 MVC 和其他几个文件来完成应用的配置。 使用 Bot Framework 的所有机器人需要该配置调用，但是，在生成机器人时，该调用已在示例或 VSIX 模板中定义。 应用启动时，运行时会调用 `ConfigureServices` 和 `Configure`。

#### <a name="bot-controller"></a>机器人控制器

采用标准 MVC 结构的控制器可让你确定消息和 HTTP POST 请求的路由方式。 对于我们的机器人，我们会将传入的请求传递到前面[活动处理堆栈](#the-activity-processing-stack)部分所述的适配器 *process async activity* 方法。 在该调用中，指定可能需要的机器人和其他任何授权信息。

控制器实现 `ControllerBase`，保存 `Startup.cs` 中设置的适配器和机器人（此处通过依赖项注入来使用该适配器和机器人），并在收到传入的 HTTP POST 时，将所需的信息传递给机器人。

在此处可以看到路由和控制器属性继续处理的类。 这些属性可以帮助框架适当路由消息，以及了解要使用哪个控制器。 如果更改路由属性中的值，会更改仿真器或其他通道用来访问机器人的终结点。

```cs
// This ASP Controller is created to handle a request. Dependency Injection will provide the Adapter and IBot
// implementation at runtime. Multiple different IBot implementations running at different endpoints can be
// achieved by specifying a more specific type for the bot constructor argument.
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="indexjs"></a>index.js

`index.js` 用于设置可将活动转发到机器人逻辑的机器人和托管服务。

#### <a name="required-libraries"></a>所需的库

在 `index.js` 文件的最上面，可以看到所需的一系列模块或库。 通过这些模块可以访问可能要包含在应用程序中的函数集。

```javascript
const dotenv = require('dotenv');
const path = require('path');
const restify = require('restify');

// Import required bot services.
// See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter } = require('botbuilder');

// This bot's main dialog.
const { MyBot } = require('./bot');

// Import required bot configuration.
const ENV_FILE = path.join(__dirname, '.env');
dotenv.config({ path: ENV_FILE });
```

#### <a name="set-up-services"></a>设置服务

后面的部件设置可让机器人与用户通信和发送响应的服务器与适配器。 服务器将侦听配置文件中指定的端口，或者故障回复到 _3978_ 以便与仿真器建立连接。 适配器充当机器人的指挥者，可以定向传入和传出的通信、身份验证，等等。

```javascript
// Create HTTP server
const server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, () => {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open the emulator select "Open Bot"`);
});

// Create adapter.
// See https://aka.ms/about-bot-adapter to learn more about how bots work.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    await context.sendActivity(`Oops. Something went wrong!`);
};

// Create the main dialog.
const myBot = new MyBot();
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>将请求转发到机器人逻辑

适配器的 `processActivity` 将传入活动发送到机器人逻辑。
`processActivity` 中的第三个参数是在收到的[活动](#the-activity-processing-stack)由适配器预先处理并通过任何中间件路由后，将要调用的以执行机器人逻辑的函数处理程序。 可以使用作为参数传递给函数处理程序的轮次上下文变量来提供有关传入的活动、发送方和接收方、通道、聊天等的信息。活动处理将路由到机器人的 `run` 方法。 `run` 在 `ActivityHandler` 中定义；它执行某种错误检查，然后根据收到的活动类型调用机器人的事件处理程序。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await myBot.run(context);
    });
});
```

---

## <a name="manage-bot-resources"></a>管理机器人资源

需要适当管理机器人资源，例如连接的服务的应用 ID、密码、密钥或机密。 有关如何执行此操作的详细信息，请参阅[管理机器人资源](bot-file-basics.md)。

## <a name="additional-resources"></a>其他资源

- 若要了解机器人中的状态的作用，请参阅[管理状态](bot-builder-concept-state.md)。
