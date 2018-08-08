---
title: 通过网上聊天测试机器人服务 | Microsoft Docs
description: 了解如何使用 Azure 门户中的网上聊天控件测试机器人服务。
keywords: 通过网上聊天执行测试, Azure 门户
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0c358f4e53f3fd64cce3635f644cc0f2f612e983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297055"
---
# <a name="test-in-web-chat"></a>通过网页聊天执行测试
机器人服务包括[网上聊天控件](bot-service-channel-connect-webchat.md)，可帮助你方便地测试机器人。 

## <a name="test-a-bot-in-the-azure-portal-with-web-chat"></a>通过网上聊天在 Azure 门户中测试机器人
登录到 [Azure 门户](https://portal.azure.com)，并打开机器人的边栏选项卡。 在“机器人管理”部分中，单击“通过网上聊天执行测试”。 机器人服务将加载网上聊天控件，并连接到机器人。

![“通过网页聊天执行测试”UI](~/media/test-in-webchat/test-in-webchat.png)

可以输入文本与机器人聊天。 如果机器人支持语音，你可以单击麦克风按钮与机器人交谈。 如果机器人支持附件，你可以上传附件，例如图像。 了解如何使用 [Bot Builder SDK](bot-builder-overview-getstarted.md) 向机器人添加功能。

> [!NOTE]
> 如果几分钟后未完全加载网上聊天控件，请尝试刷新页面。

## <a name="next-steps"></a>后续步骤
现在你已了解如何在 Azure 中测试机器人，了解如何利用 Bot Framework Emulator 更深入地测试和调试功能。

> [!div class="nextstepaction"]
> [Bot Framework Emulator](bot-service-debug-emulator.md)