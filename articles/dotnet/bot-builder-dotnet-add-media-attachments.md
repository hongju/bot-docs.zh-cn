---
title: 向消息添加媒体附件 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 向消息添加媒体附件。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8fd9b181676eee24b1e9c64c79663d0d0ac8abfa
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997314"
---
# <a name="add-media-attachments-to-messages"></a>向消息添加媒体附件

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

用户与机器人之间的消息交换可以包含媒体附件（例如图像、视频、音频、文件）。 <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> 对象的 `Attachments` 属性包含一组 <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a> 对象，表示消息中的媒体附件和富卡。 

> [!NOTE]
> [向消息添加富卡](bot-builder-dotnet-add-rich-card-attachments.md)。

## <a name="add-a-media-attachment"></a>添加媒体附件  

若要向消息添加媒体附件，请为 `message` 活动创建 `Attachment` 对象，并设置 `ContentType`、`ContentUrl` 和 `Name` 属性。 

[!code-csharp[Add media attachment](../includes/code/dotnet-add-attachments.cs#addMediaAttachment)]

如果附件是图像、音频或视频，连接器服务将以一种让[通道](bot-builder-dotnet-channeldata.md)可以在会话中呈现附件的方式将此附件数据传递给通道。 如果附件是文件，该文件 URL 将呈现为该会话中的超链接。

## <a name="additional-resources"></a>其他资源

- [使用通道检查器预览功能][inspector]
- [活动概述](bot-builder-dotnet-activities.md)
- [创建消息](bot-builder-dotnet-create-messages.md)
- [向消息添加富卡](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment 类</a>

[inspector]: ../bot-service-channel-inspector.md


