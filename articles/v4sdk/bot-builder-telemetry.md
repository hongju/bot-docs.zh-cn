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
ms.openlocfilehash: 4c268bc40b7dc3315232d8f695bdb79343b15e21
ms.sourcegitcommit: c7d2e939ec71f46f48383c750fddaf6627b6489d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/07/2019
ms.locfileid: "55795578"
---
# <a name="add-telemetry-to-your-bot"></a>将遥测功能添加到机器人
在 Bot Framework SDK 版本 4.2 中，遥测日志记录功能已添加到机器人产品。  这样，机器人应用程序便可将事件数据发送到 Application Insights 等服务。

本文档介绍如何将机器人与新的遥测功能相集成。  

## <a name="using-bot-configuration-option-1-of-2"></a>使用机器人配置（选项 1/2）
可通过两种方法配置机器人。  第一种方法假设你要与 Application Insights 相集成。

机器人配置文件包含有关机器人在运行时要使用的外部服务的元数据。  例如，CosmosDB、Application Insights 和语言理解 (LUIS) 服务连接与元数据存储在此文件中。   

如果你想要一个“库存式”的 Application Insights 并且不想要指定额外的特定于 Application Insights 的配置（例如遥测初始化表达式），请在初始化期间传入机器人配置对象。   这是最简单的初始化方法，并且可将 Application Insights 配置为开始跟踪请求、对其他服务的外部调用，以及跨服务的关联事件。

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**机器人配置中不包含 Application Insights**如果机器人配置不包含 Application Insights，该怎么办？  没问题，它将默认为 null 客户端，方法不会在其上调用任何操作。

**多个 Application Insights**机器人配置中包含多个 Application Insights 节？  可以指定要在机器人配置中使用 Application Insights 服务的哪个实例。

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>重写遥测客户端（选项 2/2）

若要自定义 Application Insights 客户端或将事件记录到完全独立的服务，必须以不同的方式配置系统。

**修改 Application Insights 配置**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**使用自定义遥测**若要将 Bot Framework 生成的遥测事件记录到完全独立的系统，请创建派生自基接口的新类并进行配置。  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>将自定义日志记录添加到机器人

为机器人配置新的遥测日志记录支持后，可以开始将遥测功能添加到机器人。  `BotTelemetryClient`（在 C# 中为 `IBotTelemetryClient`）提供多个方法用于记录不同类型的事件。  选择适当的事件类型可以利用 Application Insights 的现有报告（如果使用 Application Insights）。  对于常规方案，通常使用 `TraceEvent`。  使用 `TraceEvent` 记录的数据将进入 Kusto 中的 `CustomEvent` 表。

如果在机器人中使用对话，每个基于对话的对象（包括提示）将包含新的 `TelemetryClient` 属性。  这是用于执行日志记录的 `BotTelemetryClient`。  此属性不只是提供便利，本文稍后将会提到，如果设置此属性，`WaterfallDialogs` 将生成事件。

### <a name="identifiers-and-custom-events"></a>标识符和自定义事件

将事件记录到 Application Insights 时，生成的事件包含无需填充的默认属性。  例如，`user_id` 和 `session_id` 属性包含在每个自定义事件（使用 `TraceEvent` API 生成）中。  此外，还会添加 `activitiId`、`activityType` 和 `channelId`。

>注意：不会为自定义遥测客户端提供这些值。

属性 |Type | 详细信息
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [机器人活动 ID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [机器人活动类型](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [机器人活动通道 ID](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>WaterfallDialog 事件

除了生成你自己的事件以外，SDK 中的 `WaterfallDialog` 对象现在还会生成事件。 以下部分将会介绍从 Bot Framework 内部生成的事件。 在 `WaterfallDialog` 中设置 `TelemetryClient` 属性后，将存储这些事件。

下面演示了如何修改一个示例 (BasicBot)，该示例采用 `WaterfallDialog` 记录遥测事件。  BasicBot 采用一种通用模式，其中的 `WaterfallDialog` 放在 `ComponentDialog` (`GreetingDialog`) 中。

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
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


## <a name="events-generated-by-the-bot-framework-service"></a>Bot Framework 服务生成的事件

除 `WaterfallDialog`（从机器人代码生成事件）以外，Bot Framework 通道服务还会记录事件。  这可以帮助你诊断通道问题或整个机器人的故障。

### <a name="customevent-activity"></a>CustomEvent:"Activity"
**记录自：** 收到消息时由通道服务记录的通道服务。

### <a name="exception-bot-errors"></a>异常："Bot Errors"
**记录自：** 调用机器人时由通道记录的通道服务返回非 2XX Http 响应。

## <a name="additional-events"></a>其他事件

[企业模板](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)是可以任意复制的开源代码。  可以根据报告需求重复使用和修改此模板中的多个组件。

### <a name="customevent-botmessagereceived"></a>CustomEvent:BotMessageReceived 
**记录自：** TelemetryLoggerMiddleware（**企业示例**）

当机器人收到新消息时记录。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ConversationID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Text（对于 PII 是可选的）
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- 区域设置

### <a name="customevent-botmessagesend"></a>CustomEvent:BotMessageSend 
**记录自：** TelemetryLoggerMiddleware（**企业示例**）

当机器人发送消息时记录。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ConversationID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ReplyToID
- Channel（源通道 - 例如 Skype、Cortana、Teams）
- RecipientId
- ConversationName
- 区域设置
- Text（对于 PII 是可选的）
- RecipientName（对于 PII 是可选的）

### <a name="customevent-botmessageupdate"></a>CustomEvent:BotMessageUpdate
**记录自：** 机器人更新消息（这种情况很少见）时记录的 TelemetryLoggerMiddleware

### <a name="customevent-botmessagedelete"></a>CustomEvent:BotMessageDelete
**记录自：** 机器人删除消息（这种情况很少见）时记录的 TelemetryLoggerMiddleware

### <a name="customevent-luisintentinentname"></a>CustomEvent:LuisIntent.INENTName 
**记录自：** TelemetryLuisRecognizer（**企业示例**）

记录来自 LUIS 服务的结果。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ConversationID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- 意向
- IntentScore
- 问题
- ConversationId
- SentimentLabel
- SentimentScore
- *LUIS 实体*
- **NEW** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent:QnAMessage
**记录自：** TelemetryQnaMaker（**企业示例**）

记录来自 QnA Maker 服务的结果。

- UserID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ConversationID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityID（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- Channel（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- ActivityType（[从遥测初始化表达式](https://aka.ms/telemetry-initializer)）
- 用户名
- ConversationId
- OriginalQuestion
- 问题
- Answer
- Score（*可选*：如果找到了知识）

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

