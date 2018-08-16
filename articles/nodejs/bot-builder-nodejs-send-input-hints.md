---
title: 向消息添加输入提示 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 向消息添加输入提示。
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b5efb024c01437867b6ab1cf99b2f544077eee0c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298095"
---
# <a name="add-input-hints-to-messages"></a>向消息添加输入提示
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

通过为消息指定输入提示，可以指示在将消息传送到客户端后，机器人是接受、预期还是忽略用户输入。 对于许多通道，这使客户端可以相应地设置用户输入控件的状态。 例如，如果消息的输入提示指示机器人忽略用户输入，则客户端可以关闭麦克风并禁用输入框以防止用户提供输入。

## <a name="accepting-input"></a>接受输入

若要指示机器人已被动准备好输入但未等待用户的响应，请将消息的输入提示设置为 `builder.InputHint.acceptingInput`。 在许多通道上，这将导致客户端的输入框启用，麦克风关闭但仍可供用户访问。 例如，如果用户按住麦克风按钮，Cortana 将打开麦克风以接受用户的输入。 下面的代码示例可创建一条消息，指示机器人正在接受用户输入。

[!code-javascript[IMessage.speak](../includes/code/node-input-hints.js#InputHintAcceptingInput)]

## <a name="expecting-input"></a>预期输入

若要指示机器人正在等待用户的响应，请将消息的输入提示设置为 `builder.InputHint.expectingInput`。 在许多通道上，这将导致客户端的输入框启用，麦克风打开。 下面的代码示例可创建一个提示，指示机器人正在预期用户输入。

[!code-javascript[Prompt](../includes/code/node-input-hints.js#InputHintExpectingInput)]

## <a name="ignoring-input"></a>忽略输入

若要指示机器人没有准备好接收用户的输入，请将消息的输入提示设置为 `builder.InputHint.ignoringInput`。 在许多通道上，这将导致客户端的输入框禁用，麦克风关闭。 下面的代码示例使用 `session.say()` 方法发送一条消息，指示机器人正在忽略用户输入。

[!code-javascript[Session.say()](../includes/code/node-input-hints.js#InputHintIgnoringInput)]

## <a name="default-values-for-input-hint"></a>输入提示的默认值

如果未为消息设置输入提示，Bot Builder SDK 将使用以下逻辑自动进行设置： 

- 如果机器人发送提示，则消息的输入提示将指定机器人正在预期输入。</li>
- 如果机器人发送单条消息，则消息的输入提示将指定机器人正在接受输入。</li>
- 如果机器人发送了一系列连续的消息，那么系列中除最后消息之外的所有消息的输入提示将指定机器人正在忽略输入，并且系列中最后消息的输入提示将指定机器人正在接受输入。

## <a name="additional-resources"></a>其他资源

- [向消息添加语音](bot-builder-nodejs-text-to-speech.md)
- [Bot Builder SDK for Node.js 参考][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

