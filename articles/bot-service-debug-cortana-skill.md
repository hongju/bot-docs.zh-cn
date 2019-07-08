---
title: 测试 Cortana 技能 |Microsoft Docs
description: 了解如何通过调用 Cortana 技能测试 Cortana 机器人。
keywords: Bot Framework SDK, 注册机器人, cortana
author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/01/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7d07317afb6d89c2d22d6f4983f7b21c3a1a053c
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496667"
---
# <a name="test-a-cortana-skill"></a>测试 Cortana 技能

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]
 
如果使用 Bot Framework SDK 生成了 Cortana 技能，则可通过从 Cortana 调用它来对其进行测试。 以下说明将指导你完成测试 Cortana 技能所需的步骤。

## <a name="register-your-bot"></a>注册机器人
如果在 Azure 中使用机器人服务[创建了机器人](~/bot-service-quickstart.md)，则已注册机器人，可跳过此步骤。

如果已在其他位置部署了机器人，或如果想在本地测试机器人，则必须[注册](bot-service-quickstart-registration.md)机器人，以便将其连接到 Cortana。 在注册过程中，需要提供机器人的消息终结点  。 如果选择在本地测试机器人，则需运行 [ngrok](http://ngrok.com) 等隧道软件来获取本地机器人的终结点。

## <a name="get-messaging-endpoint-using-ngrok"></a>使用 ngrok 获取消息终结点

若要在本地运行机器人，则可通过运行 [ngrok](https://ngrok.com) 等隧道软件来获取用于测试的终结点。 要使用 ngrok 获取终结点，请在控制台窗口中键入： 

```cmd
 ngrok.exe http 3979 -host-header="localhost:3979"
``` 

这将配置并显示 ngrok 转接链接，该链接将请求转发给在端口 3978 上运行的机器人。 转接链接的 URL 应如下所示：`https://0d6c4024.ngrok.io`。  将 `/api/messages` 附加到链接，将消息终结点 URL 设置为如下格式：`https://0d6c4024.ngrok.io/api/messages`。 

在机器人“[设置](~/bot-service-manage-settings.md)”边栏选项卡的“配置”部分中输入此终结点 URL  。

## <a name="enable-speech-recognition-priming"></a>启用语音识别启动
如果机器人使用语言理解 (LUIS) 应用，请确保将 LUIS 应用程序 ID 与注册的机器人服务相关联。 这有助于机器人识别 LUIS 模型中定义的口语表达。 有关详细信息，请参阅[语音启动](~/bot-service-manage-speech-priming.md)。

## <a name="add-the-cortana-channel"></a>添加 Cortana 通道
要将 Cortana 添加为通道，请按照[将机器人连接到 Cortana](bot-service-channel-connect-cortana.md) 中列出的步骤进行操作。

## <a name="test-using-web-chat-control"></a>使用网上聊天控件进行测试

要使用机器人服务中的集成网上聊天控件测试机器人，请单击“通过网上聊天执行测试”并键入消息以验证机器人是否正常工作  。

## <a name="test-using-emulator"></a>使用模拟器进行测试

要使用[模拟器](~/bot-service-debug-emulator.md)测试机器人，请执行以下操作：

1. 运行机器人。
2. 打开模拟器并填写必要信息。 要查找机器人的 AppID 和 AppPassword，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)   。 
3. 单击“连接”将模拟器连接到机器人  。
4. 键入消息以验证机器人是否正在工作。

## <a name="test-using-cortana"></a>使用 Cortana 进行测试
可通过向 Cortana 说出调用短语来调用 Cortana 技能。 
1. 打开 Cortana。
2. 在 Cortana 中打开笔记本，然后单击“关于我”，查看用于 Cortana 的帐户  。 确保使用与注册机器人时相同的 Microsoft 帐户登录。 
   ![登录到 Cortana 的笔记本](~/media/cortana/cortana-notebook.png)
2. 单击 Cortana 应用中的麦克风按钮或 Windows 中的“询问我任何问题”搜索框，并说出机器人的[调用短语][InvocationNameGuidelines]。 调用短语包括调用名，它是要调用技能的唯一标识  。 例如，如果技能的调用名是“Northwind Photo”，则正确的调用短语包括“要求 Northwind Photo...”或“告知 Northwind Photo...”。

   为 Cortana 配置机器人时，将指定机器人的调用名  。
   ![配置 Cortana 通道时输入调用名](~/media/cortana/cortana-invocation-name-callout.png)

3. 如果 Cortana 识别出调用短语，则会在 Cortana 的画布中启动机器人。 

## <a name="troubleshoot"></a>故障排除

如果 Cortana 技能无法启动，请进行以下检查：
* 确保使用在 Bot Framework 门户中注册机器人时所用的相同 Microsoft 帐户登录 Cortana。
* 单击“通过网上聊天执行测试”，打开“聊天”窗口并在其中键入消息，检查机器人是否正常工作   。
* 检查调用名是否符合[指导原则][InvocationNameGuidelines]。 如果调用名超过三个字词、难以发音或听起来像其他词，Cortana 可能难以识别。
* 如果技能使用 LUIS 模型，请确保[启用语音识别启动](~/bot-service-manage-speech-priming.md)。

有关如何在 Cortana 仪表板中启用技能调试的其他疑难解答提示和信息，请参阅[启用 Cortana 技能调试][Cortana-TestBestPractice]。 


## <a name="next-steps"></a>后续步骤

测试了 Cortana 技能并确认其按所需方式工作后，即可将其部署到一组 beta 版本测试人员或向公众发布。 有关详细信息，请参阅[发布 Cortana 技能][Cortana-Publish]。

## <a name="additional-resources"></a>其他资源
* [Cortana 技能套件][CortanaGetStarted]

[CortanaGetStarted]: /cortana/getstarted

[BFPortal]: https://dev.botframework.com/
[CortanaDevCenter]: https://developer.microsoft.com/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines 


[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice
[Cortana-Publish]: /cortana/skills/publish-skill
