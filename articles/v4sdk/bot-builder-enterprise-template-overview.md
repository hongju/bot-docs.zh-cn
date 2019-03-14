---
title: 企业机器人模板详细概述 | Microsoft Docs
description: 了解企业机器人模板背后的设计决策
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 582ec21d36e15fcbaef2d26616a9e55a04d8e5f2
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/07/2019
ms.locfileid: "57568214"
---
# <a name="enterprise-bot-template---overview"></a>Enterprise Bot Template - 概述

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

Enterprise Bot Template 为创建聊天体验所需的最佳做法和服务提供坚实的基础，在减少工作量的同时提高了质量标准。 该模板利用 [Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) 和 [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools)提供以下功能：

Feature      | 说明 |
------------ | -------------
介绍 | 在聊天开始时使用[自适应卡]()的介绍消息
键入指示器  | 在聊天期间自动显示的视觉键入指示器，长时间运行的操作会重复显示它
基 LUIS 模型  | 支持常用意向，例如“取消”、“帮助”、“升级”等。
基对话 | 对话流，用于捕获基本用户信息以及“取消”和“帮助”意向的中断逻辑
基响应  | 基意向和对话的文本和语音响应
常见问题解答 | 与 [QnA Maker](https://www.qnamaker.ai) 集成可回答知识库中的一般问题 
聊天内容 | 一个专业聊天模型，为常见查询提供标准答案（[了解详情](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)）
调度程序 | 一个集成的 [Dispatch](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) 模型，用于确定给定话语应该由 LUIS 还是由 QnA Maker 处理。
语言支持 | 提供英语、法语、意大利语、德语、西班牙语和中文支持
脚本 | Azure 存储中存储的所有聊天的脚本
遥测  | [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) 集成，用于收集所有聊天的遥测数据
分析 | 一个示例 PowerBI 仪表板，可以从这里开始了解聊天体验。
自动化部署 | 轻松地使用 [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools)部署上述所有服务

# <a name="background"></a>背景

## <a name="introduction-card"></a>介绍卡
介绍卡概述机器人的功能并提供方便你入门的示例话语，因此可以提高聊天质量。 它还将机器人的个性呈现给用户。

## <a name="base-luis-model-with-common-intents"></a>包含常见意向的基 LUIS 模型
包括在模板中的基 LUIS 模型涵盖了一系列最常见意向，这些意向是大多数机器人需要处理的。 以下意向可以在机器人中直接使用：

意向       | 示例话语 |
-------------|-------------|
取消       |取消、没关系|
升级     |我可以与某人通话吗？|
完成任务   |完成、全部完成|
返回       |返回|
帮助         |你可以帮我吗？|
重复       |你能再说一次吗？|
SelectAny    |这其中的任何项|
选择项   |第一项|
SelectNone   |这些项都不是|
显示下一个     |显示更多项|
显示上一个 |显示上一项|
重新开始    |*restart*|
停止         |*stop*|

## <a name="qna-maker"></a>QnA Maker

使用 [QnA Maker](https://www.qnamaker.ai/)，非开发人员可以生成一系列在机器人中使用的问答对。 该知识可以在 QnA Maker 门户中从常见问题解答数据源和产品手册导入，也可以手动创建。

模板中提供两个示例 QnA Maker 模型，一个用于常见问题解答，一个用于[专业聊天内容](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/chit-chat-knowledge-base)。 

## <a name="dispatch"></a>Dispatch

Dispatch 服务用于管理多个 LUIS 模型和 QnA Maker 知识库之间的路由，其方法是提取每项服务的话语，然后创建一个集中调度的 LUIS 模型。

这样机器人就能快速确定哪个组件应该处理某个给定的话语，确保在进行意向处理时将 QnA Maker 数据置于层次结构的顶部而不是底部进行处理。

此 Dispatch 工具还可以用来对模型进行评估，突出显示多个服务中的重叠话语和不一致。

## <a name="telemetry"></a>遥测

了解机器人聊天内容就能了解用户参与程度、用户所使用的具体功能，以及机器人无法处理的问题。

[Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) 捕获机器人服务的操作遥测数据以及特定于聊天的事件。 可以使用 [Power BI](https://powerbi.microsoft.com/en-us/what-is-power-bi/) 之类的工具将这些事件聚合成可操作的信息。 我们提供了一个示例 Power BI 仪表板，该仪表板可以与演示此功能的 Enterprise Bot Template 配合使用。

# <a name="next-steps"></a>后续步骤
请参阅[入门](bot-builder-enterprise-template-getting-started.md)，了解如何创建并部署 Enterprise Bot。 

# <a name="resources"></a>资源
Enterprise Bot Template 的完整源代码可在 [GitHub](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template) 上找到。
