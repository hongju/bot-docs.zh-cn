---
ms.openlocfilehash: 0570ec6a44c9fe1b007c1fd1b8c335288baa63cb
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464848"
---
## <a name="prerequisites"></a>先决条件
- [Visual Studio 2017 或更高版本](https://www.visualstudio.com/downloads)
- [面向 C# 的 Bot Framework SDK v4 模板](https://aka.ms/bot-vsix)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)
- 了解 [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) 和 [C# 中的异步编程](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/index)

## <a name="create-a-bot"></a>创建机器人
安装在先决条件部分中下载的 BotBuilderVSIX.vsix 模板。

在 Visual Studio 中使用 **Echo 机器人 (Bot Framework v4)** 模板创建一个新的机器人项目。

![Visual Studio 项目](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> 如果需要，请将项目生成类型更改为 ``.Net Core 2.1``。 另外，如果需要，请更新 `Microsoft.Bot.Builder` [NuGet 程序包](https://docs.microsoft.com/nuget/quickstart/install-and-use-a-package-in-visual-studio)。

得益于该模板，项目包含在此快速入门中创建机器人所需的所有代码。 实际上不需要编写任何其他代码。

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中启动机器人

单击运行按钮时，Visual Studio 将生成应用程序，将其部署到 localhost，然后启动 Web 浏览器以显示应用程序的 `default.htm` 页。 此时，机器人在本地运行。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎使用”选项卡中的“新建机器人配置”链接  。 
2. 为机器人填写字段。 使用机器人主页的地址（通常为 http://localhost:3978) ，并将路由信息“/ api/messages”追加到此地址。
3. 然后，单击“保存并连接”  。

## <a name="interact-with-your-bot"></a>与机器人交互

向机器人发送消息，机器人将回复消息。

![正在运行的模拟器](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> 如果发现消息无法发送，则可能需要重启计算机，因为 ngrok 尚未在系统上获得所需特权（只需授权一次）。
