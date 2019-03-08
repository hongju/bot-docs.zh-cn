---
title: 使用 Bot Framework Emulator 测试和调试机器人 | Microsoft Docs
description: 了解如何使用 Bot Framework Emulator 桌面应用程序检查、测试和调试机器人。
keywords: 脚本, msbot 工具, 语言服务, 语音识别
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: 2266dcb936205a20e1d19d97983a3b802fbe2736
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224865"
---
# <a name="debug-with-the-emulator"></a>使用模拟器进行调试

Bot Framework Emulator 是一个桌面应用程序，允许机器人开发人员在本地或远程测试和调试机器人。 使用此模拟器，你可以和机器人聊天并检查机器人发送和接收的消息。 模拟器会显示消息，因为它们会出现在网上聊天 UI 中，并在与机器人交换消息时记录 JSON 请求和响应。 在将机器人部署到云之前，请使用模拟器在本地运行和测试。 可以使用模拟器测试机器人，即使还没有使用 Azure 机器人服务[创建](./bot-service-quickstart.md)它或将其配置为在任何通道上运行。

## <a name="prerequisites"></a>先决条件
- 安装[模拟器](https://aka.ms/Emulator-wiki-getting-started)
- 安装 [ngrok][ngrokDownload] 隧道软件

## <a name="connect-to-a-bot-running-on-localhost"></a>连接到在 localhost 上运行的机器人

若要连接到本地运行的机器人，请单击“打开机器人”并选择 .bot 文件。 

![模拟器 UI](media/emulator-v4/emulator-welcome.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用检查器查看消息活动的详细信息

向机器人发送消息，机器人应该回复。 可以单击聊天窗口中的消息气泡，并使用窗口右侧的“检查器”功能检查原始 JSON 活动。 选中时，消息气泡将变为黄色，并在聊天窗口左侧显示活动 JSON 对象。 JSON 信息包含密钥元数据，包括 channelID、活动类型、聊天 ID、文本消息、终结点 URL 等等。可以检查用户发送的活动，以及机器人响应的活动。 

![模拟器消息活动](media/emulator-v4/emulator-view-message-activity-02.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>使用机器人脚本保存和加载会话

模拟器中的活动可以另存为脚本。 在打开的实时聊天窗口中，选择“将脚本另存为”来命名脚本文件。 可以随时使用“重新开始”按钮清除会话，并重新启动到机器人的连接。  

![模拟器保存脚本](media/emulator-v4/emulator-live-chat.png)

若要加载脚本，只需选择“文件”>“打开脚本文件”，然后选择该脚本。 新“脚本”窗口将打开并在输出窗口中呈现消息活动。 

![模拟器加载脚本](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>添加服务 

可以直接从模拟器轻松地将 LUIS 应用、QnA 知识库或调度模型添加到 .bot 文件中。 加载 .bot 文件后，选择模拟器窗口最左侧的服务按钮。 你将在“服务”菜单下看到用于添加 LUIS、QnA Maker 和 Dispatch 的选项。 

若要添加服务应用，只需单击 **+** 按钮，然后选择要添加的服务即可。 系统将提示你登录 Azure 门户，以便将服务添加到机器人文件，并将服务连接到机器人应用程序。 

![LUIS 连接](media/emulator-v4/emulator-connect-luis-btn.png)

当连接这两种服务时，可以返回到实时聊天窗口并验证服务是否已连接和正常工作。 

![QnA 已连接](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>检查服务

使用新 v4 模拟器，还可以检查来自 LUIS 和 QnA 的 JSON 响应。 将机器人与已连接的语言服务结合使用，可以选择右下角“LOG”窗口中的“跟踪”。 此新工具还提供直接从模拟器更新语言服务的功能。 

![LUIS 检查器](media/emulator-v4/emulator-luis-inspector.png)

使用已连接的 LUIS 服务，你会注意到跟踪链接指定了 Luis 跟踪。 选中时，将看到 LUIS 服务中的原始响应，其中包括意向、实体以及指定分数。 此外可以选择为用户表达重新分配意向。 

![QnA 检查器](media/emulator-v4/emulator-qna-inspector.png)

使用已连接的 QnA 服务，日志将显示“QnA 跟踪”，选择后，可以预览与该活动相关联的问题和答案对，以及置信度分数。 可以在此处为答案添加备用问题表述。

## <a name="configure-ngrok"></a>配置 ngrok

如果使用的是 Windows 且正在防火墙或其他网络边界后运行 Bot Framework Emulator，并且想要连接到远程托管的机器人，必须安装和配置 ngrok 隧道软件。 Bot Framework Emulator 与 ngrok 隧道软件（由 [inconshreveable][inconshreveable] 开发）紧密集成，并且可以在需要时自动启动。

打开“模拟器设置”，输入到 ngrok 的路径，选择是否绕过 ngrok 以获取本地地址，然后单击“保存”。

![ngrok 路径](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>其他资源

Bot Framework Emulator 为开放源代码。 可以[参与][EmulatorGithubContribute]开发并[提交 bug 报告和建议][EmulatorGithubBugs]。

有关疑难解答，请参阅[排查常见问题](bot-service-troubleshoot-bot-configuration.md)和该部分中的其他疑难解答文章。

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
