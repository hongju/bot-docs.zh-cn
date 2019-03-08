---
title: 使用脚本文件调试机器人 | Microsoft Docs
description: 了解如何使用脚本文件来帮助调试机器人。
keywords: 调试, 常见问题解答, 脚本文件, 模拟器
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservices: sdk
ms.date: 2/26/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 997ad82e15a0fcd67d47b2fd6495c8e88a5ea127
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224815"
---
# <a name="debug-your-bot-using-transcript-files"></a>使用脚本文件调试机器人
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

若要成功地测试和调试机器人，其中的一个关键是必须能够记录并检查运行机器人时出现的一组条件。 本文讨论如何创建并使用一个机器人脚本文件，目的是提供一组详细的适用于测试和调试的用户交互和机器人响应。

## <a name="the-bot-transcript-file"></a>机器人脚本文件
机器人脚本文件是一个专用的 JSON 文件，保留了用户和机器人之间的交互。 脚本文件不仅保留消息内容，而且保留交互详细信息，例如用户 ID、通道 ID、通道类型、通道功能、交互时间，等等。上述所有信息均可用于查找和解决测试或调试机器人时出现的问题。 

## <a name="creatingstoring-a-bot-transcript-file"></a>创建/存储机器人脚本文件
本文介绍如何使用 Microsoft 的 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) 创建机器人脚本文件。 脚本文件也可以编程方式创建；有关该方法的详细信息，可参阅[此文](./bot-builder-howto-v4-storage.md#blob-transcript-storage)。 在本文中，我们将使用适用于[带计数器的 EchoBot](https://aka.ms/EchoBot-With-Counter) 的 Bot Framework 示例代码。该代码经过修改，可以请求用户的姓名和电话号码。不过，任何能够使用 Microsoft 的 Bot Framework Emulator 访问的代码都可以用来创建脚本文件。

若要开始此过程，请确保要测试的机器人代码在开发环境中运行。 启动 Bot Framework Emulator，选择“打开机器人”按钮，将模拟器连接到代码的机器人配置文件，如下图所示。

![将模拟器连接到代码](./media/emulator_open_bot_configuration.png)

将模拟器连接到运行的代码以后，请向机器人发送模拟的用户交互，对代码进行测试。 在此示例中，我们传入了用户的姓名和电话号码。 在输入需要保留的所有用户交互以后，请使用 Bot Framework Emulator 创建并保存包含该聊天的脚本文件。 

在“实时聊天”选项卡（见下）中，选择“保存脚本”按钮。 

![选择“保存脚本”](./media/emulator_transcript_save.png)

选择脚本文件的位置和名称，然后选择“保存”按钮。

![将脚本另存为 ursula](./media/emulator_transcript_saveas_ursula.png)

现在，你输入的用于通过模拟器来测试代码的所有用户交互和机器人响应都已保存到一个脚本文件中。该文件可以稍后重新加载，用于调试用户和机器人之间的交互。

## <a name="retrieving-a-bot-transcript-file"></a>检索机器人脚本文件
若要使用 Bot Framework Emulator 检索机器人脚本文件，请在模拟器左上角的“资源”部分选择“脚本”列表控件，如下所示。 接下来，选择要检索的脚本文件。 在此示例中，我们将检索名为“Ursula_User_transcript”的脚本文件。 选择脚本文件时，会自动将整个保留的聊天加载到名为“脚本”的新选项卡中。

![检索保存的脚本](./media/emulator_transcript_retrieve.png)

## <a name="debug-using-transcript-file"></a>使用脚本文件进行调试
加载脚本文件后，就可以调试捕获的用户和机器人之间的交互了。 为此，请直接单击在“日志”部分记录的任何事件或活动，如模拟器右下区域所示。 在下面显示的示例中，我们选择了用户在发送消息“Hello”时的首次交互。 这样做时，脚本文件中有关此特定交互的所有信息都会以 JSON 格式显示在模拟器的“检查器”窗口中。 从下往上查看这其中的一些值时，可以看到：
* 交互类型为消息。
* 发送消息的时间。
* 已发送的包含“Hello”的纯文本。
* 消息发送到机器人。
* 用户 ID 和信息。
* 通道 ID、功能和信息。

![使用脚本进行调试](./media/emulator_transcript_debug.png)

信息如此详细，因此可以一步步地追溯这些交互中的用户输入和机器人响应，通过调试来了解到底是机器人没有按预期方式响应用户，还是根本就没有响应。 有了这些值以及导致交互失败的步骤的记录以后，就可以对代码进行步进调试，找出机器人没有按预期响应的位置，并解决相应的问题。

可以通过许多方式来测试和调试机器人的代码和用户交互，将脚本文件和 Bot Framework Emulator 配合使用的方式只是其中的一种。 如需更多的机器人测试和调试方式，请参阅下面列出的其他资源。

## <a name="additional-resources"></a>其他资源

如需其他的测试和调试信息，请参阅：

* [机器人测试和调试指南](./bot-builder-testing-debugging.md)
* [使用 Bot Framework Emulator 进行调试](../bot-service-debug-emulator.md)
* [排查常见问题](../bot-service-troubleshoot-bot-configuration.md)和该部分中的其他疑难解答文章。
* [在 Visual Studio 中进行调试](https://docs.microsoft.com/en-us/visualstudio/debugger/index)
