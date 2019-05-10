---
title: 将机器人连接到 GroupMe | Microsoft Docs
description: 了解如何配置机器人与 GroupMe 的连接。
keywords: 机器人通道, GroupMe, 创建 GroupMe, 凭据
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a2004293ff10cfbc7132f58b7c0c834a2012cfd1
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563580"
---
# <a name="connect-a-bot-to-groupme"></a>将机器人连接到 GroupMe

可以对机器人进行配置，以与使用 GroupMe 群组消息传递应用的用户进行通信。

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>注册 GroupMe 帐户

若没有 GroupMe 帐户，请[注册新帐户](https://web.groupme.com/signup)。

## <a name="create-a-groupme-application"></a>创建 GroupMe 应用程序

为机器人[创建 GroupMe 应用程序](https://dev.groupme.com/applications/new)。

使用此回调 URL：`https://groupme.botframework.com/Home/Login`

![创建应用](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>收集凭据

1. 在“重定向 URL”字段中，复制“client_id=”后的值。
2. 复制“访问令牌”值。

![复制客户端 ID 和访问令牌](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>提交凭据

1. 在 dev.botframework.com 上，粘贴刚复制到“客户端 ID”字段中的“client_id”值。
2. 将“访问令牌”值粘贴到“访问令牌”字段。
2. 单击“ **保存**”。

![输入凭据](~/media/channels/GM-StepClientIDToken.png)
