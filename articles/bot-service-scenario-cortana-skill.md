---
title: Cortana 技能机器人方案 | Microsoft Docs
description: 使用 Bot Framework 探索 Cortana 技能机器人方案。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 049dffd2adc700323bec943e090d369a14ff696b
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574843"
---
# <a name="cortana-skills-bot-scenario"></a>Cortana 技能机器人方案

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Cortana 技能机器人扩展了 Cortana，可以轻松地在日历的上下文中使用语音来进行汽车维护预约。

Cortana 是个人助理。 使用你的声音和自定义 Cortana 技能机器人的自然界面，可以让 Cortana 与组织（如汽车商店）交谈，以帮助你预约。 该服务可以提供服务、可用时间和持续时间的列表。 Cortana 可以查看你的日历以了解你在冲突的时间是否有其他安排，如果没有，则创建预约并将其添加到你的日历中。

![Cortana 技能机器人关系图](~/media/scenarios/bot-service-scenario-cortana-skill.png)

下面是 Cortana 技能机器人针对汽车商店的逻辑流程：

1. 用户从其电脑或移动设备访问 Cortana。
2. 用户使用文本或语音命令，要求进行汽车维护预约。
3. 由于机器人与 Cortana 集成，因此它可以访问用户的日历并将逻辑应用于请求。
4. 使用该信息，机器人可以查询汽车服务以获得有效的预约。
5. 根据提供的上下文选项，用户可以进行预约。
6. Application Insights 收集运行时遥测来帮助提高机器人性能和使用率。

## <a name="sample-bot"></a>示例机器人
使用 Cortana 技能机器人，它完全取决于个人背景。 使用 Cortana，你可以根据自己的位置使用自己的声音要求“Bob 的汽车维护”来为你的汽车提供服务。 使用通过 Cortana 公开的个人信息，机器人可以根据用户与机器人交谈时的位置来确认位置。

可以从[常见 Bot Framework 方案的示例](https://aka.ms/bot/scenarios)下载或克隆此示例机器人的源代码。

## <a name="components-youll-use"></a>将使用的组件
Cortana 机器人使用以下组件：
-   Cortana
-   Application Insights

### <a name="cortana"></a>Cortana
现在，可以通过创建 Cortana 技能为机器人添加支持。 可以使用 Cortana 技能工具包为 Cortana 构建新功能（称为“技能”）。 技能是一种允许 Cortana 做更多事情的构造。 你可以构建与机器人集成的技能，使 Cortana 能够完成任务并做好事情。 在调用过程中，Cortana 可以（征得用户同意后）在运行时将有关用户的信息传递给技能，以便技能可以相应地自定义其体验。 Cortana 的上下文知识使机器人很有用，甚至可能更聪明。 某些类型的技能一经调用可以操纵 Cortana 的界面以在技能和最终用户之间进行聊天。 技能发布后，用户可以在 Windows 10 周年更新+（桌面和移动版）、iOS 和 Android 的 Cortana 上查看和使用你的技能。

### <a name="application-insights"></a>Application Insights
Application Insights 可帮助你通过应用程序性能管理 (APM) 和即时分析获取可付诸实施的见解。 开箱即可获得丰富的性能监视、功能强大的警报和易于使用的仪表板，帮助确保机器人可用且行为符合预期。 你可以快速了解是否有问题，然后执行根本原因分析以便查找并解决问题。

## <a name="next-steps"></a>后续步骤
接下来，了解企业效率机器人方案。

> [!div class="nextstepaction"]
> [企业效率机器人方案](bot-service-scenario-enterprise-productivity.md)
