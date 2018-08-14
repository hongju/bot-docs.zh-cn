---
title: 向机器人发送消息 | Microsoft Docs
description: 了解如何使用 Direct Line API v1.1 向机器人发送消息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 3bc56d08f45ffd1e389a2dca1868a788d65e087e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298372"
---
# <a name="send-a-message-to-the-bot"></a>向机器人发送消息

> [!IMPORTANT]
> 本文介绍如何使用 Direct Line API v1.1 向机器人发送消息。 如果要在客户端应用程序和机器人之间创建新连接，请改用 [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-send-activity.md)。

通过使用 Direct Line 1.1 协议，客户端可以与机器人交换消息。 这些消息将转换为机器人支持的架构（Bot Framework v1 或 Bot Framework v3）。 客户端可为每个请求发送一条消息。 

## <a name="send-a-message"></a>发送消息

若要将消息发送给机器人，客户端必须创建 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 对象来定义消息，然后发出 `POST` 请求到 `https://directline.botframework.com/api/conversations/{conversationId}/messages`，指定请求正文中的 Message 对象。

以下代码片段提供了 Send Message 请求和响应的示例。

### <a name="request"></a>请求

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>响应

将消息传递到机器人时，服务通过反映机器人状态代码的 HTTP 状态代码进行响应。 如果机器人生成错误，则会向客户端返回 HTTP 500 响应（“内部服务器错误”）以响应其 Send Message 请求。 如果 POST 成功，则该服务返回 HTTP 204 状态代码。 响应正文中不返回任何数据。 可以通过[轮询](bot-framework-rest-direct-line-1-1-receive-messages.md)获取客户端的消息和来自机器人的任何消息。 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>Send Message 请求/响应的总时间

将消息发布到 Direct Line 聊天的总时间是以下时间的总和：

- HTTP 请求从客户端传输到 Direct Line 服务的传输时间
- Direct Line 中的内部处理时间（通常少于 120ms）
- 从 Direct Line 服务到机器人的传输时间
- 机器人中的处理时间
- HTTP 响应传输回客户端的传输时间

## <a name="send-attachments-to-the-bot"></a>向机器人发送附件

在某些情况下，客户端可能需要向机器人发送附件，如图像或文档。 客户端可以通过以下方法向机器人发送附件：[指定使用](#send-by-url) `POST /api/conversations/{conversationId}/messages` 发送的 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 对象中的附件的 URL，或使用 `POST /api/conversations/{conversationId}/upload` [上传附件](#upload-attachments)。

## <a id="send-by-url"></a>通过 URL 发送附件

若要使用 `POST /api/conversations/{conversationId}/messages` 发送作为 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 对象一部分的一个或多个附件，请指定消息的 `images` 数组和/或 `attachments` 数组内的附件 URL。

## <a id="upload-attachments"></a>通过上传发送附件

通常情况下，客户端可能在设备上包含要发送给机器人的图像或文档，但没有对应这些文件的 URL。 在此情况下，客户端可以发出 `POST /api/conversations/{conversationId}/upload` 请求通过上传向机器人发送附件。 请求的格式和内容将取决于客户端是[发送单个附件](#upload-one-attachment)还是[发送多个附件](#upload-multiple-attachments)。

### <a id="upload-one-attachment"></a>通过上传发送单个附件

若要通过上传发送附件，请发出此请求： 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

在此请求 URI 中，将 {conversationId} 替换为聊天 ID，将 {userId} 替换为发送消息的用户的 ID。 在请求标头中，设置 `Content-Type` 以指定附件的类型，设置 `Content-Disposition` 以指定附件的文件名。

以下代码片段提供了 Send (single) Attachment 请求和响应的示例。

#### <a name="request"></a>请求

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>响应

如果请求成功，上传完成后将向机器人发送消息，服务将返回 HTTP 204 状态代码。

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a>通过上传发送多个附件

若要通过上传发送多个附件，向 `/api/conversations/{conversationId}/upload` 终结点 `POST` 多部分请求。 设置对 `multipart/form-data` 请求的 `Content-Type` 标头并包含各部分的 `Content-Type` 标头和 `Content-Disposition` 标头，以指定每个附件的类型和文件名。 在请求 URI 中，将 `userId` 参数设置为发送消息的用户的 ID。 

可以通过添加指定 `Content-Type` 标头值 `application/vnd.microsoft.bot.message` 的部件在请求中包含 [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) 对象。 这使客户端能够自定义包含附件的消息。 如果请求包含 Message，在发送之前，由有效负载的其他部分指定的附件要先添加为该 Message 的附件。 

以下代码片段提供了 Send (multiple) Attachment 请求和响应的示例。 在此示例中，请求发送包含一些文本和单张图像附件的消息。 其他部分可以添加到请求中，以在此消息中包含多个附件。

#### <a name="request"></a>请求

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>响应

如果请求成功，上传完成后将向机器人发送消息，服务将返回 HTTP 204 状态代码。

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-1-1-concepts.md)
- [身份验证](bot-framework-rest-direct-line-1-1-authentication.md)
- [启动聊天](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [从机器人接收消息](bot-framework-rest-direct-line-1-1-receive-messages.md)