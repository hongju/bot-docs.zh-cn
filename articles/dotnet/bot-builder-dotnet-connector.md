---
title: 发送和接收活动 | Microsoft Docs
description: 了解如何通过 Bot Framework SDK for .NET 使用 Connector 服务跨各种通道与用户交换信息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4df2dcc8857c2af9a69c18e6acf8c8d064e1e043
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405692"
---
# <a name="send-and-receive-activities"></a>发送和接收活动

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Framework Connector 提供单个 REST API，让机器人能够跨多个通道（如 Skype、电子邮件、Slack 等）进行通信。 它通过从机器人到通道以及从通道到机器人的消息中继来促进机器人与用户之间的通信。 

本文介绍如何通过 Bot Framework SDK for .NET 使用 Connector 在通道上交换机器人与用户之间的信息。 

> [!NOTE]
> 虽然可以通过专门使用本文中描述的技术来构建机器人，但是 Bot Framework SDK 提供了[对话框](bot-builder-dotnet-dialogs.md)和 [FormFlow](bot-builder-dotnet-formflow.md) 之类的附加功能，这些功能可以简化管理聊天流和状态的过程，并使融合认知服务（例如语言理解）变得更加简单。

## <a name="create-a-connector-client"></a>创建连接器客户端

[ConnectorClient][ConnectorClient]类包含机器人用来在某频道上与用户通信的方法。 当机器人从 Connector 接收 <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> 对象时，它应该使用为该活动指定的 `ServiceUrl` 来创建它随后用于生成响应的连接器客户端。 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> 由于通道的终结点可能不稳定，因此机器人应尽可能将通信定向到 Connector 在 `Activity` 对象中指定的终结点（而不是依赖于缓存的终结点）。 
>
> 如果机器人需要发起会话，它可以为指定的通道使用缓存的终结点（因为在该场景中不会有传入的 `Activity` 对象），但它应该经常刷新缓存的终结点。 

## <a id="create-reply"></a>创建回复

Connector 使用 [Activity](bot-builder-dotnet-activities.md) 对象在机器人和通道（用户）之间来回传递信息。 每个活动都包含用于将消息路由到合适目的地的信息，以及有关创建消息的人员（`From` 属性）、消息的上下文以及消息的接收者（`Recipient` 属性）的信息。

当机器人从 Connector 接收活动时，传入活动的 `Recipient` 属性指定机器人在该会话中的身份。 因为某些通道（例如，Slack）在将机器人添加到会话时会为其分配新身份，所以机器人应始终将传入活动的 `Recipient` 属性的值用作其响应中的 `From` 属性的值。

虽然你可以自行从头开始创建和初始化传出的 `Activity` 对象，但最好是使用 Bot Framework SDK 提供的一种更简单的创建回复的方法。 通过使用传入活动的 `CreateReply` 方法，你只需指定响应的消息文本并创建传出活动，其中 `Recipient`、`From` 和 `Conversation` 属性将自动填充。

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>发送回复

创建回复后，可以通过调用连接器客户端的 `ReplyToActivity` 方法发送回复。 Connector 将使用合适的通道语义传递回复。 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> 如果机器人正在回复用户的消息，应始终使用 `ReplyToActivity` 方法。

## <a name="send-a-non-reply-message"></a>发送（不回复）消息 

如果机器人是会话的一部分，它可以通过调用 `SendToConversation` 方法发送一条消息，该消息不是对用户的任何消息的直接回复。 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

可以使用 `CreateReply` 方法初始化新消息（这将自动设置消息的 `Recipient`、`From` 和 `Conversation` 属性）。 或者，可以使用 `CreateMessageActivity` 方法来创建新消息并自行设置所有属性值。

> [!NOTE]
> Bot Framework 未对机器人可能发送的消息数量施加任何限制。 但是，大多数通道都会强制实施限制，以限制机器人在短时间内发送大量的消息。 此外，如果机器人快速连续地发送多条消息，则通道可能无法始终以正确的顺序呈现消息。

## <a name="start-a-conversation"></a>启动会话

可能有机器人需要与一个或多个用户发起会话的时候。 可以通过调用 `CreateDirectConversation` 方法（适用于与单个用户的私人会话）或 `CreateConversation` 方法（适用于与多个用户的群组会话）检索 `ConversationAccount` 对象来开始会话。 然后，通过调用 `SendToConversation` 方法创建消息并发送。 若要使用 `CreateDirectConversation` 方法或 `CreateConversation` 方法，必须首先使用目标通道的服务 URL（如果从以前的消息中保留了此 URL，则可以从缓存中检索）[创建连接器客户端](#create-a-connector-client)。 

> [!NOTE]
> 并非所有通道都支持群组会话。 若要确定一个通道是否支持群组会话，请参阅通道的文档。

此代码示例使用 `CreateDirectConversation` 方法创建与单个用户的私人会话。

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

此代码示例使用 `CreateConversation` 方法创建与多个用户的群组会话。

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>其他资源

- [活动概述](bot-builder-dotnet-activities.md)
- [创建消息](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET 参考</a>
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">ConnectorClient 类</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient
