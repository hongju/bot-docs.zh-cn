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
ms.date: 11/08/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eab8e2f9d437748d0bb0fefd31c03c8fb350c6b1
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645697"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>向机器人添加自然语言理解

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

理解用户在会话和上下文中表达的含义是一项艰巨的任务，但可以让机器人更自然地进行聊天。 使用语言理解（称为 LUIS）能够实现此目标，使机器人能够识别用户消息的意向，接收用户更自然的语言，并更好地指导会话流程。 如果需要了解 LUIS 的更多背景知识，请参阅机器人的[语言理解](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)。

## <a name="prerequisites"></a>先决条件
本主题将指导你设置一个可使用 LUIS 识别多个不同意向的简单机器人。 本文中的代码基于采用 LUIS 的 NLP [C#](https://aka.ms/cs-luis-sample) 和 [JavaScript](https://aka.ms/js-luis-sample) 示例。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 门户中创建 LUIS 应用

首先，在 [luis.ai](https://www.luis.ai) 上注册一个帐户，然后遵照[此处](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app)的说明在 LUIS 门户中创建一个 LUIS 应用。 若要创建自己的要在本文中使用的示例 LUIS 应用版本，请在 LUIS 门户中[导入](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)此 `LUIS.Reminders.json` 文件 ([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json)) 以生成 LUIS 应用，然后[训练](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)并[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)此应用。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>获取用于连接 LUIS 应用的值

发布 LUIS 应用后，可以在机器人中访问它。 需要记录多个值才能在机器人中访问该 LUIS 应用。 可以使用 LUIS 门户或 CLI 工具检索该信息。

#### <a name="using-luis-portal"></a>使用 LUIS 门户
- 在 [luis.ai](https://www.luis.ai) 中选择已发布的 LUIS 应用。
- 打开已发布的 LUIS 应用后，选择“管理”选项卡。
- 在左侧选择“应用程序信息”选项卡，记录“应用程序 ID”(<YOUR_APP_ID>) 的显示值。
- 在左侧选择“密钥和终结点”选项卡，记录“创作密钥”(<YOUR_AUTHORING_KEY>) 的显示值。 请注意，<YOUR_SUBSCRIPTION_KEY> 与 <YOUR_AUTHORING_KEY> 相同。 向下滚动到页面底部，记录“区域”(<YOUR_REGION>) 和“终结点”(<YOUR_ENDPOINT>) 的显示值。

#### <a name="using-cli-tools"></a>使用 CLI 工具

可以使用 [luis](https://aka.ms/botbuilder-tools-luis) 和 [msbot](https://aka.ms/botbuilder-tools-msbot-readme) BotBuilder CLI 工具获取有关 LUIS 应用的元数据，并将其添加到 **.bot** 文件中。

1. 打开终端或命令提示符，导航到机器人项目的根目录。
2. 确保已安装 `luis` 和 `msbot` 工具。

    ```shell
    npm install luis msbot
    ```

3. 运行 `luis init` 创建 LUIS 资源文件 (**.luisrc**)。 出现提示时，请提供 LUIS 创作密钥和区域。 暂时不需要输入应用 ID。
4. 运行以下命令，下载元数据并将其添加到机器人的配置文件。
    如果已加密配置文件，则需要提供机密密钥才能更新该文件。

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>将机器人配置为使用你的 LUIS 应用

初始化机器人时，先添加对 LUIS 应用的引用。 然后，我们可以在机器人逻辑中调用它。

### <a name="prerequisite"></a>先决条件

在开始编写代码之前，请确保已获取 LUIS 应用所需的包。

# <a name="ctabcs"></a>[C#](#tab/cs)

将以下 [NuGet 包](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)添加到机器人。

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

LUIS 功能位于 `botbuilder-ai` 包中。 可通过 npm 向项目添加以下包：

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

下载此处提供的 [NLP LUIS 示例代码](https://aka.ms/cs-luis-sample)并将其打开。 根据需要修改代码。 

首先，在 `BotConfiguration.bot` 文件中添加访问 LUIS 应用所需的信息，包括应用程序 ID、创作密钥、订阅密钥、终结点和区域。 前面已在发布的 LUIS 应用中保存了这些值。

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

接下来，初始化 BotService 类 `BotServices.cs` 的新实例，以便从 `.bot` 文件中获取上述信息。 将以下代码添加到 `BotServices.cs` 文件。

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
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

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

然后，在 `ConfigureServices` 中添加以下代码，以便在 `Startup.cs` 文件中将 LUIS 应用注册为单一实例。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
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

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

接下来，我们需要向机器人提供此 LUIS 实例。 打开 `LuisBot.cs`，并在文件顶部添加以下代码。

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在示例中，启动代码位于 **index.js** 文件中，机器人逻辑代码位于 **bot.js** 文件中，其他配置信息位于 **nlp-with-luis.bot** 文件中。

遵照说明创建 LUIS 应用并更新 **.bot** 文件之后，**nlp-with-luis.bot** 文件应包含 LUIS 应用的服务条目。

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

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

## <a name="extract-entities"></a>提取实体

除了识别意向外，LUIS 应用还可以提取实体，这些实体是实现用户请求的重要词汇。 例如，对于天气机器人，LUIS 应用可能能够从用户的消息中提取天气报告的位置。

构造聊天结构的一种常用方法是识别用户消息中的任何实体，并提示找不到的任何所需的实体。 然后，后续步骤会处理对提示的响应。

<!--Snip
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
```
/Snip-->

从聊天中的多个步骤收集实体等信息时，在你的状态中保存所需信息会很有帮助。 如果发现某个实体，则可以将其添加到适当的状态字段中。 在聊天中，如果当前步骤已填写了关联的字段，则可以跳过提示输入该信息的步骤。

## <a name="additional-resources"></a>其他资源

有关使用 LUIS 的示例，请查看适用于 [[C#](https://aka.ms/cs-luis-sample)] 或 [[JavaScript](https://aka.ms/js-luis-sample)] 的项目。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用调度工具组合 LUIS 和 QnA 服务](./bot-builder-tutorial-dispatch.md)
