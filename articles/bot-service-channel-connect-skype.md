---
title: 将机器人连接到 Skype | Microsoft Docs
description: 了解如何通过 Skpye 接口配置机器人以进行访问。
keywords: Skype, 机器人通道, 配置 Skype, 发布, 连接到通道
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 2/1/2018
ms.openlocfilehash: 5dc4063125855113f813b8873b01df84c90e197e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297862"
---
# <a name="connect-a-bot-to-skype"></a>将机器人连接到 Skpye

Skype 通过即时消息、电话和视频通话让你与用户保持联系。 通过生成用户可通过 Skype 接口发现并与之进行互动的机器人来扩展此功能。

要添加 Skype 通道，请在 [Azure 门户](https://portal.azure.com/)中打开机器人，单击“通道”边栏选项卡，然后单击“Skype”。 这会转到“配置 Skype”设置页。 填写有关机器人的所有必要信息，然后单击“保存”以连接 Skype 通道。 接受“服务条款”，Skype 通道将添加到机器人中。

![添加 Skype 通道](~/media/channels/skype-addchannel.png)

## <a name="web-control"></a>Web 控件

为了将机器人嵌入到你的网站中，可通过单击“Web 控件”部分的“获取嵌入代码”按钮来获取代码。

## <a name="messaging"></a>消息传递

此部分配置机器人在 Skype 中发送和接收消息的方式。

## <a name="calling"></a>呼叫

此部分在机器人中配置 Skype 的呼叫功能。 可指定是否为机器人启用“呼叫”，如果启用，则可指定是否使用 IVR 功能或实时媒体功能。

## <a name="groups"></a>组

此部分配置机器人是否可添加到组以及它在组中传送消息的行为方式，此部分还可用于启用呼叫机器人的组视频呼叫。

## <a name="publish"></a>发布

此部分配置机器人的发布设置。 标有 * 的所有字段都是必填字段。

预览版中的机器人限 100 个联系人。 如果需要超过 100 个联系人，请提交机器人进行审核。 单击“提交以供审核”，如果接受，你的机器人会在 Skype 中自动成为可搜索的机器人。 如果申请未获批准，你将收到有关获得批准所需要执行的更改的通知。

## <a name="next-steps"></a>后续步骤

* [Skype for Business](bot-service-channel-connect-skypeforbusiness.md)
