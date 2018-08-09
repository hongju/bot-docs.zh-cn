---
title: 开始使用 Bot Builder 生成机器人 | Microsoft Docs
description: 开始使用 Bot Builder 生成功能强大的机器人。
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21186c5d3b0769311e4703ca1dab2f48a0a0081a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574883"
---
# <a name="develop-bots-with-bot-builder"></a>使用 Bot Builder 开发机器人

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Builder 提供 SDK、库、示例和工具来帮助你生成和调试机器人。 使用机器人服务生成机器人时，机器人由 Bot Builder SDK 提供支持。 还可以使用 Bot Builder SDK 通过 C# 或 Node.js 从头开始创建机器人。 Bot Builder 包括用于测试机器人的 Bot Framework Emulator 和用于预览机器人在不同通道上的用户体验的 Channel Inspector。

如果要使用所需的任何编程语言生成机器人，可以使用 REST API。

## <a name="bot-builder-sdk-for-net"></a>用于 .NET 的 Bot Builder SDK
用于 .NET 的 Bot Builder SDK 利用 C# 为 .NET 开发人员提供了一种熟悉的方法来编写机器人。 它是一个强大的用于构造机器人的框架，可以处理自由形式的交互和更多的引导式聊天，用户可以从中选择可能的值。 

[.NET 快速入门](~/dotnet/bot-builder-dotnet-quickstart.md)将指导你使用用于 .NET 的 Bot Builder SDK 创建机器人。

用于 .NET 的 Bot Builder SDK 以 [NuGet 包](https://www.nuget.org/packages/Microsoft.Bot.Builder/)的形式提供。

> [!IMPORTANT]
> 用于 .NET 的 Bot Builder SDK 需要 .NET Framework 4.6 或更高版本。 如果要将 SDK 添加到面向较低版本 .NET Framework 的现有项目，则需要先将项目更新为面向 .NET Framework 4.6。

若要在现有 Visual Studio 项目中安装 SDK，请完成以下步骤：

1. 在“解决方案资源管理器”中，右键单击项目名称，然后选择“管理 NuGet 包...”。
2. 在“浏览”选项卡上的搜索框中键入“Microsoft.Bot.Builder”。
3. 在结果列表中选择“Microsoft.Bot.Builder”，单击“安装”，然后接受更改。

还可以从 GitHub 下载用于 .NET 的 Bot Builder SDK [源代码](https://github.com/Microsoft/BotBuilder/tree/master/CSharp)。

### <a name="visual-studio-project-templates"></a>Visual Studio 项目模板
下载 Visual Studio 项目模板以加速机器人开发。

* [用于 Visual Studio 的机器人模板][bot-template]，用于使用 C# 开发机器人
* [用于 Visual Studio 的 Cortana 技能模板][cortana-template]，用于使用 C# 开发 Cortana 技能

> [!TIP]
> <a href="/visualstudio/ide/how-to-locate-and-organize-project-and-item-templates" target="_blank">详细了解</a>如何安装 Visual Studio 2017 项目模板。

## <a name="bot-builder-sdk-for-nodejs"></a>用于 Node.js 的 Bot Builder SDK
用于 Node.js 的 Bot Builder SDK 为 Node.js 开发人员提供了一种熟悉的方法来编写机器人。 可以使用它来构建各种聊天用户界面，从简单的提示到自由格式的聊天。 用于 Node.js 的 Bot Builder SDK 使用 Restify（一种用于构建 Web 服务的流行框架）来创建机器人的 Web 服务器。 此 SDK 也与 Express 兼容。 

[Node.js 快速入门](~/nodejs/bot-builder-nodejs-quickstart.md)将指导你使用用于 Node.js 的 Bot Builder SDK 创建机器人。 

用于 Node.js 的 Bot Builder SDK 以 npm 包的形式提供。 若要安装用于 Node.js 的 Bot Builder SDK 及其依赖项，首先要为机器人创建一个文件夹，导航到该文件夹，然后运行以下 **npm** 命令：

```nodejs
npm init
```

接下来，通过运行以下 **npm** 命令，安装用于 Node.js 的 Bot Builder SDK 和 <a href="http://restify.com/" target="_blank">Restify</a> 模块：

```nodejs
npm install --save botbuilder
npm install --save restify
```

还可以从 GitHub 下载用于 Node.js 的 Bot Builder SDK [源代码](https://github.com/Microsoft/BotBuilder/tree/master/Node)。

## <a name="rest-api"></a>REST API

可以使用 Bot Framework REST API 通过任何编程语言创建机器人。 [REST 快速入门](rest-api/bot-framework-rest-connector-quickstart.md)将指导你使用 REST 创建机器人。

Bot Framework 中有三个 REST API：

 - [Bot Connector REST API][connectorAPI] 使机器人能够向/从 [Bot Framework 门户](https://dev.botframework.com/)中配置的通道发送/接收消息。 

- [Bot State REST API][stateAPI] 使机器人能够存储和检索与通过 Bot Connector REST API 进行的聊天关联的状态。

- [Direct Line REST API][directLineAPI] 使你可以将自己的应用程序（例如客户端应用程序、网上聊天控件或移动应用）直接连接到单个机器人。

## <a name="bot-framework-emulator"></a>Bot Framework Emulator
Bot Framework Emulator 是一个桌面应用程序，可用于在本地或远程测试和调试机器人。 使用模拟器，你可以与机器人聊天并检查机器人发送和接收的消息。 [详细了解 Bot Framework Emulator](~/bot-service-debug-emulator.md) 并针对平台[下载模拟器](http://emulator.botframework.com)。

## <a name="bot-framework-channel-inspector"></a>Bot Framework Channel Inspector
使用 Bot Framework [Channel Inspector](bot-service-channel-inspector.md) 快速了解不同通道上的机器人功能是什么样子。

[bot-template]: http://aka.ms/bf-bc-vstemplate
[cortana-template]: https://aka.ms/bf-cortanaskill-template


[connectorAPI]: https://docs.botframework.com/en-us/restapi/connector/#navtitle
 
[stateAPI]: https://docs.botframework.com/en-us/restapi/state/#navtitle

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle
