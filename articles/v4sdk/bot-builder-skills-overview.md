---
title: Bot Framework 技能概述 | Microsoft Docs
description: 详细了解 Bot Framework 技能
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8ed5841f9bfe874de26f1aecbb0e4460e541668b
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215289"
---
# <a name="virtual-assistant---skills-overview"></a>虚拟助手 - 技能概述

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

## <a name="overview"></a>概述

开发人员可以通过将可重用的聊天功能（称为技能）拼接到一起来构造聊天体验。

在企业中，这可能包括创建一个父机器人，将不同团队拥有的子机器人组合在一起，或者在更大的范围内利用其他开发人员提供的常见功能。 使用此预览版技能时，开发人员可以创建新的机器人（通常通过虚拟助理模板来进行），然后使用一个命令行操作来添加/删除技能，纳入所有调度和配置更改。     

技能本身是远程调用的机器人，可以使用技能开发人员模板（.NET、TS）来简化新技能的创建。

技能的一个主要设计目标是保留一致的活动协议，确保开发体验尽可能与任何常规 V4 SDK 机器人接近。 

![技能方案](./media/enterprise-template/skills-scenarios.png)

## <a name="bot-framework-skills"></a>Bot Framework 技能

目前我们提供以下 Bot Framework 技能，这些技能由 Microsoft Graph 提供支持，并以多种语言发布。

![技能方案](./media/enterprise-template/skills-at-build.png)

| 名称 | 说明 |
| ---- | ----------- |
|[日历技能](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-calendar.md)|将日历功能添加到助理。 由 Microsoft Graph 和 Google 提供支持。|
|[电子邮件技能](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-email.md)|将电子邮件功能添加到助理。 由 Microsoft Graph 和 Google 提供支持。|
|[待办事项技能](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-todo.md)|将任务管理功能添加到助理。 由 Microsoft Graph 提供支持。|
|[兴趣点技能](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-pointofinterest.md)|查找兴趣点和行车路线。 由 Azure Maps 和 FourSquare 提供支持。|
|[汽车技能](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/automotive.md)|行业垂直技能，用于展示如何启用汽车功能控件。|
|[试验性技能](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/experimental.md)|新闻、餐厅预订和天气。|

## <a name="getting-started"></a>入门

请参阅[入门](https://github.com/Microsoft/AI/tree/master/docs#tutorials)，了解如何利用现有的技能并构建自己的技能。
