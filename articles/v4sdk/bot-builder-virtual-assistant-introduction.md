---
title: 虚拟助手概述 | Microsoft Docs
description: 了解如何创建自己的虚拟助手
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 13/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99c37812a5c13fe2409a68cbb8614cf8144d0711
ms.sourcegitcommit: b94c4286f6f64955fd51ccf4a68109c43db0e47d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65083690"
---
# <a name="virtual-assistant-overview"></a>虚拟助理概述

## <a name="overview"></a>概述

客户和合作伙伴强烈要求我们提供一款根据其品牌定制的、针对其用户个性化的，并且可在各种画布和设备中使用的聊天助手。 <br/><br/> 开源虚拟助理解决方案继续了 Microsoft 实现 Bot Framework SDK 时使用的开源方法，为你提供一套核心的基本功能，让你能够完全控制最终用户体验。 <br/><br/> 此模板纳入了以前的企业模板并汇集了在积累聊天体验过程中确定的所有最佳做法和支持组件，大大简化了新机器人项目的创建，包括：基本聊天意向、Dispatch 集成、QnA Maker、Application Insights 和自动化部署。

我们坚信客户应该拥有并丰富其客户关系和见解。 因此，任何虚拟助手都允许客户和合作伙伴通过 GitHub 上的开源代码对用户体验进行完全的控制。 名称、语音和个性可以根据组织需求进行更改。 我们的虚拟助手解决方案简化了助手创建过程，数分钟即可入门，然后进一步使用端到端开发工具。

虚拟助手的功能范围很广，通常会为最终用户提供各种功能。 为了提高开发人员的工作效率，并且为了实现一个包含可重用聊天体验在内的充满活力的生态系统，我们为开发人员提供可重用聊天技能的初始示例。 这些技能可以添加到聊天应用程序中，开启特定的聊天体验，例如查找兴趣点，与日历、任务、电子邮件和许多其他的方案交互。 技能可完全自定义，包含的语言模型适合多种语言、对话框和代码。

![虚拟助手图](./media/enterprise-template/customassistantdiagram.jpg)

## <a name="get-started"></a>入门

如需更多详细信息，请浏览[虚拟助理和技能](https://github.com/Microsoft/AI)文档。

## <a name="whats-in-the-box"></a>现成内容 

虚拟助理模板汇集了我们通过构建聊天体验确定的一些最佳做法，并自动集成了我们发现的对 Bot Framework 开发人员非常有益的组件。 本部分介绍关键决策的一些背景知识，帮助用户了解模板的工作方式。

虚拟助理模板现在纳入了以前的企业模板功能，包括多语言形式的基本聊天意向、调度、QnA 和聊天见解。 下述与助手相关的功能是目前提供的，其他功能仍在计划中。我们会与客户及合作伙伴密切合作，及时将路线图告知他们。

Feature | 说明 |
------------ | -------------
登记 | 示例 OnBoarding 流允许助手问候用户并收集初始信息。
事件处理体系结构 | 使用虚拟助手上下文中的事件，在 Web 浏览器或者汽车或扬声器之类的设备上托管助手的客户端应用程序就可以交换有关用户或设备事件的信息，同时还可以在收到事件后执行设备操作。
链接的帐户 | 在语音引导方案中，让用户通过语音命令输入适用于支持系统的用户名和密码不可行。 因此，可以通过单独的助手体验让用户有机会登录并提供可供虚拟助手检索以后使用的令牌的权限。
技能支持 | 目前存在一大系列的常用功能，这些功能需要每个开发人员自行构建。 我们的虚拟助手解决方案包含新的技能功能，只需通过配置即可将新功能插入虚拟助手中。此外还提供针对技能的身份验证机制，用于请求下游活动的令牌。
兴趣点技能 | 预览版兴趣点 (PoI) 技能提供一个全面的语言模型，用于查找兴趣点和请求指导。 此技能目前提供集成到 Azure Maps 中的功能。
日历技能 | 预览版日历技能提供了一个适用于常见的日历相关活动的语言模型；此技能目前集成到 Microsoft Graph (Office 365/Outlook.com) 中，很快会提供对 Google API 的支持。
电子邮件技能 | 预览版电子邮件技能提供了一个适用于常见的电子邮件相关活动的语言模型；此技能目前集成到 Microsoft Graph (Office 365/Outlook.com) 中，很快会提供对 Google API 的支持。
待办事项技能 | 预览版待办事项技能提供了一个适用于常见的任务相关活动的语言模型；此技能目前集成到 OneNote 中，很快会提供对 Microsoft Graph (outlookTask) 的支持。
设备集成 | 使用 Azure 机器人服务 SDK (DirectLine) 和自适应卡片以及语音 SDK 可以轻松地跨平台集成到设备。 其他设备集成示例和平台（包括 Edge）已列入计划。
测试工具 | 除了 Bot Framework Emulator，还提供基于 WebChat 的测试工具，因此可以测试更复杂的身份验证方案。 通过简单的基于控制台的测试工具，演示了如何交换消息才能确保轻松集成设备。
自动化部署 | 助手所需的所有 Azure资源均自动部署：机器人注册、Azure 应用服务、LUIS、QnAMaker、内容审查器、CosmosDB、Azure 存储和 Application Insights。 此外，还将创建、训练和发布适用于所有技能的 LUIS 模型、QnAMaker 和调度模型，以便立即测试。
汽车语言模型 | 涵盖核心领域（例如电话、导航以及车内功能控制）的汽车语言模型即将推出。

## <a name="example-scenarios"></a>示例方案

虚拟助手可以扩展到许多行业方案。 一些示例方案显示在下面，供参考。

- 汽车行业：支持语音的个人助手在集成到汽车中以后，可以为最终用户提供执行传统汽车操作（例如，导航、收音机）的功能；另外还有专注工作效率的方案，例如在迟到的情况下，移动会议就很有用；可以将项目添加到任务列表中；可以提供前摄性体验，让汽车根据事件来建议要完成的任务，例如启动引擎、回家或启用巡航控制。 自适应卡片在通过一键通或唤醒词交互执行的机头单元和语音集成中渲染。

- 酒店：支持语音的个人助手在集成到酒店房间设备以后，可以提供一大系列专注酒店的方案（例如，续房、请求延迟退房、房间服务），包括接待功能，以及查找附近酒店和名胜的功能。 允许选择性地链接到生产帐户为你带来更多个性化体验，例如建议的闹钟服务、天气警报、了解住宿时间模式。 由目前聊天室中的最新电视个性化体验演变而来。

- 企业：支持语音和文本的带品牌员工助手体验集成到企业设备和现有聊天画布（例如，Teams、WebChat、Slack）中，使员工可以管理其日历、查找可用的会议室、查找具有特定技能的人员，或者执行与 HR 相关的操作。

## <a name="virtual-assistant-principles"></a>虚拟助手原则

### <a name="your-data-your-brand-and-your-experience"></a>数据、品牌和体验
最终用户体验的所有方面都由你做主并控制。 这包括品牌、名称、语音、个性、响应和虚拟形象。 虚拟助手和支持技能的源代码会完整地提供给你，允许你根据需要对其进行调整。

虚拟助手会部署在 Azure 订阅中。 因此，由助手生成的所有数据（提问的问题、用户行为等）全都包含在 Azure 订阅中。 有关详细信息，请更具体地参阅[认知服务 Azure 信任云](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices)和[信任中心的 Azure 部分](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure)。

### <a name="write-it-once-embed-it-anywhere"></a>编写一次之后，即可将其嵌入任意位置
虚拟助手利用了 Microsoft 聊天 AI 平台，因此可以通过任何 Framework [通道](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0)（例如 WebChat、FaceBook Messenger、Skype 等）来显示。 

另外，我们可以通过 [Direct Line](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) 通道将体验嵌入到桌面和移动应用，包括汽车、扬声器、闹钟等设备。

### <a name="enterprise-grade-solutions"></a>企业级解决方案
虚拟助手解决方案在 Azure 机器人服务、语言理解认知服务、统一语音以及一大系列的 Azure 支持组件基础上构建，这意味着你可以受益于 [Azure 全球基础结构](https://azure.microsoft.com/en-gb/global-infrastructure/)，包括 ISO 27018，HIPPA，PCI DSS，SOC 1、2、3 认证。

另外，语言理解支持由 LUIS 认知服务提供，后者支持[此处列出](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages)的一大系列的语言。 [Translator 认知服务](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/)提供其他机器翻译功能，进一步扩大虚拟助手的应用范围。

### <a name="integrated-and-context-aware"></a>集成和上下文感知
虚拟助手可以集成到设备和生态系统中，实现完全集成的智能体验。 通过这种上下文感知，可以开发更智能的体验，并可提供比任何其他的可能方式更进一步的个性化。

### <a name="3rd-party-assistant-integration"></a>第三方助手集成
虚拟助手可以用来提供你自己的唯一体验，但对于某些类型的问题，也可将其提供给最终用户选择的数字助手进行处理。

### <a name="flexible-integration"></a>灵活的集成
我们的虚拟助手体系结构很灵活，可以与你投到基于设备的语音或自然语言处理功能的现有投资集成，当然也可以与现有的后端系统和 API 集成。

### <a name="adaptive-cards"></a>自适应卡片
[自适应卡片](https://adaptivecards.io/)允许虚拟助手返回用户体验元素（例如，卡片、图像、按钮）以及文本库响应。 如果设备或聊天画布有一个屏幕，则可以跨很大范围的设备和平台来渲染这些自适应卡片，根据需要提供支持的用户体验。 [此处](https://adaptivecards.io/samples/)提供自适应卡片的示例，[此处](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started)的文档介绍渲染选项。

### <a name="skills"></a>技能
除了基本助手，还有一大系列的常用功能，这些功能需要每个开发人员自行构建。 工作效率是一个很好的示例，说明了每个组织需要创建语言模型 (LUIS)、对话框（代码）、集成（代码）和语言生成（响应）功能，以便启用常见的日历、任务或电子邮件体验。

由于需要支持多种语言，这一点随后就进一步复杂化了，结果就是任何组织在生成自己的助手时，都有大量工作需要完成。

我们的虚拟助手解决方案包含新的技能功能，只需通过配置即可将新功能插入虚拟助手中。 

每个技能的所有方面（语言模型、对话框、集成代码和语言生成）完全可以由开发人员自定义，因为完整的源代码在 GitHub 上与虚拟助手一起提供。

## <a name="getting-started"></a>入门

虚拟助手解决方案在[此 GitHub 存储库](https://github.com/Microsoft/AI/)中提供。虚拟助手团队会定期更新此存储库。 同一存储库中提供了更详细的文档。可以通过 GitHub 反馈机制直接提供问题/反馈。