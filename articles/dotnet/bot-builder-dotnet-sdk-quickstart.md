---
title: 通过 Bot Builder SDK for .NET 创建机器人 | Microsoft Docs
description: 通过功能强大的机器人构造框架 Bot Builder SDK for .NET 创建机器人。
keywords: Bot Builder SDK, 创建机器人, 快速入门, .NET, 入门, C# 机器人
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5f3a02783242697fccf267bef2d56ed453880c67
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707973"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>通过 Bot Builder SDK for .NET 创建机器人
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入门将指导你使用 C# 模板构建机器人，然后使用 Bot Framework Emulator 对其进行测试。 

## <a name="prerequisites"></a>先决条件
- Visual Studio [2017](https://www.visualstudio.com/downloads)
- 面向 [C#](https://botbuilder.myget.org/feed/aitemplates/package/vsix/BotBuilderV4.fbe0fc50-a6f1-4500-82a2-189314b7bea2) 的 Bot Builder SDK v4 模板
- Bot Framework [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index) 中的异步编程

## <a name="create-a-bot"></a>创建机器人
安装在先决条件部分中下载的 BotBuilderVSIX.vsix 模板。 

在 Visual Studio 中创建一个新的机器人项目。

![Visual Studio 项目](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 如果需要，请更新 [NuGet 包](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

得益于模板，项目包含本快速入门中创建机器人所需的所有代码。 实际上不需要编写任何其他代码。

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中启动机器人

单击运行按钮时，Visual Studio 将生成应用程序，将其部署到 localhost，然后启动 Web 浏览器以显示应用程序的 `default.htm` 页。 此时，机器人在本地运行。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。 
2. 选择位于创建 Visual Studio 解决方案的目录中的 .bot 文件。

## <a name="interact-with-your-bot"></a>与机器人交互

向机器人发送消息，机器人将回复消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [机器人工作原理](../v4sdk/bot-builder-basics.md) 
