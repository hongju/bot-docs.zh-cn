---
ms.openlocfilehash: 4af367b04f84d935936b5752cf9dbc863430105c
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464842"
---
## <a name="prerequisites"></a>先决条件

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- [Yeoman](http://yeoman.io/)：使用生成器自己创建机器人
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- 了解 [restify](http://restify.com/) 和 JavaScript 中的异步编程

> [!NOTE]
> 仅当使用 Windows 作为开发操作系统时，才需要安装下面列出的 Windows 生成工具。
> 对某些安装，restify 安装步骤是提供与 node-gyp 相关的错误。
> 如果存在这种情况，可以尝试使用提升的权限运行此命令。
> 如果系统上已安装 python，此调用也可能会挂起而不会退出：

> ```bash
> # only run this command if you are on Windows. Read the above note. 
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>创建机器人

创建机器人并初始化其包

1. 打开终端或权限提升的命令提示符。
1. 如果尚未为 JavaScript 机器人创建目录，请创建一个目录，并切换到该目录。 （我们将为你的 JavaScript 机器人创建一个通用的目录，不过，在本教程中，我们只会创建一个目录。）

   ```bash
   mkdir myJsBots
   cd myJsBots
   ```

1. 确保 npm 版本为最新。

   ```bash
   npm install -g npm
   ```

1. 接下来，安装 Yeoman 和 JavaScript 生成器。

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. 然后，使用生成器创建 echo 机器人。

   ```bash
   yo botbuilder
   ```

Yeoman 会提示你输入用来创建机器人的一些信息。 对于本教程，请使用默认值。

- 为机器人输入名称。 (my-chat-bot)
- 输入说明。 （演示 Microsoft Bot Framework 的核心功能）
- 选择机器人的语言。 (JavaScript)
- 选择要使用的模板。 （回显机器人 - https://aka.ms/bot-template-echo)

得益于模板，项目包含本快速入门中创建机器人所需的所有代码。 实际上不需要编写任何其他代码。

> [!NOTE]
> 如果选择创建 `Core` 机器人，则需要 LUIS 语言模型。 可以在 [luis.ai](https://www.luis.ai) 上创建一个。 创建模型以后，请更新配置文件。

## <a name="start-your-bot"></a>启动机器人

在终端或命令提示符中，将目录切换到为机器人创建的目录，并使用 `npm start` 启动机器人。 此时，机器人在本地运行。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

1. 启动 Bot Framework Emulator。
2. 单击模拟器“欢迎使用”选项卡中的“新建机器人配置”链接  。 
3. 为机器人填写字段。 使用机器人的欢迎页地址（通常为 http://localhost:3978) ，并将路由信息“/ api/messages”附加到此地址。
4. 然后，单击“保存并连接”  。

向机器人发送消息，机器人将回复消息。
![模拟器运行](../media/emulator-v4/js-quickstart.png)
