---
title: 将机器人连接到 Teams | Microsoft Docs
description: 了解如何配置可供通过 Teams 访问的机器人。
keywords: Teams, 机器人通道, 配置 Teams
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
ms.openlocfilehash: d2609e4294416691e156ba3dbd09eabc8e0d3423
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/05/2019
ms.locfileid: "67592226"
---
# <a name="connect-a-bot-to-teams"></a>将机器人连接到 Teams

要添加 Microsoft Teams 通道，请在 [Azure 门户](https://portal.azure.com)中打开机器人，单击“通道”边栏选项卡，然后单击“Teams”   。

![添加 Teams 通道](media/teams/connect-teams-channel.png)

接下来，单击“保存”。 

![保存 Teams 通道](media/teams/save-teams-channel.png)

添加 Teams 通道后，请转到“通道”  页，然后单击“获取机器人嵌入代码”  。

![获取嵌入代码](media/teams/get-embed-code.png)

- 复制“获取机器人嵌入代码”对话框中显示的 _https_ 代码部分  。 例如，`https://teams.microsoft.com/l/chat/0/0?users=28:b8a22302e-9303-4e54-b348-343232` 。 

- 在浏览器中，粘贴此地址，然后选择用于将机器人添加到 Teams 的 Microsoft Teams 应用（客户端或 Web）。 应能够看到机器人已作为联系人列出，你可在 Microsoft Teams 中向其发送消息并接收其消息。 

## <a name="additional-information"></a>其他信息
有关 Microsoft Teams 特定信息，请参阅 Teams [文档](https://docs.microsoft.com/en-us/microsoftteams/platform/overview)。 
