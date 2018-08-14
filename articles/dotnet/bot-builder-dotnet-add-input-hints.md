---
title: 向消息添加输入提示 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 向消息添加输入提示。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2f56a855990675ccae4845c13541150ab205379a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298362"
---
# <a name="add-input-hints-to-messages"></a>向消息添加输入提示
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

通过为消息指定输入提示，在将消息传送到客户端后你可以指示机器人是接受、期待还是忽略用户输入。 对于许多通道，这使客户端可以相应地设置用户输入控件的状态。 例如，如果消息的输入提示指示机器人忽略用户输入，则客户端可以关闭麦克风并禁用输入框以防止用户提供输入。

## <a name="accepting-input"></a>接受输入

若要指示机器人被动地准备好接收输入但未等待用户的响应，请将消息的输入提示设置为 `InputHints.AcceptingInput`。 在许多通道上，这将导致客户端的输入框启用并且麦克风关闭，但仍可供用户访问。 例如，如果用户按住麦克风按钮，Cortana 将打开麦克风以接受来自用户的输入。 下面的代码示例可创建一条消息，指示机器人接受用户输入。

[!code-csharp[Accepting input](../includes/code/dotnet-input-hints.cs#InputHintAcceptingInput)]

## <a name="expecting-input"></a>期待输入

若要指示机器人正在等待用户的响应，请将消息的输入提示设置为 `InputHints.ExpectingInput`。 在许多通道上，这将导致客户端的输入框启用并且麦克风打开。 下面的代码示例可创建一条消息，指示机器人期待用户输入。

[!code-csharp[Expecting input](../includes/code/dotnet-input-hints.cs#InputHintExpectingInput)]

## <a name="ignoring-input"></a>忽略输入
 
若要指示机器人尚未准备好接收用户的输入，请将消息的输入提示设置为 `InputHints.IgnorningInput`。 在许多通道上，这将导致客户端的输入框禁用并且麦克风关闭。 下面的代码示例可创建一条消息，指示机器人忽略用户输入。

[!code-csharp[Ignoring input](../includes/code/dotnet-input-hints.cs#InputHintIgnoringInput)]

## <a name="default-values-for-input-hint"></a>输入提示的默认值

如果未设置消息的输入提示，Bot Builder SDK 将使用以下逻辑自动进行设置： 

- 如果机器人发送提示，则消息的输入提示将指定机器人期待输入。</li>
- 如果机器人发送单条消息，则消息的输入提示将指定机器人接受输入。</li>
- 如果机器人发送了一系列连续的消息，那么系列消息中除最后一条之外的所有消息的输入提示将指定机器人忽略输入，并且系列消息中最后一条的输入提示将指定机器人接受输入。

## <a name="additional-resources"></a>其他资源

- [创建消息](bot-builder-dotnet-create-messages.md)
- [向消息添加语音](bot-builder-dotnet-text-to-speech.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
- <a href="/dotnet/api/microsoft.bot.connector.inputhints" target="_blank">InputHints 类</a>