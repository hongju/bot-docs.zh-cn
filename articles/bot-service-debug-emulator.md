---
title: 使用 Bot Framework Emulator 测试和调试机器人 | Microsoft Docs
description: 了解如何使用 Bot Framework Emulator 桌面应用程序检查、测试和调试机器人。
keywords: 脚本, msbot 工具, 语言服务, 语音识别
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: bc1da99c7d0f7a6431ad0c2746b8583ef7bfbd5f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298046"
---
# <a name="debug-bots-with-the-bot-framework-emulator"></a>使用 Bot Framework Emulator 调试机器人

Bot Framework Emulator 是一个桌面应用程序，允许机器人开发人员在本地或远程测试和调试机器人。 使用此模拟器，你可以和机器人聊天并检查机器人发送和接收的消息。 模拟器会显示消息，因为它们会出现在网上聊天 UI 中，并在与机器人交换消息时记录 JSON 请求和响应。 

> [!TIP] 
> 在将机器人部署到云之前，请使用模拟器在本地运行和测试。 可以使用模拟器测试机器人，即使还没有使用 Bot Framework 进行[注册](~/bot-service-quickstart-registration.md) 或将其配置为在任何通道上运行。

![模拟器 UI](media/emulator-v4/emulator-welcome.png)

## <a name="download-the-bot-framework-emulator"></a>下载 Bot Framework Emulator

可通过 [GitHub 发布页面](https://github.com/Microsoft/BotFramework-Emulator/releases)下载适用于 Mac、Windows 和 Linux 的程序包。

## <a name="connect-to-a-bot-running-on-local-host"></a>连接到本地主机上运行的机器人

![模拟器终结点](media/emulator-v4/emulator-endpoint.png)

若要连接到在本地运行的机器人，请选择左上角的“机器人资源管理器”选项卡。 单击“终结点”选项卡下的 + 图标。可以在此处指定机器人在本地运行的同一个端口的终结点，以便连接到它。 单击“提交”，将重定向到实时聊天窗口，可以在其中与机器人交互。

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用检查器查看消息活动的详细信息

![模拟器消息活动](media/emulator-v4/emulator-view-message-activity-02.png)

可以单击会话窗口中的任何消息气泡，并使用窗口右侧的“检查器”功能检查原始 JSON 活动。 选中时，消息气泡将变为黄色，并在聊天窗口左侧显示活动 JSON 对象。 此 JSON 信息包含密钥元数据，包括 channelID、活动类型、会话 id、文本消息、终结点 URL 等等。可以查看用户发送的检查活动，以及机器人响应的活动。 

此外可以使用[通道检查器](bot-service-channel-inspector.md)在特定通道上预览支持功能。


## <a name="save-and-load-conversations-with-bot-transcripts"></a>使用机器人脚本保存和加载会话

模拟器中的消息活动可以另存为脚本。 在打开的实时聊天窗口中，可以选择“将脚本另存为”来命名和选择输出脚本文件的位置。 

>[!TIP]
> 可以随时使用“重新开始”按钮清除会话，并重新启动到机器人的连接。  

![模拟器保存脚本](media/emulator-v4/emulator-live-chat.png)

若要加载脚本，只需选择“文件” --> “打开脚本文件”，然后选择该脚本。 新“脚本”窗口将打开并在输出窗口中呈现消息活动。 

![模拟器加载脚本](media/emulator-v4/emulator-load-transcript.png)

## <a name="author-transcripts-with-chatdown"></a>使用 Chatdown 创建脚本

[Chatdown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown) 工具是一个脚本生成器，它使用 [markdown](https://daringfireball.net/projects/markdown/syntax) 文件生成活动脚本。 可以完全采用 markdown 格式编写你自己的脚本，并将它们另存为 .chat 文件以生成脚本。 这可用于在机器人开发过程中快速创建模拟会话方案。  

### <a name="prerequisites"></a>先决条件

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) v.4 或更高版本 
- [Node.js](https://nodejs.org/en/)
 
Chatdown 可作为 npm 模块提供，它需要 Node.js。 若要安装 Chatdown，请在全局范围内安装到计算机。 

```
npm install -g chatdown
```
### <a name="create-and-load-transcript-transcript-files"></a>创建和加载脚本文件 ###

下面举例说明了如何创建 .chat 文件。 这些文件为 markdown 格式，其中包含两个部分：
- 标头：定义会话参与者（用户、机器人）
- 参与者之间的来回会话

```
user=John Doe
bot=Bot

bot: Hello!
user: hey
bot: [Typing][Delay=3000]
What can I do for you?
user: Actually nevermind, goodbye.
bot: bye!
```
[单击此处](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown/Examples)查看 .chat 文件的更多示例。 

若要生成脚本文件，请使用 CLI 中的 chatdown 命令，输入 .chat 文件的名称，后跟“>”和输出脚本文件的名称。 

```
chatdown sample.chat > sample.transcript
```
## <a name="manage-bot-resources-with-the-msbot-tool"></a>使用 MSBot 工具管理机器人资源

新 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 工具允许你创建 .bot 文件，该文件将机器人使用的不同服务的相关元数据存储在一个位置。 此文件还使机器人从 CLI 连接到这些服务。该工具可作为 npm 模块使用，安装以运行：

```
npm install -g msbot 
```
![MSBot CLI 窗口](media/emulator-v4/msbot-cli-window.png)


若要创建机器人文件，请在 CLI 中输入“msbot init”后跟机器人的名称，并输入目标 URL 终结点，例如：

```shell
msbot init --name az-cli-bot --endpoint http://localhost:3984/api/messages
```
![Botfile](media/emulator-v4/botfile-generated.png)

>注意：本指南中使用的机器人是简单的回显机器人，从 Azure CLI 机器人扩展生成。 [单击此处](https://github.com/Microsoft/botbuilder-tools/tree/master/AzureCli)了解有关使用 Azure CLI 生成机器人的详细信息。 

使用 .bot 文件，现在可以轻松将机器人加载到模拟器。 .bot 文件还需要用于为机器人注册不同的终结点和语言组件。 

![Bot-File-Dropdown](media/emulator-v4/bot-file-dropdown.png)

## <a name="add-language-services"></a>添加语言服务 

可以直接从模拟器将 LUIS 应用或 QnA 知识库轻松注册到 .bot 文件。 加载 .bot 文件后，选择模拟器窗口最左侧的服务按钮。 将看到“服务”菜单下的选项，可用于添加 LUIS、QnA Maker、Dispatch、终结点和 Azure 机器人服务。 

要添加 LUIS 应用，只需单击 LUIS 菜单上的 + 按钮，输入 LUIS 应用凭证，然后单击“提交”。 这会将 LUIS 应用程序注册到 .bot 文件，并将服务连接到机器人应用程序。 

![LUIS 连接](media/emulator-v4/emulator-connect-luis-btn.png)

同样，若要添加 QnA 知识库，只需单击 QnA 菜单上的 + 按钮，输入 QnA Maker 知识库凭据，然后单击“提交”。 知识库现在将注册到 .bot 文件，并且可供使用。 

![QnA 连接](media/emulator-v4/emulator-connect-qna-btn.png)

当连接这两种服务时，可以返回到实时聊天窗口并验证服务是否已连接和正常工作。 

![QnA 已连接](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-language-services"></a>检查语言服务

使用新 v4 模拟器，还可以检查来自 LUIS 和 QnA 的 JSON 响应。 将机器人与已连接的语言服务结合使用，可以选择右下角“LOG”窗口中的“跟踪”。 此新工具还提供直接从模拟器更新语言服务的功能。 

![LUIS 检查器](media/emulator-v4/emulator-luis-inspector.png)

使用已连接的 LUIS 服务，你会注意到跟踪链接指定了 Luis 跟踪。 选中时，将看到 LUIS 服务中的原始响应，其中包括意向、实体以及指定分数。 此外可以选择为用户表达重新分配意向。 

![QnA 检查器](media/emulator-v4/emulator-qna-inspector.png)

使用已连接的 QnA 服务，日志将显示“QnA 跟踪”，选择后，可以预览与该活动相关联的问题和答案对，以及置信度分数。 可以在此处为答案添加备用问题表述。

[!TIP]
> 这些功能仅适用于 v4 SDK 机器人 


## <a name="speech-recognition"></a>语音识别
Bot Framework Emulator 支持通过[认知服务语音 API](/azure/cognitive-services/Speech/home) 进行语音识别。 这样，你可以通过开发过程中模拟器的语音练习启用了语音的机器人或 Cortana 技能。 Bot Framework Emulator 为每个机器人每天提供最多三小时的免费语音识别。 

## <a id="ngrok"></a> 安装和配置 ngrok

如果使用的是 Windows 且正在防火墙或其他网络边界后运行 Bot Framework Emulator，并且想要连接到远程托管的机器人，必须安装和配置 ngrok 隧道软件。 Bot Framework Emulator 与 [ngrok][ngrokDownload] 隧道软件（由 [inconshreveable][inconshreveable] 开发）紧密集成，并且可以在需要时自动启动。

若要在 Windows 上安装 **ngrok** 并配置模拟器以使用它，请完成以下步骤： 

1. 将 [ngrok][ngrokDownload] 可执行文件下载到本地计算机。

2. 打开模拟器的“应用设置”对话框，输入到 ngrok 的路径，选择是否绕过 ngrok 以获取本地地址，然后单击“保存”。

![ngrok 路径](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>其他资源

Bot Framework Emulator 为开放源代码。 可以[参与][EmulatorGithubContribute]开发并[提交 bug 报告和建议][EmulatorGithubBugs]。


[EmulatorGithub]: https://github.com/Microsoft/BotFramework-Emulator
[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
[BotFrameworkDevPortal]: https://dev.botframework.com/


[EmulatorConnectPicture]: ~/media/emulator/emulator-connect_localhost_credentials.png
[EmulatorNgrokPath]: ~/media/emulator/emulator-configure_ngrok_path.png
[EmulatorNgrokMonitor]: ~/media/emulator/emulator-testbot-ngrok-monitoring.png
[EmulatorUI]: ~/media/emulator/emulator-ui-new.png

[TroubleshootingGuide]: ~/bot-service-troubleshoot-general-problems.md
[TroubleshootingAuth]: ~/bot-service-troubleshoot-authentication-problems.md
[NodeGetStarted]: ~/nodejs/bot-builder-nodejs-quickstart.md
[CSGetStarted]: ~/dotnet/bot-builder-dotnet-quickstart.md
