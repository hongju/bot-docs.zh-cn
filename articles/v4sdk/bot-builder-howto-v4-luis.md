---
title: 将 LUIS 用于语言理解 | Microsoft Docs
description: 了解如何借助 Bot Builder SDK 将 LUIS 用于自然语言理解。
keywords: 语言理解, LUIS, 意向, 识别器, 实体, 中间件
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/30/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9a04a73e8ac8716ec528586e069aa2d16d90e28f
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2018
ms.locfileid: "39298562"
---
# <a name="using-luis-for-language-understanding"></a>将 LUIS 用于语言理解

理解用户在会话和上下文中表达的含义是一项艰巨的任务，但可以让你的机器人更自然地进行会话。 使用语言理解（称为 LUIS）能够实现此目标，使机器人能够识别用户消息的意向，接收用户更自然的语言，并更好地指导会话流程。 如果需要了解 LUIS 与机器人集成的更多背景信息，请参阅[机器人语言理解](./bot-builder-concept-LUIS.md). 

本主题将指导你设置可使用 LUIS 识别多个不同意向的简单机器人。

## <a name="installing-packages"></a>安装程序包

首先，确保拥有 LUIS 所需的包。

# <a name="ctabcs"></a>[C#](#tab/cs)

在以下 NuGet 包的 v4 预发布版本中[添加引用](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)：


* `Microsoft.Bot.Builder.Ai.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

可以通过 npm 在项目中添加对 botbuilder 和 botbuilder-ai 包的引用：

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="set-up-middleware-to-use-your-luis-app"></a>设置中间件以使用 LUIS 应用

首先，设置 LUIS 应用，它是你在 [www.luis.ai](https://www.luis.ai) 上创建的服务。 可以对该 LUIS 应用定型它应能识别的某些意向。 有关如何创建 LUIS 应用的详细信息，请访问 [LUIS 网站](https://www.luis.ai)。

对于本示例，你将只使用可识别“帮助”、“取消”和“天气”意向的演示 LUIS 应用；应用 ID 已包含在示例代码中。 需要拥有认知服务密钥，可通过登录到 [www.luis.ai](https://www.luis.ai) 并从“用户设置” > “创作密钥”中复制密钥来获取。

> [!NOTE] 
> 若要创建本示例中使用的公共 LUIS 应用的副本，请复制 [FirstSimpleBotExample.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS 文件。 然后，[导入 LUIS 应用](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)、[定型](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)并[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)该应用。 将示例代码中的公共 APP ID 替换为你的新 LUIS 应用的应用 ID。

将机器人配置为在收到用户的每条消息时调用 LUIS 应用，只需将其添加到机器人的中间件堆栈即可。 中间件将识别结果存储在上下文对象中，然后可以由机器人逻辑访问。

# <a name="ctabcs"></a>[C#](#tab/cs)
首先使用 Echo 机器人模板，然后打开 Startup.cs。 

为 `Microsoft.Bot.Builder.Ai.LUIS` 添加一个 `using` 语句

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
// add this
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;
```

在 `Startup.cs` 文件中更新 `ConfigureServices` 方法，添加连接到 LUIS 应用的 `LuisRecognizerMiddleware` 对象。 

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "eb0bf5e0-b468-421b-9375-fdfb644c512e",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

<!--TODO: DOES PUBLIC APP WORK WITH KEYS IN DIFFERENT REGIIONS? --> 在 [https://www.luis.ai](https://www.luis.ai) 中粘贴你的订阅密钥来替代 `<subscriptionKey>`。

> [!NOTE] 
> 如果你使用的是自己的 LUIS 应用，而不是公共 LUIS，可以从 [https://www.luis.ai](https://www.luis.ai) 获取 LUIS 应用的 ID、订阅密钥和 URL。 
>
>通过登录到 [LUIS 网站](https://www.luis.ai)、转到“发布”选项卡，然后在“资源和密钥”下查找“终结点”一列，可以找到要在 `LuisModel` 中使用的基 URL。 基 URL 是终结点 URL 的一部分，位于订阅 ID 和其他参数前面。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

首先在 [LuisRecognizer](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.luisrecognizer.md) 类中要求提供/导入并为 LUIS 模型创建一个实例：

```javascript
const { LuisRecognizer } = require('botbuilder-ai');

const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription key>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
```

将模型添加到中间件堆栈：

```javascript
adapter.use(model);
```


> [!NOTE] 
> 如果你使用的是自己的 LUIS 应用，而不是公共 LUIS，可以从 [https://www.luis.ai](https://www.luis.ai) 获取 LUIS 应用的 ID、订阅 ID 和 URL。 

---



LUIS 语言理解现已插入你的机器人。 接下来，让我们看看如何从上下文对象中存储的 LUIS 模型中获取意向。


## <a name="get-the-intent-from-the-turn-context"></a>从轮次上下文中获取意向

然后，可以使用每个会话轮次的上下文从机器人访问 LUIS 的结果。

# <a name="ctabcs"></a>[C#](#tab/cs)

要让机器人根据 LUIS 应用检测到的意向发送回复，请使用以下内容替换 `OnTurn` 中的代码：

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot1
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, checks the /// intent from the LUIS recognizer, and sends a reply based on the recognized intent
        /// </summary>
        /// <param name="context">Turn-scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                var result = context.Services.Get<RecognizerResult>
                    (LuisRecognizerMiddleware.LuisRecognizerResultKey);
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
                        // Cancel the process.
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const results = model.get(context);
            const topIntent = LuisRecognizer.topIntent(results);
            switch (topIntent) {

                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;
                case 'None':                    
                    await context.sendActivity("Sorry, I don't understand.")
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
识别的任何实体将作为实体名称的映射返回到值，并通过 `results.entities` 访问。 在创建 LuisRecognizer 时，可以通过传递 `verbose=true` 设置来返回其他实体元数据。 然后，可以通过 `results.entities.$instance` 访问添加的元数据。

---

<!-- TODO: SHOW RUNNING THE FIRST BOT --> 尝试在 Bot Framework Emulator 中运行机器人，并对它说出类似于“天气”、“帮助”和“取消”的内容。

![运行机器人](./media/how-to-luis/run-luis-bot.png)


## <a name="using-a-luis-recognizer-with-conversation-state"></a>使用处于会话状态的 LUIS 识别器
<!-- TBD, complete example --> 如果你对用户的回复超过一个轮次，则可以决定通过保存会话状态来跟踪你在会话中的位置。 可以使用来自 LUIS 的意向来帮助设置会话状态数据，例如会话主题是否已启动或已完成。

LUIS 识别器中间件在机器人的每一个轮次中运行，这样你就可以获得用户收到的每条消息的意向。 如果你想要基于意向启动多轮次会话流，一种方法是跳过更改主题的逻辑，直到完成当前主题。

# <a name="ctabcs"></a>[C#](#tab/cs)

在 EchoState.cs 中，将 EchoState 更改为以下内容：

```csharp
public class ConversationStateInfo 
{
    public bool WeatherTopicStarted  { get; set; }
    public bool HelpTopicStarted  { get; set; }
    public bool CancelTopicStarted  { get; set; }
}
```

在 Startup.cs 中，将 ConversationState 的初始化更改为使用 `ConversationStateInfo`。
```cs
    options.Middleware.Add(new ConversationState<ConversationStateInfo>(dataStore));
```

在 EchoBot.cs 中，编辑 `OnTurn`：
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        var text = context.Activity.Text;
        var conversationState = context.GetConversationState<ConversationStateInfo>() ?? new ConversationStateInfo();

        // Here, you can add some other logic based on the topic flags in conversation state
        // For example, if you know that a particular topic was started in a previous turn,
        // you can send the reply for that topic and bypass getting the intent from LUIS 
        if (conversationState.WeatherTopicStarted)
        {
            // Set this flag to false since this reply concludes the topic.
            conversationState.WeatherTopicStarted = false;
            // Assume that they responded to the prompt with a location.
            await context.SendActivity($"The weather in {text} is sunny.");
        }
        else
        {
            var result = context.Services.Get<RecognizerResult>
            (LuisRecognizerMiddleware.LuisRecognizerResultKey);
            var topIntent = result?.GetTopScoringIntent();
            switch ((topIntent != null) ? topIntent.Value.intent : null)
            {
                case null:
                    await context.SendActivity("Failed to get results from LUIS.");
                    break;
                case "None":
                    await context.SendActivity("Sorry, I don't understand.");
                    break;
                case "Weather":
                    conversationState.WeatherTopicStarted = true;
                    await context.SendActivity($"Looks like you want a weather forecast. What city do you want the forecast for?");

                    break;
                case "Help":
                    conversationState.HelpTopicStarted = true;
                    await context.SendActivity("<here's some help>");
                    break;
                case "Cancel":
                    // Cancel the process.
                    conversationState.CancelTopicStarted = true;
                    await context.SendActivity("<cancelling the process>");
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                    break;
            }
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在添加 `LuisRecognizer` 后，添加会话状态中间件。

```javascript
const model = new LuisRecognizer({
    // This appID is for a public app that's made available for demo purposes
    // You can use it by providing your LUIS subscription key
     appId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // replace subscriptionKey with your Authoring Key
    // your key is at https://www.luis.ai under User settings > Authoring Key 
    subscriptionKey: '<your subscription>',
    // The serviceEndpoint URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com'
});
adapter.use(model)

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

然后，就可以保存指示已开始主题的状态了。

```javascript
// Listen for incoming activities 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async(context) => {
        if (context.activity.type === 'message') {
            var utterance = context.activity.text;
            
            // Check topic flags in conversation state 
            if (conversationState.weatherTopicStarted) 
            {
                // Assume the user's message is a reply to the bot's prompt for a location
                await context.sendActivity(`The weather in ${utterance} is sunny.`);
                // This conversation flow is now finished. Set flag to false,
                // so that on the next turn the user can ask for another weather forecast.
                conversationState.WeatherTopicStarted = false;
            }
            // To add more steps to the other topics
            // you could check the topic flags here
            else 
            {
                const results = model.get(context);
                const topIntent = LuisRecognizer.topIntent(results);
                switch (topIntent) {
                    case 'None':
                        //Add app logic when there is no result
                        await context.sendActivity("<null case>")
                        break;
                    case 'Cancel':
                        conversationState.cancelTopicStarted = true;
                        await context.sendActivity("<cancelling the process>")
                        break;
                    case 'Help':
                        conversationState.helpTopicStarted = true;
                        await context.sendActivity("<here's some help>");
                        break;
                    case 'Weather':
                        conversationState.weatherTopicStarted = true;
                        await context.sendActivity("Looks like you want a weather forecast. What city do you want the forecast for?");
                        break;
                    default:
                        // Add app logic for the recognition results.
                        await context.sendActivity(`Received this intent: ${topIntent}`);
                }
            }
        }
    });
});
```

---

尝试在 Bot Framework Emulator 中运行机器人，并注意“天气”现在是两个轮次的会话流。

![运行机器人](./media/how-to-luis/run-luis-bot-2step-weather.png)


## <a name="using-luis-with-dialogs"></a>通过对话框使用 LUIS

如果使用来自 LUIS 的意向触发多轮次会话流，那么使用对话框来封装此流会很有帮助。 此示例机器人使用 LUIS 应用，该应用检测用于触发家庭自动化对话框或天气对话框的意向。

> [!NOTE] 
> 若要创建本示例中使用的公共 LUIS 应用的副本，请复制 [WeatherOrHomeAutomation.json](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/WeatherOrHomeAutomation.json) LUIS 文件。 然后，[导入 LUIS 应用](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app)、[定型](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train)并[发布](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp)该应用。 将示例代码中的公共 APP ID 替换为你的新 LUIS 应用的应用 ID。

# <a name="ctabcs"></a>[C#](#tab/cs)

首先，在 Startup.cs 中修改 `ConfigureServices`，为 LUIS 应用添加中间件。 将 `EchoBot` 重命名为 `LuisDialogBot`。
在本示例中，LUIS 应用作为 `LuisRecognizerMiddleware` 添加，检测 `homeautomation` 或 `weather` 意向，以触发这些名称的对话框。 

```csharp
public void ConfigureServices(IServiceCollection services)
{  
    // Rename EchoBot to LuisDialogBot
    services.AddBot<LuisDialogBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration); 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Use Dictionary<string, object> for the conversation state type
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));

        // Add LUIS recognizer as middleware
        options.Middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel(
                    // This appID is for a public app that's made available for demo purposes
                    "428affb6-7650-46ac-9184-68c00a4f1729",
                    // You can use it by replacing <subscriptionKey> with your Authoring Key
                    // which you can find at https://www.luis.ai under User settings > Authoring Key
                    "<subscriptionKey>",
                    // The location-based URL begins with "https://<region>.api.cognitive.microsoft.com", where region is the region associated with the key you are using. Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
                    new Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"))));
    });
}
```

将 EchoBot.cs 重命名为 LuisDialogBot.cs，然后将 `EchoBot` 类重命名为 `LuisDialogBot`。 然后，在 LuisDialogBot.cs 中添加以下 `using` 语句。

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Ai.LUIS;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
using Microsoft.Bot.Schema;
```

在 LuisDialogBot.cs 中，将以下代码添加到 `LuisDialogBot` 类。

```csharp
    public class LuisDialogBot : IBot
    {
        private DialogSet _dialogs;

        public LuisDialogBot()
        {
            _dialogs = new DialogSet();

            _dialogs.Add("homeautomation", CreateHomeAutomationWaterfall());
            _dialogs.Add("weather", CreateWeatherWaterfall());
            _dialogs.Add("weather_city", new Builder.Dialogs.TextPrompt());
        }

        // App ID for a separate LUIS app used to tell if the user wants to turn the lights on or off
        // The `homeautomation` dialogs uses results from this app to determine the "on/off" argument. 
        private static LuisModel luisHomeAutomation =
            new LuisModel("76feb726-515b-44c4-acc9-adb216965a58", "SUBSCRIPTION-KEY", new System.Uri("https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/"));

        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {

                // Create a dialog context
                var state = ConversationState<Dictionary<string, object>>.Get(context);
                var dc = _dialogs.CreateContext(context, state);

                // Run the next dialog step
                await dc.Continue();
                
                // Check if any dialog has responded on this turn
                if (!context.Responded)
                {

                    var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
                    var topIntent = luisResult?.GetTopScoringIntent();
                    var utterance = context.Activity.Text;
                    var dialogArgs = new Dictionary<string, object>();

                    switch ((topIntent != null) ? topIntent.Value.intent.ToLowerInvariant() : null)
                    {
                        case "homeautomation":
                            // The Homeautomation_TurnOn and Homeautomation_TurnOff dialogs 
                            // use results from a separate LUIS app.
                            // The results determine the "on/off" argument to pass to the dialog. 
                            var recognizerHomeAutomation = new LuisRecognizer(luisHomeAutomation);
                            RecognizerResult recognizerResult = await recognizerHomeAutomation.Recognize(utterance, System.Threading.CancellationToken.None);
                            var topHomeAutoIntent = recognizerResult.GetTopScoringIntent().intent;


                            dialogArgs.Add("IntentFromHomeAuto", "");
                            switch ((topHomeAutoIntent != null) ? topHomeAutoIntent.ToLowerInvariant() : null)
                            {
                                case "homeautomation_turnon":
                                    dialogArgs["Intent_HomeAutomation"] = "on";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case "homeautomation_turnoff":
                                    dialogArgs["Intent_HomeAutomation"] = "off";
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                                case null:
                                    await dc.Begin("homeautomation", null);
                                    break;
                                default:
                                    dialogArgs["Intent_HomeAutomation"] = topHomeAutoIntent;
                                    await dc.Begin("homeautomation", dialogArgs);
                                    break;
                            }
                            break;
                        case "weather":
                            dialogArgs.Add("LuisResult", luisResult);
                            await dc.Begin("weather", dialogArgs);
                            break;
                        case null:
                            await context.SendActivity($"Couldn't get a result from LUIS. You said: {utterance}");
                            break;
                        default:
                            // The intent didn't match any case, so just display the recognition results.
                            await context.SendActivity($"you said: {utterance}");
                            await context.SendActivity($"Recognized intent: {topIntent.Value.intent}.");

                            break;

                    }

                }
            }
        }

    }


```

在 `OnTurn` 后，在 `LuisDialogBot` 类中添加代码用于实现对话框。

```csharp
        // The home automation waterfall has one step
        private WaterfallStep[] CreateHomeAutomationWaterfall()
        {
            return new WaterfallStep[] {
                TurnLightsOnOrOff
            };
        }

        // The weather waterfall has two steps
        private WaterfallStep[] CreateWeatherWaterfall()
        {
            return new WaterfallStep[] {
                AskWeatherLocation,
                SendWeatherReport
            };
        }

        /// <summary>
        /// This is the first step and only of the home automation dialog.
        /// </summary>
        /// <param name="dc"></param>
        /// <param name="args">Can be "on", "off", another string for an intent name, or null.
        /// null indicates that no result was received from which to get an intent name.</param>
        /// <param name="next"></param>
        /// <returns></returns>
        private async Task TurnLightsOnOrOff(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var intentFromHomeAutomation = args["Intent_HomeAutomation"];
            if (args != null)
            {
                switch (intentFromHomeAutomation)
                {
                    case "on":
                    case "off":
                        await dc.Context.SendActivity($"Turning {intentFromHomeAutomation} your lights!");
                        break;
                    default:
                        await dc.Context.SendActivity($"Intent detected by homeautomation was: {intentFromHomeAutomation}, but the home automation system doesn't support that yet.");
                        break;
                }

            }
            else
            {
                await dc.Context.SendActivity($"You said {dc.Context.Activity.AsMessageActivity().Text}. Unable to get a result from which to determine on/off/other operation");
            }

            await dc.End();

        }

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }

        // This the second step of the weather dialog
        private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            if (args != null)
            {
                if (!dialogState.ContainsKey("Location"))
                {   // Interpret args as a reply to prompt only if they didn't give location
                    TextResult locationResult = (TextResult)args;
                    dialogState.Add("Location", locationResult.Text);
                }

            }

            // You can add some logic that uses the location 
            // to get the weather from a weather service instead of hard-coding it
            await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");

            dc.ActiveDialog.State = new Dictionary<string, object>(); // clear the dialog state 
            await dc.End();
        }
```

添加此帮助程序函数。

```cs
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

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

你可能需要安装对话框库（如果尚未这样做）。 

```cmd
npm install --save botbuilder-dialogs
```

首先，创建 LUIS 应用，并使用 `adapter.use` 将其添加到机器人：

```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage, TurnContext } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');
const { DialogSet } = require('botbuilder-dialogs');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Create LuisRecognizer 
// The LUIS application is public, meaning you can use your own subscription key to test the applications.
const luisRecognizer = new LuisRecognizer({
    appId: '1fefd4a7-ae5b-4e63-99f7-0cf499a1423b',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// Add the recognizer to your bot
adapter.use(luisRecognizer);

// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the intents detected by the LUIS app
const dialogs = new DialogSet();
```

接下来，添加对话框：

```javascript

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.homeAutomationTurnOn counts how many times this dialog was called 
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation_TurnOn" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.weatherGetForecast counts how many times this dialog was called  
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather_GetForecast" dialog.`);

        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        // state.None counts how many times this dialog was called        
        state.None = state.None ? state.None + 1 : 1;
        await dialogContext.context.sendActivity(`${state.None}: You reached the "None" dialog.`);

        await dialogContext.end();
    }
]);
```

在机器人中，基于已识别的意向调用每个对话框：
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our LUIS application
            const luisResults = luisRecognizer.get(context);

            // Extract the top intent from LUIS and use it to select which dialog to start
            // "NotFound" is the intent name for when no top intent can be found.
            const topIntent = LuisRecognizer.topIntent(luisResults, "NotFound");

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'homeautomation':                    
                        await dc.begin("HomeAutomation_TurnOn", luisResults);
                        break;
                    case 'weather':                    
                        await dc.begin("Weather_GetForecast", luisResults);
                        break;
                    case 'None':
                        await dc.begin("None");
                        break;
                    case 'NotFound':
                        await context.sendActivity(`Sorry, I didn't get any results from LUIS.`);
                        break;
                    default:
                        // handle intents for which we have no dialog
                        await context.sendActivity(`You reached the ${topIntent} intent.`);
                        break;
                }
            }
            
            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dialog bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});
```

尝试在 Bot Framework Emulator 中运行机器人，并对它说出类似于“打开灯”和“西雅图的天气”的内容。

![运行机器人](./media/how-to-luis/run-luis-bot-js-single-step-dialog.png)

---

## <a name="extract-entities"></a>提取实体

除了识别意向外，LUIS 应用还可以提取实体，这些实体是实现用户请求的重要词汇。 例如，在天气对话框的示例中，LUIS 应用可能能够从用户的消息中提取天气报告的位置。 

使用对话框的常用方法是识别用户消息中的任何实体，并提示找不到的任何所需的实体。 然后，后续对话框步骤处理对提示的响应。

# <a name="ctabcs"></a>[C#](#tab/cs)

假设用户发出的信息是“西雅图天气怎么样”？ 对于本示例中的机器人，[LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) 使用 [`Entities` 属性](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-)为你提供 [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult)，该属性的结构如下：

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

你添加到 `LuisDialogBot` 类的以下帮助程序函数从 LUIS 获取 `RecognizerResult` 中的实体。

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

从对话框中的多个步骤收集实体等信息时，在对话框特定的状态下保存所需信息会很有帮助。 例如，在 `AskWeatherLocation` 对话框中，从传递到对话框的 LUIS 结果中提取实体。 如果发现 `Weather_Location` 实体，则将其添加到对话框状态。

```cs

        // This is the first step of the weather dialog
        private async Task AskWeatherLocation(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
        {
            var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
            
            if (args["LuisResult"] is RecognizerResult luisResult)
            {
                var location = GetEntity<string>(luisResult, "Weather_Location");
                if (!string.IsNullOrEmpty(location))
                {
                    dialogState.Add("Location", location);
                }
            }

            // Save info back to the dialog instance
            dc.ActiveDialog.State = dialogState;

            if (!dialogState.ContainsKey("Location"))
            {
                await dc.Prompt("weather_city", "What city do you want the weather for?");
            }
            else
            {
                // We've set the location parameter for the weather report,
                // so go to the next step in the waterfall
                await dc.Continue();
            }
        }
```

在对话框的第二步中，可以从对话框实例的状态获取上一步中保存的信息，以及用户对位置提示的回复，并使用它向用户发送天气报告。

```cs

// This the second step of the weather dialog
private async Task SendWeatherReport(DialogContext dc, IDictionary<string, object> args, SkipStepFunction next)
{
    var dialogState = dc.ActiveDialog.State as IDictionary<string, object>;
    if (args != null)
    {
        if (!dialogState.ContainsKey("Location"))
        {   // Interpret args as a reply to prompt only if they didn't give location
            TextResult locationResult = (TextResult)args;
            dialogState.Add("Location", locationResult.Text);
        }

    }

    // You can add some logic that uses the location and date
    // to get the weather from a weather service instead of hard-coding it
    await dc.Context.SendActivity($"The weather forecast for '{dialogState["Location"]}' is sunny and 70 degrees F.");
    
    // clear the dialog state 
    dc.ActiveDialog.State = new Dictionary<string, object>(); 
    await dc.End();
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

例如，假设用户发出的信息是“西雅图天气怎么样”？ 对于本示例中的机器人，[LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) 使用 `entities` 属性为你提供 [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult)，该属性的结构如下：

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

此 `findEntities` 函数查找由 LUIS 应用识别的任何 `Weather_Location` 实体。

<!-- TODO: Turn into a waterfall -->

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

从 `Weather_GetForecast` 对话框调用 `findEntities`。

```javascript
// Pass the LUIS recognizer result to the args parameter
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
```

从传递到 `dc.begin` 的 LUIS 结果中提取实体。

```javascript
await dc.begin("Weather_GetForecast", luisResults);
```
---

## <a name="additional-resources"></a>其他资源

有关 LUIS 的更多背景信息，请参阅[语言理解](./bot-builder-concept-luis.md)。

有关如何生成这些示例中使用的 LUIS 应用的信息，请参阅[用于 Bot Builder 的 LUIS 应用](https://aka.ms/luis-bot-examples)。

LUIS 可与其他认知服务结合使用，使机器人的功能更加强大。 请参阅[使用调度工具合并 LUIS 应用和 QnA 服务](./bot-builder-tutorial-dispatch.md)，了解如何在机器人中将 QnA 与语言理解 (LUIS) 相结合。

## <a name="next-steps"></a>后续步骤


> [!div class="nextstepaction"]
> [生成强类型 LUIS 结果](./bot-builder-howto-v4-luisgen.md)


