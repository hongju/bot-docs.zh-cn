---
title: 向消息添加语音 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for Node.js 向消息添加语音。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 58086bfb29846e39a219beb2f7f0e8d977415559
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904912"
---
# <a name="add-speech-to-messages"></a>向消息添加语音

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

如果要为支持语音的通道（如 Cortana）构建机器人，可以构造可指定机器人要说出的文本的消息。 还可以通过指定[输入提示](bot-builder-nodejs-send-input-hints.md)来尝试影响客户端麦克风的状态，以指示机器人是在接受、期望还是忽略用户输入。

## <a name="specify-text-to-be-spoken-by-your-bot"></a>指定机器人要说的文本

使用 Bot Builder SDK for Node.js，有多种方法可以指定机器人要在支持语音的通道上说出的文本。 可以设置 `IMessage.speak` 属性并使用 `session.send()` 方法发送消息、使用 `session.say()` 方法（传递可指定显示文本、语音文本和选项的参数）发送消息，或者使用内置提示（指定 `speak` 和 `retrySpeak` 选项）发送消息。

### <a id="message-speak"></a> IMessage.speak 

如果你创建的消息将通过 `session.send()` 方法发送，请将 `speak` 属性设置为指定机器人要说出的文本。 以下代码示例创建一条消息，指定要说出的文本，并指示机器人[正在接受用户输入](bot-builder-nodejs-send-input-hints.md)。

[!code-javascript[IMessage.speak](../includes/code/node-text-to-speech.js#IMessageSpeak)]

### <a id="session-say"></a> session.say()

除了要显示的文本和其他选项外，作为使用 `session.send()` 的替代方法，可以调用 `session.say()` 方法创建并发送可指定要说出的文本的消息。 该方法定义如下：

`session.say(displayText: string, speechText: string, options?: object)`

| 参数 | Description |
|----|----|
| `displayText` | 要显示的文本。 |
| `speechText` | 要说出的文本（纯文本或 <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a> 格式）。 |
| `options` | [IMessage][IMessage] 对象，可以包含附件或[输入提示](bot-builder-nodejs-send-input-hints.md)。 |

以下代码示例发送一条消息，指定要显示的文本和要说出的文本，并指示机器人[正在忽略用户输入](bot-builder-nodejs-send-input-hints.md)。

[!code-javascript[Session.say()](../includes/code/node-text-to-speech.js#SessionSay)]

### <a id="prompt-options"></a> 提示选项

使用任何内置提示，可以设置 `speak` 和 `retrySpeak` 选项以指定机器人要说出的文本。 以下代码示例创建一个提示，指定要显示的文本、最初要说出的文本以及等待用户输入后要说出的文本。 它指示机器人[正在期望用户输入](bot-builder-nodejs-send-input-hints.md)并使用 [SSML](#ssml) 格式设置来指定应该以适度的强调说出“确保”一词。

[!code-javascript[Prompt](../includes/code/node-text-to-speech.js#Prompt)]

## <a id="ssml"></a> 语音合成标记语言 (SSML)

若要指定机器人要说出的文本，可以使用纯文本字符串或格式设置为语音合成标记语言 (SSML) 的字符串，后者是一种基于 XML 的标记语言，可用于控制机器人语音的各种特征（如语音、语速、音量、发音、音高等）。 有关 SSML 的详细信息，请参阅<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言参考</a>。

> [!TIP]
> 使用 <a href="https://www.npmjs.com/search?q=ssml" target="_blank">SSML 库</a>创建格式正确的 SSML。

## <a name="input-hints"></a>输入提示

在支持语音的通道上发送消息时，可以尝试通过同时包含输入提示来影响客户端麦克风的状态，以指示机器人是在接受、期望还是忽略用户输入。 有关详细信息，请参阅[向消息添加输入提示](bot-builder-nodejs-send-input-hints.md)。

## <a name="sample-code"></a>代码示例 

有关演示如何使用 Bot Builder SDK for .NET 创建支持语音的机器人的完整示例，请参阅 GitHub 中的<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">掷骰子示例</a>。

## <a name="additional-resources"></a>其他资源

- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言 (SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">掷骰子示例 (GitHub)</a>
- [Bot Builder SDK for Node.js 参考][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
