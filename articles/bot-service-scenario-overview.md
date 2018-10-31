---
title: 机器人服务方案概述 |Microsoft Docs
description: 了解使用机器人服务成功生成的功能强大机器人的主要方案。
author: BrianRandell
ms.author: v-brra
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e195f83eefd5f162b74f8891f3b174efc8934700
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997959"
---
# <a name="bot-scenarios"></a>机器人方案

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

关于使用机器人服务成功生成的功能强大的机器人，本主题介绍了几种主要方案。

可以从[常见 Bot Framework 方案示例](https://aka.ms/bot/scenarios)中下载或克隆所有方案机器人示例的源代码。

## <a name="commerce-bot-scenario"></a>商业机器人方案
[商业机器人](bot-service-scenario-commerce.md)方案描述的机器人使用酒店的礼宾服务代替人们通常进行的传统电子邮件和电话交互。 机器人利用认知服务，从后端服务集成收集上下文，通过文本和语音更好地处理客户请求。

在商业机器人方案中，客户可以向酒店请求礼宾服务。 她通过 Azure Active Directory v2 身份验证终结点进行身份验证。 机器人可以查看客户的预订情况并提供不同的服务选项。 例如，客户要预订挨着泳池的小屋。 机器人使用语言理解智能服务 (LUIS) 来分析请求，然后引导用户完成预订小屋的流程。

## <a name="cortana-skill-bot-scenario"></a>Cortana 技能机器人方案
[Cortana 技能机器人](bot-service-scenario-cortana-skill.md)方案利用 Cortana。 利用语音自然界面和自定义 Cortana 技能机器人，可以让 Cortana 与组织（如移动式汽车装饰设计公司）交谈，根据你呼叫时所在的位置帮助进行预约。 机器人可提供服务列表、可用时间和持续时间。

## <a name="enterprise-productivity-bot-scenario"></a>企业效率机器人方案
[企业效率机器人](bot-service-scenario-enterprise-productivity.md)方案介绍如何将机器人与 Office 365 日历和其他服务集成，从而提高工作效率。

将机器人与 Office 365 集成可以更快、更轻松地与其他人创建会议请求。 在此过程中，可以访问 Dynamics CRM 等其他服务。 此示例提供了通过 Azure Active Directory 进行身份验证从而与 Office 365 集成所需的代码。 它为外部服务提供模拟入口点，作为读者的练习。

## <a name="information-bot-scenario"></a>信息机器人方案
此[信息机器人](bot-service-scenario-informational.md)可使用认知服务 QnA Maker 回答知识集中定义的问题或常见问题，并能使用 Azure 搜索回答更加开放的问题。

通常，通过搜索可以轻松找到深藏在 SQL Server 等结构化数据存储中的信息。 假设可通过简单的聊天式命令查找客户的订单状态。 使用认知服务 QnA Maker，向用户呈现一组有效的搜索选项，例如查找客户、查看客户的最新订单等。通过定义 QnA 格式，用户可以轻松提出 Azure 搜索（可查找存储在 SQL 数据库中的数据）支持的问题。

## <a name="iot-bot-scenario"></a>IoT 机器人方案
此[物联网 (IoT) 机器人](bot-service-scenario-internet-things.md)可让你轻松控制家里的设备，例如使用交互式聊天命令控制 Philips Hue 灯。

这个简单的机器人可以与免费的 If This Then That (IFTTT) 服务结合使用，从而控制 Philips Hue 灯。 Philips Hue 是一个 IoT 设备，可通过其公开的 API 在本地进行控制。 但是，此 API 不支持从本地网络外部进行一般性访问。 不过，IFTTT 是“[Hue 的搭档](http://www2.meethue.com/en-us/friends-of-hue/ifttt/)”，因此公开了许多可以发出的控制命令，例如开灯和关灯、改变灯光颜色或光线强度。

## <a name="next-steps"></a>后续步骤
现已大概了解这些方案，可深入研究各个方案。

> [!div class="nextstepaction"]
> [商业机器人](bot-service-scenario-commerce.md)
