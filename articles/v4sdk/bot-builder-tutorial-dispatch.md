---
title: 将 LUIS 和 QnA 服务与 Dispatch 工具配合使用 | Microsoft Docs
description: 了解如何在机器人中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, Dispatch 工具, 多个服务
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4d029dc7361ac8a7fadb61141faf60d8a62eab3c
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736675"
---
# <a name="use-luis-and-qna-services-with-the-dispatch-tool"></a>将 LUIS 和 QnA 服务与 Dispatch 工具配合使用

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

本教程演示如何使用 Dispatch 工具生成的 LUIS 模型，将机器人与多个语言理解智能服务 (LUIS) 应用和 QnAMaker 服务集成。 本示例结合了以下服务。

| 服务类型 | 名称 | Description |
|------|------|------|
| LUIS 应用 | 家庭自动化 | 识别具有关联实体数据的家庭自动化意向。|
| LUIS 应用 | 天气 | 识别包含位置数据的 Weather.GetForecast 和 Weather.GetCondition 意向。|
| QnAMaker 服务 | 常见问题解答  | 提供有关机器人的一些简单问题的答案。 |

本文的代码摘自**使用 Dispatch 的 NLP** 示例 [[C#](https://aka.ms/dispatch-sample-cs)]。

<!-- | [JS](https://aka.ms/dispatch-sample-js)-->

请参阅[语言理解](bot-builder-concept-luis.md)获取语言服务的概述。 请参阅 [LUIS](bot-builder-howto-v4-luis.md) 和 [QnA Maker](bot-builder-howto-qna.md) 的相关文章，获取有关在机器人中实现这些功能的说明。

可以遵照示例自述文件中的说明来设置和测试机器人，或者直接跳转到[有关代码的说明](#notes-about-the-code)。

## <a name="create-the-services-and-test-the-bot"></a>创建服务并测试机器人

遵照示例自述文件中的说明操作。 我们将使用 CLI 工具来创建和发布这些服务，并在配置 (**.bot**) 文件中更新有关这些服务的信息。

1. 克隆或提取示例存储库。
1. 安装 BotBuilder CLI 工具。
1. 手动配置所需的服务。

### <a name="test-your-bot"></a>测试机器人

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用 Visual Studio 或 Visual Studio Code 启动机器人。

<!--
# [JavaScript](#tab/javascript)
-->

---

将机器人连接到 Bot Framework Emulator。

下面是包含的服务使用的输入：

* QnA Maker
  * `hi`、`good morning`
  * `what are you`、`what do you do`
* LUIS（家庭自动化）
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS（天气）
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>有关代码的说明

### <a name="packages"></a>包

本示例使用以下包。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

这些 [NuGet 包](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package)的最新 v4 版本。

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>BotBuilder CLI 工具

本示例使用这些 [BotBuilder CLI 工具](https://aka.ms/botbuilder-tools-readme)（可通过 npm 获取）来创建、训练和发布 LUIS、QnA Maker 与 Dispatch 服务，以及在机器人的配置 (**.bot**) 文件中记录有关这些服务的信息。

* [Dispatch](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot](https://aka.ms/botbuilder-tools-msbot-readme)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> 为确保使用 npm 和这些 CLI 工具的最新版本，请运行以下命令。
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

使用工具设置服务后，本示例的 **.bot** 文件应如下所示。 （可以运行 `msbot secret -n` 来加密此文件中的敏感值。）

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>从机器人连接到服务

若要连接到 Dispatch、LUIS 和 QnA Maker 服务，机器人需从 **.bot** 文件中提取信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **Startup.cs** 中，`ConfigureServices` 将读取配置文件，而 `InitBotServices` 将使用该信息来初始化服务。 每次创建机器人时，都会使用已注册的 `BotServices` 对象初始化该机器人。 下面是这两种方法的相关部分。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

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
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
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

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
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

运行 `dispatch eval` 生成一个 **Summary.html** 文件，该文件提供有关语言模型的预测性能的统计信息。

> [!TIP]
> 可在任何 LUIS 应用上运行 `dispatch eval`，而不仅仅是在 Dispatch 工具创建的 LUIS 应用上运行。

### <a name="edit-intents-for-duplicates-and-overlaps"></a>编辑重复和重叠的意向

在 Summary.html 中查看标记为重复的示例话语，并删除类似或重叠的示例。 例如，假设在 `Home Automation` LUIS 应用中，“开灯”请求将映射到“TurnOnLights”意向，而“为什么灯未打开？”请求 将映射到“None”意向，以便可将其传递给 QnA Maker。 使用 Dispatch 合并 LUIS 应用和 QnA Maker 服务时，需执行以下某个操作：

* 从原始 `Home Automation` LUIS 应用中删除“None”意向，并将该意向中的话语添加到调度程序应用中的“None”意向。
* 如果不从原始 LUIS 应用中删除“None”意向，则需要在机器人中添加逻辑，以将那些与该意向匹配的消息传递给 QnA Maker 服务。

> [!TIP]
> 有关提高语言模型性能的技巧，请参阅[语言理解的最佳做法](./bot-builder-concept-luis.md#best-practices-for-language-understanding)。

## <a name="to-clean-up-resources-from-this-sample"></a>清理本示例中的资源

本示例创建了多个应用程序和资源。 可遵照以下说明删除这些资源。

### <a name="luis-resources"></a>LUIS 资源

1. 登录到 [luis.ai](https://www.luis.ai) 门户。
1. 转到“我的应用”页。
1. 选择本示例创建的应用。
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. 单击“删除”，然后单击“确定”以确认。

### <a name="qna-maker-resources"></a>QnA Maker 资源

1. 登录到 [qnamaker.ai](https://www.qnamaker.ai/) 门户。
1. 转到“我的知识库”页。
1. 单击 `Sample QnA` 知识库对应的删除按钮，然后单击“删除”以确认。

### <a name="azure-resources"></a>Azure 资源

> [!WARNING]
> 不要删除其他任何应用或服务所依赖的任何资源。

1. 登录到 [Azure 门户](https://portal.azure.com/)。
1. 转到为示例创建的认知服务资源的“概述”页。
1. 单击“删除”，然后单击“是”以确认。

## <a name="additional-resources"></a>其他资源

* [语言理解](bot-builder-concept-luis.md)
* [设计知识机器人](../bot-service-design-pattern-knowledge-base.md)
* [配置语音启动](../bot-service-manage-speech-priming.md)
* [将 LUIS 用于语言理解](bot-builder-howto-v4-luis.md)
* [使用 QnA Maker 回答问题](bot-builder-howto-qna.md)
