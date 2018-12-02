---
title: 企业机器人模板的概述 | Microsoft Docs
description: 了解企业机器人模板的功能
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 43bc3c7606a12084690d71f8b6ea2dc3b2e5984d
ms.sourcegitcommit: 6c719b51c9e4e84f5642100a33fe346b21360e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/28/2018
ms.locfileid: "52451989"
---
# <a name="enterprise-bot-template"></a>企业机器人模板 

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

创建高质量的聊天体验需要一系列基本功能。 为了帮助你成功地构建良好的聊天体验，我们创建了一个企业机器人模板。 此模板汇集了我们通过构建聊天体验确定的所有最佳做法和支持组件。 

此模板大大简化了新机器人项目的创建工作。 该模板将利用 [Bot Builder SDK v4](https://github.com/Microsoft/botbuilder) 和 [Bot Builder 工具](https://github.com/Microsoft/botbuilder-tools)提供以下开箱即用功能。

Feature | Description |
------------ | -------------
介绍消息 | 在聊天开始时使用自适应卡的介绍消息。 它解释了机器人的功能，并提供了指导初始问题的按钮。 然后，开发人员可以根据需要自定义此项。
自动键入指示符  | 在聊天期间发送视觉输入指示符，并对长时间运行的操作重复此操作。
.bot 文件驱动的配置 | 机器人的所有配置信息（如 LUIS、Dispatcher Endpoints、Application Insights）都打包在 .bot 文件中，用于驱动机器人启动。
基本聊天意向  | 使用英语、法语、意大利语、德语、西班牙语和中文表达的基本意向（问候、告别、帮助、取消等）。 这些意向在 .LU（语言理解）文件中提供，可以轻松修改。
基本聊天响应  | 对抽象为单独 View 类的基本聊天意向的响应。 这些响应将在未来转移到新的语言生成 (LG) 文件。
不适当的内容或 PII（个人身份信息）检测  |通过在中间件组件中使用[内容审查器](https://azure.microsoft.com/en-us/services/cognitive-services/content-moderator/)来检测传入聊天中的不适当内容或 PII 数据。
脚本  | Azure 存储中存储的所有聊天内容的脚本
调度程序 | 一个集成的[调度](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-dispatch?view=azure-bot-service-4.0&tabs=csaddref%2Ccsbotconfig)模型，用于识别给定的话语是应由 LUIS + 代码处理还是应传递给 QnA Maker。
QnA Maker 集成  | 与 [QnA Maker](https://www.qnamaker.ai) 集成可回答知识库中的一般问题，这些问题可以是如何利用现有数据源（例如 PDF手册）。
聊天见解  | 与 [Application Insights](https://azure.microsoft.com/en-gb/services/application-insights/) 集成可收集所有聊天和示例 PowerBI 仪表板的遥测数据，让你开始深入了解聊天体验。

此外，将自动部署机器人所需的所有 Azure 资源：机器人注册、Azure 应用服务、LUIS、QnA Maker、内容审查器、CosmosDB、Azure 存储和 Application Insights。 此外，还将创建、训练和发布基本 LUIS、QnA Maker 和调度模型，以便立即测试基本意向和路由。

创建模板并执行部署步骤后，可以按 F5 进行端到端测试。 这为开始你的聊天体验提供了坚实的基础，减少了每个项目必须承担的多天工作量并提高了聊天质量标准。

若要开始，请继续[创建项目](bot-builder-enterprise-template-create-project.md)。 若要详细了解驱动上述功能的最佳做法和知识，请参阅[模板详细信息](bot-builder-enterprise-template-overview-detail.md)主题。 

此 [GitHub 位置](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template)提供了企业机器人模板的完整源代码
