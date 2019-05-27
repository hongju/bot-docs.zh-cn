---
title: 将遥测功能添加到机器人 | Microsoft Docs
description: 了解如何将机器人与新的遥测功能相集成。
keywords: 遥测, appinsights, 监视机器人
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 95b56ec8e278c3d94430dc3c870803e8672fb053
ms.sourcegitcommit: 4086189a9c856fbdc832eb1a1d205e5f1b4e3acd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/16/2019
ms.locfileid: "65733326"
---
# <a name="add-telemetry-to-your-bot"></a>将遥测功能添加到机器人

[!INCLUDE[applies-to](../includes/applies-to.md)]

在 Bot Framework SDK 版本 4.2 中，遥测日志记录功能已添加到机器人产品。  这样，机器人应用程序便可将事件数据发送到 Application Insights 等服务。 第一部分将介绍这些方法，然后介绍更丰富的遥测功能。

本文档介绍如何将机器人与新的遥测功能相集成。 

## <a name="basic-telemetry-options"></a>基本遥测选项

### <a name="basic-application-insights"></a>基本 Application Insights

首先，让我们使用 Application Insights 向机器人添加基本的遥测。 有关设置的其他信息，请查看 [Application Insights 入门](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started-with-Application-Insights-for-ASP.NET-Core)的前面几个部分。   

如果你想要一个“库存式”的 Application Insights 并且不想要指定额外的特定于 Application Insights 的配置（例如遥测初始化表达式），请将以下内容添加到 `ConfigureServices()` 方法。   这是最简单的初始化方法，并且可将 Application Insights 配置为开始跟踪请求、对其他服务的外部调用，以及跨服务的关联事件。

需添加以下代码片段包含的 NuGet 包。

**Startup.cs**
```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Bot.Builder.ApplicationInsights;
using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
 
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry();

    // Add the standard telemetry client
    services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

    // Add ASP middleware to store the HTTP body, mapped with bot activity key, in the httpcontext.items
    // This will be picked by the TelemetryBotIdInitializer
    services.AddTransient<TelemetrySaveBodyASPMiddleware>();

    // Add telemetry initializer that will set the correlation context for all telemetry items
    services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

    // Add telemetry initializer that sets the user ID and session ID (in addition to other 
    // bot-specific properties, such as activity ID)
    services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
    ...
}

// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...
    app.UseBotApplicationInsights();
}
```

然后，需将 Application Insights 检测密钥存储在 `appsettings.json` 文件中，或者作为环境变量存储。 `appsettings.json` 文件包含有关机器人在运行时要使用的外部服务的元数据。  例如，CosmosDB、Application Insights 和语言理解 (LUIS) 服务连接与元数据存储在此文件中。 可以在 Azure 门户的“概览”部分（在该页上你的服务的 `Essentials` 下拉列表下，如果该页处于折叠状态）找到检测密钥。 可在[此处](~/bot-service-resources-app-insights-keys.md)找到如何获取密钥的详细信息。

此框架会为你找到密钥（如果格式设置正确）。 `appsettings.json` 条目的格式设置应如下所示：

```json
    "ApplicationInsights": {
        "InstrumentationKey": "putinstrumentationkeyhere"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    }
```

若要详细了解如何将 Application Insights 添加到 ASP.NET Core 应用程序，请参阅[此文](https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core-no-visualstudio)。 

### <a name="customize-your-telemetry-client"></a>自定义遥测客户端

若要自定义 Application Insights 客户端或将事件记录到完全独立的服务，必须以不同的方式配置系统。 通过 Nuget 下载 `Microsoft.Bot.Builder.ApplicationInsights` 包，或使用 npm 安装 `botbuilder-applicationinsights`。 可在[此处](~/bot-service-resources-app-insights-keys.md)找到如何获取 Application Insights 密钥的详细信息。

**修改 Application Insights 配置**

若要修改配置，请在添加 Application Insights 时包括 `options`。 否则，一切与上面的相同。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry(options);
    ...
}
```

`options` 对象为 `ApplicationInsightsServiceOptions` 类型。 [可以在此处找到]()有关这些选项的详细信息。

**使用自定义遥测**若要将 Bot Framework 生成的遥测事件记录到完全独立的系统，请创建派生自基接口 `IBotTelemetryClient` 的新类并进行配置。 然后，在如上所述添加遥测客户端时，请直接注入自定义客户端。 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### <a name="add-custom-logging-to-your-bot"></a>将自定义日志记录添加到机器人

为机器人配置新的遥测日志记录支持后，可以开始将遥测功能添加到机器人。  `BotTelemetryClient`（在 C# 中为 `IBotTelemetryClient`）提供多个方法用于记录不同类型的事件。  选择适当的事件类型可以利用 Application Insights 的现有报告（如果使用 Application Insights）。  对于常规方案，通常使用 `TraceEvent`。  使用 `TraceEvent` 记录的数据将进入 Kusto 中的 `CustomEvent` 表。

如果在机器人中使用对话，每个基于对话的对象（包括提示）将包含新的 `TelemetryClient` 属性。  这是用于执行日志记录的 `BotTelemetryClient`。  此属性不只是提供便利，本文稍后将会提到，如果设置此属性，`WaterfallDialogs` 将生成事件。

#### <a name="identifiers-and-custom-events"></a>标识符和自定义事件

将事件记录到 Application Insights 时，生成的事件包含无需填充的默认属性。  例如，`user_id` 和 `session_id` 属性包含在每个自定义事件（使用 `TraceEvent` API 生成）中。  此外，还会添加 `activitiId`、`activityType` 和 `channelId`。

>注意：不会为自定义遥测客户端提供这些值。

属性 |Type | 详细信息
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [机器人活动 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [机器人活动类型](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [机器人活动通道 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)

## <a name="in-depth-telemetry"></a>深度遥测功能

SDK 版本 4.4 中添加了三个新组件。  所有组件使用 `IBotTelemetryClient`（或 node.js 中的 `BotTelemetryClient`）接口写入日志，可以使用自定义的实现来重写该接口。

- 接收、发送、更新或删除消息时，Bot Framework 中间件组件 (*TelemetryLoggerMiddleware*) 会写入日志。 可以重写为自定义日志记录。
- *LuisRecognizer* 类。  可以重写为以两种方式进行自定义的日志记录 - 按调用（add/replace 属性）或派生类。
- *QnAMaker* 类。  可以重写为以两种方式进行自定义的日志记录 - 按调用（添加/替换属性）或派生类。

### <a name="telemetry-middleware"></a>遥测中间件

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

#### <a name="out-of-box-usage"></a>按原样使用

TelemetryLoggerMiddleware 是一个无需修改即可添加的 Bot Framework 组件，它可以执行日志记录，启用 Bot Framework SDK 随附的现成报告功能。 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

#### <a name="adding-properties"></a>添加属性
如果你决定添加其他属性，可以派生 TelemetryLoggerMiddleware 类。  例如，当你想要将“MyImportantProperty”属性添加到 `BotMessageReceived` 事件时。  当用户向机器人发送消息时，会记录 `BotMessageReceived`。  可通过以下方式添加其他属性：

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task OnReceiveActivityAsync(
                  Activity activity,
                  CancellationToken cancellation)
    {
        // Fill in the "standard" properties for BotMessageReceived
        // and add our own property.
        var properties = FillReceiveEventProperties(activity, 
                    new Dictionary<string, string>
                    { {"MyImportantProperty", "myImportantValue" } } );
                    
        // Use TelemetryClient to log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgReceiveEvent,
                        properties);
    }
    ...
}
```

在 Startup 中添加新类：

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

#### <a name="completely-replacing-properties--additional-events"></a>完全替换属性/其他事件

如果你决定完全替换所记录的属性，可以派生 `TelemetryLoggerMiddleware` 类（类似于上面所述的扩展属性）。   新事件的日志记录是按相同的方式执行的。

例如，下面演示了如何完全替换 `BotMessageSend` 属性并发送多个事件：

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task<RecognizerResult> OnLuisRecognizeAsync(
                  Activity activity,
                  string dialogId = null,
                  CancellationToken cancellation)
    {
        // Override properties for BotMsgSendEvent
        var botMsgSendProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        // Log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgSendEvent,
                        botMsgSendProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("activityId",
                                   activity.Id);
        secondEventProperties.Add("MyImportantProperty",
                                   "myImportantValue");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
    }
    ...
}
```
注意：如果未记录标准属性，则会导致产品随附的现成报告功能停止工作。

#### <a name="events-logged-from-telemetry-middleware"></a>从遥测中间件记录的事件
[BotMessageSend](#customevent-botmessagesend)
[BotMessageReceived](#customevent-botmessagereceived)
[BotMessageUpdate](#customevent-botmessageupdate)
[BotMessageDelete](#customevent-botmessagedelete)

### <a name="telemetry-support-luis"></a>遥测支持 LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

#### <a name="out-of-box-usage"></a>按原样使用
LuisRecognizer 是现有的 Bot Framework 组件，通过 `luisOptions` 传递 IBotTelemetryClient 接口可以启用遥测。  可根据需要重写所记录的默认属性并记录新事件。

在构造 `luisOptions` 期间，必须提供 `IBotTelemetryClient` 对象才能正常执行此操作。

```csharp
var luisOptions = new LuisPredictionOptions(
      ...
      telemetryClient,
      false); // Log personal information flag. Defaults to false.

var client = new LuisRecognizer(luisApp, luisOptions);
```

#### <a name="adding-properties"></a>添加属性
如果你决定添加其他属性，可以派生 `LuisRecognizer` 类。  例如，当你想要将“MyImportantProperty”属性添加到 `LuisResult` 事件时。  执行 LUIS 预测调用时，会记录 `LuisResult`。  可通过以下方式添加其他属性：

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   override protected Task OnRecognizerResultAsync(
           RecognizerResult recognizerResult,
           ITurnContext turnContext,
           Dictionary<string, string> properties = null,
           CancellationToken cancellationToken = default(CancellationToken))
   {
       var luisEventProperties = FillLuisEventProperties(result, 
               new Dictionary<string, string>
               { {"MyImportantProperty", "myImportantValue" } } );
        
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResultEvent,
                        luisEventProperties);
        ..
   }    
   ...
}
```

#### <a name="add-properties-per-invocation"></a>按调用添加属性
有时，在调用期间需要添加其他属性：
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties,
     CancellationToken.None).ConfigureAwait(false);
```

#### <a name="completely-replacing-properties--additional-events"></a>完全替换属性/其他事件
如果你决定完全替换所记录的属性，可以派生 `LuisRecognizer` 类（类似于上面所述的扩展属性）。   新事件的日志记录是按相同的方式执行的。

例如，下面演示了如何完全替换 `LuisResult` 属性并发送多个事件：

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    override protected Task OnRecognizerResultAsync(
             RecognizerResult recognizerResult,
             ITurnContext turnContext,
             Dictionary<string, string> properties = null,
             CancellationToken cancellationToken = default(CancellationToken))
    {
        // Override properties for LuisResult event
        var luisProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        
        // Log event
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResult,
                        luisProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("MyImportantProperty2",
                                   "myImportantValue2");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
        ...
    }
    ...
}
```
注意：如果未记录标准属性，则会导致产品随附的 Application Insights 现成报告功能停止工作。

#### <a name="events-logged-from-telemetryluisrecognizer"></a>从 TelemetryLuisRecognizer 记录的事件
[LuisResult](#customevent-luisevent)

### <a name="telemetry-qna-recognizer"></a>遥测 QnA 识别器

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


#### <a name="out-of-box-usage"></a>按原样使用
QnAMaker 类是现有的 Bot Framework 组件，它可以添加两个额外的构造函数参数来启用日志记录，从而启用 Bot Framework SDK 随附的现成报告功能。 新的 `telemetryClient` 引用执行日志记录的 `IBotTelemetryClient` 接口。  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
#### <a name="adding-properties"></a>添加属性 
如果你决定添加其他属性，可以使用两个方法来实现此目的 - 在 QnA 调用期间需要添加属性来检索回答，或者从 `QnAMaker` 类派生。  

下面演示了如何从 `QnAMaker` 类派生。  该示例演示了如何将“MyImportantProperty”属性添加到 `QnAMessage` 事件。  执行 QnA `GetAnswers` 调用时，会记录 `QnAMessage` 事件。  此外，我们记录了另一个事件“MySecondEvent”。

```csharp
class MyQnAMaker : QnAMaker 
{
   ...
   protected override Task OnQnaResultsAsync(
                 QueryResult[] queryResults, 
                 ITurnContext turnContext, 
                 Dictionary<string, string> telemetryProperties = null, 
                 Dictionary<string, double> telemetryMetrics = null, 
                 CancellationToken cancellationToken = default(CancellationToken))
   {
            var eventData = await FillQnAEventAsync(queryResults, turnContext, telemetryProperties, telemetryMetrics, cancellationToken).ConfigureAwait(false);

            // Add my property
            eventData.Properties.Add("MyImportantProperty", "myImportantValue");

            // Log QnaMessage event
            TelemetryClient.TrackEvent(
                            QnATelemetryConstants.QnaMsgEvent,
                            eventData.Properties,
                            eventData.Metrics
                            );

            // Create second event.
            var secondEventProperties = new Dictionary<string, string>();
            secondEventProperties.Add("MyImportantProperty2",
                                       "myImportantValue2");
            TelemetryClient.TrackEvent(
                            "MySecondEvent",
                            secondEventProperties);       }    
    ...
}
```

#### <a name="adding-properties-during-getanswersasync"></a>在 GetAnswersAsync 期间添加属性
如果在运行时期间需要添加属性，`GetAnswersAsync` 方法可以提供要添加到事件的属性和/或指标。

例如，若要将 `dialogId` 添加到事件，可按如下所示执行此操作：
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
`QnaMaker` 类提供重写属性（包括 PersonalInfomation 属性）的功能。

#### <a name="completely-replacing-properties--additional-events"></a>完全替换属性/其他事件
如果你决定完全替换所记录的属性，可以派生 `TelemetryQnAMaker` 类（类似于上面所述的扩展属性）。   新事件的日志记录是按相同的方式执行的。

例如，下面演示了如何完全替换 `QnAMessage` 属性：

```csharp
class MyLuisRecognizer : TelemetryQnAMaker
{
    ...
    protected override Task OnQnaResultsAsync(
         QueryResult[] queryResults, 
         ITurnContext turnContext, 
         Dictionary<string, string> telemetryProperties = null, 
         Dictionary<string, double> telemetryMetrics = null, 
         CancellationToken cancellationToken = default(CancellationToken))
    {
        // Add properties from GetAnswersAsync
        var properties = telemetryProperties ?? new Dictionary<string, string>();
        // GetAnswerAsync properties overrides - don't add if already present.
        properties.TryAdd("MyImportantProperty", "myImportantValue");

        // Log event
        TelemetryClient.TrackEvent(
                           QnATelemetryConstants.QnaMsgEvent,
                            properties);
    }
    ...
}
```
注意：如果未记录标准属性，则会导致产品随附的现成报告功能停止工作。

#### <a name="events-logged-from-telemetryluisrecognizer"></a>从 TelemetryLuisRecognizer 记录的事件
[QnAMessage](#customevent-qnamessage)


## <a name="waterfalldialog-events"></a>WaterfallDialog 事件

除了生成你自己的事件以外，SDK 中的 `WaterfallDialog` 对象现在还会生成事件。 以下部分将会介绍从 Bot Framework 内部生成的事件。 在 `WaterfallDialog` 中设置 `TelemetryClient` 属性后，将存储这些事件。

下面演示了如何修改一个示例 (CoreBot)，该示例采用 `WaterfallDialog` 记录遥测事件。  CoreBot 采用一种通用模式，其中的 `WaterfallDialog` 放在 `ComponentDialog` (`GreetingDialog`) 中。

```csharp
// IBotTelemetryClient is direct injected into our Bot
public CoreBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

为 `WaterfallDialog` 配置 `IBotTelemetryClient` 后，它将开始记录事件。

## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework 服务生成的事件

除 `WaterfallDialog`（从机器人代码生成事件）以外，Bot Framework 通道服务还会记录事件。  这可以帮助你诊断通道问题或整个机器人的故障。

### <a name="customevent-activity"></a>CustomEvent:"Activity"
**记录自：** 收到消息时由通道服务记录的通道服务。

### <a name="exception-bot-errors"></a>异常："Bot Errors"
**记录自：** 调用机器人时由通道记录的通道服务返回非 2XX Http 响应。

## <a name="all-events-generated"></a>已生成所有事件

### <a name="customevent-waterfallstart"></a>CustomEvent:"WaterfallStart" 

当 WaterfallDialog 启动时，将记录 `WaterfallStart` 事件。

- `user_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `session_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityType`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.channelId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.DialogId`（这是传入瀑布中的 dialogId（字符串）。  可将其视为“瀑布类型”）
- `customDimensions.InstanceID`（对于每个对话实例是唯一的）

### <a name="customevent-waterfallstep"></a>CustomEvent:"WaterfallStep" 

记录瀑布对话中的各个步骤。

- `user_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `session_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityType`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.channelId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.DialogId`（这是传入瀑布中的 dialogId（字符串）。  可将其视为“瀑布类型”）
- `customDimensions.StepName`（方法名称，使用 lambda 时为 `StepXofY`）
- `customDimensions.InstanceID`（对于每个对话实例是唯一的）

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent:"WaterfallDialogComplete"

瀑布对话完成时记录。

- `user_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `session_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityType`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.channelId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.DialogId`（这是传入瀑布中的 dialogId（字符串）。  可将其视为“瀑布类型”）
- `customDimensions.InstanceID`（对于每个对话实例是唯一的）

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent:"WaterfallDialogCancel" 

瀑布对话取消时记录。

- `user_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `session_id`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.activityType`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.channelId`（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- `customDimensions.DialogId`（这是传入瀑布中的 dialogId（字符串）。  可将其视为“瀑布类型”）
- `customDimensions.StepName`（方法名称，使用 lambda 时为 `StepXofY`）
- `customDimensions.InstanceID`（对于每个对话实例是唯一的）

### <a name="customevent-botmessagereceived"></a>CustomEvent:BotMessageReceived 
当机器人收到用户的新消息时记录。

如果不重写，将使用 `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()` 方法从 `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` 记录此事件。

- 会话标识符  
  - 使用 Application Insights 时，将从 `TelemetryBotIdInitializer` 记录此属性，作为在 Application Insights 中使用的**会话**标识符 (*Temeletry.Context.Session.Id*)。  
  - 对应于 Bot Framework 协议定义的[聊天 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)。
  - 记录的属性名称为 `session_id`。

- 用户标识符
  - 使用 Application Insights 时，将从 `TelemetryBotIdInitializer` 记录此属性，作为在 Application Insights 中使用的**用户**标识符 (*Telemetry.Context.User.Id*)。  
  - 此属性的值是 Bot Framework 协议定义的[通道标识符](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)与[用户 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)（连接在一起）属性的组合。
  - 记录的属性名称为 `user_id`。

- ActivityID 
  - 使用 Application Insights 时，将从 `TelemetryBotIdInitializer` 记录此属性作为事件的属性。
  - 对应于 Bot Framework 协议定义的[活动 ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id)。
  - 属性名称为 `activityId`。

- 通道标识符
  - 使用 Application Insights 时，将从 `TelemetryBotIdInitializer` 记录此属性作为事件的属性。  
  - 对应于 Bot Framework 协议的[通道标识符](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)。
  - 记录的属性名称为 `channelId`。

- 活动类型 
  - 使用 Application Insights 时，将从 `TelemetryBotIdInitializer` 记录此属性作为事件的属性。  
  - 对应于 Bot Framework 协议的[活动类型](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type)。
  - 记录的属性名称为 `activityType`。

- 文本
  - **（可选）** 将 `logPersonalInformation` 属性设置为 `true` 时记录。
  - 对应于 Bot Framework 协议的[活动文本](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text)字段。
  - 记录的属性名称为 `text`。

- Speak

  - **（可选）** 将 `logPersonalInformation` 属性设置为 `true` 时记录。
  - 对应于 Bot Framework 协议的[活动讲述](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak)字段。
  - 记录的属性名称为 `speak`。

  - 

- FromId
  - 对应于 Bot Framework 协议的[源标识符](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromId`。

- FromName
  - **（可选）** 将 `logPersonalInformation` 属性设置为 `true` 时记录。
  - 对应于 Bot Framework 协议的[源名称](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromName`。

- RecipientId
  - 对应于 Bot Framework 协议的[源名称](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromName`。

- RecipientName
  - 对应于 Bot Framework 协议的[源名称](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromName`。

- ConversationId
  - 对应于 Bot Framework 协议的[源名称](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromName`。

- ConversationName
  - 对应于 Bot Framework 协议的[源名称](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromName`。

- 区域设置
  - 对应于 Bot Framework 协议的[源名称](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)字段。
  - 记录的属性名称为 `fromName`。

### <a name="customevent-botmessagesend"></a>CustomEvent:BotMessageSend 
**记录自：** TelemetryLoggerMiddleware 

当机器人发送消息时记录。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- SessionID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ReplyToID
- RecipientId
- ConversationName
- 区域设置
- RecipientName（对于 PII 是可选的）
- Text（对于 PII 是可选的）
- Speak（对于 PII 是可选的）


### <a name="customevent-botmessageupdate"></a>CustomEvent:BotMessageUpdate
**记录自：** 机器人更新消息（这种情况很少见）时记录的 TelemetryLoggerMiddleware
- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- SessionID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- RecipientId
- ConversationId
- ConversationName
- 区域设置
- Text（对于 PII 是可选的）


### <a name="customevent-botmessagedelete"></a>CustomEvent:BotMessageDelete
**记录自：** 机器人删除消息（这种情况很少见）时记录的 TelemetryLoggerMiddleware
- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- SessionID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- RecipientId
- ConversationId
- ConversationName

### <a name="customevent-luisevent"></a>CustomEvent:LuisEvent
**记录自：** LuisRecognizer

记录来自 LUIS 服务的结果。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- SessionID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ApplicationId
- 意向
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Entities（JSON 格式）
- Question（对于 PII 是可选的）

## <a name="customevent-qnamessage"></a>CustomEvent:QnAMessage
**记录自：** QnAMaker

记录来自 QnA Maker 服务的结果。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- SessionID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Username（对于 PII 是可选的）
- Question（对于 PII 是可选的）
- MatchedQuestion
- QuestionId
- Answer
- Score
- ArticleFound

## <a name="querying-the-data"></a>查询数据
使用 Application Insights 时，所有数据（甚至包括跨服务的数据）将关联到一起。  我们可以通过查询来查看成功的请求，并查看该请求的所有关联事件。  
以下查询告知最近的请求：
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

在第一个查询中选择一些 `operation_Id`，然后查看更多信息：

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

这会提供单个请求在不同时间的细分，以及每个调用的持续时间桶。
![示例调用](media/performance_query.png)

> 注意：“Activity”`customEvent` 事件时间戳是无序的，因为这些事件是以异步方式记录的。

## <a name="create-a-dashboard"></a>创建仪表板

最简单的测试方法是使用 [Azure 门户的模板部署页](https://portal.azure.com/#create/Microsoft.Template)创建仪表板。  
- 单击“在编辑器中生成自己的模板”
- 复制并粘贴这些帮助你创建仪表板的 .json 文件之一：
  - [系统运行状况仪表板](https://aka.ms/system-health-appinsights)
  - [聊天运行状况仪表板](https://aka.ms/conversation-health-appinsights)
- 单击“保存”
- 填充 `Basics`： 
   - 订阅：<your test subscription>
   - 资源组：<a test resource group>
   - 位置：<such as West US>
- 填充 `Settings`：
   - Insights 组件名称：<例如 `core672so2hw`>
   - Insights 组件资源组：<例如 `core67`>
   - 仪表板名称：<例如 `'ConversationHealth'` 或 `SystemHealth`>
- 单击 `I agree to the terms and conditions stated above`
- 单击 `Purchase`
- 验证
   - 单击 [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - 从上面选择资源组（例如 `core67`）。
   - 如果看不到新资源，请查看“部署”以确定是否发生了任何失败。
   - 下面是通常会看到的失败：
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

若要查看数据，请转到 Azure 门户。 单击左侧的“仪表板”，然后从下拉列表中选择创建的仪表板。

## <a name="additional-resources"></a>其他资源
可以参考下面这些实现遥测功能的示例：
- C#
  - [LUIS 和 AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA 和 AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS 和 AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA 和 AppInsights](https://aka.ms/qna-with-appinsights-js)

