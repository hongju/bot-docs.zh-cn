---
title: Bot Builder SDK for .NET | Microsoft Docs
description: 开始使用 Bot Builder SDK for .NET，这是一种强大的、易于使用的用于构造机器人的框架。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 801925e2c179392804d9707e62bfaef082c8e81b
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996695"
---
# <a name="bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Builder SDK for .NET 是一个强大的用于构造机器人的框架，可以处理自由形式的交互和更多的引导式聊天，用户可以从中选择可能的值。 它易于使用并利用 C# 为 .NET 开发人员提供了一种熟悉的方法来编写机器人。

通过 SDK，可以构建利用以下 SDK 功能的机器人： 

- 功能强大的对话系统，具有独立且可组合的对话
- 内容简单（例如是/否、字符串、数字和枚举）的内置提示
- 内置对话，利用 <a href="http://luis.ai" target="_blank">LUIS</a> 等功能强大的 AI 框架
- 用于自动生成机器人（来自 C# 类）的 FormFlow，引导用户完成聊天，根据需要提供帮助、导航、说明和确认

> [!IMPORTANT]
> 2017 年 7 月 31 日，已在 Bot Framework 安全协议中实现中断性变更。 要防止这些更改对机器人产生负面影响，必须确保应用程序使用的是 Bot Builder SDK v3.5 或更高版本。 如果使用 2017 年 1 月 5 日（Bot Builder SDK v3.5 的发布日期）之前获得的 SDK 构建了机器人，请确保升级到 Bot Builder SDK v3.5 或更高版本。

## <a name="get-the-sdk"></a>获取 SDK

SDK 可作为 NuGet 包和 <a href="https://github.com/Microsoft/BotBuilder" target="_blank">GitHub</a> 上的开放源代码提供。

> [!IMPORTANT]
> Bot Builder SDK for .NET 需要 .NET Framework 4.6 或更高版本。 如果要将 SDK 添加到面向较低版本 .NET Framework 的现有项目，则需要先将项目更新为面向 .NET Framework 4.6。

若要在 Visual Studio 项目中安装 SDK，请完成以下步骤：

1. 在“解决方案资源管理器”中，右键单击项目名称，然后选择“管理 NuGet 包...”。
2. 在“浏览”选项卡上的搜索框中键入“Microsoft.Bot.Builder”。
3. 在结果列表中选择“Microsoft.Bot.Builder”，单击“安装”，然后接受更改。

## <a name="get-code-samples"></a>获取代码示例

此 SDK 包括的[示例源代码](bot-builder-dotnet-samples.md)使用 Bot Builder SDK for .NET 的功能。

## <a name="next-steps"></a>后续步骤

可查看本部分中的文章，了解有关使用 Bot Builder SDK for .NET 构建机器人的详细信息，请从以下内容开始：

- [快速入门](bot-builder-dotnet-quickstart.md)：按照此分步教程中的指示，快速构建并测试简单的机器人。
- [关键概念](bot-builder-dotnet-concepts.md)：了解 Bot Builder SDK for .NET 中的关键概念。

如果遇到问题或对 Bot Builder SDK for .NET 有任何建议，请参阅[客户支持](../bot-service-resources-links-help.md)获取可用资源列表。 
