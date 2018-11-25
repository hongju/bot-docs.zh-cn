---
title: 机器人的工作原理 | Microsoft Docs
description: 介绍 Bot Builder SDK 中的活动和 http 工作原理。
keywords: 聊天流, 轮次, 机器人聊天, 对话, 提示, 瀑布, 对话集
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 15cd6c998abf37b1c7b9a9e2659b7390370f7f10
ms.sourcegitcommit: d92fd6233295856052305e0d9e3cba29c9ef496e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2018
ms.locfileid: "51715121"
---
# <a name="how-bots-work"></a>机器人的工作原理

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人是用户使用文本、图形（例如卡片或图像）或语音通过聊天的方式进行交互的应用。 用户与机器人之间的每次交互会生成一个活动。 机器人服务在用户的机器人联网应用（例如 Facebook、Skype、Slack 等，称为“通道”）与机器人之间发送信息。 每个通道可以在发送的活动中包含其他信息。 在创建机器人之前，必须了解机器人如何使用活动对象来与其用户通信。 首先，让我们了解在运行简单的聊天机器人时交换的活动。

![活动示意图](media/bot-builder-activity.png)

此处演示了两种活动类型：聊天更新和消息。

当某一方参与聊天时，Bot Framework Service 可以发送聊天更新。 例如，使用 Bot Framework Emulator 开始聊天时，会看到两个聊天更新活动（一个活动指出用户正在参与聊天，另一个活动指出机器人正在参与聊天）。 若要区分这些聊天更新活动，请检查“成员已添加”属性是否包含除机器人以外的成员。 

消息活动承载参与方之间的聊天信息。 在聊天机器人示例中，消息活动承载简单的文本，通道呈现这些文本。 或者，消息活动可能承载要讲述的文本、建议的操作或要显示的卡片。

在此示例中，机器人创建并发送了一个消息活动，以响应它收到的入站消息活动。 但是，机器人可通过其他方式响应收到的消息活动；机器人经常通过在消息活动中发送一些欢迎文本来响应聊天更新活动。 在[欢迎用户](bot-builder-welcome-user.md)中可以找到更多信息。

### <a name="http-details"></a>HTTP 详细信息

活动通过 HTTP POST 请求从 Bot Framework Service 抵达机器人。 机器人使用 200 HTTP 状态代码响应入站 POST 请求。 从机器人发送到通道的活动通过单独的 HTTP POST 发送到 Bot Framework Service。 而此请求将通过 200 HTTP 状态代码得到确认。

协议不会指定这些 POST 请求及其确认的发送顺序。 但是，为了适应常见的 HTTP 服务框架，这些请求通常会嵌套，这意味着，出站 HTTP 请求将在入站 HTTP 请求的范围内从机器人发出。 上图演示了此模式。 由于后端之间存在两个不同的 HTTP 连接，安全模型必须提供这两个连接。

### <a name="defining-a-turn"></a>定义轮次

在聊天时，人们通常一次说一句话，并且会轮流说话。 使用机器人时，通常是由机器人对用户输入进行响应。 在 Bot Builder SDK 中，一轮通话既包含用户传给机器人的活动，又包含机器人发回用户的作为即时响应的活动。 可以将一个轮次视为给定活动抵达时的相关处理。

轮次上下文对象提供有关活动的信息，例如发送方和接收方、通道，以及处理该活动所需的其他数据。 使用该对象还能在机器人的不同层中处理轮次期间添加信息。

轮次上下文是 SDK 中最重要的抽象之一。 轮次上下文不仅将入站活动传递到所有中间件组件将和应用程序逻辑，而且还提供所需的机制让中间件组件和应用程序逻辑发送出站活动。

## <a name="the-activity-processing-stack"></a>活动处理堆栈

让我们深入查看上图，并重点关注消息活动的抵达。

![活动处理堆栈](media/bot-builder-activity-processing-stack.png)

在上述示例中，机器人使用包含相同文本消息的另一个消息活动回复了原始消息活动。 处理工作从 HTTP POST 请求开始，该请求包含作为 JSON 有效负载承载的活动，抵达 Web 服务器。 在 C# 中，这通常是一个 ASP.NET 项目；在 JavaScript Node.js 项目中，这可能是 Express 或 Restify 等常用框架之一。

适配器（SDK 的集成组件）是 SDK 运行时的核心。 活动以 JSON 形式承载在 HTTP POST 正文中。 将反序列化此 JSON 以创建 Activity 对象，然后通过调用 *process activity* 方法将此对象传递给适配器。 收到活动时，适配器会创建轮次上下文并调用中间件。 之所以称作“轮次上下文”，是因为使用了“轮次”一词来描述与活动抵达相关的所有处理。 轮次上下文是 SDK 中最重要的抽象之一，它不仅将入站活动传递到所有中间件组件和应用程序逻辑，而且还提供所需的机制让中间件组件和应用程序逻辑发送出站活动。 轮次上下文提供 _send、update 和 delete activity_ 响应方法来响应活动。 每个响应方法都在异步进程中运行。 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]


## <a name="middleware"></a>中间件
中间件非常类似于其他任何消息传送中间件，由一组线性组件构成，其中每个组件按顺序执行，并有机会对活动运行。 中间件管道的最后一个阶段是一个回调，该回调针对已由应用程序注册到适配器的机器人类调用轮次处理程序（C# 中的 `OnTurnAsync`，以及 JS 中的 `onTurn`）函数。 轮次处理程序采用轮次上下文作为参数。通常，在轮次处理程序函数内部运行的应用程序逻辑将处理入站活动的内容，在响应中生成一个或多个活动，并使用轮次上下文中的 *send activity* 函数发出这些活动。 调用轮次上下文中的 *send activity* 会导致针对出站活动调用中间件组件。 中间件组件在机器人的轮次处理程序函数之前和之后执行。 执行在本质上是嵌套的，因此，有时称作“俄罗斯套娃”或类似叫法。 有关中间件的更深入信息，请参阅[中间件主题](~/v4sdk/bot-builder-concept-middleware.md)。

## <a name="bot-structure"></a>机器人结构
以下部分介绍机器人的关键组成部分。

### <a name="prerequisites"></a>先决条件
- [C#](https://aka.ms/EchoBotWithStateCSharp) 或 [JS](https://aka.ms/EchoBotWithStateJS) **EchoBotWithCounter** 示例的副本。 此处仅显示相关代码，但你可以参考示例中的完整源代码。

# <a name="ctabcs"></a>[C#](#tab/cs)

机器人是一种 [ASP.NET Core](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1) Web 应用程序。 在 [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) 基础知识中，可以看到 **Program.cs** 和 **Startup.cs** 等文件包含类似的代码。 这些文件并非特定于机器人，所有 Web 应用都需要它们。 

### <a name="bot-logic"></a>机器人逻辑

主要机器人逻辑在派生自 `IBot` 接口的 `EchoWithCounterBot` 类中定义。 `IBot` 定义单个方法 `OnTurnAsync`。 应用程序必须实现此方法。 `OnTurnAsync` 包含 turnContext，后者提供有关传入活动的信息。 传入活动对应于入站 HTTP 请求。 活动可以属于不同的类型，因此，让我们先检查机器人是否收到了消息。 如果收到的是消息，则我们可以从轮次上下文获取聊天状态、递增轮次计数器，然后将新的轮次计数器值保存到聊天状态中。 然后使用 SendActivityAsync 调用将一条消息发回给用户。 传出活动对应于出站 HTTP 请求。

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="set-up-services"></a>设置服务

startup.cs 文件中的 `ConfigureServices` 方法从 [.bot](bot-builder-basics.md#the-bot-file) 文件加载连接服务，捕获并记录聊天轮次期间发生的任何错误，设置凭据提供程序，然后创建一个聊天状态对象用于在内存中存储聊天数据。

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

`ConfigureServices` 方法还会创建并注册 **EchoBotStateAccessors.cs** 文件中定义的、并使用 ASP.NET Core 中的依赖项注入框架传递到公共 `EchoWithCounterBot` 构造函数的 `EchoBotAccessors`。

```csharp
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

`Configure` 方法通过指定应用需使用 Bot Framework 和其他几个文件来完成应用的配置。 使用 Bot Framework 的所有机器人需要该配置调用。 应用启动时，运行时会调用 `ConfigureServices` 和 `Configure`。

### <a name="manage-state"></a>管理状态

此文件包含机器人用来保持当前状态的简单类。 其中仅包含一个用于递增计数器的 `int`。

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="accessor-class"></a>访问器类

`EchoBotAccessors` 类创建为 `Startup` 类中的单一实例，并传递给 IBot 派生的类。 在本例中为 `public class EchoWithCounterBot : IBot`。 机器人使用访问器来保存聊天数据。 `EchoBotAccessors` 构造函数传入到 Startup.cs 文件中创建的聊天对象。

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Yeoman 生成器创建 [restify](http://restify.com/) 类型的 Web 应用程序。 如果在相应文档中查看 restify 快速入门，会看到类似于生成的 **index.js** 文件的应用。 本部分主要介绍 **package.json**、**.env**、**index.js**、**bot.js** 和 **echobot-with-counter.bot** 文件。 此处未复制某些文件中的代码，但在运行机器人时会看到这些文件，或者可以参考 [Node.js echobot-with-counter](https://aka.ms/js-echobot-with-counter) 示例。

### <a name="packagejson"></a>package.json

**package.json** 指定机器人的依赖项及其关联的版本。 所有这些信息由模板和系统设置。

### <a name="env-file"></a>.env 文件

**.env** 文件指定机器人的配置信息，例如端口号、应用 ID 和密码等。 如果使用特定的技术或者在生产环境中使用此机器人，则需要将特定的密钥或 URL 添加到此配置。 但是，对于此聊天机器人，目前不需要执行任何操作；暂时可将应用 ID 和密码保持未定义状态。

若要使用 **.env** 配置文件，需在模板中包含一个附加的包。  首先，从 npm 获取 `dotenv` 包：

`npm install dotenv`

### <a name="indexjs"></a>index.js

`index.js` 用于设置可将活动转发到机器人逻辑的机器人和托管服务。

#### <a name="required-libraries"></a>所需的库

在 `index.js` 文件的最上面，可以看到所需的一系列模块或库。 通过这些模块可以访问可能要包含在应用程序中的函数集。

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>机器人配置

下一个部件从机器人配置文件加载信息。

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>机器人适配器、HTTP 服务器和机器人状态

后面的部件设置可让机器人与用户通信和发送响应的服务器与适配器。 服务器将侦听 **BotConfiguration.bot** 配置文件中指定的端口，或者故障回复到 _3978_ 以便与仿真器建立连接。 适配器充当机器人的指挥者，可以定向传入和传出的通信、身份验证，等等。

我们还会创建一个使用 `MemoryStorage` 作为存储提供程序的状态对象。 此状态定义为 `ConversationState`，仅表示它正在保留聊天的状态。 `ConversationState` 将存储所需的信息，在本例中，该信息只是内存中的一个轮次计数器。

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>机器人逻辑

适配器的 `processActivity` 将传入活动发送到机器人逻辑。
`processActivity` 中的第三个参数是在收到的[活动](#the-activity-processing-stack)由适配器预先处理并通过任何中间件路由后，将要调用的以执行机器人逻辑的函数处理程序。 可以使用作为参数传递给函数处理程序的轮次上下文变量来提供有关传入的活动、发送方和接收方、通道、聊天等的信息。活动处理将路由到聊天机器人的 `onTurn`。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>聊天机器人

所有活动处理将路由到此类的 `onTurn` 处理程序。 创建类时已传入状态对象。 构造函数使用此状态对象创建 `this.countProperty` 访问器，以保存此机器人的轮次计数器。

在每个轮次，我们先检查机器人是否收到了消息。 如果机器人未收到消息，则我们回显收到的活动类型。 接下来，创建一个状态变量用于保存机器人聊天的信息。 如果计数变量为 `undefined`，则它设置为 1（启动机器人时存在这种情况），或者每收到新的消息，就递增计数变量。 将计数和发送的消息回显给用户。 最后，设置计数并保存对状态所做的更改。

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

## <a name="the-bot-file"></a>bot 文件

**.bot** 文件包含信息，其中包括终结点、应用 ID、密码，以及对机器人所用服务的引用。 此文件是在通过模板构建机器人时创建的，但你可以通过仿真器或其他工具创建自己的文件。 可以指定在[仿真器](../bot-service-debug-emulator.md)中测试机器人时要使用的 .bot 文件。

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>其他资源

若要了解机器人文件在资源管理中发挥的作用，请参阅[机器人文件](bot-file-basics.md)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [创建机器人](~/bot-service-quickstart.md)
