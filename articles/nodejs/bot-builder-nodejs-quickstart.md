---
title: 使用 Bot Builder SDK for Node.js 创建机器人 | Microsoft Docs
description: 使用功能强大的机器人构造框架 Bot Builder SDK for Node.js 创建机器人。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6159997ec5ea3dbd3188ba2ea4b6207b5d9db08f
ms.sourcegitcommit: 97bb24f15041caccef4ca5736aa14f144881e0c6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2018
ms.locfileid: "39567546"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-nodejs"></a>使用 Bot Builder SDK for Node.js 创建机器人

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [机器人服务](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Bot Builder SDK for Node.js 是一个用于开发机器人的框架。 它易于使用，并模拟 Express 和 Restify 等框架，为 JavaScript 开发人员提供熟悉的方法来编写机器人。

本教程将逐步指导你使用 Bot Builder SDK for Node.js 来构建机器人。 可以在控制台窗口中使用 Bot Framework Emulator 测试机器人。

## <a name="prerequisites"></a>先决条件
通过完成以下先决任务开始：

1. 安装 [Node.js](https://nodejs.org)。
2. 为机器人创建一个文件夹。
3. 从命令提示符或终端导航至刚刚创建的文件夹。
4. 运行以下 npm 命令：

```nodejs
npm init
```

按照屏幕上的提示输入有关机器人的信息，npm 将创建一个包含你提供的信息的 package.json 文件。 

## <a name="install-the-sdk"></a>安装 SDK
接下来，通过运行以下 npm 命令，安装 Bot Builder SDK for Node.js：

```nodejs
npm install --save botbuilder
```

安装 SDK 后，你就可以编写你的第一个机器人。

对于第一个机器人，你将创建一个简单回显任何用户输入的机器人。 若要创建机器人，请按以下步骤操作：

1. 在之前为机器人创建的文件夹中，创建一个名为 app.js 的新文件。
2. 在文本编辑器或所选择的 IDE 中打开 app.js。 将以下代码添加到该文件： 

   [!code-javascript[consolebot code sample Node.js](../includes/code/node-getstarted.js#consolebot)]

3. 保存文件。 现在，就可以运行和测试机器人了。

### <a name="start-your-bot"></a>启动机器人

在控制台窗口中导航到机器人的目录并启动机器人：

```nodejs
node app.js
```

机器人现在在本地运行。 通过在控制台窗口中键入一些消息来测试机器人。
你应该看到机器人通过以带有“您说：”前缀的消息回显你的消息来响应你发送的每条消息。

## <a name="install-restify"></a>安装 Restify

控制台机器人是很好的基于文本的客户端，但是为了使用任何 Bot Framework 通道（或在模拟器中运行机器人），机器人将需要在 API 终结点上运行。 通过运行以下 npm 命令安装 <a href="http://restify.com/" target="_blank">restify</a>：

```nodejs
npm install --save restify
```

安装 Restify 后，就可以对机器人进行一些更改了。

## <a name="edit-your-bot"></a>编辑机器人

需要对 app.js 文件进行一些更改。 

1. 添加一行以要求 `restify` 模块。
2. 将 `ConsoleConnector` 更改为 `ChatConnector`。
3. 包括你的 Microsoft 应用 ID 和应用密码。
4. 让连接器侦听 API 终结点。

   [!code-javascript[echobot code sample Node.js](../includes/code/node-getstarted.js#echobot)]

5. 保存文件。 现在，就可以在模拟器中运行和测试机器人了。

> [!NOTE] 
> 不需要提供 Microsoft 应用 ID 或 Microsoft 应用密码即可在 Bot Framework Emulator 中运行机器人。

## <a name="test-your-bot"></a>测试机器人
接下来，使用 [Bot Framework Emulator](../bot-service-debug-emulator.md) 测试机器人以查看它的实际运行情况。 模拟器是一个桌面应用程序，允许你在 localhost 上测试和调试机器人或通过隧道远程运行。

首先，需要[下载](https://emulator.botframework.com)并安装模拟器。 下载完成后，启动可执行文件并完成安装过程。

### <a name="start-your-bot"></a>启动机器人

安装模拟器后，在控制台窗口中导航到机器人的目录并启动机器人：

```nodejs
node app.js
```
   
机器人现在在本地运行。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
启动机器人后，启动模拟器，然后连接机器人：

1. 单击模拟器窗口中的“新建机器人配置”链接。 

2. 在地址栏中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。

3. 单击“保存并连接”。 无需指定 Microsoft 应用 ID 和 Microsoft 应用密码。 可以暂时将这些字段留空。 以后当你注册机器人时，将获得此信息。

### <a name="try-out-your-bot"></a>试用机器人

现在机器人在本地运行并连接到模拟器，通过在模拟器中键入一些消息来测试机器人。
你应该看到机器人通过以带有“您说：”前缀的消息回显你的消息来响应你发送的每条消息。

你已经使用 Bot Builder SDK for Node.js 成功创建了第一个机器人！

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [Bot Builder SDK for Node.js](bot-builder-nodejs-overview.md)
