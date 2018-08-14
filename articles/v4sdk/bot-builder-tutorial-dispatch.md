---
title: 使用适合 LUIS 和 QnA Maker 的 Dispatch 工具 | Microsoft Docs
description: 了解如何在机器人中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, Dispatch 工具, 多个服务
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6294021355f82ff53a2ea99db4fb19f44cf13029
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2018
ms.locfileid: "39298559"
---
## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>将多个 LUIS 应用和 QnA 服务与 Dispatch 工具集成

本教程演示如何使用 Dispatch 工具生成的 LUIS 模型，将机器人与多个语言理解智能服务 (LUIS) 应用和 QnAMaker 服务集成。 

假设开发了以下服务，并且想要创建一个集成所有这些服务的机器人。

| 服务类型 | 名称 | Description |
|------|------|------|
| LUIS 应用 | 家庭自动化 | 识别 HomeAutomation.TurnOn、HomeAutomation.TurnOff 和 HomeAutomation.None 意向。|
| LUIS 应用 | 天气 | 识别 Weather.GetForecast 和 Weather.GetCondition 意向。|
| QnAMaker 服务 | 常见问题解答  | 提供有关家庭自动化照明系统问题的答案 |

让我们先创建应用和服务，然后将其集成在一起。

> [!NOTE]
> 需要在 Dispatch 的同一 Azure 位置创建这三个应用才能成功访问它们。 下面的 Dispatch 代码示例使用示例位置美国西部。

## <a name="create-the-luis-apps"></a>创建 LUIS 应用

创建 HomeAutomation 和 Weather LUIS 应用的最快方法是下载 [homeautomation.json][HomeAutomationJSON] 和 [weather.json][WeatherJSON] 文件。 然后转到 [LUIS 网站](https://www.luis.ai/home)并登录。 单击“我的应用” > “导入新应用”，然后选择 homeautomation.json 文件。 将新应用命名为 `homeautomation`。 单击“我的应用” > “导入新应用”，然后选择 weather.json 文件。 将这一新应用命名为 `weather`。

## <a name="create-the-qna-cognitive-service-in-azure"></a>在 Azure 中创建 QnA 认知服务

QnA Maker 服务涉及两个部分，即 Azure 中的认知服务，以及使用认知服务发布的问答对知识库。

若要在 Azure 中创建认知服务，请通过 https://portal.azure.com 登录 Azure 门户，并执行以下操作：

1. 单击“所有服务”。
1. 搜索 `Cognitive` 并选择“认知服务”。
1. 单击 **“添加”**。
1. 搜索 `QnA` 并选择“QnA Maker”。
1. 在“QnA Maker”边栏选项卡上，单击“创建”。
1. 填写信息并创建 QnA Maker 服务。

    <!-- TODO: Add screenshot.-->

    * 输入服务的名称。 在本教程中，我们将使用 `SmartLightQnA`。
    * 选择要使用的订阅。
    * 选择要使用的管理定价层；我们将使用 F0（免费）层。
    * 创建一个资源组，或选择要使用的新资源组。 在本教程中，我们将创建一个新的 `SmartLightQnA` 资源组。
    * 选择搜索定价层；我们将使用 B（基本）层。
    * 选择搜索位置；我们将使用 `West US`。
    * 输入要使用的应用程序名称；我们将保留默认的 `SmartLightQnA`。
    * 选择网站位置；我们将使用 `West US`。
    * 保持启用 App Insights 的默认设置。
    * 选择 App Insights 位置；我们将使用 `West US 2`。
    * 单击“创建”以创建 QnA Maker 服务。
    * Azure 创建并开始部署服务。

1. 部署服务后，查看通知并单击“转到资源”，导航到服务的边栏选项卡。
1. 单击“密钥”以获取密钥。

    * 复制服务名称和第一个密钥。 将在以下步骤中使用这些信息。
    * 将获得两个密钥，以便可一次重写其中一个密钥而无需中断服务。

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>创建并发布 QnA Maker 知识库

转到 [QnA Maker 网站](https://qnamaker.ai)并登录。 选择“创建知识库”，创建一个名为“常见问题解答”的新知识库。 单击“选择文件”按钮，然后上传 [TSV 文件示例][FAQ_TSV]。 单击“创建”，创建服务后，单击“发布”。

## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>使用 Dispatch 工具创建调度程序 LUIS 应用

接下来，创建 LUIS 应用以合并创建的每个服务。

通过在 Node.js 命令提示符中运行以下命令来安装 [Dispatch 工具][DispatchTool]。

```
npm install -g botdispatch
```

运行以下命令，初始化名为 `CombineWeatherAndLights` 的 Dispatch 工具。 将 [LUIS 创作密钥](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys)换为 `"YOUR-LUIS-AUTHORING-KEY"`。

```
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

对于创建的每个 LUIS 应用，获取 LUIS 应用 ID。 可在 [LUIS 站点](https://www.luis.ai/home)“我的应用”下找到每个应用的应用 ID；单击应用名称，然后单击“设置”查看应用程序 ID。 

然后为创建的每个 LUIS 应用运行 `dispatch add` 命令。

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

对创建的 QnA Maker 服务运行 `dispatch add` 命令。 `-key` 参数应为 Azure 门户中的密钥，完成[在 Azure 中创建 QnA 认知服务](./bot-builder-tutorial-dispatch.md#create-the-qna-cognitive-service-in-azure)这一步骤时应保存该密钥。

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-QNA-SUBSCRIPTION-KEY"
```

运行 `dispatch create`：

```
dispatch create
```

这将创建名为 CombineWeatherAndLights 的调度程序 LUIS 应用。 可在 [https://www.luis.ai/home](https://www.luis.ai/home) 中看到新应用。 

![LUIS.ai 中的调度程序应用](media/tutorial-dispatch/dispatch-app-in-luis.png)

单击新应用。 在“意向”下，可看到 `l_homeautomation`、`l_weather` 和 `q_faq` 意向。

![LUIS.ai 中的调度程序意向](media/tutorial-dispatch/dispatch-intents-in-luis.png)

单击“训练”按钮以训练 LUIS 应用，并使用“发布”选项卡进行[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)。 单击“设置”，复制要在机器人中使用的新应用的 ID。

## <a name="create-the-bot"></a>创建机器人

现在，可将调度程序应用的意向连接到机器人中的逻辑，该逻辑将消息路由到原始 LUIS 应用和 QnAMaker 服务。

可使用 Bot Builder SDK 附带的示例作为起始点。

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

以 [LUIS Dispatch 示例][DispatchBotCS]中的代码开始。 在 Visual Studio 中，[将 Nuget 包更新](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)为以下的最新预发布版本：

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA`（QnA Maker 需要）
* `Microsoft.Bot.Builder.Ai.Luis`（LUIS 需要）

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

下载 [LUIS Dispatch 示例][DispatchBotJs]。  使用 npm 安装所需包（包括 LUIS 和 QnA Maker 的 `botbuilder-ai` 包）：

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


设置示例以使用调度程序应用程序。

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

在 [LUIS Dispatch 示例][DispatchBotCS]中的 appsettings.json 中，编辑以下字段。

| 名称 | Description |
|------|------|
| `Luis-SubscriptionKey` |  LUIS 订阅密钥。 可以是终结点密钥或创作密钥，如[此处](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys)所述。 | 
| `Luis-ModelId-Dispatcher` | Dispatch 工具生成的 LUIS 应用的应用 ID。 | 
| `Luis-ModelId-HomeAutomation` | 从 homeautomation.json 创建的应用的应用 ID  | 
| `Luis-ModelId-Weather` | 从 weather.json 创建的应用的应用 ID | 
| `QnAMaker-Endpoint-Url` | 对于预览版 QnA Maker 服务，应将此设置为 https://westus.api.cognitive.microsoft.com/qnamaker/v2.0。 <br/>对于新的 (GA) QnA Maker 服务，请将此设置为 https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker。|
| `QnAMaker-SubscriptionKey` | QnA Maker 订阅密钥。 | 
| `QnAMaker-KnowledgeBaseId` | 在 [QnAMaker 门户](https://qnamaker.ai)创建的知识库的 ID。| 



在 Startup.cs 中，查看 `ConfigureServices` 方法。 它包含代码，可使用刚生成的 LUIS 应用的应用 ID 初始化 `LuisRecognizerMiddleware`。

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
`GetLuisConfiguration` 和 `GetQnAMakerConfiguration` 方法从 appsettings.json 获取 LUIS 和 QnA 配置。
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>调度消息

查看 LuisDispatchBot.cs，其中机器人将消息调度到 LUIS 应用或 QnA Maker 以获取子组件。 

在 `DispatchToTopIntent` 中，如果调度程序应用检测到 `l_homeautomation` 或 `l_weather` 意向，它会调用 `DispatchToTopIntent` 方法创建一个 `LuisRecognizer` 来调用原始 `homeautomation` 和 `weather` 应用。 如果机器人检测到 `q_faq` 意向，或者 `none` 意向用作后备情况，则会调用查询 QnAMaker 的方法。

> [!NOTE] 
> 如果意向名称 `l_homeautomation`、`l_weather` 或 `q_faq` 与使用 Dispatch 创建的 LUIS 应用不匹配，请进行编辑，使其与 [LUIS 门户](https://www.luis.ai)中显示的小写版意向名称相匹配。

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

`DispatchToQnAMaker` 方法将用户的消息发送到 QnA Maker 服务。 在运行机器人之前，请确保已在 [QnA Maker 门户](https://qnamaker.ai)中发布该服务。

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

`DispatchToLuisModel` 方法将用户的消息发送到原始 `homeautomation` 和 `weather` LUIS 应用。 在运行机器人之前，请确保已在 [LUIS 门户](https://www.luis.ai)中发布这些 LUIS 应用。

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

`RecognizeAsync` 方法调用 `LuisRecognizer` 以从 LUIS 应用获取结果。

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

以 [Dispatch 机器人示例][DispatchBotJs]中的代码开始。 打开 app.js，并可选择将 `appId` 字段替换为创建的 LUIS 应用的 ID。 如果保留最初的 `appId` 字段，则将使用为演示而创建的公共 LUIS 应用。

```javascript
// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

在以下代码中，将 `endpointKey` 和 `knowledgeBaseID` 替换为 QnAMaker 中的密钥和知识库 ID。 对于预览版 QnA Maker 服务，`host` 应设置为 `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0`，对于新的 (GA) QnA Maker 服务，应设置为 `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker`。

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

app.js 中的其余代码通过启动相应的对话来处理 `l_homeautomation`、`l_weather` 和 `None` 意向。 对于 `q_faq` 意向，它将回复 QnAMaker 服务中的答案。

> [!NOTE] 
> 如果意向名称 `l_homeautomation`、`l_weather` 或 `q_faq` 与使用 Dispatch 创建的 LUIS 应用不匹配，请进行编辑，使其与 [LUIS 门户](https://www.luis.ai)中显示的意向名称相匹配。

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>运行机器人

使用 [Bot Framework Emulator](../bot-service-debug-emulator.md) 测试机器人。 发送“开灯”等消息，将消息调度到家庭自动化 LUIS 应用；发送“获取西雅图天气”等消息，调度到天气 LUIS 应用。

> [!NOTE] 
> 在运行机器人之前，请确保已在 [LUIS 门户](https://www.luis.ai)中发布创建的所有 LUIS 应用，并检查是否已在 [QnA Maker 门户](https://qnamaker.ai)中发布 QnA Maker 服务。

![将消息发送到调度机器人](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>评估调度程序的性能

有时会在 LUIS 应用和 QnA Maker 服务中提供用户消息作为示例，而针对这些输入，Dispatch 生成的合并 LUIS 应用将无法很好地执行相关操作。 使用 `eval` 选项检查应用性能。 

```
dispatch eval
```

运行 `dispatch eval` 生成一个 Summary.html 文件，该文件提供有关新语言模型性能的统计信息。

> [!TIP] 
> 实际上，可在任何 LUIS 应用上运行 `dispatch eval`，而不仅仅是在 Dispatch 工具创建的 LUIS 应用上运行。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>编辑重复和重叠的意向

在 Summary.html 中查看标记为重复的示例话语，并删除类似或重叠的示例。 例如，假设在用于家庭自动化的 `homeautomation` LUIS 应用中，“开灯”请求将映射到“TurnOnLights”意向，而“为什么灯未打开？”请求 将映射到“None”意向，以便可将其传递给 QnA Maker。 使用 Dispatch 合并 LUIS 应用和 QnA Maker 服务时，需执行以下某个操作： 

* 从原始 `homeautomation` LUIS 应用中删除“None”意向，并将该意向中的话语添加到调度程序应用中的“None”意向。
* 如果不从原始 LUIS 应用中删除“None”意向，则需要在机器人中添加逻辑，以将那些与该意向匹配的消息传递给 QnA Maker 服务。

> [!TIP] 
> 有关提高语言模型性能的技巧，请参阅[语言理解的最佳做法](./bot-builder-concept-luis.md#best-practices-for-language-understanding)。

## <a name="additional-resources"></a>其他资源

* [将 LUIS 用于聊天流][luis-v4-how-to]

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js



