---
title: 活动概述 | Microsoft Docs
description: 了解 Bot Builder SDK for .NET 中可用的不同活动类型。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 834702024c99873ca9f0bbedb53a24a16ba55878
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2018
ms.locfileid: "42756474"
---
# <a name="activities-overview"></a>活动概述

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-builder-sdk-for-net"></a>Bot Builder SDK for .NET 中的活动类型

Bot Builder SDK for .NET 支持以下活动类型。

| Activity.Type | 接口 | Description |
|------|------|------|
| [message](#message) | IMessageActivity | 表示机器人和用户之间的通信。 |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | 指示机器人已添加到聊天中、其他成员已添加到聊天或从聊天中删除，或者聊天元数据已更改。 |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | 指示已将机器人添加到用户的联系人列表或已从其中删除。 |
| [typing](#typing) | ITypingActivity | 指示位于聊天另一端的用户或机器人正在编写答复。 | 
| [deleteUserData](#deleteuserdata) | 不适用 | 向机器人表明用户已请求机器人删除其可能存储的所有用户数据。 |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | 指示聊天结束。 |
| [event](#event) | IEventActivity | 表示发送到用户不可见的机器人的通信。 |
| [invoke](#invoke) | IInvokeActivity | 表示发送到机器人以请求它执行特定操作的通信。 保留此活动类型以供 Microsoft Bot Framework 内部使用。 |
| [messageReaction](#messagereaction) | IMessageReactionActivity | 指示用户已对现有活动做出反应。 例如，用户单击消息上的“赞”按钮。 |

## <a name="message"></a>message

机器人将发送“消息”活动向用户传达信息，并接收来自用户的“消息”活动。 某些消息可能只包含纯文本，而其他消息可能包含更丰富的内容，例如[要说的文本](bot-builder-dotnet-text-to-speech.md)、[建议的操作](bot-builder-dotnet-add-suggested-actions.md)、[媒体附件](bot-builder-dotnet-add-media-attachments.md)、[丰富的卡片](bot-builder-dotnet-add-rich-card-attachments.md)和[特定于通道的数据](bot-builder-dotnet-channeldata.md)。 有关常用消息属性的信息，请参阅[创建消息](bot-builder-dotnet-create-messages.md)。

## <a name="conversationupdate"></a>conversationUpdate

无论何时将机器人添加到聊天，其他成员已添加到聊天或从聊天中删除，或者聊天元数据已更改，机器人都会收到 **conversationUpdate** 活动。 

如果成员已添加到聊天中，则活动的 `MembersAdded` 属性将包含一组 `ChannelAccount` 对象以标识新成员。 

若要确定机器人是否已添加到聊天中（即，是新成员之一），请评估活动的 `Recipient.Id` 值（即机器人 ID）是否与 `MembersAdded` 数组中任何帐户的 `Id` 属性相匹配。

如果成员已从聊天中删除，则 `MembersRemoved` 属性将包含一组 `ChannelAccount` 对象，以标识已删除的成员。 

> [!TIP]
> 如果机器人收到 **conversationUpdate** 活动，指示用户已加入聊天，你可以选择通过向该用户发送欢迎消息来使其响应。 

## <a name="contactrelationupdate"></a>contactRelationUpdate

每次将机器人添加到用户的联系人列表或从中删除时，机器人都会收到 contactRelationUpdate 活动 。 活动的 `Action` 属性的值 (add | remove) 指示是已将机器人添加到用户的联系人列表还是已将其从中删除。

## <a name="typing"></a>typing

机器人会收到 typing 活动，它指示用户正在键入答复。 机器人可发送 typing 活动向用户表明它正在积极处理请求或编写答复。 

## <a name="deleteuserdata"></a>deleteUserData

当用户请求删除机器人之前为其保留的任何数据时，机器人会收到 deleteUserData 活动。 如果机器人收到此类活动，则应删除之前为发出请求的用户存储的任何个人身份信息 (PII)。

## <a name="endofconversation"></a>endOfConversation 

机器人会收到 endOfConversation 活动，它指示用户已结束聊天。 机器人可发送 endOfConversation 活动向用户表明聊天即将结束。 

## <a name="event"></a>event

机器人可能会从外部流程或服务收到 **event** 活动，该活动用于向机器人传达用户无法看到的信息。 **event** 活动的发送者通常不希望机器人以任何方式确认收到。

## <a name="invoke"></a>Invoke

机器人可能会收到 **invoke** 活动，该活动表示请求机器人执行特定操作。 **invoke** 活动的发送者通常希望机器人通过 HTTP 响应确认收到。 此活动类型保留供 Microsoft Bot Framework 内部使用。

## <a name="messagereaction"></a>messageReaction

当用户对现有活动做出反应时，某些通道会向机器人发送 **messageReaction** 活动。 例如，用户单击消息上的“赞”按钮。 **ReplyToId** 属性将指示用户响应的活动。

**messageReaction** 活动可以对应于通道定义的任意数量的 **messageReactionTypes**。 例如，将“Like”或“PlusOne”作为通道可以发送的反应类型。 

## <a name="additional-resources"></a>其他资源

- [发送和接收活动](bot-builder-dotnet-connector.md)
- [创建消息](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
