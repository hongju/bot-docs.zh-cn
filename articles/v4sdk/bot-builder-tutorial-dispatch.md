---
title: 使用多个 LUIS 和 QnA 模型 | Microsoft Docs
description: 了解如何在机器人中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, Dispatch 工具, 多个服务, 路由意向
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c798c26f108458e1caeb16aa22c02c6e7c70fb61
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323653"
---
# <a name="use-multiple-luis-and-qna-models"></a>使用多个 LUIS 和 QnA 模型

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本教程演示当机器人支持对不同的方案使用多个 LUIS 模型和 QnA Maker 服务时，如何使用 Dispatch 服务来路由话语。 在本例中，我们将围绕家庭自动化和天气信息为 Dispatch 配置多个聊天 LUIS 模型，并配置 QnA Maker 服务以基于输入的 FAQ 文本文件来回答问题。 本示例结合了以下服务。

| 名称 | Description |
|------|------|
| 家庭自动化 | 一个可以识别包含关联实体数据的家庭自动化意向的 LUIS 应用。|
| 天气 | 一个可以识别包含位置数据的 `Weather.GetForecast` 和 `Weather.GetCondition` 意向的 LUIS 应用。|
| 常见问题解答  | 一个可以提供有关机器人的一些简单问题的答案的 QnA Maker 知识库。 |

## <a name="prerequisites"></a>先决条件

- 本文中的代码基于**采用 Dispatch 的 NLP** 示例。 需要获取 [C# ](https://aka.ms/dispatch-sample-cs) 或 [JS](https://aka.ms/dispatch-sample-js) 示例的副本。
- 需要了解[机器人基础知识](bot-builder-basics.md)、[自然语言处理](bot-builder-howto-v4-luis.md)、[QnA Maker](bot-builder-howto-qna.md) 和 [.bot](bot-file-basics.md) 文件。
- 用于测试的 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download)。

## <a name="create-the-services-and-test-the-bot"></a>创建服务并测试机器人

可以按照 [C#](https://aka.ms/dispatch-sample-readme-cs) 或 [JS](https://aka.ms/dispatch-sample-readme-js) 的**自述文件**说明操作，以便通过命令行界面调用创建此机器人，也可执行下面的步骤，通过 Azure、LUIS 和 QnAMaker 用户界面手动创建机器人。

 ### <a name="create-your-bot-using-service-ui"></a>通过服务 UI 创建机器人
 
若要开始手动创建机器人，请将 GitHub [BotFramework-Samples](https://aka.ms/botdispatchgitsamples) 存储库中的下述 4 个文件下载到本地文件夹：[home-automation.json](https://aka.ms/dispatch-home-automation-json)、[weather.json](https://aka.ms/dispatch-weather-json)、[nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json)、[QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv) 若要完成此操作，一种方法是打开上面的 GitHub 存储库链接，单击“BotFramework-Samples”，然后将存储库“克隆或下载”到本地计算机。 请注意，这些文件所在的存储库不同于在先决条件中提到的示例。

### <a name="manually-create-luis-apps"></a>手动创建 LUIS 应用

登录到 [LUIS Web 门户](https://www.luis.ai/)。 在“我的应用”部分，选择“导入新应用”选项卡。 此时会出现以下对话框：

![导入 LUIS json 文件](./media/tutorial-dispatch/import-new-luis-app.png)

选择“选择应用文件”按钮，然后选择已下载文件“home-automation.json”。 将可选名称字段留空。 选择“完成”。

当 LUIS 打开“家庭自动化”应用以后，请选择“训练”按钮。 这样就会使用刚刚通过“home-automation.json”文件导入的话语集来训练应用。

训练完成后，请选择“发布”按钮。 此时会出现以下对话框：

![发布 LUIS 应用](./media/tutorial-dispatch/publish-luis-app.png)

选择“生产”环境，然后选择“发布”按钮。

发布新的 LUIS 应用以后，请选择“管理”选项卡。在“应用程序信息”页中，记录 `Application ID` 和 `Display name` 值。 在“密钥和终结点”页中，记录 `Authoring Key` 和 `Region` 值。 这些值稍后由“nlp-with-dispatch.bot”文件使用。

完成后，即可训练并发布 LUIS 天气应用和 LUIS 调度应用，方法是针对已下载到本地的“weather.json”和“nlp-with-dispatchDispatch.json”文件重复这些相同的步骤。

### <a name="manually-create-qna-maker-app"></a>手动创建 QnA Maker 应用

若要设置 QnA Maker 知识库，第一步是在 Azure 中设置 QnA Maker 服务。 为此，请按[此处](https://aka.ms/create-qna-maker)提供的分步说明操作。 现在登录到 [QnAMaker Web 门户](https://qnamaker.ai)。 向下移动到步骤 2

![创建 QnA 的步骤 2](./media/tutorial-dispatch/create-qna-step-2.png)

并选择
1. Azure AD 帐户。
1. Azure 订阅名称。
1. 为 QnA Maker 服务创建的名称。 （如果你的 Azure QnA 服务一开始没有显示在此下拉列表中，请尝试刷新页面。） 

移动到步骤 3

![创建 QnA 的步骤 3](./media/tutorial-dispatch/create-qna-step-3.png)

为 QnA Maker 知识库提供一个名称。 对于本示例，我们将使用名称“sample-qna”。

移动到步骤 4

![创建 QnA 的步骤 4](./media/tutorial-dispatch/create-qna-step-4.png)

选择“+ 添加文件”选项，然后选择已下载文件“QnAMaker.tsv”

可以通过其他选项向知识库添加聊天个性化内容，但我们的示例不包括该选项。

选择“保存并训练”，完成后，选择“发布”选项卡来发布应用。

发布 QnA Maker 应用以后，请选择“设置”选项卡，然后向下滚动到“部署详细信息”。 记录 _Postman_ 示例 HTTP 请求中的以下值。

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
这些值稍后由“nlp-with-dispatch.bot”文件使用。

### <a name="manually-update-your-bot-file"></a>手动更新 .bot 文件

创建所有服务应用以后，需将每个应用的信息添加到“nlp-with-dispatch.bot”文件中。 打开以前下载的 C# 或 JS 示例文件中的此文件。 将以下值添加到 "type": "luis" 或 "type": "dispatch" 所在的每个节

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

对于 "type": "qna" 所在的节，请添加以下值：

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

当所有更改都已完成后，保存此文件。

### <a name="test-your-bot"></a>测试机器人

现在，请使用模拟器运行示例。 打开模拟器以后，选择“nlp-with-dispatch.bot”文件。

为便于参考，下面提供了所包含的服务使用的某些问题和命令：

* QnA Maker
  * `hi`、`good morning`
  * `what are you`、`what do you do`
* LUIS（家庭自动化）
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS（天气）
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>从机器人连接到服务

若要连接到 Dispatch、LUIS 和 QnA Maker 服务，机器人需从 **.bot** 文件中提取信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，`ConfigureServices` 将读取配置文件，而 `InitBotServices` 将使用该信息来初始化服务。 每次创建机器人时，都会使用已注册的 `BotServices` 对象初始化该机器人。 下面是这两种方法的相关部分。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
以下代码初始化机器人对外部服务的引用。 例如，此处创建了 LUIS 和 QnaMaker 服务。 这些外部服务是使用 `BotConfiguration` 类基于“.bot”文件的内容配置的。

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

示例代码使用预定义的命名常量来标识 .bot 文件的各个节。 如果已修改 _nlp-with-dispatch.bot_ 文件的原始示例命名中的任何节名称，请确保找到 **bot.js**、**homeAutomation.js**、**qna.js** 或 **weather.js** 文件中的关联常量声明，并将该条目更改为已修改的名称。  
```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

在 **bot.js** 中，包含在配置文件 _nlp-with-dispatch.bot_ 中的信息用于将调度机器人连接到各种服务。 每个构造函数都会根据上面详述的节名称来查找并使用配置文件的相应节。

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
---

### <a name="calling-the-services-from-your-bot"></a>从机器人调用服务

机器人逻辑根据组合的 Dispatch 模型检查用户输入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **NlpDispatchBot.cs** 文件中，机器人的构造函数获取我们在启动时注册的 `BotServices` 对象。

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

在机器人的 `OnTurnAsync` 方法中，根据 Dispatch 模型检查来自用户的传入消息。

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **bot.js** `onTurn` 方法中，我们检查是否有来自用户的传入消息。 如果收到类型 _ActivityType.Message_，则表明该消息是通过机器人的 _dispatchRecognizer_ 发送出去的。

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
---

### <a name="working-with-the-recognition-results"></a>处理识别结果

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

当模型生成结果时，它会指示哪个服务最适合用于处理话语。 此机器人中的代码将请求路由到相应的服务，然后汇总被调用服务返回的响应。

```csharp
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

// Dispatches the turn to the request QnAMaker app.
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}


// Dispatches the turn to the requested LUIS model.
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

当模型生成结果时，它会指示哪个服务最适合用于处理话语。 此机器人中的代码将请求路由到相应的服务。

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 
 // In homeAutomation.js
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
    
// In weather.js
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
    
// In qna.js
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>编辑意向以提升性能

待机器人运行以后，即可删除类似的或重叠的话语，改进机器人的性能。 例如，假设在 `Home Automation` LUIS 应用中，“开灯”请求将映射到“TurnOnLights”意向，而“为什么灯未打开？”请求 将映射到“None”意向，以便可将其传递给 QnA Maker。 使用 Dispatch 合并 LUIS 应用和 QnA Maker 服务时，需执行以下某个操作：

* 从原始 `Home Automation` LUIS 应用中删除“None”意向，改将该意向中的话语添加到调度程序应用中的“None”意向。
* 如果不从原始 LUIS 应用中删除“None”意向，则需在机器人中添加逻辑，将那些与“None”意向匹配的消息传递给 QnA Maker 服务。

上述两项操作中的任一操作都会减少机器人使用消息“找不到答案”来回应用户的次数。 

## <a name="additional-resources"></a>其他资源

**更新或创建新的 LUIS 模型：** 此示例基于预先配置的 LUIS 模型。 有关如何更新此模型或创建新的 LUIS 模型的其他信息，可参阅[此文](https://aka.ms/create-luis-model#updating-your-cognitive-models
)。

**删除资源：** 此示例将创建许多应用程序和资源，可以使用下面列出的步骤将其删除，但不应删除其他任何应用或服务依赖的资源。 

_LUIS 资源_
1. 登录到 [luis.ai](https://www.luis.ai) 门户。
1. 转到“我的应用”页。
1. 选择本示例创建的应用。
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. 单击“删除”，然后单击“确定”以确认。

_QnA Maker 资源_
1. 登录到 [qnamaker.ai](https://www.qnamaker.ai/) 门户。
1. 转到“我的知识库”页。
1. 单击 `Sample QnA` 知识库对应的删除按钮，然后单击“删除”以确认。

**最佳做法：** 若要改进本示例中使用的服务，请参阅 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) 和 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices) 的最佳做法。