---
title: 企业效率机器人方案 | Microsoft Docs
description: 使用 Bot Framework 探索企业效率机器人方案。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0dc869aa1464c086b6596ee83d8e6e488d8a8a55
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298030"
---
# <a name="enterprise-productivity-bot-scenario"></a>企业效率机器人方案
企业机器人展示了如何通过将机器人与 Office 365 日历和其他服务集成来提高工作效率。

无需打开一堆窗口即可快速访问客户信息，这就是企业效率机器人所实现的功能。 通过使用简单的聊天命令，销售代表可以查找客户并通过 Graph API 和 Office 365 查看他们的下一个约会。 从那里，他们可以访问存储在 Dynamics CRM 中的客户特定信息，例如，获取案例或创建新案例。

![企业机器人关系图](~/media/scenarios/bot-service-scenario-enterprise-bot.png)

以下介绍企业效率机器人的逻辑流程：

1. 员工访问企业效率机器人。
2. Azure Active Directory 验证员工的身份。
3. 企业效率机器人能够通过 Azure Graph 查询员工的 Office 365 日历。
4. 使用从日历收集的数据，机器人访问 Dynamics CRM 中的案例信息。
5. 信息将返回给可在不离开机器人的情况下筛选数据的员工。
6. Application Insights 收集运行时遥测来帮助提高机器人性能和使用率。

可以从[常见 Bot Framework 方案的示例](https://aka.ms/bot/scenarios)下载或克隆此示例机器人的源代码。

## <a name="sample-bot"></a>示例机器人
由于可从多种通道访问机器人，因此你可以在办公桌上通过公司门户或 Skype 随时随地地使用机器人，只需进行身份验证即可。 通过 Azure AD 集成，企业效率机器人知道如果你能访问它，即表示你已经过 Azure AD 身份验证。 从此处，你可以要求机器人查看与特定客户的下一次约会的具体时间。 机器人通过 Graph API 查询 Office 365 来获取此信息。 然后，如果在接下来的七天内有约会，机器人会查询 CRM 以查找客户的最新案例。 机器人以没有找到任何案例或以打开和关闭的案例数进行响应。 从此处，你可以要求机器人按类型列出案例并深入了解各个案例。

## <a name="components-youll-use"></a>将使用的组件
企业效率机器人使用以下组件：
-   用于身份验证的 Azure AD
-   Office 365 的 Graph API
-   Dynamics CRM
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) 是 Microsoft 提供的基于多租户云的目录和标识管理服务。 作为机器人开发工具，Azure AD 让你可以快速轻松地与世界各地数百万组织使用的世界一流标识管理解决方案集成，从而专注于构建机器人。 通过定义 Azure AD 应用，你可以控制谁有权访问你的机器人及其公开的数据，而无需实现自己复杂的身份验证和授权系统。

### <a name="graph-api-to-office-365"></a>Office 365 的 Graph API
Microsoft Graph 通过 https://graph.microsoft.com 的单个终结点公开 Office 365 和其他 Microsoft 云服务的多个 API。 Microsoft Graph 使你和机器人可以更加轻松地执行查询。 API 公开多个 Microsoft 云服务的数据，这些服务包括属于 Office 365 的 Exchange Online、Azure Active Directory、SharePoint 等。 可以使用 API 在实体和关系之间进行导航。 可以从使用 SDK 或 REST 终结点的机器人中使用此 API，也可以从本机支持 Android、iOS、Ruby、UWP、Xamarin 等的其他应用使用此 API。

### <a name="dynamics-crm"></a>Dynamics CRM
Dynamics CRM 是客户参与平台。 使用机器人和 CRM 的 API，可以生成丰富的交互式机器人，此机器人可以访问存储在 CRM 中的丰富数据。 Dynamics CRM 的强大功能可供机器人创建案例、检查状态、进行知识管理搜索等。

### <a name="application-insights"></a>Application Insights
Application Insights 可帮助你通过应用程序性能管理 (APM) 和即时分析获取可付诸实施的见解。 开箱即可获得丰富的性能监视、功能强大的警报和易于使用的仪表板，帮助确保机器人可用且行为符合预期。 可以快速了解是否有问题，然后执行根本原因分析以便查找并解决问题。

## <a name="next-steps"></a>后续步骤
接下来，了解信息机器人方案。

> [!div class="nextstepaction"]
> [信息机器人方案](bot-service-scenario-informational.md)
