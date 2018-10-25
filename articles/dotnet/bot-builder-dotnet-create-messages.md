---
title: 通过 Bot Builder SDK for .NET 创建消息 | Microsoft Docs
description: 了解 Bot Builder SDK for .NET 中的常用消息属性。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f3f40e41605bfd309d480465a05b91acf5a52d5c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000384"
---
# <a name="create-messages"></a>创建消息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

机器人将发送消息[活动](bot-builder-dotnet-activities.md)向用户传递信息，同样，也将收到来自用户的消息活动。 某些消息可能只包含纯文本，而其他消息可能包含更丰富的内容，例如[要说的话](bot-builder-dotnet-text-to-speech.md)、[建议的操作](bot-builder-dotnet-add-suggested-actions.md)、[媒体附件](bot-builder-dotnet-add-media-attachments.md)、[资讯卡](bot-builder-dotnet-add-rich-card-attachments.md)和[通道特定的数据](bot-builder-dotnet-channeldata.md)。 

本文介绍了一些常用的消息属性。

## <a name="customizing-a-message"></a>自定义消息

要更好地控制消息的文本格式，可以使用 [Activity](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) 对象创建自定义消息并设置必要的属性，然后将其发送给用户。

此示例演示如何创建自定义 `message` 对象并设置 `Text`、`TextFormat` 和 `Local` 属性。

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

消息的 `TextFormat` 属性可用于指定文本的格式。 `TextFormat` 属性可以设置为“纯本文”、“markdown”或“xml”。 `TextFormat` 的默认值为“markdown”。 

## <a name="attachments"></a>附件

消息活动的 `Attachments` 属性可用于发送和接收简单的媒体附件（图像、音频、视频及文件）和资讯卡。 有关详细信息，请参阅[向消息添加媒体附件](bot-builder-dotnet-add-media-attachments.md)和[向消息添加资讯卡](bot-builder-dotnet-add-rich-card-attachments.md)。

## <a name="entities"></a>实体

消息的 `Entities` 属性是一组开放式的 <a href="http://schema.org/" target="_blank">schema.org</a> 对象，它允许在通道和机器人之间交换通用上下文元数据。

### <a name="mention-entities"></a>Mention 实体

许多通道支持机器人或用户在聊天上下文中“提及”某人。 要在消息中提及某位用户，请使用 `Mention` 对象填充消息的 `Entities` 属性。 `Mention` 对象包含以下属性： 

| 属性 | 说明 | 
|----|----|
| 类型 | 实体（“mention”）的类型 | 
| Mentioned | `ChannelAccount` 对象，它指示提及了哪位用户 | 
| 文本 | `Activity.Text` 属性中的文本，它表示 mention 本身（可能为空或为 NULL） |

此代码示例演示如何向 `Entities` 集合添加 `Mention` 实体。

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> 在尝试确定用户意向时，机器人可能希望忽略消息中提到它的部分。 调用 `GetMentions` 方法并评估答复中返回的 `Mention` 对象。

### <a name="place-objects"></a>Place 对象

可在消息中传输<a href="https://schema.org/Place" target="_blank">位置相关信息</a>，方法是使用 `Place` 对象或 `GeoCoordinates` 对象填充消息的 `Entities` 属性。 

`Place` 对象包含以下属性：

| 属性 | 说明 | 
|----|----|
| 类型 | 实体（“Place”）的类型 |
| 地址 | 说明或 `PostalAddress` 对象（未来值） | 
| 地域 | 地理坐标 | 
| HasMap | 地图或 `Map` 对象的 URL（未来值） |
| 名称 | 位置的名称 |

`GeoCoordinates` 对象包含以下属性：

| 属性 | 说明 | 
|----|----|
| 类型 | 实体（“GeoCoordinates”）类型 |
| 名称 | 位置的名称 |
| 经度 | 位置的经度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| 纬度 | 位置的纬度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 
| Elevation | 位置的海拔高度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) | 

此代码示例演示如何向 `Entities` 集合添加 `Place` 实体：

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>使用实体

要使用实体，请使用 `dynamic` 关键字或强类型类。

此代码示例演示如何使用 `dynamic` 关键字处理消息的 `Entities` 属性中的实体：

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

此代码示例演示如何使用强类型类处理消息的 `Entities` 属性中的实体：

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>通道数据

消息活动的 `ChannelData` 的属性可用于实现通道特定的功能。 有关详细信息，请参阅[实现通道特定的功能](bot-builder-dotnet-channeldata.md)。

## <a name="text-to-speech"></a>文本到语音转换

消息活动的 `Speak` 属性可用于指定机器人要在启用了语音的通道上说出的文本。 消息活动的 `InputHint` 属性可用于控制客户端麦克风和输入框（如有）的状态。 有关详细信息，请参阅[向消息添加语音](bot-builder-dotnet-text-to-speech.md)。

## <a name="suggested-actions"></a>建议的操作

消息活动的 `SuggestedActions` 属性可用于显示用户可点击以提供输入的按钮。 与资讯卡中显示的按钮（即使在点击后仍然可见且可供用户访问）不同，建议的操作窗格中显示的按钮将在用户进行选择后消失。 有关详细信息，请参阅[向消息添加建议的操作](bot-builder-dotnet-add-suggested-actions.md)。

## <a name="next-steps"></a>后续步骤

机器人和用户可互发消息。 消息更复杂时，机器人可随附消息向客户发送资讯卡。 资讯卡包含大多数机器人常需的许多演示和交互方案。

> [!div class="nextstepaction"]
> [随附消息发送资讯卡](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>其他资源

- [活动概述](bot-builder-dotnet-activities.md)
- [发送和接收活动](bot-builder-dotnet-connector.md)
- [向消息添加媒体附件](bot-builder-dotnet-add-media-attachments.md)
- [向消息添加资讯卡](bot-builder-dotnet-add-rich-card-attachments.md)
- [向消息添加语音](bot-builder-dotnet-text-to-speech.md)
- [向消息添加建议的操作](bot-builder-dotnet-add-suggested-actions.md)
- [实现通道特定的功能](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">IMessageActivity 接口</a>

