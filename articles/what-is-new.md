---
title: 新增功能 | Microsoft Docs
description: 了解 Bot Framework 中的新增功能。
keywords: bot framework, azure 机器人服务
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2c8284bf1a78c4f8dd9fb5cc3dcb346ac99ad936
ms.sourcegitcommit: 710d279898db587abb1e81d13628177a4e182293
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/06/2019
ms.locfileid: "66751301"
---
# <a name="whats-new-in-bot-framework"></a>Bot Framework 中的新增功能

[!INCLUDE[applies-to](includes/applies-to.md)]

Bot Framework SDK v4 是一个[开源 SDK][1a]，可让开发人员使用其偏好的编程语言来建模和生成复杂的聊天。

本文汇总了 Bot Framework 与 Azure 机器人服务中的重要新功能和改进。

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK 中 IsInRole 中的声明 |[4.4.3][1] | [4.4.0][2] | [4.4.0b1（预览版）][3] | [4.0.0a6（预览版）][3a]|
|文档 | [文档][5] |[文档][5] |  | |
|示例 |[.NET Core][6]、[WebAPI][10] |[Node.js][7]、[TypeScript][8]、[es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[4]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/en-us/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>Bot Framework SDK（新功能！ 预览版）

- [自适应对话][47] | [文档][48] | [C# 示例][49]：自适应对话可让开发人员生成在聊天过程中可以动态变化的聊天。  一直以来，开发人员都是事先制定好整个聊天流，这会限制聊天的灵活性。  自适应对话可让开发人员更灵活地应对上下文的变化，并在聊天过程中将新的步骤或整个子对话插入到聊天中。 

- [Language Generation][43] | [文档][44] | [C# 示例][45]：Language Generation 可让开发人员从其代码和资源文件中提取嵌入的字符串，并通过 Language Generation 运行时和文件格式管理这些文件。  Language Generation 可让客户定义某个短语的多个变体、基于上下文执行简单的表达式和引用聊天内存，并可让我们不断引入其他功能，以创建更自然的聊天体验。

- [通用表达式语言][40] | [api][41]：自适应对话和 Language Generation 都依赖于使用通用表达式语言来实现机器人聊天。

[40]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/common-expression-language#readme
[41]:https://github.com/Microsoft/BotBuilder-Samples/blob/master/experimental/common-expression-language/api-reference.md
[43]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation#readme
[44]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/docs
[45]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/csharp_dotnetcore
[46]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/javascript_nodejs/13.core-bot
[47]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme
[48]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs
[49]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore
[50]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/declarative

## <a name="botkit"></a>Botkit
[Botkit][100] 是一个开发人员工具和 SDK，用于生成适用于主流消息平台的聊天机器人、应用和自定义集成。 Botkit 机器人可以通过 `hear()` 接收触发器，通过 `ask()` 提出问题，并通过 `say()` 讲出答复。 开发人员可以使用此语法来生成对话 - 该语法现在与最新版本的 Bot Framework SDK 兼容。 

此外，Botkit 随附了 6 个平台适配器，可让 Javascript 机器人应用程序直接与消息平台通信：[Slack][102]、[Webex Teams][103]、[Google Hangouts][104]、[Facebook Messenger][105]、[Twilio][106] 和 [Web chat][107]。

Botkit 是 Microsoft Bot Framework 的一部分，根据 [MIT 开放源代码许可][101]发行

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme

## <a name="bot-framework-solutions-new-in-preview"></a>Bot Framework 解决方案（新功能！ 预览版）

[Bot Framework 解决方案存储库](https://github.com/Microsoft/AI#readme)是用于帮助构建助手式高级聊天体验的一组模板、解决方案加速器和技能的基地库。

| 名称 | 说明 |  
|:------------|:------------| 
|[**虚拟助手**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | 客户强烈要求我们提供一款根据其品牌定制的、针对其用户个性化的，并且可在各种画布和设备中使用的聊天助手。 <br/><br/> 企业模板大大简化了新机器人项目的创建，包括：基本聊天意向、Dispatch 集成、QnA Maker、Application Insights 和自动化部署。|
|[**技能**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| 开发人员可以通过将可重用的聊天功能（称为技能）拼接到一起来构造聊天体验。 技能本身是远程调用的机器人，可以使用技能开发人员模板（.NET、TS）来简化新技能的创建。 
|[**分析**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| 使用 Conversational AI Analytics 解决方案获取机器人运行状况和行为的关键见解。 查看可用的遥测数据、Application Insights 示例查询和 Power BI 仪表板可以了解机器人与用户之间的整个聊天情景。 |

## <a name="azure-bot-service"></a>Azure 机器人服务
使用 Azure 机器人服务可以托管智能化的企业级机器人，同时可以获得数据的完整所有权和控制权。 开发人员可以在 Skype、Microsoft Teams、Cortana、Web Chat 等服务中注册机器人以及将机器人连接到用户。 [Azure][27]  |  [文档][28] | [连接到通道][29] 

* **Direct Line JS 客户端**：若要在 Azure 机器人服务中使用 Direct Line 通道且不使用 WebChat 客户端，可以在自定义应用程序中使用 Direct Line JS 客户端。 有关详细信息，请访问 [Github][30]。

<a name="ABS-whats-new"></a>

* **新功能！Direct Line 语音通道**：我们正在将 Bot Framework 与 Microsoft 的语音服务相结合，以提供一个通道用于在客户端与机器人应用程序之间双向流式传输语音和文本。  有关详细信息，请参阅如何[将语音通道添加到机器人](https://docs.microsoft.com/en-us/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0)。

[27]:https://azure.microsoft.com/en-us/services/bot-service/
[28]:https://docs.microsoft.com/en-us/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0
[30]:https://github.com/Microsoft/BotFramework-DirectLineJS/blob/master/README.md


## <a name="bot-framework-emulator"></a>Bot Framework Emulator
[Bot Framework Emulator][60] 是一个跨平台桌面应用程序，可让机器人开发人员测试和调试使用 Bot Framework SDK 生成的机器人。 可以使用 Bot Framework Emulator 测试计算机本地运行的机器人，或连接到远程运行的机器人。

- [下载最新版本][61] | [文档][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>Bot Inspector（新功能！ 预览版）

Bot Framework Emulator 已发布 Beta 版的 Bot Inspector 新功能。 使用此功能可以在 Microsoft Teams、Slack、Cortana、Facebook Messenger、Skype 等通道上调试和测试 Bot Framework SDK v4 机器人。由于运行了聊天，消息将镜像到 Bot Framework Emulator，在其中可以检查机器人收到的消息数据。 此外，还会呈现通道与机器人之间任意给定轮次的机器人状态快照。 详细了解 [Bot Inspector](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md)

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0


## <a name="related-services"></a>相关服务

### <a name="language-understanding"></a>语言理解 
用于生成自然语言体验的基于机器学习的服务。 快速创建可持续改进的企业就绪自定义模型。 [使用语言理解服务 (LUIS)][30]，应用程序可以理解用户以自己的语言表达的内容。

<a name="LUIS-whats-new"></a>

- **新功能！角色、外部实体和动态实体**：LUIS 中添加了多项功能，可让开发人员从文本中提取更详细的信息，使用户能够以更少的工作量生成更智能化的解决方案。 LUIS 还将角色扩展到了所有实体类型，允许根据上下文使用不同的子类型将相同的实体分类。 开发人员现在可以更精细地控制他们可以在 LUIS 中执行的操作，包括在运行时通过动态列表和外部实体识别与更新模型。 动态列表在预测时用于追加到列表实体，允许精确匹配用户特定的信息。 单独的补充实体提取器将与外部实体配合运行，可将该信息追加到 LUIS 作为其他模型的强信号。

- **新功能！分析仪表板**：LUIS 正在发布一个更详细的、视觉信息更丰富的综合性分析仪表板。 其用户友好的设计突显了大多数用户在设计应用程序时遇到的常见问题，并提供有关如何解决这些问题的简单说明，以帮助用户更深入地了解其模型的质量和潜在的数据问题；另外它还提供有关如何采用最佳做法的指导。

[文档][31] | [将语言理解添加到机器人][32] 

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme
[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33] 是一个基于云的 API 服务，用于创建基于数据的聊天式问答层。 使用 QnA Maker 可在几分钟内基于 FAQ URL、结构化文档、产品手册或编辑内容生成、训练和发布简单的问答机器人。

<a name="QnA-whats-new"></a>

- **新功能！提取管道**：现在，可以从 URL、文件和 Sharepoint 提取分层信息
- **新功能！智能**：区分上下文的排名模型，主动学习建议
- **新功能！聊天**：QnA Maker 中的多轮次聊天。

[文档][34]  | [将 qnamaker 添加到机器人][35] 

[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/qnamaker-docs-home
[35]:https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

### <a name="speech-services"></a>语音服务
[语音服务](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/)可将音频转换为文本，使用统一语音服务执行语音翻译和文本转语音的操作。 使用语音服务可将语音集成到机器人、创建自定义唤醒词，并以多种语言创作。

### <a name="adaptive-cards"></a>自适应卡片
[自适应卡片](https://adaptivecards.io)是一个开放标准，可让开发人员以平时的一致方式交换卡内容。Bot Framework 开发人员可以使用自适应卡片来创建优异的跨通道聊天体验。

## <a name="additional-information"></a>其他信息
- 有关详细[信息](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new)，请访问 GitHub 页。
