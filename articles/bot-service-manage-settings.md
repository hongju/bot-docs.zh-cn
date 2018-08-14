---
title: 配置机器人设置 | Microsoft Docs
description: 了解如何使用 Azure 门户为机器人配置各种选项。
keywords: 配置机器人设置, 显示名称, 图标, Application Insights, 设置边栏选项卡
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 531e37f2186de2e315f11362dcefcc30a2ab6879
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297863"
---
# <a name="configure-bot-settings"></a>配置机器人设置

在“设置”边栏选项卡上可以查看和修改机器人设置，例如显示名称、图标和 Application Insights。

![机器人设置边栏选项卡](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

以下是“设置”边栏选项卡上的字段列表：

| 字段 | Description |
| :---  | :---        |
| 图标 | 用于直观地识别通道中的机器人的自定义图标，以及 Skype、Cortana 和其他服务的图标。 此图标必须为 PNG 格式，且大小不得超过 30K。 可随时更改此值。 |
| 显示名称 | 通道和目录中机器人的名称。 可随时更改此值。 35 个字符限制。 |
| 机器人句柄 | 机器人的唯一标识符。 使用机器人服务创建机器人后，无法更改此值。 |
| 消息传送终结点 | 与机器人通信的终结点。 |
| Microsoft 应用 ID | 机器人的唯一标识符。 不能更改此值。 可单击“管理”链接生成新密码。 |
| Application Insights 检测密钥 | 机器人遥测数据的唯一密钥。 如果要接收此机器人的机器人遥测数据，请将 Azure Application Insights 密钥复制到此字段。 此值是可选的。 在 Azure 门户中创建的机器人将为它们生成此密钥。 有关此字段的更多详细信息，请参阅 [Application Insights 密钥](~/bot-service-resources-app-insights-keys.md)。 |
| Application Insights API 密钥 | 机器人分析的唯一密钥。 如果要在仪表板中查看有关机器人的分析，请将 Azure Application Insights API 密钥复制到此字段。 此值是可选的。 有关此字段的更多详细信息，请参阅 [Application Insights 密钥](~/bot-service-resources-app-insights-keys.md)。 |
| Application Insights 应用程序 ID | 机器人分析的唯一密钥。 如果要在仪表板中查看有关机器人的分析，请将 Azure Insights 应用程序 ID 密钥复制到此字段。 此值是可选的。 在 Azure 门户中创建的机器人将为它们生成此密钥。 有关此字段的更多详细信息，请参阅 [Application Insights 密钥](~/bot-service-resources-app-insights-keys.md)。 |

> [!NOTE]
> 更改机器人的设置后，单击边栏选项卡顶部的“保存”按钮以保存新的机器人设置。

## <a name="next-steps"></a>后续步骤
既然已了解如何为机器人服务配置设置，请了解如何配置语音启动。
> [!div class="nextstepaction"]
> [语音启动](bot-service-manage-speech-priming.md)