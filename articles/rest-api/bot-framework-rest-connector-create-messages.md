---
title: 使用 Bot Connector API 创建消息 | Microsoft Docs
description: 了解 Bot Connector API 中的常用消息属性。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2fbff9c6d7fe1e06fa87e5b2695056dbc1414570
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298496"
---
# <a name="create-messages"></a>创建消息

机器人将发送“消息”类型的 [Activity][Activity] 对象向用户传递信息，同样也将收到来自用户的消息活动。 某些消息可能只包含纯文本，而其他消息可能包含更丰富的内容，例如[要说的话](bot-framework-rest-connector-text-to-speech.md)、[建议的操作](bot-framework-rest-connector-add-suggested-actions.md)、[媒体附件](bot-framework-rest-connector-add-media-attachments.md)、[资讯卡](bot-framework-rest-connector-add-rich-cards.md)和[通道特定的数据](bot-framework-rest-connector-channeldata.md)。 本文介绍了一些常用的消息属性。

## <a name="message-text-and-formatting"></a>消息文本和格式设置

可使用 plain、markdown 或 xml 设置消息文本的格式。 `textFormat` 属性的默认格式是 markdown，并使用 Markdown 格式设置标准解释文本。 文本格式支持的级别因通道而异。 要了解目标通道是否支持要使用的功能，请使用 [Channel Inspector][ChannelInspector] 预览该功能。 

[Activity][Activity] 对象的 `textFormat` 属性可用于指定文本的格式。 例如，要创建只包含纯文本的基本消息，请将 [Activity][Activity] 对象的 `textFormat` 属性设置为“plain”，将 `text` 属性设置为消息内容，并将 `locale` 属性设置为发送方的区域设置。 

要查看常用文本格式的列表，请参阅[文本格式](../bot-service-channel-inspector.md#text-formatting)。

## <a name="attachments"></a>附件

[Activity][Activity] 对象的 `attachments` 的属性可用于发送简单的媒体附件（图像、音频、视频、文件）和资讯卡。 有关详细信息，请参阅[向消息添加媒体附件](bot-framework-rest-connector-add-media-attachments.md)和[向消息添加资讯卡](bot-framework-rest-connector-add-rich-cards.md)。

## <a name="entities"></a>实体

[Activity][Activity] 对象的 `entities` 属性是一组开放式 <a href="http://schema.org/" target="_blank">schema.org</a> 对象，它允许在通道和机器人之间交换通用上下文元数据。

### <a name="mention-entities"></a>Mention 实体

许多通道支持机器人或用户在聊天上下文中“提及”某人。 要在消息中提及某位用户，请使用 [Mention][Mention] 对象填充消息的 `entities` 属性。 

### <a name="place-entities"></a>位置实体

要在消息中传达<a href="https://schema.org/Place" target="_blank">与位置相关的信息</a>，请使用 [Place][Place] 对象填充消息的 `entities` 属性。 

## <a name="channel-data"></a>通道数据

[Activity][Activity] 对象的 `channelData` 的属性可用于实现通道特定的功能。 有关详细信息，请参阅[实现通道特定的功能](bot-framework-rest-connector-channeldata.md)。

## <a name="text-to-speech"></a>文本到语音转换

[Activity][Activity] 对象的 `speak` 属性可用于指定机器人要在启用语音的通道上说出的文本，[Activity][Activity] 对象的 `inputHint` 属性可用于影响客户端麦克风的状态。 有关详细信息，请参阅[向消息添加语音](bot-framework-rest-connector-text-to-speech.md)和[向消息添加输入提示](bot-framework-rest-connector-add-input-hints.md)。

## <a name="suggested-actions"></a>建议的操作

[Activity][Activity] 对象的 `suggestedActions` 属性可用于提供用户能点击进行输入的按钮。 与资讯卡中显示的按钮（即使在点击后仍然可见且可供用户访问）不同，建议的操作窗格中显示的按钮将在用户进行选择后消失。 有关详细信息，请参阅[向消息添加建议的操作](bot-framework-rest-connector-add-suggested-actions.md)。

## <a name="additional-resources"></a>其他资源

- [使用 Channel Inspector 预览功能][ChannelInspector]
- [活动概述](bot-framework-rest-connector-activities.md)
- [发送和接收消息](bot-framework-rest-connector-send-and-receive-messages.md)
- [向消息添加媒体附件](bot-framework-rest-connector-add-media-attachments.md)
- [向消息添加资讯卡](bot-framework-rest-connector-add-rich-cards.md)
- [向消息添加语音](bot-framework-rest-connector-text-to-speech.md)
- [向消息添加输入提示](bot-framework-rest-connector-add-input-hints.md)
- [向消息添加建议的操作](bot-framework-rest-connector-add-suggested-actions.md)
- [实现通道特定的功能](bot-framework-rest-connector-channeldata.md)

[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting
