---
title: 物联网机器人方案 | Microsoft Docs
description: 使用 Bot Framework 探索物联网机器人方案。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3b65f323427760fa43586f471aefb6811ef3e675
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574514"
---
# <a name="internet-of-things-iot-bot-scenario"></a>物联网 (IoT) 机器人方案

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

这款物联网 (IoT) 机器人可让你轻松控制家中的设备，例如使用语音或交互式聊天命令的 Philips Hue 灯。

人们喜欢谈论自己的事情。 自从第一个电视遥控器问世以来，人们喜欢不必移动即可影响环境。 该款 IoT 机器人允许用户通过简单的聊天命令或语音管理 Philips Hue。 此外，使用聊天时，可为用户提供与颜色相关的视觉选项。

![物联网机器人关系图](~/media/scenarios/bot-service-scenario-iot-bot.png)

下面是 IoT 机器人的逻辑流：

1. 用户登录到 Skype 并访问 IoT 机器人。
2. 用户使用语音让机器人通过 IoT 设备打开灯光。
3. 请求转至有权访问 IoT 设备网络的第三方服务。
4. 命令的结果将返回给用户。
5. Application Insights 收集运行时遥测来帮助提高机器人性能和使用率。

## <a name="sample-bot"></a>示例机器人
IoT 机器人将允许你快速使用来自 Skype 或 Slack 等通道的聊天命令来控制 Hue。 为便于远程访问，将调用预定义为配合 Hue 使用的 IFTTT 小程序。

可以从[常见 Bot Framework 方案的示例](https://aka.ms/bot/scenarios)下载或克隆此示例机器人的源代码。

## <a name="components-youll-use"></a>将使用的组件
物联网 (IoT) 机器人使用以下组件：
-   Philips Hue
-   如果…那么… (IFTTT)
-   Application Insights

### <a name="philips-hue"></a>Philips Hue
连接了 Philips Hue 的灯泡和电桥让你可以完全控制照明。 无论想对照明做些什么，Hue 都可满足你。 Hue 有一个可从本地网络使用的 API。 然而，你希望能使用友好的机器人接口从任何位置访问控制 Hue 的设备和灯。 因此，将通过 IFTTT 访问 Hue。

### <a name="ifttt"></a>IFTTT
IFTTT 是基于 Web 的免费服务，人们使用它来创建简单条件语句链，称为小程序。 可从机器人触发小程序，让它代表你执行某些操作。 有许多预定义的 Hue 小程序可用于开灯、关灯、更改场景等。

### <a name="application-insights"></a>Application Insights
Application Insights 可帮助你通过应用程序性能管理 (APM) 和即时分析获取可付诸实施的见解。 开箱即可获得丰富的性能监视、功能强大的警报和易于使用的仪表板，帮助确保机器人可用且行为符合预期。 你可以快速了解是否有问题，然后执行根本原因分析以便查找并解决问题。
