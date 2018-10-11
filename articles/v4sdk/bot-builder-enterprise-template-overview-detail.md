---
title: 企业机器人模板详细概述 | Microsoft Docs
description: 了解企业机器人模板背后的设计决策
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6f295794ca7d3cc17688337e70df2a52cdb665ed
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46708562"
---
# <a name="enterprise-template---detailed-overview"></a>企业模板 - 详细概述

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

企业机器人模板汇集了我们通过构建聊天体验确定的一些最佳做法，并自动集成了我们发现的对 Azure 机器人服务开发人员非常有益的组件。 本部分介绍关键决策的一些背景知识，帮助用户了解模板的工作方式。

## <a name="introduction-card"></a>介绍卡

许多聊天体验存在的一个关键问题是最终用户不知道如何开始，导致出现机器人可能不太适合回答的一般问题。 第一印象非常重要！ 可以通过介绍卡向最终用户介绍机器人的功能，并建议一些可供用户在初次提问时使用的初始问题。 这也是一个展示机器人个性的好机会。

我们提供了一张简单的介绍卡作为标准，你可以根据需要对其进行修改。

## <a name="basic-language-understanding-luis-intents"></a>基本语言理解 (LUIS) 意向

每个机器人都应该能够基本上理解聊天语言。 例如，每个机器人都应该能够轻松地进行问候。 通常，开发人员需要创建这些基本意向并提供开始训练所需的初始训练数据。 企业机器人模板提供入门所需的示例 LU 文件，避免每个项目每次都必须创建这些文件，确保提供现成的基本功能。

LU 文件通过英语、法语、意大利语、德语和西班牙语提供以下意向。

> 问候、帮助、取消、重启、升级、确认是、确认否、确认更多、下一步、告别

[LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 格式类似于 MarkDown，可以轻松地进行修改以及进行源代码管理。 可以使用 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 工具将 .LU 文件转换为 LUIS 模型，然后通过门户或关联的 [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI（命令行）工具将其发布到 LUIS 订阅。

## <a name="content-moderator"></a>内容审查器

内容审查器用于检测可能存在的亵渎内容并检查是否存在个人身份信息 (PII)。 可以将其集成到机器人中，使机器人能够对亵渎内容作出反应，或者在用户共享 PII 信息时作出反应。 例如，如果检测到 PII 信息，机器人可以向人道歉并移交该信息，或者不存储遥测记录。

将会提供一个中间件组件对文本进行筛选，并通过 TurnState 对象上的 ```TextModeratorResult``` 来显示。

## <a name="telemetry"></a>遥测

事实证明，提供对机器人用户参与度的见解非常有价值。 此见解有助于了解用户参与程度、他们使用机器人的哪些功能（意向），以及人们提问的哪些问题机器人无法回答 - 突出显示机器人的知识缺口，该缺口也许可以通过某些方式（例如新的 QnAMaker 文章）弥补。

集成 Application Insights 这一功能可以用来提供现成的重大操作/技术见解，但也可用来捕获特定的机器人相关事件 - 发送和接收的消息，以及 LUIS 和 QnAMaker 操作。

机器人级别遥测本质上关联到技术和操作遥测，让你可以检查给定用户问题的解答方式，反之亦然。

将中间件组件和围绕 QnAMaker 的包装类以及 LuisRecognizer SDK 类组合使用即可轻松地收集一组一致的事件。 这些一致的事件随后可供 Applicatin Insights 工具以及 PowerBI 之类的工具使用。

每个使用企业机器人模板创建的项目都提供了一个示例 PowerBI 仪表板。 有关详细信息，请参阅 [PowerBI](bot-builder-enterprise-template-powerbi.md) 部分。

## <a name="dispatcher"></a>调度程序

在第一波聊天体验中，具有良好效果的关键设计模式是利用语言理解 (LUIS) 和 QnAMaker。 将使用机器人为最终用户执行的任务来训练 LUIS，而使用更常规的知识来训练 QnAMaker。

所有传入话语（问题）都会路由到 LUIS 进行分析。 如果给定话语的意向未确定，则会将该意向标记为“None”意向。 然后会使用 QnAMaker 来尝试为最终用户找出一个解答。

虽然此模式很有用，但在两种关键情况下，可能会遇到问题。

- 有时候，如果 LUIS 模型中的话语和 QnAMaker 中的略有重叠，则可能导致奇怪行为的发生：LUIS 可能会在本应将某个问题转给 QnaMaker 处理的时候尝试处理该问题。
- 当存在两个或多个 LUIS 模型时，机器人就必须调用每个模型并执行某种形式的意向评估比较，以便确定将给定话语发送至何处。 由于没有通用的基线，因此跨模型进行分数比较不是那么有效，导致用户体验很糟。

[Dispatcher](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig) 是用于解决此问题的很好的解决方案，它可以从每个配置的 LUIS 模型提取话语，从 QnAMaker 提取问题，然后创建一个集中调度的 LUIS 模型。

这样机器人就能快速确定哪个 LUIS 模型或组件应该处理某个给定的话语，确保将 QnAMaker 数据视为顶级意向处理，而不是像此前那样仅仅作为 None 意向处理。

此调度工具还可以进行评估，突出显示各 LUIS 模型之间的混淆和重叠情况，以及突出显示 QnAMaker 知识库的问题，然后再进行部署。

Dispatcher 在每个使用企业机器人模板创建的项目的核心使用。 调度模型在 `MainDialog` 类中使用，可确定目标是 LUIS 模型还是 QnA。 如果目标是 LUIS 模型，则会调用辅助 LUIS 模型，正常返回意向和实体。

## <a name="qnamaker"></a>QnAMaker

使用 [QnAMaker](https://www.qnamaker.ai/)，非开发人员可以通过问答对的形式来策展常规知识。 该知识可以在 QnaMaker 门户中以交互方式从常见问题解答数据源以及产品手册导入。

QnA 条目的示例集以 [LU](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) 文件格式在 QnA 文件夹 CogSvcModels 中提供。 然后，将 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 用作部署脚本的一部分来创建 QnAMaker JSON 文件，该文件随后可供 [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI（命令行）工具用来将项目发布到 QnAMaker 知识库。
