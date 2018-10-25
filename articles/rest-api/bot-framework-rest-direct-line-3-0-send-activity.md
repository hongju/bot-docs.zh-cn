---
title: 发送活动机器人 | Microsoft Docs
description: 了解如何使用 Direct Line API v3.0 向机器人发送活动。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 290a2733b96a458eb3529b0b0854703631e05f22
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000034"
---
# <a name="send-an-activity-to-the-bot"></a>向机器人发送活动

使用 Direct Line 3.0 协议时，客户端和机器人可以交换多种不同类型的[活动](bot-framework-rest-connector-activities.md)，其中包括消息活动、键入活动和机器人支持的自定义活动。 客户端可以按请求发送单个活动。 

## <a name="send-an-activity"></a>发送活动

若要将某个活动发送到机器人，客户端必须创建 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 对象来定义活动，然后发出 `POST` 请求到 `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities`，指定请求正文中的 Activity 对象。

以下代码片段提供了 Send Activity 请求和响应的示例。

### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>响应

将活动传递到机器人时，服务通过反映机器人状态代码的 HTTP 状态代码进行响应。 如果机器人生成错误，则会向客户端返回 HTTP 502 响应（“错误网关”）以响应其 Send Activity 请求。 如果 POST 成功，响应将包含 JSON 有效负载，它指定已发送到机器人的 Activity ID。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>Send Activity 请求/响应的总时间

将消息发布到 Direct Line 会话的总时间是以下时间的总和：

- HTTP 请求从客户端传输到 Direct Line 服务的传输时间
- Direct Line 中的内部处理时间（通常少于 120 ms）
- 从 Direct Line 服务到机器人的传输时间
- 机器人中的处理时间
- HTTP 响应传输回客户端的传输时间

## <a name="send-attachments-to-the-bot"></a>向机器人发送附件

在某些情况下，客户端可能需要向机器人发送附件，如图像或文档。 客户端可以通过以下方法向机器人发送附件：[指定 URL](#send-by-url)（该 URL 是使用 `POST /v3/directline/conversations/{conversationId}/activities` 发送的 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 对象中的附件的 URL），或使用 `POST /v3/directline/conversations/{conversationId}/upload` [上传附件](#upload-attachments)。

## <a id="send-by-url"></a> 通过 URL 发送附件

若要使用 `POST /v3/directline/conversations/{conversationId}/activities` 将一个或多个附件作为 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 对象的一部分发送，只需在 Activity 对象中包含一个或多个 [Attachment](bot-framework-rest-connector-api-reference.md#attachment-object) 对象，并设置每个 Attachment 对象的 `contentUrl` 属性来指定 HTTP、HTTPS 或附件的 `data` URI。

## <a id="upload-attachments"></a> 通过上传发送附件

通常情况下，客户端可能在设备上包含要发送给机器人的图像或文档，但没有对应这些文件的 URL。 在此情况下，客户端可以发出 `POST /v3/directline/conversations/{conversationId}/upload` 请求通过上传向机器人发送附件。 请求的格式和内容将取决于客户端是[发送单个附件](#upload-one-attachment)还是[发送多个附件](#upload-multiple-attachments)。

### <a id="upload-one-attachment"></a>通过上传发送单个附件

若要通过上传发送附件，请发出此请求： 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

在此请求 URI 中，将 {conversationId} 替换为会话 ID，将 {userId} 替换为发送消息的用户的 ID。 `userId` 参数是必需的。 在请求标头中，设置 `Content-Type` 以指定附件的类型，设置 `Content-Disposition` 以指定附件的文件名。

以下代码片段提供了 Send (single) Attachment 请求和响应的示例。

#### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>响应

如果请求成功，消息 会在上传完成时发送给机器人，客户端收到的响应将包含已发送 Activity 的 ID。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a> 通过上传发送多个附件

若要通过上传发送多个附件，向 `/v3/directline/conversations/{conversationId}/upload` 终结点 `POST` 多部分请求。 设置对 `multipart/form-data` 请求的 `Content-Type` 标头并包含各部分的 `Content-Type` 标头和 `Content-Disposition` 标头，以指定每个附件的类型和文件名。 在请求 URI 中，将 `userId` 参数设置为发送消息的用户的 ID。 

可以通过添加指定 `Content-Type` 标头值 `application/vnd.microsoft.activity` 的部件在请求中包含 [Activity](bot-framework-rest-connector-api-reference.md#activity-object) 对象。 如果请求包含 Activity，在发送之前，由有效负载的其他部分指定的附件要先添加为该 Activity 的附件。 如果请求不包含 Activity，将创建空的 Activity 以充当在其中发送指定附件的容器。

以下代码片段提供了 Send (multiple) Attachments 请求和响应的示例。 在此示例中，请求发送包含一些文本和单张图像附件的消息。 其他部分可以添加到请求中，以在此消息中包含多个附件。

#### <a name="request"></a>请求

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>响应

如果请求成功，消息 Activity 会在上传完成时发送给机器人，客户端收到的响应将包含已发送 Activity 的 ID。

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0004"
}
```

## <a name="additional-resources"></a>其他资源

- [关键概念](bot-framework-rest-direct-line-3-0-concepts.md)
- [身份验证](bot-framework-rest-direct-line-3-0-authentication.md)
- [启动会话](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [重新连接到会话](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [从机器人接收活动](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [结束会话](bot-framework-rest-direct-line-3-0-end-conversation.md)
