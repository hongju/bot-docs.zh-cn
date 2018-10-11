---
title: 实体和活动类型 | Microsoft Docs
description: 实体和活动类型。
keywords: 提及实体, 活动类型, 使用实体
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/01/2018
ms.openlocfilehash: f6bf1d99922351a66a4e5401e744fad190746747
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389802"
---
# <a name="entities-and-activity-types"></a>实体和活动类型

实体是活动的一部分，提供有关活动或会话的更多信息。

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>实体

消息的 entities 属性是一组开放式 <a href="http://schema.org/" target="_blank">schema.org</a> 对象，它允许在通道和机器人之间交换公共上下文元数据。

### <a name="mention-entities"></a>Mention 实体

许多通道支持机器人或用户在会话上下文中“提及”某人的功能。
要在消息中提及某个用户，请使用 mention 对象填充消息的 entities 属性。
mention 对象包含以下属性：

| 属性 | Description |
|----|----|
| Type | 实体（“mention”）的类型 |
| Mentioned | 指示提及了哪个用户的通道帐户对象 | 
| 文本 | activity.text 属性中的文本，表示 mention 本身（可能为空或为 null） |

此代码示例演示如何将提及实体添加到实体集合。

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> 在尝试确定用户意向时，机器人可能希望忽略消息中提到它的部分。 调用 `GetMentions` 方法并评估答复中返回的 `Mention` 对象。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>Place 对象

<a href="https://schema.org/Place" target="_blank">与位置相关的信息</a>可以在消息中传输，具体方法是使用 Place 对象或 GeoCoordinates 对象填充消息的 entities 属性。

place 对象包含以下属性：

| 属性 | Description |
|----|----|
| Type | 实体（“Place”）的类型 |
| 地址 | 说明或邮寄地址对象（未来） |
| 地域 | 地理坐标 |
| HasMap | 地图或地图对象的 URL（未来） |
| 名称 | 位置的名称 |

geoCoordinates 对象包含以下属性：

| 属性 | Description |
|----|----|
| Type | 实体（“GeoCoordinates”）类型 |
| 名称 | 位置的名称 |
| 经度 | 位置的经度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| 经度 | 位置的纬度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |
| Elevation | 位置的海拔高度 (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

此代码示例演示如何将位置实体添加到实体集合：

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>使用实体

# <a name="ctabcs"></a>[C#](#tab/cs)

要使用实体，请使用 `dynamic` 关键字或强类型类。

此代码示例演示如何使用 `dynamic` 关键字处理消息的 `Entities` 属性中的实体：

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

此代码示例演示如何使用强类型类处理消息的 `Entities` 属性中的实体：

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

此代码示例演示如何处理消息的 `entity` 属性中的实体：

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>活动类型

此代码示例显示如何处理 message 类型的活动：

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

活动可以是最常见的 message 之后的几种不同类型。 有几种活动类型：

| Activity.Type | 接口 | Description |
|-----|-----|-----|
| [message](#message) | IMessageActivity (C#) <br> Activity (JS) | 表示机器人和用户之间的通信。 |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity (C#) <br> Activity (JS) | 指示已将机器人添加到用户的联系人列表或已从其中删除。 |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity (C#) <br> Activity (JS) | 指示机器人已添加到会话中、其他成员已添加到会话或从会话中删除，或者会话元数据已更改。 |
| [deleteUserData](#deleteuserdata) | 不适用 | 向机器人表明用户已请求机器人删除其可能存储的所有用户数据。 |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity (C#) <br> Activity (JS) | 指示会话结束。 |
| [event](#event) | IEventActivity (C#) <br> Activity (JS) | 表示发送到用户不可见的机器人的通信。 |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity (C#) <br> Activity (JS) | 表示在通道的组织单位（例如客户租户或“团队”）内安装或卸载机器人。 |
| [invoke](#invoke) | IInvokeActivity (C#) <br> Activity (JS) | 表示发送到机器人以请求它执行特定操作的通信。 保留此活动类型以供 Microsoft Bot Framework 内部使用。 |
| [messageReaction](#messagereaction) | IMessageReactionActivity (C#) <br> Activity (JS) | 指示用户已对现有活动做出反应。 例如，用户单击消息上的“赞”按钮。 |
| [typing](#typing) | ITypingActivity (C#) <br> Activity (JS) | 指示位于聊天另一端的用户或机器人正在编写答复。 |

## <a name="message"></a>message

<!-- Only the last link is different. -->
::: moniker range="azure-bot-service-3.0"
机器人将发送消息活动向用户传达信息，并接收来自用户的消息活动。
某些消息可能只包含纯文本，而其他消息可能包含更丰富的内容，例如[要说的文本](v4sdk/bot-builder-howto-send-messages.md#send-a-spoken-message)、[建议的操作](v4sdk/bot-builder-howto-add-suggested-actions.md)、[媒体附件](v4sdk/bot-builder-howto-add-media-attachments.md)、[富卡](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card)和[特定于通道的数据](~/dotnet/bot-builder-dotnet-channeldata.md)。
::: moniker-end
::: moniker range="azure-bot-service-4.0"
机器人将发送消息活动向用户传达信息，并接收来自用户的消息活动。
某些消息可能只包含纯文本，而其他消息可能包含更丰富的内容，例如[要说的文本](v4sdk/bot-builder-howto-send-messages.md#send-a-spoken-message)、[建议的操作](v4sdk/bot-builder-howto-add-suggested-actions.md)、[媒体附件](v4sdk/bot-builder-howto-add-media-attachments.md)、[富卡](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card)和[特定于通道的数据](~/v4sdk/bot-builder-channeldata.md)。
::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

无论何时将机器人添加到用户的联系人列表或从用户的联系人列表中删除，机器人都会收到 contact relation update 活动。 活动的 action 属性的值 (add | remove) 指示是已将机器人添加到用户的联系人列表还是已将其从列表中删除。

## <a name="conversationupdate"></a>conversationUpdate

无论何时将机器人添加到会话，其他成员已添加到会话或从会话中删除，或者会话元数据已更改，机器人都会收到 conversation update 活动。

如果成员已添加到会话中，则活动的 members added 属性将包含一组通道帐户对象以标识新成员。

若要确定机器人是否已添加到会话中（即，是新成员之一），请评估活动的收件人 ID 值（即机器人 ID）是否与 members added 数组中任何帐户的 Id 属性相匹配。

如果成员已从会话中删除，则 members removed 属性将包含一组通道帐户对象以标识已删除的成员。

> [!TIP]
> 如果机器人收到 conversation update 活动，指示用户已加入会话，可以选择通过向该用户发送欢迎消息来使其响应。

## <a name="deleteuserdata"></a>deleteUserData

当用户请求删除机器人之前为其保留的任何数据时，机器人会收到 delete user data 活动。 如果机器人收到此类活动，则应删除之前为发出请求的用户存储的任何个人身份信息 (PII)。

## <a name="endofconversation"></a>endOfConversation

机器人接收到 end of conversation 活动以指示用户已结束会话。 机器人可以发送 end of Conversation 活动以向用户指示正在结束会话。

## <a name="event"></a>event

机器人可能会从外部流程或服务收到 event 活动，该活动用于向机器人传达用户无法看到的信息。 event 活动的发送者通常不希望机器人以任何方式确认收到。

## <a name="installationupdate"></a>installationUpdate

安装更新活动表示在通道的组织单位（例如客户租户或“团队”）内安装或卸载机器人。 安装更新活动通常不表示添加或删除通道。 在通道内的租户、团队或其他组织单位中添加或删除机器人时，通道可以发送安装活动。 在通道内安装或删除机器人时，通道不应当发送安装活动。

## <a name="invoke"></a>invoke

机器人可能会收到 invoke 活动，该活动表示请求机器人执行特定操作。
invoke 活动的发送者通常希望机器人通过 HTTP 响应确认收到。
保留此活动类型以供 Microsoft Bot Framework 内部使用。

## <a name="messagereaction"></a>messageReaction

当用户对现有活动做出反应时，某些通道会向机器人发送消息反应活动。 例如，用户单击消息上的“赞”按钮。 replyToId 属性将指示用户响应的活动。

message reaction 活动可以对应于通道定义的任意数量的消息反应类型。 例如，将“Like”或“PlusOne”作为通道可以发送的反应类型。

## <a name="typing"></a>typing

机器人会收到 typing 活动，该活动指示用户正在键入响应。
机器人可以发送 typing 活动，向用户表明它正在努力完成请求或编译响应。

::: moniker range="azure-bot-service-3.0"
## <a name="additional-resources"></a>其他资源

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
::: moniker-end
