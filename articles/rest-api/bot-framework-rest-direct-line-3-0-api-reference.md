---
title: API 参考 - Direct Line API 3.0 | Microsoft Docs
description: 了解 Direct Line API 3.0 中的标头、HTTP 状态代码、架构、操作和对象。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: d69f1f658520790ff429ecd25a190319e321164d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998100"
---
# <a name="api-reference---direct-line-api-30"></a>API 参考 - Direct Line API 3.0

可以使用 Direct Line API 3.0 通过客户端应用程序与机器人通信。 Direct Line API 3.0 通过 HTTPS 使用行业标准的 REST 和 JSON。

## <a name="base-uri"></a>基本 URI

若要访问 Direct Line API 3.0，请对所有 API 请求使用以下基本 URI：

`https://directline.botframework.com`

## <a name="headers"></a>标头

除了标准 HTTP 请求标头，Direct Line API 请求还必须包含一个 `Authorization` 标头，用于指定对发出请求的客户端进行身份验证所需的机密或令牌。 使用以下格式指定 `Authorization` 标头：

```http
Authorization: Bearer SECRET_OR_TOKEN
```

若要详细了解如何获取可供客户端用于进行 Direct Line API 请求身份验证的机密或令牌，请参阅[身份验证](bot-framework-rest-direct-line-3-0-authentication.md)。

## <a name="http-status-codes"></a>HTTP 状态代码

随每个响应返回的 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 状态代码</a>指示相应请求的结果。 

| HTTP 状态代码 | 含义 |
|----|----|
| 200 | 请求成功。 |
| 201 | 请求成功。 |
| 202 | 已接受请求，将进行处理。 |
| 204 | 请求成功，但未返回任何内容。 |
| 400 | 请求格式不正确或者其他方面不正确。 |
| 401 | 客户端未获授权，无法发出请求。 通常情况下，出现此状态代码是因为 `Authorization` 标头缺失或格式不正确。 |
| 403 | 不允许客户端执行请求的操作。 如果请求指定了之前有效但现已过期的令牌，则 [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) 对象中已返回的[错误](bot-framework-rest-connector-api-reference.md#error-object)的 `code` 属性将设置为 `TokenExpired`。 |
| 404 | 找不到所请求的资源。 通常情况下，此状态代码指示请求 URI 无效。 |
| 500 | Direct Line 服务中发生了内部服务器错误。 |
| 502 | 机器人不可用或返回了错误。 这是常见错误代码。 |

> [!NOTE]
> HTTP 状态代码 101 在 WebSocket 连接路径中使用，尽管 WebSocket 客户端很有可能会处理此代码。

### <a name="errors"></a>错误

指定 4xx 范围或 5xx 范围内的 HTTP 状态代码的任何响应都将在响应正文中包含 [ErrorResponse](bot-framework-rest-connector-api-reference.md#errorresponse-object) 对象，该对象提供错误相关信息。 如果收到 4xx 范围中的错误响应，请检查 ErrorResponse 对象以确定错误原因并在重新提交请求之前解决问题。

> [!NOTE]
> 在 ErrorResponse 对象内的 `code` 属性中指定的 HTTP 状态代码和值是稳定的。 在 ErrorResponse 对象内的 `message` 属性中指定的值可能会随时间而变化。

以下片段显示了示例请求和生成的错误响应。

#### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>响应
```http
HTTP/1.1 502 Bad Gateway
[other headers]
```
```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>令牌操作 
使用以下操作创建或刷新可供客户端用来访问单个聊天的令牌。

| Operation | Description |
|----|----|
| [生成令牌](#generate-token) | 生成适用于新聊天的令牌。 | 
| [刷新令牌](#refresh-token) | 刷新一个令牌。 | 

### <a name="generate-token"></a>生成令牌
生成对一个聊天有效的令牌。 
```http 
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [Conversation](#conversation-object) 对象 | 

### <a name="refresh-token"></a>刷新令牌
刷新令牌。 
```http 
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [Conversation](#conversation-object) 对象 | 

## <a name="conversation-operations"></a>聊天操作 
使用以下操作与机器人开启聊天，并在客户端和机器人之间交换活动。

| Operation | Description |
|----|----|
| [启动聊天](#start-conversation) | 与机器人开启新的聊天。 | 
| [获取聊天信息](#get-conversation-information) | 获取现有聊天的相关信息。 此操作生成客户端可用于[重新连接](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)到聊天的新 WebSocket 流 URL。 |
| [获取活动](#get-activities) | 从机器人检索活动。 |
| [发送活动](#send-an-activity) | 向机器人发送活动。 | 
| [上传和发送文件](#upload-send-files) | 以附件形式上传和发送文件。 |

### <a name="start-conversation"></a>启动聊天
与机器人开启新的聊天。 
```http 
POST /v3/directline/conversations
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [Conversation](#conversation-object) 对象 | 

### <a name="get-conversation-information"></a>获取聊天信息
获取有关现有聊天的信息，并生成客户端可用于[重新连接](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)到聊天的新 WebSocket 流 URL。 可以选择提供请求 URI 中的 `watermark` 参数，以指示客户端看到的最新消息。
```http 
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [Conversation](#conversation-object) 对象 | 

### <a name="get-activities"></a>获取活动
从机器人检索指定聊天的活动。 可以选择提供请求 URI 中的 `watermark` 参数，以指示客户端看到的最新消息。 

```http 
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [ActivitySet](#activityset-object) 对象。 此响应包含的 `watermark` 用作 `ActivitySet` 对象的属性。 客户端应通过增加 `watermark` 值来浏览可用活动，直到不返回任何活动。 | 

### <a name="send-an-activity"></a>发送活动
向机器人发送活动。 
```http 
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **请求正文** | [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 对象 |
| **返回** | 将 [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object) 发送到机器人，其中包含指定活动 ID 的 `id` 属性。 | 

### <a id="upload-send-files"></a> 上传和发送文件
以附件形式上传和发送文件。 在请求 URI 中设置 `userId` 参数，以便指定发送附件的用户的 ID。
```http 
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **请求正文** | 对于单个附件，请在请求正文中填充文件内容。 对于多个附件，请创建一个多部件请求正文，将其中的一个部件用于每个附件，并选择性地将另一个部件用于 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 对象，该对象应该充当指定附件的容器。 有关详细信息，请参阅[向机器人发送活动](bot-framework-rest-direct-line-3-0-send-activity.md)。 |
| **返回** | 将 [ResourceResponse](bot-framework-rest-connector-api-reference.md#resourceresponse-object) 发送到机器人，其中包含指定活动 ID 的 `id` 属性。 | 

> [!NOTE]
> 上传的文件在 24 小时后删除。

## <a name="schema"></a>架构

Direct Line 3.0 架构包含所有由 [Bot Framework v3 架构](bot-framework-rest-connector-api-reference.md#objects)指定的对象以及 `ActivitySet` 对象和 `Conversation` 对象。

### <a name="activityset-object"></a>ActivitySet 对象 
定义一组活动。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **activities** | [Activity](bot-framework-rest-connector-api-reference.md#activity-object)[] | Activity 对象的数组。 |
| **watermark** | 字符串 | 集内活动的最大水印。 客户端可以使用 `watermark` 值来指示[从机器人检索活动](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get)时或[生成 WebSocket 流 URL](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) 时所看到的最新消息。 |

### <a name="conversation-object"></a>Conversation 对象
定义 Direct Line 聊天。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **conversationId** | 字符串 | 一个 ID，可以唯一标识指定的令牌所适用的聊天。 |
| **expires_in** | 数字 | 令牌过期前需经历的秒数。 |
| **streamUrl** | 字符串 | 聊天的消息流的 URL。 |
| **token** | 字符串 | 对指定的聊天有效的令牌。 |

### <a name="activities"></a>活动

对于客户通过 Direct Line 从机器人接收的每个[活动](bot-framework-rest-connector-api-reference.md#activity-object)：

- 保留附件卡。
- 以私有链接方式隐藏已上传的附件的 URL。
- `channelData` 属性已保留且未经修改。 

客户端可以从机器人[接收](bot-framework-rest-direct-line-3-0-receive-activities.md)多个活动作为 [ActivitySet](#activityset-object) 的一部分。 

客户端通过 Direct Line 向机器人发送[活动](bot-framework-rest-connector-api-reference.md#activity-object)时：

- `type` 属性指定它正在发送的类型活动（通常是消息）。
- 必须使用客户端选择的用户 ID 填充 `from` 属性。
- 附件可能包含现有资源的 URL 或通过 Direct Line 附件终结点上传的 URL。
- `channelData` 属性已保留且未经修改。

客户端可以为每个请求[发送](bot-framework-rest-direct-line-3-0-send-activity.md)一个活动。 
