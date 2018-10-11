---
title: 将 LUIS 用于语言理解 | Microsoft Docs
description: 了解如何借助 Bot Builder SDK 将 LUIS 用于自然语言理解。
keywords: 语言理解, LUIS, 意向, 识别器, 实体, 中间件
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4a7cfba6f588c95dbebf4886ffd7e432d99c3ae
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389796"
---
# <a name="using-luis-for-language-understanding"></a>将 LUIS 用于语言理解

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

理解用户在会话和上下文中表达的含义是一项艰巨的任务，但可以让你的机器人更自然地进行会话。 使用语言理解（称为 LUIS）能够实现此目标，使机器人能够识别用户消息的意向，接收用户更自然的语言，并更好地指导会话流程。 如果需要了解 LUIS 与机器人集成的更多背景信息，请参阅[机器人语言理解](./bot-builder-concept-LUIS.md). 

本主题将指导你设置可使用 LUIS 识别多个不同意向的简单机器人。

## <a name="installing-packages"></a>安装程序包

首先，确保拥有 LUIS 所需的包。

# <a name="ctabcs"></a>[C#](#tab/cs)

[添加](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)对以下 NuGet 程序包的 v4 版本的引用：


* `Microsoft.Bot.Builder.AI.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

使用 npm 在项目中安装 botbuilder 和 botbuilder-ai 程序包：

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`

---

## <a name="set-up-your-luis-app"></a>设置 LUIS 应用

首先，设置一个 _LUIS 应用_，它是在 [luis.ai](https://www.luis.ai) 上创建的服务。 可以对该 LUIS 应用定型它应能识别的某些意向。 有关如何创建 LUIS 应用的详细信息，请访问 LUIS 网站。

对于本示例，你将只使用可识别“帮助”、“取消”和“天气”意向的演示 LUIS 应用；应用 ID 已包含在示例代码中。 你需要拥有一个认知服务密钥，可以通过登录到 [www.luis.ai](https://www.luis.ai) 并从“用户设置” > “创作密钥”中复制密钥来获取该密钥。

> [!NOTE]
> 若要创建本示例中使用的公共 LUIS 应用的副本，请复制 [JSON](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS 文件。 然后，[导入](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) LUIS 应用，[训练](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)并[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)该应用。 将示例代码中的公共 APP ID 替换为你的新 LUIS 应用的应用 ID。


### <a name="configure-your-bot-to-call-your-luis-app"></a>对机器人进行配置以调用 LUIS 应用。

# <a name="ctabcs"></a>[C#](#tab/cs)

虽然可以在每个轮次中同时创建并调用 LUIS 应用，但更好的编码做法是将 LUIS 服务注册为单一实例，然后将其作为参数传递给机器人的构造函数。 此处，我们将展示该方法，因为它稍微有点复杂。

首先使用 Echo 机器人模板，然后打开 Startup.cs。 

为 `Microsoft.Bot.Builder.AI.LUIS` 添加一个 `using` 语句

```csharp
// add this
using Microsoft.Bot.Builder.AI.LUIS;
```

在 `ConfigureServices` 的末尾在状态初始化之前添加以下代码。 这将从 `appsettings.json` 文件获取信息，但那些字符串可以从 `.bot` 文件获取（就像本文末尾链接的示例那样），用于测试时，还可以硬编码。

单一实例将一个新的 `LuisRecognizer` 返回到构造函数。

```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        // Get LUIS information from appsettings.json.
        var section = this.Configuration.GetSection("Luis");
        var luisApp = new LuisApplication(
            applicationId: section["AppId"],
            endpointKey: section["SubscriptionKey"],
            azureRegion: section["Region"]);

        // Specify LUIS options. These may vary for your bot.
        var luisPredictionOptions = new LuisPredictionOptions
        {
            IncludeAllIntents = true,
        };

        return new LuisRecognizer(
            application: luisApp,
            predictionOptions: luisPredictionOptions,
            includeApiResults: true);
    });
```

粘贴来自 [luis.ai](https://www.luis.ai) 的订阅密钥来替代 `<subscriptionKey>`。 可以通过以下方法轻松找到此密钥：在右上角单击你的帐户名称，转到“设置”，在这里，它被称为**创作密钥**。

> [!NOTE]
> 如果你使用的是自己的 LUIS 应用，而不是公共应用，则可以从 [luis.ai](https://www.luis.ai) 获取 LUIS 应用的 ID、订阅密钥和 URL。 可以在你的应用页面上的“发布”和“设置”选项卡下找到这些项。
>
>通过登录到 [luis.ai](https://www.luis.ai)、转到“发布”选项卡，然后在“资源和密钥”下查找“终结点”一列，可以找到要在 `LuisModel` 中使用的基 URL。 基 URL 是终结点 URL 的一部分，位于订阅 ID 和其他参数前面。

接下来，我们需要向机器人提供此 LUIS 实例。 打开 **EchoBot.cs**，并在文件顶部添加以下代码。 为方便你查看，还包括了类标题和状态项，但这里我们不会对它们进行解释。

```csharp
public class EchoBot : IBot
{
    /// <summary>
    /// Gets the Echo Bot state.
    /// </summary>
    private IStatePropertyAccessor<EchoState> EchoStateAccessor { get; }

    /// <summary>
    /// Gets the LUIS recognizer.
    /// </summary>
    private LuisRecognizer Recognizer { get; } = null;

    public EchoBot(ConversationState state, LuisRecognizer luis)
    {
        EchoStateAccessor = state.CreateProperty<EchoState>("EchoBot.EchoState");

        // The incoming luis variable is the LUIS Recognizer we added above.
        this.Recognizer = luis ?? throw new ArgumentNullException(nameof(luis));
    }

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

首先，按照 JavaScript [快速入门](../javascript/bot-builder-javascript-quickstart.md)中的步骤创建一个机器人。 这里，我们将 LUIS 信息硬编码到机器人中，但可以从 `.bot` 文件中拉取该信息，就像本文末尾链接的示例中所做的那样。

在新机器人中，编辑 **app.js** 来要求提供 `LuisRecognizer` 类并为 LUIS 模型创建实例：

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

const luisApplication = {
    // This appID is for a public app that's made available for demo purposes
    // You can use it or use your own LUIS "Application ID" at https://www.luis.ai under "App Settings".
     applicationId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // Replace endpointKey with your "Subscription Key"
    // your key is at https://www.luis.ai under Publish > Resources and Keys, look in the Endpoint column
    // The "subscription-key" is embeded in the Endpoint link. 
    endpointKey: '<your subscription key>',
    // You can find your app's region info embeded in the Endpoint link as well.
    // Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    azureRegion: 'westus'
}

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
}

// Create the bot that handles incoming Activities.
const luisBot = new LuisBot(luisApplication, luisPredictionOptions);
```

然后，在机器人 `LuisBot` 的构造函数中，让应用程序创建 LuisRecognizer 实例。

```javascript
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {object} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {object} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }
```

> [!NOTE] 
> 如果你使用的是自己的 LUIS 应用，而不是公共应用，可以从 [https://www.luis.ai](https://www.luis.ai) 获取应用程序 ID、订阅密钥和区域。 可以在你的应用页面上的“发布”和“设置”选项卡下找到这些项。

---

现在，已为你的机器人配置了 LUIS 语言理解。 接下来，让我们看看如何从 LUIS 中获取意向。

## <a name="get-the-intent-by-calling-luis"></a>通过调用 LUIS 获取意向

你的机器人通过调用 LUIS 识别器从 LUIS 获取结果。

# <a name="ctabcs"></a>[C#](#tab/cs)

要让机器人根据 LUIS 应用检测到的意向发送回复，请调用 `LuisRecognizer` 来获取 `RecognizerResult`： 每当需要获取 LUIS 意向时都可以在代码中执行此操作。

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
// add this reference
using Microsoft.Bot.Builder.AI.LUIS;

namespace EchoBot
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Echo bot turn handler 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurnAsync(ITurnContext context, System.Threading.CancellationToken token)
        {            
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Call LUIS recognizer
                var result = this.Recognizer.RecognizeAsync(context, System.Threading.CancellationToken.None);
                
                var topIntent = result?.GetTopScoringIntent();

                switch ((topIntent != null) ? topIntent.Value.intent : null)
                {
                    case null:
                        await context.SendActivity("Failed to get results from LUIS.");
                        break;
                    case "None":
                        await context.SendActivity("Sorry, I don't understand.");
                        break;
                    case "Help":
                        await context.SendActivity("<here's some help>");
                        break;
                    case "Cancel":
                        // Cancel the process.
                        await context.SendActivity("<cancelling the process>");
                        break;
                    case "Weather":
                        // Report the weather.
                        await context.SendActivity("The weather today is sunny.");
                        break;
                    default:
                        // Received an intent we didn't expect, so send its name and score.
                        await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                        break;
                }
            }
        }
    }    
}

```

在表达中识别的任何意向将作为意向名称的映射返回到分数，并且可以从 `result.Intents` 访问。 提供静态 `LuisRecognizer.topIntent()` 方法帮助简化查找结果集的最高得分意向的过程。

识别的任何实体将作为实体名称的映射返回到值，并使用 `results.entities` 进行访问。 在创建 LuisRecognizer 时，可以通过传递 `verbose=true` 设置来返回其他实体元数据。 然后，可以使用 `results.entities.$instance` 访问添加的元数据。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

编辑用于侦听传入活动的代码，以便它调用 `LuisRecognizer` 来获取 `RecognizerResult`。

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;
            
            switch (topIntent) {
                case 'None':
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;                        
                case 'null':
                    await context.sendActivity("Failed to get results from LUIS.")
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.sendActivity(`The top intent was ${topIntent}`);
            }
        }
    });
});
```

在表达中识别的任何意向将作为意向名称的映射返回到分数，并且可以从 `results.intents` 访问。 提供静态 `LuisRecognizer.topIntent()` 方法帮助简化查找结果集的最高得分意向的过程。


---

尝试在 Bot Framework Emulator 中运行机器人，并对它说出“天气”、“帮助”和“取消”之类的内容。

![运行机器人](./media/how-to-luis/run-luis-bot.png)

## <a name="extract-entities"></a>提取实体

除了识别意向外，LUIS 应用还可以提取实体，这些实体是实现用户请求的重要词汇。 例如，对于天气机器人，LUIS 应用可能能够从用户的消息中提取天气报告的位置。

构造聊天结构的一种常用方法是识别用户消息中的任何实体，并提示找不到的任何所需的实体。 然后，后续步骤会处理对提示的响应。

# <a name="ctabcs"></a>[C#](#tab/cs)

假设用户发出的信息是“西雅图天气怎么样”？ `LuisRecognizer` 通过 `Entities` 属性向你提供 `RecognizerResult`，该属性具有以下结构：

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

可以向机器人添加以下帮助程序函数来通过 LUIS 从 `RecognizerResult` 外部获取实体。 它将需要使用 `Newtonsoft.Json.Linq` 库，必须将该库添加到 **using** 语句中。

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

从聊天中的多个步骤收集实体等信息时，在你的状态中保存所需信息会很有帮助。 如果发现某个实体，则可以将其添加到适当的状态字段中。 在聊天中，如果当前步骤已填写了关联的字段，则可以跳过提示输入该信息的步骤。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

假设用户发出的信息是“西雅图天气怎么样”？ `LuisRecognizer` 通过 `entities` 属性向你提供 `RecognizerResult`，该属性具有以下结构：

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

此 `findEntities` 函数查找由 LUIS 应用识别的与传入的 `entityName` 匹配的任何实体。


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

从聊天中的多个步骤收集实体等信息时，在你的状态中保存所需信息会很有帮助。 如果发现某个实体，则可以将其添加到适当的状态字段中。 在聊天中，如果当前步骤已填写了关联的字段，则可以跳过提示输入该信息的步骤。

## <a name="additional-resources"></a>其他资源

有关使用 LUIS 的示例，请查看适用于 [[C#](https://aka.ms/cs-luis-sample)] 或 [[JavaScript](https://aka.ms/js-luis-sample)] 的项目。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用调度工具组合 LUIS 和 QnA 服务](./bot-builder-tutorial-dispatch.md)
