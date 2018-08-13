---
title: 管理状态数据 | Microsoft Docs
description: 了解如何使用 Bot State 服务来存储和检索状态数据。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 1557941d4e5413108ea3ce788f7d5d684252b657
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297652"
---
# <a name="manage-state-data"></a>管理状态数据

Bot State 服务使机器人能够在特定聊天的上下文中存储和检索与用户、聊天或特定用户关联的状态数据。 对于通道上的每个用户、通道上的每个聊天，以及通道上聊天上下文中的每个用户，最多可以存储 32 KB 的数据。 状态数据可用于多种目的，例如确定先前聊天中断的位置或仅按名称来问候返回的用户。 如果存储用户的首选项，则可以在下次聊天时使用该信息自定义聊天。 例如，可以提醒用户注意有关他们感兴趣的主题的新闻文章，或者在约会可用时提醒用户。 

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境，该 API 可能会在将来的版本中弃用。 建议更新机器人代码以使用内存中存储进行测试，或者将 Azure 扩展之一用于生产性机器人。 有关详细信息，请参阅针对 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 实现的“管理状态数据”主题。

## <a id="concurrency"></a> 数据并发

为了控制通过 Bot State 服务存储的数据的并发，此框架将实体标记 (ETag) 用于 `POST` 请求。 此框架不使用 ETag 的标准标头。 相反，请求和响应的正文使用 `eTag` 属性来指定 ETag。 

例如，如果发出一个用于从存储检索用户数据的 `GET` 请求，则响应会包含 `eTag` 属性。 如果更改数据并通过发出 `POST` 请求将更新后的数据保存到存储中，则请求可能会包含 `eTag` 属性，该属性指定的值与之前在 `GET` 响应中接收的值相同。 如果在 `POST` 请求中指定的 ETag 与存储中的当前值相符，则服务器会保存用户的数据并使用“HTTP 200 成功”进行响应，同时在响应正文中指定新的 `eTag` 值。 如果在 `POST` 请求中指定的 ETag 与存储中的当前值不符，则服务器会使用“HTTP 412 前提条件失败”进行响应，指示用户在存储中的数据自上次保存或检索后已更改。

> [!TIP]
> `GET` 响应会始终包含 `eTag` 属性，但你不需要在 `GET` 请求中指定 `eTag` 属性。 `eTag` 属性值为星号 (`*`) 指示尚未针对包含通道、用户和聊天在内的指定组合保存数据。
>
> 可以选择在 `POST` 请求中指定 `eTag` 属性。 
> 如果并发性对于机器人来说不是问题，请勿在 `POST` 请求中包含 `eTag` 属性。 

## <a id="save-user-data"></a> 保存用户数据

若要在特定通道中保存用户的状态数据，请发出以下请求：

```http
POST /v3/botstate/{channelId}/users/{userId}
```

在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{userId}** 替换为该通道中用户的 ID。 机器人以前从用户处接收的消息中的 `channelId` 和 `from` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问用户的数据，不需从消息中提取。

将请求的正文设置为 [BotData][BotData] 对象，其中的 `data` 属性指定需保存的数据。 如果将实体标记用于[并发控制](#concurrency)，则请将 `eTag` 属性设置为上次保存或检索用户数据时在响应中接收的 ETag（具体取决于保存或检索的数据中哪一种更新）。 如果不将实体标记用于并发，则请不要在请求中包括 `eTag` 属性。

> [!NOTE]
> 只有在指定的 ETag 与服务器的 ETag 匹配或者在请求中未指定 ETag 的情况下，此 `POST` 请求才会覆盖存储中的用户数据。

### <a name="request"></a>请求 

以下示例显示了一个请求，该请求保存特定通道中某个用户的数据。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
POST https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "a1b2c3d4"
}
```

### <a name="response"></a>响应 

此响应会包含一个 [BotData][BotData] 对象，其 `eTag` 值是新的。

## <a name="get-user-data"></a>获取用户数据

若要获取以前在特定通道中为用户保存的状态数据，请发出以下请求：

```http
GET /v3/botstate/{channelId}/users/{userId}
```

在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{userId}** 替换为该通道中用户的 ID。 机器人以前从用户处接收的消息中的 `channelId` 和 `from` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问用户的数据，不需从消息中提取。

### <a name="request"></a>请求

以下示例显示一个请求，该请求获取的数据是以前为此用户保存的。 在此示例请求中，`https://smba.trafficmanager.net/apis` 表示基本 URI；机器人发出的请求的基本 URI 可能不同。 有关设置基本 URI 的详细信息，请参阅 [API 参考](bot-framework-rest-connector-api-reference.md#base-uri)。

```http
GET https://smba.trafficmanager.net/apis/v3/botstate/abcd1234/users/12345678
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

### <a name="response"></a>响应

响应会包含一个 [BotData][BotData] 对象，其中，`data` 属性包含的数据是以前为此用户保存的，`eTag` 属性包含的 ETag 是可以在后续请求中用于保存此用户的数据的。 如果以前未为此用户保存过数据，则 `data` 属性会为 `null`，`eTag` 属性会包含星号 (`*`)。

以下示例演示了对 `GET` 请求的响应：

```json
{
    "data": [
        {
            "trail": "Lake Serene",
            "miles": 8.2,
            "difficulty": "Difficult",
        },
        {
            "trail": "Rainbow Falls",
            "miles": 6.3,
            "difficulty": "Moderate",
        }
    ],
    "eTag": "xyz12345"
}
```

## <a name="save-conversation-data"></a>保存聊天数据

若要在特定通道中保存聊天的状态数据，请发出以下请求：

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{conversationId}** 替换为聊天的 ID。 机器人以前发送到聊天上下文或在其中接收的消息中的 `channelId` 和 `conversation` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问聊天的数据，不需从消息中提取。

将请求正文设置为 [BotData][BotData] 对象，如[保存用户数据](#save-user-data)中所述。

> [!IMPORTANT]
> 由于[删除用户数据](#delete-user-data)操作不会删除使用“保存聊天数据”操作存储的数据，因此，不得使用此操作来存储用户的个人身份信息 (PII)。

### <a name="response"></a>响应

此响应会包含一个 [BotData][BotData] 对象，其 `eTag` 值是新的。

## <a name="get-conversation-data"></a>获取聊天数据

若要获取以前在特定通道中为聊天保存的状态数据，请发出以下请求：

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}
```

在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{conversationId}** 替换为聊天的 ID。 机器人以前在聊天上下文中发送或接收的消息中的 `channelId` 和 `conversation` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问聊天的数据，不需从消息中提取。

### <a name="response"></a>响应

此响应会包含一个 [BotData][BotData] 对象，其 `eTag` 值是新的。

## <a name="save-private-conversation-data"></a>保存私密聊天数据

若要在特定聊天的上下文中保存用户的状态数据，请发出以下请求：

```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{conversationId}** 替换为聊天的 ID，将 **{userId}** 替换为该通道中用户的 ID。 机器人以前在聊天上下文中从用户接收的消息中的 `channelId`、`conversation` 和 `from` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问聊天的数据，不需从消息中提取。

将请求正文设置为 [BotData][BotData] 对象，如[保存用户数据](#save-user-data)中所述。

### <a name="response"></a>响应

此响应会包含一个 [BotData][BotData] 对象，其 `eTag` 值是新的。

## <a name="get-private-conversation-data"></a>获取私密聊天数据

若要获取以前在特定聊天的上下文中为用户保存的状态数据，请发出以下请求： 

```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId}
```

在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{conversationId}** 替换为聊天的 ID，将 **{userId}** 替换为该通道中用户的 ID。 机器人以前在聊天上下文中从用户接收的消息中的 `channelId`、`conversation` 和 `from` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问聊天的数据，不需从消息中提取。

### <a name="response"></a>响应

此响应会包含一个 [BotData][BotData] 对象，其 `eTag` 值是新的。

## <a name="delete-user-data"></a>删除用户数据

若要在特定通道中删除用户的状态数据，请发出以下请求：

```http
DELETE /v3/botstate/{channelId}/users/{userId}
```
在此请求 URI 中，请将 **{channelId}** 替换为通道的 ID，将 **{userId}** 替换为该通道中用户的 ID。 机器人以前从用户处接收的消息中的 `channelId` 和 `from` 属性会包含这些 ID。 也可选择将这些值缓存在某个安全的位置，这样就可以在以后访问用户的数据，不需从消息中提取。

> [!IMPORTANT]
> 此操作删除以前使用[保存用户数据](#save-user-data)操作或[保存私密聊天数据](#save-private-conversation-data)操作为用户存储的数据。 它不删除以前使用[保存聊天数据](#save-conversation-data)操作存储的数据。 因此，不得使用“保存聊天数据”操作来存储用户的个人身份信息 (PII)。

机器人在收到 [deleteUserData](bot-framework-rest-connector-activities.md#deleteuserdata) 类型的[活动][Activity]或 [contactRelationUpdate](bot-framework-rest-connector-activities.md#contactrelationupdate) 类型的活动（指示已从用户的联系人列表中删除机器人）时，应执行“删除用户数据”操作。

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-connector-concepts.md)
- [活动概述](bot-framework-rest-connector-activities.md)

[BotData]: bot-framework-rest-connector-api-reference.md#botdata-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
