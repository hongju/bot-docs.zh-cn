---
title: 商业机器人方案 | Microsoft Docs
description: 使用 Bot Framework 探索商业机器人方案。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 15745bc25013df2fd18b0a2045ae2314d6c361e2
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574853"
---
# <a name="commerce-bot-scenario"></a>商业机器人方案

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

[商业机器人](bot-service-scenario-commerce.md)方案描述了一种机器人，它使用酒店的礼宾服务代替人们通常进行的传统电子邮件和电话交互。 机器人利用认知服务通过文本和语音更好地处理客户请求，并通过与后端服务集成收集上下文。

![应用程序机器人关系图](~/media/scenarios/bot-service-scenario-commerce-bot.png)

以下是充当酒店礼宾员的商业机器人的逻辑流程：

1. 客户使用酒店移动应用。
2. 用户使用 Azure AD B2C 进行身份验证。
3. 用户使用自定义应用程序机器人请求信息。 
4. 认知服务帮助处理自然语言请求。
5. 响应由可使用自然聊天精简问题的客户进行审阅。
6. 用户对结果感到满意之后，应用程序机器人将更新客户的预订。
7. Application Insights 收集运行时遥测来帮助提高机器人性能和使用率。

## <a name="sample-bot"></a>示例机器人
示例商业机器人是围绕虚构的酒店礼宾服务进行设计的。 用 C# 编写的客户使用连锁店的会员服务移动应用通过酒店向 Azure AD B2C 进行身份验证后，即可访问机器人。 连锁店在 SQL 数据库中存储预订。 客户可以使用自然短语问题，例如“在我住宿期间租用泳池小屋要多少钱”。 而机器人则获得有关客人住宿的酒店和住宿持续时间的上下文。 此外，语言理解 (LUIS) 服务使机器人可以轻松地从一个简单的短语（如“泳池小屋”）获取上下文。 机器人提供解答，然后可以为客人预订小屋，并提供有关天数和小屋类型的选项。 一旦机器人拥有所有必要的数据，它就会预订请求。 客人也可以使用他们的声音发出相同的请求。

可以从[常见 Bot Framework 方案的示例](https://aka.ms/bot/scenarios)下载或克隆此示例机器人的源代码。

## <a name="components-youll-use"></a>将使用的组件
商业机器人使用以下组件：
-   用于身份验证的 Azure AD
-   认知服务：LUIS
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) 是 Microsoft 提供的基于多租户云的目录和标识管理服务。 作为机器人开发人员，Azure AD 让你可以快速轻松地与世界各地数百万组织使用的世界一流标识管理解决方案集成，从而专注于构建机器人。 Azure AD 支持使用 B2C 连接器，目的是确定使用外部 ID（例如 Google、Facebook 或 Microsoft 帐户）的用户。 Azure AD 消除了你必须管理用户凭据的责任，而让你专注于机器人解决方案，因为知道你可以将机器人的用户与应用程序公开的正确数据相关联。

### <a name="cognitive-services-luis"></a>认知服务：LUIS
作为认知服务技术系列的成员，语言理解 (LUIS) 为你的应用带来了机器学习的强大功能。 目前，LUIS 支持多种语言，使机器人能够理解人们想要的内容。 与 LUIS 集成时，你表达意向并定义机器人理解的实体。 然后，通过使用示例话语训练机器人，教机器人理解这些意向和实体。 你可以使用短语列表和正则表达式功能调整集成，以便机器人尽可能流利，满足你的特定聊天需求。

### <a name="application-insights"></a>Application Insights
Application Insights 可帮助你通过应用程序性能管理 (APM) 和即时分析获取可付诸实施的见解。 开箱即可获得丰富的性能监视、功能强大的警报和易于使用的仪表板，帮助确保机器人可用且行为符合预期。 你可以快速了解是否有问题，然后执行根本原因分析以便查找并解决问题。

## <a name="next-steps"></a>后续步骤
接下来，了解 Cortana 技能机器人方案。

> [!div class="nextstepaction"]
> [Cortana 技能机器人方案](bot-service-scenario-cortana-skill.md)
