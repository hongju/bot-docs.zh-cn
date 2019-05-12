---
title: 实体和活动类型 | Microsoft Docs
description: 实体和活动类型。
keywords: 提及实体, 活动类型, 使用实体
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: 9fa9a23f4d14667aeb97d304498b415f2c8041d1
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033057"
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

| 属性 | 说明 |
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

| 属性 | 说明 |
|----|----|
| Type | 实体（“Place”）的类型 |
| 地址 | 说明或邮寄地址对象（未来） |
| 地域 | 地理坐标 |
| HasMap | 地图或地图对象的 URL（未来） |
| 名称 | 位置的名称 |

geoCoordinates 对象包含以下属性：

| 属性 | 说明 |
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

有多种活动类型；活动可以是最常见的消息之后的几种不同类型。 [活动架构页](https://aka.ms/botSpecs-activitySchema)上提供了说明和更多详细信息。

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>其他资源

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity 类</a>
::: moniker-end
