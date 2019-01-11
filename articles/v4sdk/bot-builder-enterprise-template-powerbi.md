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
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152170"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>企业机器人模板 - 使用 PowerBI 仪表板和 Application Insights 进行聊天分析

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

部署机器人并开始处理消息后，你将看到遥测数据流入资源组中的 Application Insights 实例。 

可以在 Azure 门户中的“Application Insights”边栏选项卡中查看此遥测数据，并使用 Log Analytics。 此外，PowerBI 可以使用相同的遥测技术来提供有关机器人使用情况的更一般业务见解。

[聊天 AI 遥测](https://aka.ms/botPowerBiTemplate)上提供了示例 PowerBI 仪表板。 

这仅用于示例目的，并演示了如何开始创建自己的见解。 随着时间的推移，我们将增强这些可视化效果。 


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
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
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
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```