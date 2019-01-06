---
title: 排查机器人的 HTTP 500 错误 | Microsoft Docs
description: 如何在已部署机器人中排查 HTTP 500 错误。
keywords: 排查, HTTP 500, 问题。
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/20/2018
ms.openlocfilehash: 8ab1cd34f2cc239602db423bccd131d9df39222a
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735995"
---
# <a name="troubleshoot-http-500-errors"></a>排查 HTTP 500 错误

排查 500 错误的第一步是启用 Application Insights。

luis-with-appinsights ([C#](https://aka.ms/cs-luis-with-appinsights-sample) / [JS](https://aka.ms/js-luis-with-appinsights-sample)) 和 qna-with-appinsights ([C#](https://aka.ms/qna-with-appinsights) / [JS](https://aka.ms/js-qna-with-appinsights-sample)) 示例演示支持 Azure Application Insights 的机器人。 请参阅 [Conversation Analytics Tememetry](https://aka.ms/botPowerBiTemplate)（聊天分析遥测），了解如何向现有的机器人添加 Application Insights。

## <a name="enable-application-insights-on-aspnet"></a>在 ASP.Net 上启用 Application Insights

如需基本的 Application Insights 支持，请参阅如何[为 ASP.NET 网站设置 Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net)。 Bot Framework（从 v4.2 开始）提供另一级别的 Application Insights 遥测，但它不是诊断 HTTP 500 错误所必需的。

## <a name="enable-application-insights-on-nodejs"></a>在 Node.js 上启用 Application Insights

如需基本的 Application Insights 支持，请参阅如何[使用 Application Insights 监视 Node.js 服务和应用](https://docs.microsoft.com/azure/application-insights/app-insights-nodejs)。 Bot Framework（从 v4.2 开始）提供另一级别的 Application Insights 遥测，但它不是诊断 HTTP 500 错误所必需的。

## <a name="query-for-exceptions"></a>查询异常

若要分析 HTTP 状态代码 500 错误，最简单的方法是从异常开始。

以下查询会告知你最新的异常：

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

从第一个查询中选择一些操作 ID，查找更多信息：

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

如果只有 `exceptions`，请分析详细信息，看其是否对应于代码中的行。 如果只看到来自通道连接器 (`Microsoft.Bot.ChannelConnector`) 的异常，则请参阅[无 Application Insights 事件](#no-application-insights-events)，确保 Application Insights 已正确设置且代码在记录事件。

## <a name="no-application-insights-events"></a>无 Application Insights 事件

如果收到 500 错误且 Application Insights 中没有来自机器人的更多事件，则请检查以下事项：

### <a name="ensure-bot-runs-locally"></a>确保机器人在本地运行

首先使用模拟器确保机器人在本地运行。

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>确保正在复制配置文件（仅限 .NET）

确保在部署过程中将 `.bot` 配置文件和 `appsettings.json` 文件正确打包。

#### <a name="application-assemblies"></a>应用程序集

确保在部署过程中将 Application Insights 程序集正确打包。

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

确保在部署过程中将 `.bot` 配置文件和 `appsettings.json` 文件正确打包。

#### <a name="appsettingsjson"></a>appsettings.json

在 `appsettings.json` 文件中，确保检测密钥已设置。

## <a name="aspnet-web-apitabdotnetwebapi"></a>[ASP.NET Web API](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "botFilePath": "mybot.bot",
    "botFileSecret": "<my secret>",
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-bot-config-file"></a>验证 .bot 配置文件

确保 .bot 文件中包含一个 Application Insights 密钥。

```json
    {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    },
```

### <a name="check-logs"></a>检查日志

Bot ASP.Net 和 Node 会在服务器级别发出可以检查的日志。

#### <a name="set-up-a-browser-to-watch-your-logs"></a>设置一个用于查看日志的浏览器

1. 在 [Azure 门户](http://portal.azure.com/)中打开机器人。
1. 打开“应用服务设置/所有应用服务设置”页，查看所有服务设置。
1. 打开应用服务的“监视/诊断日志”页。
   - 将“应用程序日志记录(文件系统)”已启用。 如果更改此设置，请确保单击“保存”。
1. 切换到“监视/日志流”页。
   - 选择“Web 服务器日志”，确保看到一条指示已连接的消息。 它应该如下所示：

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     让此窗口保持打开状态。

#### <a name="set-up-browser-to-restart-your-bot-service"></a>设置浏览器，以便重启机器人服务

1. 使用单独的浏览器，在 Azure 门户中打开机器人。
1. 打开“应用服务设置/所有应用服务设置”页，查看所有服务设置。
1. 切换到应用服务的“概览”页，单击“重启”。
   - 系统会提示你是否确定执行该操作，此时请选择“是”。
1. 返回到第一个浏览器窗口，查看日志。
1. 验证是否收到新日志。
   - 如果没有任何活动，请重新部署机器人。
   - 然后切换到“应用程序日志”页，查看是否存在任何错误。
