---
title: 向消息添加语音 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 向消息添加语音。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dc542c7e85b3a79e1071edebea65d93c99742beb
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000354"
---
# <a name="add-speech-to-messages"></a>向消息添加语音

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

如果要为支持语音的通道（如 Cortana）构建机器人，可以构造可指定机器人要说出的文本的消息。 还可以通过指定[输入提示](bot-builder-dotnet-add-input-hints.md)来尝试影响客户端麦克风的状态，以指示机器人是在接受、期望还是忽略用户输入。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>指定机器人要说的文本

使用 Bot Builder SDK for .NET，有多种方法可以指定机器人要在支持语音的通道上说的文本。 可以设置[消息][IMessageActivity]的 `Speak` 属性、调用 `IDialogContext.SayAsync()` 方法，或者在使用内置提示发送消息时指定提示选项 `speak` 和 `retrySpeak`。

### <a id="message-speak"></a> IMessageActivity.Speak

如果要创建[消息][IMessageActivity]并设置其各个属性，则可以设置消息的 `Speak` 属性以指定机器人要说的文本。 以下代码示例创建一条消息，指定要显示的文本和要说出的文本，并指示机器人[正在接受用户输入](bot-builder-dotnet-add-input-hints.md)。

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

如果使用[对话](bot-builder-dotnet-dialogs.md)，除了要显示的文本和其他选项外，还可以调用 `SayAsync()` 方法创建并发送指定要说出的文本的消息。 以下代码示例创建一条消息，指定要显示的文本和要说出的文本。

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> 提示选项

使用任何内置提示，可以设置 `speak` 和 `retrySpeak` 选项以指定机器人要说出的文本。 以下代码示例创建一个提示，指定要显示的文本、最初要说出的文本以及等待用户输入后要说出的文本。 它使用 [SSML](#ssml) 格式设置来指示应该以适度的强调说出“确保”一词。

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a> 语音合成标记语言 (SSML)

若要指定机器人要说出的文本，可以使用纯文本字符串或格式设置为语音合成标记语言 (SSML) 的字符串，后者是一种基于 XML 的标记语言，可用于控制机器人语音的各种特征（如语音、语速、音量、发音、音高等）。 有关 SSML 的详细信息，请参阅<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言参考</a>。

## <a name="input-hints"></a>输入提示

在支持语音的通道上发送消息时，可以尝试通过同时包含输入提示来影响客户端麦克风的状态，以指示机器人是在接受、期望还是忽略用户输入。 有关详细信息，请参阅[向消息添加输入提示](bot-builder-dotnet-add-input-hints.md)。

## <a name="sample-code"></a>代码示例 

有关演示如何使用 Bot Builder SDK for .NET 创建支持语音的机器人的完整示例，请参阅 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">掷骰子技能示例</a>。

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-builder-dotnet-create-messages.md)
- [向消息添加输入提示](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言 (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-RollerSkill" target="_blank">掷骰子技能示例 (GitHub)</a>
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">活动类</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 接口</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">DialogContext 类</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">提示类</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity

