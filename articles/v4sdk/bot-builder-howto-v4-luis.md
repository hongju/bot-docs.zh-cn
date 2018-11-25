---
title: 向机器人添加自然语言理解 | Microsoft Docs
description: 了解如何借助 Bot Builder SDK 将 LUIS 用于自然语言理解。
keywords: 语言理解, LUIS, 意向, 识别器, 实体, 中间件
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: faf26b1c4ba87061631f217ee074283759f77c97
ms.sourcegitcommit: 392c581aa2f59cd1798ee2136b6cfee56aa3ee6d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/20/2018
ms.locfileid: "52156696"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>向机器人添加自然语言理解

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

理解用户在会话和上下文中表达的含义是一项艰巨的任务，但可以让机器人更自然地进行聊天。 使用语言理解（称为 LUIS）能够实现此目标，使机器人能够识别用户消息的意向，接收用户更自然的语言，并更好地指导会话流程。 本主题将指导你设置一个可使用 LUIS 识别多个不同意向的简单机器人。 
## <a name="prerequisites"></a>先决条件
- [luis.ai](https://www.luis.ai) 帐户
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- 本文中的代码基于**采用 LUIS 的 NLP** 示例。 需要获取 [C# ](https://aka.ms/cs-luis-sample) 或 [JS](https://aka.ms/js-luis-sample) 示例的副本。 
- 了解[机器人基础知识](bot-builder-basics.md)、[自然语言处理](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)和 [.bot](bot-file-basics.md) 文件。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 门户中创建 LUIS 应用
登录到 LUIS 门户，以创建自己的示例 LUIS 应用版本。 可在“我的应用”中创建和管理应用程序。 

1. 选择“导入新应用”。 
1. 单击“选择应用文件(JSON 格式)...” 
1. 选择示例的 `CognitiveModels` 文件夹中的 `reminders.json` 文件。 在“可选名称”中，输入 **LuisBot**。 此文件包含三个意向：Calendar-Add、Calendar-Find 和 None。 当用户向机器人发送消息时，我们将使用这些意向来了解其意图。 
1. [训练](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)应用。
1. 将应用[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)到生产环境。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>获取用于连接 LUIS 应用的值

发布 LUIS 应用后，可以在机器人中访问它。 需要记录多个值才能在机器人中访问该 LUIS 应用。 可以使用 LUIS 门户检索该信息。

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>从 LUIS.ai 门户检索应用程序信息
.bot 文件将所有服务引用合并到一个位置。 检索的信息将添加到下一部分所述的 .bot 文件。 
1. 在 [luis.ai](https://www.luis.ai) 中选择已发布的 LUIS 应用。
1. 打开已发布的 LUIS 应用后，选择“管理”选项卡。
1. 在左侧选择“应用程序信息”选项卡，记录“应用程序 ID”(<YOUR_APP_ID>) 的显示值。
1. 在左侧选择“密钥和终结点”选项卡，记录“创作密钥”(<YOUR_AUTHORING_KEY>) 的显示值。 请注意，订阅密钥与创作密钥相同。 
1. 向下滚动到页尾，记录“区域”(<YOUR_REGION>) 的显示值。
1. 记录“终结点”(<YOUR_ENDPOINT>) 的显示值。

#### <a name="update-the-bot-file"></a>更新机器人文件
在 `nlp-with-luis.bot` 文件中添加访问 LUIS 应用所需的信息，包括应用程序 ID、创作密钥、订阅密钥、终结点和区域。 前面已在发布的 LUIS 应用中保存了这些值。

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="configure-your-bot-to-use-your-luis-app"></a>将机器人配置为使用你的 LUIS 应用

接下来，在 `BotServices.cs` 中初始化 BotService 类的新实例，以便从 `.bot` 文件中获取上述信息。 使用 `BotConfiguration` 类配置外部服务。

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

然后，在 `ConfigureServices` 方法中使用以下代码，在 `Startup.cs` 文件中将 LUIS 应用注册为单一实例。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);
        
        // ...
    });
}
```

接下来，在 `Luis.cs` 文件中，机器人会获取此 LUIS 实例。

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在示例中，启动代码位于 **index.js** 文件中，机器人逻辑代码位于 **bot.js** 文件中，其他配置信息位于 **nlp-with-luis.bot** 文件中。

在 **bot.js** 文件中，我们读取配置信息以生成 LUIS 服务并初始化机器人。
如配置文件中所示，将 `LUIS_CONFIGURATION` 的值更新为 LUIS 应用的名称。

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

然后创建 HTTP 服务器并侦听传入请求，此类请求会生成对机器人逻辑的调用。

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

现在，为机器人配置了 LUIS。 接下来，让我们看看如何从 LUIS 中获取意向。

## <a name="get-the-intent-by-calling-luis"></a>通过调用 LUIS 获取意向

你的机器人通过调用 LUIS 识别器从 LUIS 获取结果。

# <a name="ctabcs"></a>[C#](#tab/cs)

若要让机器人根据 LUIS 应用检测到的意向发送回复，请调用 `LuisRecognizer` 来获取 `RecognizerResult`。 每当需要获取 LUIS 意向时都可以在代码中执行此操作。

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

在表达中识别的任何意向将作为意向名称的映射返回到分数，并且可以从 `recognizerResult.Intents` 访问。 提供静态 `recognizerResult?.GetTopScoringIntent()` 方法帮助简化查找结果集的最高得分意向的过程。

识别的任何实体将作为实体名称的映射返回到值，并使用 `recognizerResults.entities` 进行访问。 在创建 LuisRecognizer 时，可以通过传递 `verbose=true` 设置来返回其他实体元数据。 然后，可以使用 `recognizerResults.entities.$instance` 访问添加的元数据。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **bot.js** 文件中，将用户的输入传递到 LUIS 识别器的 `recognize` 方法，以从 LUIS 应用获取答案。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

LUIS 识别器将返回有关话语与可用意向的匹配程度的信息。 结果对象的 `luisResult.intents` 属性包含已评分意向的数组。 结果对象的 `luisResult.topScoringIntent` 属性包含评分最高的意向及其评分。

---

<!--
## Extract entities

Besides recognizing intent, a LUIS app can also extract entities, which are important words for fulfilling a user's request. For example, for a weather bot, the LUIS app might be able to extract the location for the weather report from the user's message.

A common way to structure your conversation is to identify any entities in the user's message, and prompt for any of the required entities that are not found. Then, the subsequent steps handle the response to the prompt.


# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}


When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

/Snip -->

## <a name="test-the-bot"></a>测试机器人

1. 在计算机本地运行示例。 如需说明，请参阅 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md) 示例的自述文件。

1. 在仿真器中键入一条消息，如下所示。 

![测试 nlp 示例](~/media/emulator-v4/nlp-luis-sample-testing.png)

机器人将会响应分数最高的意向（在本例中为 `Calendar-Add` 意向）。 回顾前文，在 luis.ai 门户中导入的 `reminders.json` 文件定义了意向。

预测分数表示 LUIS 对预测结果的置信度。 预测分数在零 (0) 到一 (1) 之间。 例如，一个置信度很高的 LUIS 分数可以是 0.99。 置信度低的分数可以是 0.01。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 QnA Maker 回答问题](./bot-builder-howto-qna.md)
