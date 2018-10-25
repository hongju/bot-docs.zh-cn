---
title: 使用 Bot Builder SDK for JavaScript 创建机器人 | Microsoft Docs
description: 使用 Bot Builder SDK for JavaScript 快速创建机器人。
keywords: 快速入门, bot builder sdk, 入门
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aa13889cea2a26bf094a919f5d05905d65f7661f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998846"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>通过 Bot Builder SDK for JavaScript 创建机器人

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

本快速入门将介绍如何使用 Yeoman Bot Builder 生成器和 Bot Builder SDK for JavaScript 构建机器人，然后通过 Bot Framework 模拟器进行测试。 

## <a name="prerequisites"></a>先决条件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/)，可以使用生成器为你创建机器人
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator)
- 了解 [restify](http://restify.com/) 和 JavaScript 中的异步编程

> [!NOTE]
> 对某些安装，restify 安装步骤是提供与 node-gyp 相关的错误。
> 如果是这种情况，则尝试运行 `npm install -g windows-build-tools`。

## <a name="create-a-bot"></a>创建机器人

打开提升的命令提示符，创建一个目录，并初始化机器人相应的包。

```bash
md myJsBots
cd myJsBots
```

确保 npm 版本为最新。
```bash
npm i npm
```

接下来，安装 Yeoman 和 JavaScript 生成器。

```bash
npm install -g yo
npm install -g generator-botbuilder
```

然后，使用生成器创建 echo 机器人。

```bash
yo botbuilder
```

Yeoman 会提示你输入用来创建机器人的一些信息。

- 为机器人输入名称。
- 输入说明。
- 选择机器人语言，`JavaScript` 或 `TypeScript`。
- 选择 `Echo` 模板。

得益于模板，项目包含本快速入门中创建机器人所需的所有代码。 实际上不需要编写任何其他代码。

> [!NOTE]
> 对于基础机器人，需要一个 LUIS 语言模型。 可以在 [luis.ai](https://www.luis.ai) 上创建一个。 创建模型以后，请更新 .bot 文件。 你的 bot 文件看起来应该与此[文件](../v4sdk/bot-builder-service-file.md)类似。 

## <a name="start-your-bot"></a>启动机器人

在 Powershell/Bash 中，将目录更改为为机器人创建的目录，然后通过 `npm start` 启动它。 此时，机器人在本地运行。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
1. 启动模拟器。
2. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。
3. 选择位于创建项目的目录中的 .bot 文件。

向机器人发送消息，机器人将回复消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [机器人工作原理](../v4sdk/bot-builder-basics.md) 
