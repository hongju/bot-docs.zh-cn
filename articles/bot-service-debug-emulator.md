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
ms.openlocfilehash: 307a6bf697e274391336a0d216c64da85232616d
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033362"
---
# <a name="debug-with-the-emulator"></a>使用模拟器进行调试

Bot Framework Emulator 是一个桌面应用程序，允许机器人开发人员在本地或远程测试和调试机器人。 使用此模拟器，你可以和机器人聊天并检查机器人发送和接收的消息。 模拟器会显示消息，因为它们会出现在网上聊天 UI 中，并在与机器人交换消息时记录 JSON 请求和响应。 在将机器人部署到云之前，请使用模拟器在本地运行和测试。 可以使用模拟器测试机器人，即使还没有使用 Azure 机器人服务[创建](./bot-service-quickstart.md)它或将其配置为在任何通道上运行。

## <a name="prerequisites"></a>先决条件
- 安装[模拟器](https://aka.ms/Emulator-wiki-getting-started)

## <a name="connect-to-a-bot-running-on-localhost"></a>连接到在 localhost 上运行的机器人

![模拟器 UI](media/emulator-v4/emulator-welcome.png)

若要连接到本地运行的机器人，请单击“打开机器人”，或者选择预先配置的配置文件（.bot 文件）。 不需配置文件也可连接到机器人，但如果机器人有配置文件，则模拟器也会接受。 如果机器人是使用 Microsoft 帐户 (MSA) 凭据运行的，也请输入这些凭据。

![模拟器 UI](media/emulator-v4/emulator-open-bot.png)

## <a name="view-detailed-message-activity-with-the-inspector"></a>使用检查器查看消息活动的详细信息

向机器人发送消息，机器人应该回复。 可以单击聊天窗口中的消息气泡，并使用窗口右侧的“检查器”功能检查原始 JSON 活动。 选中时，消息气泡将变为黄色，并在聊天窗口左侧显示活动 JSON 对象。 JSON 信息包含密钥元数据，包括 channelID、活动类型、聊天 ID、文本消息、终结点 URL 等等。可以检查用户发送的活动，以及机器人响应的活动。 

![模拟器消息活动](media/emulator-v4/emulator-view-message-activity-03.png)

## <a name="save-and-load-conversations-with-bot-transcripts"></a>使用机器人脚本保存和加载会话

模拟器中的活动可以另存为脚本。 在打开的实时聊天窗口中，选择“将脚本另存为”来命名脚本文件。 可以随时使用“重新开始”按钮清除会话，并重新启动到机器人的连接。  

![模拟器保存脚本](media/emulator-v4/emulator-save-transcript.png)

若要加载脚本，只需选择“文件”>“打开脚本文件”，然后选择该脚本。 新“脚本”窗口将打开并在输出窗口中呈现消息活动。 

![模拟器加载脚本](media/emulator-v4/emulator-load-transcript.png)

## <a name="add-services"></a>添加服务 

可以直接从模拟器轻松地将 LUIS 应用、QnA 知识库或调度模型添加到机器人中。 加载机器人后，选择模拟器窗口最左侧的服务按钮。 你将在“服务”菜单下看到用于添加 LUIS、QnA Maker 和 Dispatch 的选项。 

若要添加服务应用，只需单击 **+** 按钮，然后选择要添加的服务即可。 系统将提示你登录 Azure 门户，以便将服务添加到机器人文件，并将服务连接到机器人应用程序。 

> [!IMPORTANT]
> 只有在使用 `.bot` 配置文件的情况下，才能添加服务。 需单独添加服务。 有关此方面的详细信息，请参阅[管理机器人资源](v4sdk/bot-file-basics.md)；对于你要尝试添加的服务，也可参阅单独的操作方法文章。
>
> 如果不使用 `.bot` 文件，则左侧窗格不会列出服务（即使机器人使用服务），而是显示“服务不可用”。

![LUIS 连接](media/emulator-v4/emulator-connect-luis-btn.png)

当连接这两种服务时，可以返回到实时聊天窗口并验证服务是否已连接和正常工作。 

![QnA 已连接](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-services"></a>检查服务

使用新 v4 模拟器，还可以检查来自 LUIS 和 QnA 的 JSON 响应。 将机器人与已连接的语言服务结合使用，可以选择右下角“LOG”窗口中的“跟踪”。 此新工具还提供直接从模拟器更新语言服务的功能。 

![LUIS 检查器](media/emulator-v4/emulator-luis-inspector.png)

使用已连接的 LUIS 服务，你会注意到跟踪链接指定了 Luis 跟踪。 选中时，将看到 LUIS 服务中的原始响应，其中包括意向、实体以及指定分数。 此外可以选择为用户表达重新分配意向。 

![QnA 检查器](media/emulator-v4/emulator-qna-inspector.png)

使用已连接的 QnA 服务，日志将显示“QnA 跟踪”，选择后，可以预览与该活动相关联的问题和答案对，以及置信度分数。 可以在此处为答案添加备用问题表述。

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

## <a name="login-to-azure"></a>登录到 Azure

可以使用 Emulator 登录到 Azure 帐户。 若要添加和管理机器人所依赖的服务，则这特别有用。 请参阅[上面的内容](#add-services)，详细了解可以使用 Emulator 来管理的服务。

### <a name="to-login"></a>登录

![Azure 登录](media/emulator-v4/emulator-azure-login.png)

登录
- 可以单击“文件”->“使用 Azure 登录”
- 在欢迎屏幕上，单击“使用 Azure 帐户登录”。可以选择让 Emulator 保持你处于已登录状态，即使 Emulator 应用程序重启也是如此。

![Azure 登录](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>禁用收集数据

如果你决定不再允许 Emulator 收集使用情况数据，可以按照以下步骤轻松地禁用数据收集功能。

1. 导航到 Emulator 的设置页，方法是单击左侧导航栏中的“设置”按钮（齿轮图标）。

    ![禁用数据收集](media/emulator-v4/emulator-disable-data-1.png)

2. 取消选中“数据收集”部分下标记为“允许我们收集使用情况数据，帮助改进 Emulator”的复选框。

    ![禁用数据收集](media/emulator-v4/emulator-disable-data-2.png)

3. 单击“保存”按钮。

    ![禁用数据收集](media/emulator-v4/emulator-disable-data-3.png)
    
如果改变想法，只需重新选中该复选框即可启用此功能。

## <a name="additional-resources"></a>其他资源

Bot Framework Emulator 为开放源代码。 可以[参与][EmulatorGithubContribute]开发并[提交 bug 报告和建议][EmulatorGithubBugs]。

有关疑难解答，请参阅[排查常见问题](bot-service-troubleshoot-bot-configuration.md)和该部分中的其他疑难解答文章。

## <a name="next-steps"></a>后续步骤

将聊天保存到某个脚本文件即可快速地为特定的一组交互打草稿并反复进行播放，以便进行调试。

> [!div class="nextstepaction"]
> [使用脚本文件调试机器人](~/v4sdk/bot-builder-debug-transcript.md)

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
