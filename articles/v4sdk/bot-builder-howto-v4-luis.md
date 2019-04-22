---
title: 向机器人添加自然语言理解 | Microsoft Docs
description: 了解如何借助 Bot Framework SDK 将 LUIS 用于自然语言理解。
keywords: 语言理解, LUIS, 意向, 识别器, 实体, 中间件
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/28/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1f077cb5efd838f8a91a0f18a9bcc2f64455ceb6
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/12/2019
ms.locfileid: "59541113"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>向机器人添加自然语言理解

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

理解用户在会话和上下文中表达的含义是一项艰巨的任务，但可以让机器人更自然地进行聊天。 使用语言理解（称为 LUIS）能够实现此目标，使机器人能够识别用户消息的意向，接收用户更自然的语言，并更好地指导会话流程。 本主题将指导你设置一个可使用 LUIS 识别多个不同意向的简单机器人。 
## <a name="prerequisites"></a>先决条件
- [luis.ai](https://www.luis.ai) 帐户
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)
- 本文中的代码基于**采用 LUIS 的 NLP** 示例。 需要获取 [C# 示例](https://aka.ms/cs-luis-sample)或 [JS 示例](https://aka.ms/js-luis-sample)中的示例副本。 
- 了解[机器人基础知识](bot-builder-basics.md)、[自然语言处理](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis)和 [.bot](bot-file-basics.md) 文件。

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 门户中创建 LUIS 应用
登录到 LUIS 门户，以创建自己的示例 LUIS 应用版本。 可在“我的应用”中创建和管理应用程序。 

1. 选择“导入新应用”。 
1. 单击“选择应用文件(JSON 格式)...” 
1. 选择示例的 `CognitiveModels` 文件夹中的 `reminders.json` 文件。 在“可选名称”中，输入 **LuisBot**。 此文件包含三个意向：Calendar_Add、Calendar_Find 和 None。 当用户向机器人发送消息时，我们将使用这些意向来了解其意图。 若要包含实体，请参阅本文末尾处的[“可选”部分](#optional---extract-entities)。
1. [训练](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)应用。
1. 将应用[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)到生产环境。

### <a name="obtain-values-to-connect-to-your-luis-app"></a>获取用于连接 LUIS 应用的值

发布 LUIS 应用后，可以在机器人中访问它。 需要记录多个值才能在机器人中访问该 LUIS 应用。 可以使用 LUIS 门户检索该信息。

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>从 LUIS.ai 门户检索应用程序信息
.bot 文件将所有服务引用合并到一个位置。 检索的信息将添加到下一部分的 .bot 文件。 
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

### <a name="configure-your-bot-to-use-your-luis-app"></a>将机器人配置为使用你的 LUIS 应用
确保为项目安装 NuGet 包 **Microsoft.Bot.Builder.AI.Luis**。

接下来，在 `BotServices.cs` 中初始化 BotService 类的新实例，以便从 `.bot` 文件中获取上述信息。 使用 `BotConfiguration` 类配置外部服务。

```csharp
using Microsoft.Bot.Builder.AI.Luis;
using Microsoft.Bot.Configuration;

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

    // Initialize Bot Connected Services client.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);

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

接下来，在 `LuisBot.cs` 文件中，机器人会获取此 LUIS 实例。

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
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the JavaScript code.
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

### <a name="get-the-intent-by-calling-luis"></a>通过调用 LUIS 获取意向

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

### <a name="test-the-bot"></a>测试机器人

1. 在计算机本地运行示例。 如需说明，请参阅 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md) 示例的自述文件。

1. 在仿真器中键入一条消息，如下所示。 

![测试 nlp 示例输入](./media/nlp-luis-sample-message.png)

机器人将会响应分数最高的意向（在本例中为“Calendar_Add”意向）。 回顾前文，在 luis.ai 门户中导入的 `reminders.json` 文件定义了“Calendar_Add”、“Calendar_Find”和“None”意向。 

![测试 nlp 示例响应](./media/nlp-luis-sample-response.png) 

预测分数表示 LUIS 对预测结果的置信度。 预测分数在零 (0) 到一 (1) 之间。 例如，一个置信度很高的 LUIS 分数可以是 0.99。 置信度低的分数可以是 0.01。 

## <a name="optional---extract-entities"></a>可选 - 提取实体

除了识别用户意向以外，LUIS 应用还可以返回实体。 实体是与意向相关的重要单词；有时，若要实现用户的请求或者使机器人的行为更智能，实体可能至关重要。 

### <a name="why-use-entities"></a>为何使用实体

LUIS 实体可让机器人智能理解不同于标准意向的某些事物或事件。 这样，你便可以从用户收集额外的信息，让机器人以更高的智能做出响应，或者在某些情况下跳过它向用户提出的有关该信息的问题。 例如，在天气机器人中，LUIS 应用可以使用实体 _Location_ 从用户消息内部提取所请求的天气报告的位置。 这样，机器人便可以跳过用户所在位置的问题，并向用户提供更智能、更简洁的聊天。

### <a name="prerequisites"></a>先决条件

若要在本示例中使用实体，需要创建包含实体的 LUIS 应用。 遵循上面有关[创建 LUIS 应用](#create-a-luis-app-in-the-luis-portal)的部分中的步骤，但不要使用文件 `reminders.json`，而要使用 [reminders-with-entities.json](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/nlp-with-luis) 文件来生成 LUIS 应用。 此文件提供了相同的意向以及三个附加实体：Appointment、Meeting 和 Schedule。 这些实体可帮助 LUIS 确定用户消息的意向。 

### <a name="extract-and-display-entities"></a>提取并显示实体
可将以下可选代码添加到本示例应用，以便在 LUIS 使用某个实例来帮助识别用户的意向时提取并显示实体信息。 

# <a name="ctabcs"></a>[C#](#tab/cs)

可以向机器人添加以下帮助程序函数来通过 LUIS 从 `RecognizerResult` 外部获取实体。 它将需要使用 `Newtonsoft.Json.Linq` 库，必须将该库添加到 **using** 语句中。 如果在分析从 LUIS 返回的 JSON 时找到了实体信息，则 Newtonsoft _DeserializeObject_ 函数会将此 JSON 转换为动态对象，并提供对实体信息的访问。

```cs
using Newtonsoft.Json.Linq;

private string ParseLuisForEntities(RecognizerResult recognizerResult)
{
   var result = string.Empty;

   // recognizerResult.Entities returns type JObject.
   foreach (var entity in recognizerResult.Entities)
   {
       // Parse JObject for a known entity types: Appointment, Meeting, and Schedule.
       var appointFound = JObject.Parse(entity.Value.ToString())["Appointment"];
       var meetingFound = JObject.Parse(entity.Value.ToString())["Meeting"];
       var schedFound = JObject.Parse(entity.Value.ToString())["Schedule"];

       // We will return info on the first entity found.
       if (appointFound != null)
       {
           // use JsonConvert to convert entity.Value to a dynamic object.
           dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
           if (o.Appointment[0] != null)
           {
              // Find and return the entity type and score.
              var entType = o.Appointment[0].type;
              var entScore = o.Appointment[0].score;
              result = "Entity: " + entType + ", Score: " + entScore + ".";
              
              return result;
            }
        }

        if (meetingFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Meeting[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Meeting[0].type;
                var entScore = o.Meeting[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }

        if (schedFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Schedule[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Schedule[0].type;
                var entScore = o.Schedule[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }
    }

    // No entities were found, empty string returned.
    return result;
}
```

然后，可与识别到的用户意向一同显示这些检测到的实体信息。 若要显示这些信息，请在显示意向信息之后，随即将以下几行代码添加到示例代码的 _OnTurnAsync_ 任务。

```cs
// See if LUIS found and used an entity to determine user intent.
var entityFound = ParseLuisForEntities(recognizerResult);

// Inform the user if LUIS used an entity.
if (entityFound.ToString() != string.Empty)
{
   await turnContext.SendActivityAsync($"==>LUIS Entity Found: {entityFound}\n");
}
else
{
   await turnContext.SendActivityAsync($"==>No LUIS Entities Found.\n");
}
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

可将以下代码添加到机器人，以便在从 LUIS 返回的 `luisRecognizer` 结果中提取实体信息。 在代码示例文件 bot.js 的 `onTurn` 处理中，紧接在常量 _topIntent_ 的声明后面添加以下代码行。 此代码行将捕获所有返回的实体信息： 

```javascript
// Since the LuisRecognizer was configured to include the raw results, get returned entity data.
var entityData = results.luisResult.entities;

```

若要向用户显示所有返回的实体信息，请紧接在示例代码中使用的 _sendActivity_ 调用后面添加以下代码行，以便在找到 topIntent 时通知用户。

```javascript
// See if LUIS found and used an entity to determine user intent.
if (entityData.length > 0)
{
   if ((entityData[0].type == "Appointment") || (entityData[0].type == "Meeting") || (entityData[0].type == "Schedule") )
   {
      // inform user if LUIS used an entity.
      await turnContext.sendActivity(`LUIS Entity Found: Entity: ${entityData[0].entity}, Score: ${entityData[0].score}.`);
   }
}
else{
       await turnContext.sendActivity(`No LUIS Entities Found.`);
}
```

此代码首先检查 LUIS 是否在返回的结果中返回了任何实体信息，如果已返回，则显示与检测到的第一个实体相关的信息。

---

### <a name="test-bot-with-entities"></a>测试包含实体的机器人

1. 若要测试包含实体的机器人，请根据[上面](#test-the-bot)所述在本地运行该示例。

1. 在仿真器中，输入如下所示的消息。 

![测试 nlp 示例输入](./media/nlp-luis-sample-message.png)

现在，机器人的响应中将返回评分最高的意向“Calendar_Add”，加上 LUIS 用来确定用户意向的“Meetings”实体。

![测试 nlp 示例响应](./media/nlp-luis-sample-entity-response.png) 

检测实体可以帮助提高机器人的整体性能。 例如，检测“Meeting”实体（如上所示）可让应用程序调用一个专用函数，以根据用户的日历创建新会议。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 QnA Maker 回答问题](./bot-builder-howto-qna.md)
