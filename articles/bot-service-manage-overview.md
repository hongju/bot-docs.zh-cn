---
title: 管理机器人 | Microsoft Docs
description: 了解如何通过机器人服务门户管理机器人。
keywords: Azure 门户, 机器人管理, 通过网上聊天执行测试, MicrosoftAppID, MicrosoftAppPassword, 应用程序设置
author: v-ducvo
ms.author: rstand
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: e68db6c1fe9d3d136a8643652df034fb6df2858f
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645597"
---
# <a name="manage-a-bot"></a>管理机器人

本主题介绍如何使用 Azure 门户管理机器人。

## <a name="bot-settings-overview"></a>机器人设置概述

![机器人设置概述](~/media/azure-manage-a-bot/overview.png)

在“概述”边栏选项卡中，可以找到有关机器人的高级别信息。 例如，可以看到机器人的订阅 ID、定价层和消息终结点。

## <a name="bot-management"></a>机器人管理

 可以在“机器人管理”部分下发现大部分机器人的管理选项。 下面的一系列选项可帮助你管理机器人。

![机器人管理](~/media/azure-manage-a-bot/bot-management.png)

| 选项 |  说明 |
| ---- | ---- |
| **生成** | “生成”选项卡提供了对机器人进行更改的选项。 此选项不适用于仅注册机器人。 |
| **通过网上聊天执行测试** | 使用集成的网上聊天控件，帮助你快速测试机器人。 |
| **分析** | 如果为机器人启用分析，可以查看 Application Insights 为机器人收集的分析数据。 |
| **通道** | 配置机器人用来与用户进行通信的通道。 |
| **设置** | 管理各种机器人配置文件设置，例如显示名称、分析和消息终结点。 |
| **语音启动** | 管理 LUIS 应用和必应语音服务之间的连接。 |
| **机器人服务定价** | 管理机器人服务的定价层。 |

## <a name="app-service-settings"></a>应用服务设置

![应用服务设置](~/media/azure-manage-a-bot/app-service-settings.png)

“应用程序设置”边栏选项卡包含有关机器人的详细信息，如机器人的环境、ID、Application Insights 密钥、Microsoft 应用 ID 和 Microsoft 应用密码。

### <a name="microsoftappid-and-microsoftapppassword"></a>MicrosoftAppID 和 MicrosoftAppPassword

可以在“设置”边栏选项卡中找到你的机器人的 **MicrosoftAppID**。 只有在首次创建你的机器人时，才会显示机器人的 **MicrosoftAppPassword**。

![Microsoft 应用 ID 和密码](~/media/azure-manage-a-bot/app-settings.png)

> [!NOTE]
> “机器人通道注册”机器人服务附带 MicrosoftAppID，但因为没有与此类型服务相关联的应用服务，因此没有可在其中查找 MicrosoftAppPassword 的“应用程序设置”边栏选项卡。 若要获取密码，必须生成一个。 若要生成“机器人通道注册”密码，请参阅[机器人通道注册密码](bot-service-quickstart-registration.md#bot-channels-registration-password)

## <a name="next-steps"></a>后续步骤
现在你已经在 Azure 门户中探索了“机器人服务”边栏选项卡，接下来学习如何使用联机代码编辑器来自定义机器人。
> [!div class="nextstepaction"]
> [使用联机代码编辑器](bot-service-build-online-code-editor.md)
