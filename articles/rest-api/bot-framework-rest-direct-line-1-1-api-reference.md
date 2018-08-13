---
title: API 参考 - Direct Line API 1.1 | Microsoft Docs
description: 了解 Direct Line API 1.1 中的标头、HTTP 状态代码、架构、操作和对象。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2f688b9c80e762b93c2eba8f4671ff1760f624f9
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298296"
---
# <a name="api-reference---direct-line-api-11"></a>API 参考 - Direct Line API 1.1

> [!IMPORTANT]
> 本文包含 Direct Line API 1.1 的参考信息。 如果要在客户端应用程序和机器人之间创建新连接，请改用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-api-reference.md)。

可以使用 Direct Line API 1.1 通过客户端应用程序与机器人通信。 Direct Line API 1.1 通过 HTTPS 使用行业标准的 REST 和 JSON。

## <a name="base-uri"></a>基本 URI

若要访问 Direct Line API 1.1，请对所有 API 请求使用以下基本 URI：

`https://directline.botframework.com`

## <a name="headers"></a>标头

除了标准 HTTP 请求标头，Direct Line API 请求还必须包含一个 `Authorization` 标头，用于指定对发出请求的客户端进行身份验证所需的机密或令牌。 可以使用“Bearer”或“BotConnector”方案指定 `Authorization` 标头。 

**Bearer 方案**：
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**BotConnector 方案**：
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

若要详细了解如何获取可供客户端用于进行 Direct Line API 请求身份验证的机密或令牌，请参阅[身份验证](bot-framework-rest-direct-line-1-1-authentication.md)。

## <a name="http-status-codes"></a>HTTP 状态代码

随每个响应返回的 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 状态代码</a>指示相应请求的结果。 

| HTTP 状态代码 | 含义 |
|----|----|
| 200 | 请求成功。 |
| 204 | 请求成功，但未返回任何内容。 |
| 400 | 请求格式不正确或者其他方面不正确。 |
| 401 | 客户端未获授权，无法发出请求。 通常情况下，出现此状态代码是因为 `Authorization` 标头缺失或格式不正确。 |
| 403 | 不允许客户端执行请求的操作。 通常情况下，出现此状态代码是因为 `Authorization` 标头指定的令牌或机密无效。 |
| 404 | 找不到所请求的资源。 通常情况下，此状态代码指示请求 URI 无效。 |
| 500 | Direct Line 服务中出现内部服务器错误，或者机器人内部出现故障。 如果在通过 POST 方法将消息发布到机器人时收到 500 错误，则可能是因为机器人中出现的故障触发了该错误。 **这是常见错误代码。** |

## <a name="token-operations"></a>令牌操作 
使用以下操作创建或刷新可供客户端用来访问单个聊天的令牌。

| Operation | Description |
|----|----|
| [生成令牌](#generate-token) | 生成适用于新聊天的令牌。 | 
| [刷新令牌](#refresh-token) | 刷新一个令牌。 | 

### <a name="generate-token"></a>生成令牌
生成对一个聊天有效的令牌。 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 一个表示令牌的字符串 | 

### <a name="refresh-token"></a>刷新令牌
刷新令牌。 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 一个表示新令牌的字符串 | 

## <a name="conversation-operations"></a>聊天操作 
使用以下操作与机器人展开聊天，并在客户端和机器人之间交换消息。

| Operation | Description |
|----|----|
| [开始聊天](#start-conversation) | 与机器人展开新的聊天。 | 
| [Get Messages](#get-messages)（获取消息） | 从机器人检索消息。 |
| [发送消息](#send-a-message) | 向机器人发送消息。 | 
| [上传和发送文件](#upload-send-files) | 以附件形式上传和发送文件。 |

### <a name="start-conversation"></a>开始聊天
与机器人展开新的聊天。 
```http 
POST /api/conversations
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 一个[聊天](#conversation-object)对象 | 

### <a name="get-messages"></a>获取消息
从机器人检索指定聊天的消息。 设置请求 URI 中的 `watermark` 参数，以便指示客户端看到的最新消息。 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 一个 [MessageSet](#messageset-object) 对象。 此响应包含的 `watermark` 用作 `MessageSet` 对象的属性。 客户端应通过增加 `watermark` 值来按页浏览提供的消息，直到不返回任何消息。 | 

### <a name="send-a-message"></a>发送消息
向机器人发送消息。 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **请求正文** | 一个[消息](#message-object)对象 |
| **返回** | 响应正文中不返回任何数据。 如果消息发送成功，服务会使用 HTTP 204 状态代码进行响应。 客户端可能会通过[获取消息](#get-messages)操作获取其发送的消息（以及机器人发送给客户端的任何消息）。 |

### <a id="upload-send-files"></a> 上传和发送文件
以附件形式上传和发送文件。 在请求 URI 中设置 `userId` 参数，以便指定发送附件的用户的 ID。
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **请求正文** | 对于单个附件，请在请求正文中填充文件内容。 对于多个附件，请创建一个多部件请求正文，将其中的一个部件用于每个附件，并选择性地将另一个部件用于[消息](#message-object)对象，该对象应该充当指定附件的容器。 有关详细信息，请参阅[向机器人发送消息](bot-framework-rest-direct-line-1-1-send-message.md)。 |
| **返回** | 响应正文中不返回任何数据。 如果消息发送成功，服务会使用 HTTP 204 状态代码进行响应。 客户端可能会通过[获取消息](#get-messages)操作获取其发送的消息（以及机器人发送给客户端的任何消息）。 | 

> [!NOTE]
> 上传的文件在 24 小时后删除。

## <a name="schema"></a>架构

Direct Line 1.1 架构是 Bot Framework v1 架构的简化副本，其中包含以下对象。

### <a name="message-object"></a>消息对象

定义客户端发送给机器人或者从机器人接收的消息。

| 属性 | Type | Description |
|----|----|----|
| **id** | 字符串 | 用于唯一标识消息的 ID（由 Direct Line 分配）。 | 
| **conversationId** | 字符串 | 用于标识聊天的 ID。  | 
| **created** | 字符串 | 消息的创建日期和时间，以 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 格式表示。 | 
| **from** | 字符串 | 用于标识用户的 ID，而该用户是消息的发送者。 创建消息时，客户端应将此属性设置为一个稳定的用户 ID。 虽然 Direct Line 会在未提供任何用户 ID 的情况下分配一个用户 ID，但这样做通常会导致异常行为。 | 
| **text** | 字符串 | 从用户发送给机器人或从机器人发送给用户的消息文本。 | 
| **channelData** | 对象 | 一个包含特定于通道的内容的对象。 某些通道提供的功能需要其他信息，而这些信息不能使用附件架构来表示。 对于此类情况，请按通道文档中的定义将此属性设置为特定于通道的内容。 此数据在客户端和机器人之间发送时，不进行修改。 此属性必须设置为复杂对象或者保留为空。 请勿将它设置为字符串、数字或其他简单类型。 | 
| **images** | string[] | 字符串数组，其中包含的 URL 是消息所含图像的。 在某些情况下，此数组中的字符串可以是相对 URL。 如果此数组中的任何字符串不以“http”或“https”开头，请在该字符串前面预置 `https://directline.botframework.com`，形成完整的 URL。 | 
| **attachments** | [Attachment](#attachment-object)[] | **附件**对象的数组，此类对象表示消息包含的非图像附件。 此数组中的每个对象都包含 `url` 属性和 `contentType` 属性。 在客户端从机器人接收的消息中，`url` 属性有时可能会指定相对 URL。 对于不以“http”或“https”开头的 `url` 属性值，请在字符串前面预置 `https://directline.botframework.com`，形成完整的 URL。 | 

以下示例显示的消息对象包含所有可能的属性。 大多数情况下，在创建消息时，客户端只需提供 `from` 属性和至少一个内容属性（例如 `text`、`images`、`attachments` 或 `channelData`）。

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>MessageSet 对象 
定义一组消息。<br/><br/>

| 属性 | Type | Description |
|----|----|----|
| **messages** | [Message](#message-object)[] | **消息**对象的数组。 |
| **watermark** | 字符串 | 组中消息的最大水印。 客户端可以使用 `watermark` 值来指示它在[从机器人处检索消息](bot-framework-rest-direct-line-1-1-receive-messages.md)时看到的最新消息。 |

### <a name="attachment-object"></a>附件对象
定义非图像附件。<br/><br/> 

| 属性 | Type | Description |
|----|----|----|
| **contentType** | 字符串 | 附件中内容的媒体类型。 |
| **url** | 字符串 | 附件内容的 URL。 |

### <a name="conversation-object"></a>聊天对象
定义 Direct Line 聊天。<br/><br/>

| 属性 | Type | Description |
|----|----|----|
| **conversationId** | 字符串 | 一个 ID，可以唯一标识指定的令牌所适用的聊天。 |
| **token** | 字符串 | 一个令牌，适用于指定的聊天。 |
| **expires_in** | 数字 | 令牌过期前需经历的秒数。 |

### <a name="error-object"></a>错误对象
定义一个错误。<br/><br/> 

| 属性 | Type | Description |
|----|----|----|
| **code** | 字符串 | 错误代码。 下列值之一：**MissingProperty**、**MalformedData**、**NotFound**、**ServiceError**、**Internal**、**InvalidRange**、**NotSupported**、**NotAllowed**、**BadCertificate**。 |
| **message** | 字符串 | 对错误的说明。 |
| **statusCode** | 数字 | 状态代码。 |

### <a name="errormessage-object"></a>ErrorMessage 对象
标准化消息错误有效负载。<br/><br/> 


|        属性        |          Type          |                                 Description                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [错误](#error-object) | 一个<strong>错误</strong>对象，包含有关错误的信息。 |

