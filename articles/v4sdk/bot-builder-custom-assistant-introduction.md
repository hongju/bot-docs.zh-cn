---
title: 自定义助手概述 | Microsoft Docs
description: 了解如何创建自己的自定义助手
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f4b8243580ee678390177881b136a9016be4a786
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215465"
---
## <a name="custom-assistant-overview"></a>自定义助手概述

## <a name="overview"></a>概述

我们了解到，客户和合作伙伴强烈要求我们提供一个聊天助手，该聊天助手针对他们的品牌量身打造，针对其客户提供个性化内容，并且可以在各种各样的聊天画布和设备中使用。 开源自定义个人助手继续了 Microsoft 实现 Bot Framework SDK 时使用的开源方法，可以对构建在一组基本功能上的最终用户体验进行完全的控制。 另外，还可以在该体验中注入有关最终用户以及任何设备/生态系统信息的智能，实现完全集成的智能体验。

我们坚信客户应该拥有并丰富其客户关系和见解。 因此，任何自定义助手都允许客户和合作伙伴对用户体验进行完全的控制。 名称、语音和个性可以根据组织需求进行更改。 我们的自定义助手解决方案简化了助手创建过程，数分钟即可入门。 

自定义个人助手的功能范围很广，通常会为最终用户提供各种功能。 为了提高开发人员的工作效率，并且为了实现一个包含可重用聊天体验在内的充满活力的生态系统，我们为开发人员提供可重用聊天技能的初始示例。 这些技能可以添加到聊天应用程序中，开启特定的聊天体验，例如查找兴趣点，与日历、任务、电子邮件和许多其他的方案交互。 技能可完全自定义，包含的语言模型适合多种语言、对话框和代码。

目前，我们运行的是初始预览版，正与初始客户和合作伙伴在一个开源存储库中密切合作，以便推出初始体验，并使之过几个月后可以在更广泛的范围内使用。 

![自定义助手图](media/enterprise-template/CustomAssistantDiagram.jpg)

## <a name="complete-control-of-the-user-experience"></a>完全控制用户体验

最终用户体验的所有方面都由你做主并控制。 这包括品牌、名称、语音、响应和虚拟形象。 提供给你的自定义助手和支持技能的源代码完全允许你根据需要进行调整。

## <a name="complete-ownership-and-control-of-data"></a>数据的完整所有权和控制权

自定义助手会部署在 Azure 订阅中。 因此，由助手生成的所有数据（提问的问题、用户行为等）全都包含在 Azure 订阅中。 有关详细信息，请更具体地参阅[认知服务 Azure 信任云](https://www.microsoft.com/en-us/trustcenter/cloudservices/cognitiveservices)和[信任中心的 Azure 部分](https://www.microsoft.com/en-us/TrustCenter/CloudServices/Azure)。

## <a name="your-assistant-anywhere"></a>你的助手，不管你身在何处...

自定义助手利用了 Microsoft 聊天 AI 平台，因此可以通过任何 Framework [通道](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0)（例如 WebChat、FaceBook Messenger、Skype 等）来显示。另外，我们可以通过 [Direct Line](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) 通道将体验嵌入到桌面和移动应用，包括汽车、扬声器、闹钟等设备。

## <a name="built-on-enterprise-grade-technology"></a>以企业级技术为基础构建

自定义助手解决方案在 Azure 机器人服务、语言理解认知服务、统一语音以及一大系列的 Azure 支持组件基础上构建，这意味着你可以受益于 [Azure 全球基础结构](https://azure.microsoft.com/en-gb/global-infrastructure/)。

另外，语言理解支持由 LUIS 认知服务提供，后者支持[此处列出](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-supported-languages)的一大系列的语言。 [Translator 认知服务](https://azure.microsoft.com/en-us/services/cognitive-services/translator-text-api/)提供其他机器翻译功能，进一步扩大自定义助手的应用范围。

## <a name="integrated-and-context-aware"></a>集成和上下文感知

自定义助手可以集成到设备和生态系统中，实现完全集成的智能体验。 通过这种上下文感知，可以开发更智能的体验，并可提供比任何其他的可能方式更进一步的个性化。

## <a name="3rd-party-assistant-integration"></a>第三方助手集成

自定义助手可以用来提供你自己的唯一体验，但对于某些类型的问题，也可将其提供给最终用户选择的数字助手进行处理。

## <a name="flexible-integration"></a>灵活的集成

我们的自定义助手体系结构很灵活，可以与你投到基于设备的语音或自然语言处理功能的现有投资集成，当然也可以与现有的后端系统和 API 集成。

## <a name="adaptive-cards"></a>自适应卡片

[自适应卡片](https://adaptivecards.io/)允许自定义助手返回用户体验元素（例如，卡片、图像、按钮）以及文本库响应。 如果设备或聊天画布有一个屏幕，则可以跨很大范围的设备和平台来渲染这些自适应卡片，根据需要提供支持的用户体验。 [此处](https://adaptivecards.io/samples/)提供自适应卡片的示例，[此处](https://docs.microsoft.com/en-us/adaptive-cards/rendering-cards/getting-started)的文档介绍渲染选项。


## <a name="skills"></a>技能

除了基本助手，还有一大系列的常用功能，这些功能需要每个开发人员自行构建。 工作效率是一个很好的示例，说明了每个组织需要创建语言模型 (LUIS)、对话框（代码）、集成（代码）和语言生成（响应）功能，以便启用常见方案，例如兴趣点、电子邮件、日历或任务。

由于需要支持多种语言，这一点随后就进一步复杂化了，结果就是任何组织在生成自己的助手时，都有大量工作需要完成。

我们的自定义助手解决方案包含新的技能功能，只需通过配置即可将新功能插入自定义助手中。 

每个技能的所有方面（语言模型、对话框、集成代码和语言生成）完全可以由开发人员自定义，因为完整的源代码在 GitHub 上与自定义助手一起提供。

## <a name="example-scenarios"></a>示例方案

自定义助手可以扩展到许多行业方案，示例方案显示在下面，供参考：

- 汽车行业：支持语音的个人助手在集成到汽车中以后，可以为最终用户提供执行传统汽车操作（例如，导航、收音机）的功能；另外还有专注工作效率的方案，例如在迟到的情况下，移动会议就很有用；可以将项目添加到任务列表中；可以提供前摄性体验，让汽车根据事件来建议要完成的任务，例如启动引擎、回家或启用巡航控制。 自适应卡片在通过一键通或唤醒词交互执行的机头单元和语音集成中渲染。

- 酒店：支持语音的个人助手在集成到酒店房间设备以后，可以提供一大系列专注酒店的方案（例如，续房、请求延迟退房、房间服务），包括接待功能，以及查找附近酒店和名胜的功能。 允许选择性地链接到生产帐户为你带来更多个性化体验，例如建议的闹钟服务、天气警报、了解住宿时间模式。 由目前聊天室中的最新电视个性化体验演变而来。

- 企业：支持语音和文本的带品牌员工助手体验集成到企业设备和现有聊天画布（例如，Teams、WebChat、Slack）中，使员工可以管理其日历、查找可用的会议室、查找具有特定技能的人员，或者执行与 HR 相关的操作。 

## <a name="getting-started"></a>入门

目前，我们运行的是初始预览版，正与初始客户和合作伙伴在一个开源存储库中密切合作，以便推出初始体验，并使之过几个月后可以在更广泛的范围内使用。 若要登记你的意向并开始使用，请填写[此表](https://aka.ms/customassistantpreviewform)，我们会联系你。

