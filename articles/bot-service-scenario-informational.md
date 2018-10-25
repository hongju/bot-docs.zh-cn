---
title: 信息机器人方案 | Microsoft Docs
description: 使用 Bot Framework 探索信息机器人方案。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 22902f1f590661c0973d7f0427b13ee45f0d7227
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998984"
---
# <a name="information-bot-scenario"></a>信息机器人方案

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

此信息机器人可以使用认知服务 QnA Maker 回答知识集中定义的问题或常见问题，也可以使用 Azure 搜索回答更加开放的问题。

通常，通过搜索可以轻松地使淹没在 SQL Server 等结构化数据中的信息浮出表面。 想象一下按简单的会话式命令查找客户的订单状态。 使用认知服务 QnA Maker，向用户呈现一组有效的搜索选项，例如查找客户、查看客户的最近订单等。通过定义 QnA 格式，用户可以轻松地提出 Azure 搜索支持的问题，而 Azure 搜索可以查找存储在 SQL 数据库中的数据。

![信息机器人关系图](~/media/scenarios/bot-service-scenario-informational-bot.png)

下面是信息机器人的逻辑流：

1. 员工启动信息机器人。
2. Azure Active Directory 验证员工的身份。
3. 员工可以询问机器人支持哪种类型的查询。
4. 认知服务返回通过 QnA Maker 构建的常见问题解答机器人。
5. 员工定义有效的查询。
6. 机器人将查询提交到返回应用程序数据相关信息的 Azure 搜索。
7. Application Insights 收集运行时遥测来帮助提高机器人性能和使用率。

## <a name="sample-bot"></a>示例机器人
用 C# 编写的示例机器人在 Microsoft Azure 中运行，而 Microsoft Azure 使用 SQL 数据库实例的 Azure 搜索索引的数据。 机器人公开了一个问题列表，可提问关于如何使用认知服务 QnA Maker 解析问题（回答）的信息。 然后，机器人的用户可以键入一个查询，该查询通过 Azure 搜索在索引的数据库的广泛或特定区域中查找数据。 此示例提供了一个包含客户和订单信息的简单数据库。 Application Insights 跟踪机器人的使用情况，并帮你监控机器人是否出现异常。 机器人作为 Azure AD 应用发布，以便你可以限制谁有权访问该信息。

可以从[常见 Bot Framework 方案的示例](https://aka.ms/bot/scenarios)下载或克隆此示例机器人的源代码。

## <a name="components-youll-use"></a>将使用的组件
信息机器人使用以下组件：
-   用于身份验证的 Azure AD
-   认知服务：QnA Maker
-   Azure 搜索
-   Application Insights

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Azure Active Directory (Azure AD) 是 Microsoft 提供的基于多租户云的目录和标识管理服务。 作为机器人开发工具，Azure AD 让你可以快速轻松地与世界各地数百万组织使用的世界一流标识管理解决方案集成，从而专注于构建机器人。 通过定义 Azure AD 应用，你可以控制谁有权访问你的机器人及其公开的数据，而无需实现自己复杂的身份验证和授权系统。

### <a name="cognitive-services-qna-maker"></a>认知服务：QnA Maker
认知服务 QnA Maker 可帮助你提供常见问题数据源，而你的用户可以从你的机器人查询此数据源。 当处理存储在不同系统中的大量信息时，它可以帮助用户过滤掉信息源和信息集。 单个 SQL 数据库可以拥有大量信息，当应用自由格式搜索时，这会返回一大堆信息。 通过首先使用 QnA Maker，你可以为机器人用户定义路线图，以便他们知道如何提出可以通过 Azure 搜索检索到的智能问题。

### <a name="azure-search"></a>Azure 搜索
Azure 搜索是一项云搜索服务，面向可让你的搜索索引快速生效的应用。 在 Microsoft Azure 上运行，你可以根据使用需求轻松扩展和缩小。 你可以将搜索结果连接到业务目标，从而很好地控制隐藏在数据库中的搜索排名和表面数据。

### <a name="application-insights"></a>Application Insights
Application Insights 可帮助你通过应用程序性能管理 (APM) 和即时分析获取可付诸实施的见解。 开箱即可获得丰富的性能监视、功能强大的警报和易于使用的仪表板，帮助确保机器人可用且行为符合预期。 可以快速了解是否有问题，然后执行根本原因分析以便查找并解决问题。

## <a name="next-steps"></a>后续步骤
接下来，了解物联网机器人方案。

> [!div class="nextstepaction"]
> [物联网机器人方案](bot-service-scenario-internet-things.md)
