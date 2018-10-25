---
title: 将机器人连接到 Skype | Microsoft Docs
description: 了解如何配置机器人，以便通过 Skype 接口进行访问。
keywords: Skype, 机器人通道, 配置 Skype, 发布, 连接到通道
author: v-ducvo
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/11/2018
ms.openlocfilehash: 58c8145d359d292dd33972a3dd59af997e7b8393
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999500"
---
# <a name="connect-a-bot-to-skype"></a>将机器人连接到 Skpye

Skype 通过即时消息、电话和视频通话让你与用户保持联系。 通过生成用户可通过 Skype 接口发现并与之进行互动的机器人来扩展此功能。

要添加 Skype 通道，请在 [Azure 门户](https://portal.azure.com/)中打开机器人，单击“通道”边栏选项卡，然后单击“Skype”。

![添加 Skype 通道](~/media/channels/skype-addchannel.png)

这会转到“配置 Skype”设置页。

![配置 Skype 通道](~/media/channels/skype_configure.png)

需在“Web 控件”、“消息”、“呼叫”、“组”和“发布”中配置设置。 让我们逐个进行配置。

## <a name="web-control"></a>Web 控件

若要将机器人嵌入到网站中，请单击“Web 控件”部分的“获取嵌入代码”按钮。 此时会转到“开发人员版 Skype”页。 按照该处的说明获取嵌入代码。

## <a name="messaging"></a>消息传递

此部分配置机器人在 Skype 中发送和接收消息的方式。

## <a name="calling"></a>呼叫

此部分在机器人中配置 Skype 的呼叫功能。 可指定是否为机器人启用“呼叫”，如果启用，则可指定是否使用 IVR 功能或实时媒体功能。

## <a name="groups"></a>组

此部分配置机器人是否可添加到组以及它在组中传送消息的行为方式，此部分还可用于启用呼叫机器人的组视频呼叫。

## <a name="publish"></a>发布

此部分配置机器人的发布设置。 标有 * 的所有字段都是必填字段。

预览版中的机器人限 100 个联系人。 如果需要超过 100 个联系人，请提交机器人进行审核。 单击“提交以供审核”，如果接受，你的机器人会在 Skype 中自动成为可搜索的机器人。 如果申请未获批准，你将收到有关获得批准所需要执行的更改的通知。

完成配置以后，请单击“保存”并接受“服务条款”。 Skype 通道现在已添加到机器人中。

## <a name="next-steps"></a>后续步骤

* [Skype for Business](bot-service-channel-connect-skypeforbusiness.md)
