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
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336267"
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

使用仿真器遵照 [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) 或 [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md) **自述文件**中的说明生成并运行示例。 

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

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

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>评估调度程序的性能

有时会在 LUIS 应用和 QnA Maker 服务中提供用户消息作为示例，而针对这些输入，Dispatch 生成的合并 LUIS 应用将无法很好地执行相关操作。 可以使用 `eval` 选项检查应用的性能。

```shell
dispatch eval
```

运行 `dispatch eval` 生成一个 **Summary.html** 文件，该文件提供有关语言模型的预测性能的统计信息。 可在任何 LUIS 应用上运行 `dispatch eval`，而不仅仅是在 Dispatch 工具创建的 LUIS 应用上运行。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>编辑重复和重叠的意向

在 Summary.html 中查看标记为重复的示例话语，并删除类似或重叠的示例。 例如，假设在 `Home Automation` LUIS 应用中，“开灯”请求将映射到“TurnOnLights”意向，而“为什么灯未打开？”请求 将映射到“None”意向，以便可将其传递给 QnA Maker。 使用 Dispatch 合并 LUIS 应用和 QnA Maker 服务时，需执行以下某个操作：

* 从原始 `Home Automation` LUIS 应用中删除“None”意向，并将该意向中的话语添加到调度程序应用中的“None”意向。
* 如果不从原始 LUIS 应用中删除“None”意向，则需要在机器人中添加逻辑，以将那些与该意向匹配的消息传递给 QnA Maker 服务。


## <a name="additional-resources"></a>其他资源 

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
