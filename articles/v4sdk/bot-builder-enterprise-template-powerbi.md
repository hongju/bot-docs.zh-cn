---
title: 使用 Power BI 进行聊天分析 | Microsoft Docs
description: 了解企业机器人模板如何利用 Application Insights 通过 PowerBI 启用见解
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 73e19047ea64839f52bb20ea1eceee93803210bc
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645478"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>企业机器人模板 - 使用 PowerBI 仪表板和 Application Insights 进行聊天分析

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

部署机器人并开始处理消息后，你将看到遥测数据流入资源组中的 Application Insights 实例。 

可以在 Azure 门户中的“Application Insights”边栏选项卡中查看此遥测数据，并使用 Log Analytics。 此外，PowerBI 可以使用相同的遥测技术来提供有关机器人使用情况的更一般业务见解。

在创建的项目的 PowerBI 文件夹中提供了一个示例 Power BI 仪表板。 这仅用于示例目的，并演示了如何开始创建自己的见解。 随着时间的推移，我们将增强这些可视化效果。 

## <a name="getting-started"></a>入门

- 从[此处](https://powerbi.microsoft.com/en-us/desktop/)下载 PowerBI Desktop
 
- 检索机器人使用的 Application Insights 资源的 ```Application Id```。 可以通过导航到 Application Insights Azure 边栏选项卡的“配置”部分下的“API 访问”页来获取此信息。

双击解决方案的 PowerBI 文件夹中提供的 PowerBI 模板文件。 系统将提示你输入上一步中检索到的 ```Application Id```。 如果系统提示你使用 Azure 订阅凭据完成身份验证，则可能需要单击“组织帐户”设置才能登录。

显示的仪表板现在将链接到 Application Insights 实例，如果已发送和接收消息，应该在仪表板中看到初始见解。

>请注意，由于当前部署脚本在发布 LUIS 模型时未启用情绪，因此情绪可视化不会显示数据。 如果[重新发布](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app) LUIS 模型并启用情绪，这将起作用。

## <a name="middleware-processing"></a>中间件处理

提供围绕 QnAMaker 和 LuisRecognizer 类的遥测包装器，以确保无论情况如何都能实现一致的遥测输出，并使标准仪表板能够在每个项目中正常工作。

```TelemetryLuisRecognizer``` 和 ```TelemetryQnAMaker``` 都提供构造函数的属性，使开发人员能够禁止记录用户名和原始消息。 但是，这会减少可用的见解数量。

## <a name="telemetry-captured"></a>捕获的遥测数据

通过使用企业模板中默认启用的 ```TelemetryLuisRecognizer``` 和 ```TelemetryQnAMaker``` 捕获 4 个不同的遥测事件。 

项目使用的每个 LUIS 意向都将以“LuisIntent.”为前缀 以实现通过仪表板轻松识别意向。

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```