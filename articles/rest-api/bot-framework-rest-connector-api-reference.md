---
title: API 参考 | Microsoft Docs
description: 了解 Bot Connector 服务和 Bot State 服务中的标头、操作、对象和错误。
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/25/2018
ms.openlocfilehash: 81192c9b5806d467c2a1fd292ee3d5db539e9ead
ms.sourcegitcommit: 15f7fa40b7e0a05507cdc66adf75bcfc9533e781
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916864"
---
# <a name="api-reference"></a>API 参考

> [!NOTE]
> REST API 不等同于 SDK。 REST API 旨在允许进行标准 REST 通信，但是与 Bot Framework 交互的首选方法是 SDK。 

在 Bot Framework 中，Bot Connector 服务使机器人能够在 Bot Framework 门户中配置的通道上与用户交换消息，而 Bot State 服务使机器人能够存储和检索与机器人使用 Bot Connector 服务执行的聊天相关的状态数据。 这两种服务都通过 HTTPS 使用行业标准 REST 和 JSON。

> [!IMPORTANT]
> 建议不要将 Bot Framework State Service API 用于生产环境，该 API 可能会在将来的版本中弃用。 建议更新机器人代码以使用内存中存储进行测试，或者将 Azure 扩展之一用于生产机器人。 有关详细信息，请参阅针对 [.NET](~/dotnet/bot-builder-dotnet-state.md) 或 [Node](~/nodejs/bot-builder-nodejs-state.md) 实现的“管理状态数据”主题。

## <a name="base-uri"></a>基本 URI

当用户向机器人发送消息时，传入请求包含一个具有 `serviceUrl` 属性的 [Activity](#activity-object) 对象，该属性指定机器人应将其响应发送到的终结点。 若要访问 Bot Connector 服务或 Bot State 服务，请使用 `serviceUrl` 值作为 API 请求的基本 URI。 

例如，假设机器人在用户向机器人发送消息时收到以下活动。

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

用户消息中的 `serviceUrl` 属性表示机器人应将其响应发送到终结点 `https://smba.trafficmanager.net/apis`；这将是机器人在此聊天上下文中发出的任何后续请求的基 URI。 如果机器人需要向用户发送主动消息，请务必保存 `serviceUrl` 的值。

以下示例展示了机器人为响应用户消息发出的请求。 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have several times available on Saturday!",
    "replyToId": "bf3cc9a2f5de..."
}
```

## <a name="headers"></a>标头

### <a name="request-headers"></a>请求标头

除了标准 HTTP 请求标头，发出的每个 API 请求还必须包含一个 `Authorization` 标头，用于指定对机器人进行身份验证所需的访问令牌。 使用以下格式指定 `Authorization` 标头：

```http
Authorization: Bearer ACCESS_TOKEN
```

有关如何获取机器人访问令牌的详细信息，请参阅[验证机器人向 Bot Connector 服务发出的请求](bot-framework-rest-connector-authentication.md#bot-to-connector)。

### <a name="response-headers"></a>响应标头

除了标准 HTTP 响应标头，每个响应还包含一个 `X-Correlating-OperationId` 标头。 此标头的值是与 Bot Framework 日志条目（其中包含有关请求的详细信息）对应的 ID。 每当收到错误响应时，都应捕获此标头的值。 如果无法独立解决问题，请在报告问题时将此值包含在向支持团队提供的信息中。

## <a name="http-status-codes"></a>HTTP 状态代码

随每个响应返回的 <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">HTTP 状态代码</a>指示相应请求的结果。 

| HTTP 状态代码 | 含义 |
|----|----|
| 200 | 请求成功。 |
| 201 | 请求成功。 |
| 202 | 已接受请求，将进行处理。 |
| 204 | 请求成功，但未返回任何内容。 |
| 400 | 请求格式不正确或者其他方面不正确。 |
| 401 | 机器人未获授权，无法发出请求。 |
| 403 | 不允许机器人执行请求的操作。 |
| 404 | 找不到所请求的资源。 |
| 500 | 发生了内部服务器错误。 |
| 503 | 服务不可用。 |

### <a name="errors"></a>错误

指定 4xx 范围或 5xx 范围内的 HTTP 状态代码的任何响应都将在响应正文中包含 [ErrorResponse](#errorresponse-object) 对象，该对象提供错误相关信息。 如果收到 4xx 范围内的错误响应，请检查 **ErrorResponse** 对象以确定错误原因，并在重新提交请求之前解决问题。

## <a name="conversation-operations"></a>聊天操作 
以下操作可用于创建聊天、发送消息（活动）和管理聊天内容。

| Operation | Description |
|----|----|
| [创建聊天](#create-conversation) | 创建新的聊天。 | 
| [发送到聊天](#send-to-conversation) | 将活动（消息）发送到指定聊天的末尾。 | 
| [回复活动](#reply-to-activity) | 将活动（消息）发送到指定聊天，作为对指定活动的回复。 | 
| [获取聊天成员](#get-conversation-members) | 获取指定聊天的成员。 |
| [获取聊天分页成员](#get-conversation-paged-members) | 获取指定聊天的成员，一次一页。 |
| [获取活动成员](#get-activity-members) | 获取指定聊天中指定活动的成员。 | 
| [更新活动](#update-activity) | 更新现有活动。 | 
| [删除活动](#delete-activity) | 删除现有活动。 | 
| [将附件上传到通道](#upload-attachment-to-channel) | 将附件直接上传到通道的 blob 存储中。 |

### <a name="create-conversation"></a>创建聊天
创建新的聊天。 
```http 
POST /v3/conversations
```

| | |
|----|----|
| **请求正文** | [Conversation](#conversation-object) 对象 |
| **返回** | [ResourceResponse](#resourceresponse-object) 对象 | 

### <a name="send-to-conversation"></a>发送到聊天
将活动（消息）发送到指定的聊天。 活动将根据通道的时间戳或语义追加到聊天的末尾。 若要回复聊天中的特定消息，请改用[回复活动](#reply-to-activity)。
```http
POST /v3/conversations/{conversationId}/activities
```

| | |
|----|----|
| **请求正文** | [Activity](#activity-object) 对象 |
| **返回** | [Identification](#identification-object) 对象 | 

### <a name="reply-to-activity"></a>回复活动
将活动（消息）发送到指定聊天，作为对指定活动的回复。 如果通道支持，将添加该活动作为对另一个活动的回复。 如果通道不支持嵌套回复，则此操作的行为类似于[发送到聊天](#send-to-conversation)。
```http
POST /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **请求正文** | [Activity](#activity-object) 对象 |
| **返回** | [Identification](#identification-object) 对象 | 

### <a name="get-conversation-members"></a>获取聊天成员
获取指定聊天的成员。
```http
GET /v3/conversations/{conversationId}/members
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [ChannelAccount](#channelaccount-object) 对象数组 | 

### <a name="get-conversation-paged-members"></a>获取聊天分页成员
获取指定聊天的成员，一次一页。
```http
GET /v3/conversations/{conversationId}/pagedmembers
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 一个 [ChannelAccount](#channelaccount-object) 对象数组和一个可用于获取更多值的继续标记|

### <a name="get-activity-members"></a>获取活动成员
获取指定聊天中指定活动的成员。
```http
GET /v3/conversations/{conversationId}/activities/{activityId}/members
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [ChannelAccount](#channelaccount-object) 对象数组 | 

### <a name="update-activity"></a>更新活动
某些通道允许编辑现有活动，以反映机器人聊天的新状态。 例如，可以在用户单击某个按钮后从聊天消息中删除按钮。 如果成功，此操作会更新指定聊天中的指定活动。 
```http
PUT /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **请求正文** | [Activity](#activity-object) 对象 |
| **返回** | [Identification](#identification-object) 对象 | 

### <a name="delete-activity"></a>删除活动
某些通道允许删除现有活动。 如果成功，此操作会从指定聊天中删除指定活动。
```http
DELETE /v3/conversations/{conversationId}/activities/{activityId}
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 指示操作结果的 HTTP 状态代码。 响应正文中未指定任何内容。 | 

### <a name="upload-attachment-to-channel"></a>将附件上传到通道
将指定聊天的附件直接上传到通道的 blob 存储中。 这样就可以将数据存储在兼容的存储中。 
```http 
POST /v3/conversations/{conversationId}/attachments
```

| | |
|----|----|
| **请求正文** | [AttachmentUpload](#attachmentupload-object) 对象。 |
| **返回** | [ResourceResponse](#resourceresponse-object) 对象。 **ID** 属性指定可与[获取附件信息](#get-attachment-info)操作和[获取附件](#get-attachment)操作一起使用的附件 ID。 | 

## <a name="attachment-operations"></a>附件操作 
以下操作可用于检索附件相关信息以及文件本身的二进制数据。

| Operation | Description |
|----|----|
| [获取附件信息](#get-attachment-info) | 获取有关指定附件的信息，包括文件名、文件类型和可用视图（例如，原始视图或缩略图）。 |
| [获取附件](#get-attachment) | 获取指定附件的指定视图作为二进制内容。 | 

### <a name="get-attachment-info"></a>获取附件信息 
获取有关指定附件的信息，包括文件名、类型和可用视图（例如，原始视图或缩略图）。
```http
GET /v3/attachments/{attachmentId}
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [AttachmentInfo](#attachmentinfo-object) 对象 | 

### <a name="get-attachment"></a>获取附件
获取指定附件的指定视图作为二进制内容。
```http
GET /v3/attachments/{attachmentId}/views/{viewId}
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 二进制内容，表示指定附件的指定视图 | 

## <a name="state-operations"></a>状态操作
以下操作可用于存储和检索状态数据。

| Operation | Description |
|----|----|
| [设置用户数据](#set-user-data) | 存储通道上特定用户的状态数据。 | 
| [设置聊天数据](#set-conversation-data) | 存储通道上特定聊天的状态数据。 | 
| [设置私人聊天数据](#set-private-conversation-data) | 存储通道上特定聊天上下文中特定用户的状态数据。 | 
| [获取用户数据](#get-user-data) | 检索先前为通道上所有聊天中的特定用户存储的状态数据。 | 
| [获取聊天数据](#get-conversation-data) | 检索先前为通道上的特定聊天存储的状态数据。 | 
| [获取私人聊天数据](#get-private-conversation-data) | 检索先前为通道上特定聊天上下文中的特定用户存储的状态数据。 | 
| [删除用户状态](#delete-state-for-user) | 使用[设置用户数据](#set-user-data)操作或[设置私人聊天数据](#set-private-conversation-data)操作删除先前为用户存储的状态数据。 | 

### <a name="set-user-data"></a>设置用户数据
存储指定通道上指定用户的状态数据。
```http
POST /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **请求正文** | [BotData](#botdata-object) 对象 |
| **返回** | [BotData](#botdata-object) 对象 | 

### <a name="set-conversation-data"></a>设置聊天数据
存储指定通道上指定聊天的状态数据。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}
```

| | |
|----|----|
| **请求正文** | [BotData](#botdata-object) 对象 |
| **返回** | [BotData](#botdata-object) 对象 | 

### <a name="set-private-conversation-data"></a>设置私人聊天数据
存储指定通道上指定聊天上下文中指定用户的状态数据。
```http
POST /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **请求正文** | [BotData](#botdata-object) 对象 |
| **返回** | [BotData](#botdata-object) 对象 | 

### <a name="get-user-data"></a>获取用户数据
检索先前为指定通道上所有聊天中的指定用户存储的状态数据。
```http
GET /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [BotData](#botdata-object) 对象 | 

### <a name="get-conversation-data"></a>获取聊天数据
检索先前为指定通道上的指定聊天存储的状态数据。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId} 
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [BotData](#botdata-object) 对象 | 

### <a name="get-private-conversation-data"></a>获取私人聊天数据
检索先前为指定通道上指定聊天上下文中的指定用户存储的状态数据。
```http
GET /v3/botstate/{channelId}/conversations/{conversationId}/users/{userId} 
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | [BotData](#botdata-object) 对象 | 

### <a name="delete-state-for-user"></a>删除用户状态
使用[设置用户数据](#set-user-data)操作或[设置私人聊天数据](#set-private-conversation-data)操作，删除先前为指定通道上的指定用户存储的状态数据。
```http
DELETE /v3/botstate/{channelId}/users/{userId} 
```

| | |
|----|----|
| **请求正文** | 不适用 |
| **返回** | 字符串数组 (ID) | 

## <a id="objects"></a> 架构

架构定义机器人可用来与用户通信的对象及其属性。 

| 对象 | Description |
| ---- | ---- |
| [Activity 对象](#activity-object) | 定义在机器人与用户之间交换的消息。 |
| [AnimationCard 对象](#animationcard-object) | 定义可播放动态 GIF 或短视频的卡。 |
| [Attachment 对象](#attachment-object) | 定义要包含在消息中的其他信息。 附件可以是媒体文件（例如，音频、视频、图像、文件）或资讯卡。 |
| [AttachmentData 对象](#attachmentdata-object) |描述附件数据。 |
| [AttachmentInfo 对象](#attachmentinfo-object) | 描述附件。 |
| [AttachmentView 对象](#attachmentview-object) | 定义附件视图。 |
| [AttachmentUpload 对象](#attachmentupload-object) | 定义要上传的附件。 |
| [AudioCard 对象](#audiocard-object) | 定义可播放音频文件的卡。 |
| [BotData 对象](#botdata-object) | 定义使用 Bot State 服务为用户、聊天或特定聊天上下文中的用户存储的状态数据。 |
| [CardAction 对象](#cardaction-object) | 定义要执行的操作。 |
| [CardImage 对象](#cardimage-object) | 定义要在卡上显示的图像。 |
| [ChannelAccount 对象](#channelaccount-object) | 定义通道上的机器人或用户帐户。 |
| [Conversation 对象](#conversation-object) | 定义聊天，包括聊天中包含的机器人和用户。 |
| [ConversationAccount 对象](#conversationaccount-object) | 定义通道中的聊天。 |
| [ConversationParameters 对象](#conversationparameters-object) | 定义用于创建新聊天的参数 |
| [ConversationReference 对象](#conversationreference-object) | 定义聊天中的特定点。 |
| [ConversationResourceResponse 对象](#conversationresourceresponse-object) | 包含资源的响应 |
| [Entity 对象](#entity-object) | 定义实体对象。 |
| [Error 对象](#error-object) | 定义错误。 |
| [ErrorResponse 对象](#errorresponse-object) | 定义 HTTP API 响应。 |
| [Fact 对象](#fact-object) | 定义包含事实的键值对。 |
| [Geocoordinates 对象](#geocoordinates-object) | 使用世界测地系统 (WSG84) 坐标定义地理位置。 |
| [HeroCard 对象](#herocard-object) | 定义具有大图像、标题、文本和操作按钮的卡。 |
| [Identification 对象](#identification-object) | 标识资源。 |
| [MediaEventValue 对象](#mediaeventvalue-object) |媒体事件的补充参数。|
| [MediaUrl 对象](#mediaurl-object) | 定义媒体文件源的 URL。 |
| [Mention 对象](#mention-object) | 定义聊天中提到的用户或机器人。 |
| [MessageReaction 对象](#messagereaction-object) | 定义对消息的回应。 |
| [Place 对象](#place-object) | 定义聊天中提到的位置。 |
| [ReceiptCard 对象](#receiptcard-object) | 定义包含购买收据的卡。 |
| [ReceiptItem 对象](#receiptitem-object) | 定义收据中的订单项。 |
| [ResourceResponse 对象](#resourceresponse-object) | 定义资源。 |
| [SignInCard 对象](#signincard-object) | 定义允许用户登录服务的卡。 |
| [SuggestedActions 对象](#suggestedactions-object) | 定义用户可以选择的选项。 |
| [ThumbnailCard 对象](#thumbnailcard-object) | 定义具有缩略图、标题、文本和操作按钮的卡。 |
| [ThumbnailUrl 对象](#thumbnailurl-object) | 定义图像源的 URL。 |
| [VideoCard 对象](#videocard-object) | 定义可播放视频的卡。 |
| [SemanticAction 对象](#semanticaction-object) | 定义对编程操作的引用。 |

### <a name="activity-object"></a>Activity 对象
定义在机器人与用户之间交换的消息。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **action** | 字符串 | 要应用或已应用的操作。 使用 **type** 属性确定操作的上下文。 例如，当 **type** 为 **contactRelationUpdate** 时，如果用户将机器人添加到其联系人列表中，则 **action** 属性的值为 **add**，如果他们从联系人列表中删除机器人，则该属性的值为 **remove**。 |
| **attachments** | [Attachment](#attachment-object)[] | 一个 **Attachment** 对象数组，用于定义要包含在消息中的其他信息。 各附件可以是媒体文件（例如，音频、视频、图像、文件）或资讯卡。 |
| **attachmentLayout** | 字符串 | 消息中包含的资讯卡**附件**的布局。 下列值之一：**carousel**、**list**。 有关资讯卡附件的详细信息，请参阅[向消息添加资讯卡附件](bot-framework-rest-connector-add-rich-cards.md)。 |
| **channelData** | 对象 | 一个包含特定于通道的内容的对象。 某些通道提供的功能需要其他信息，而这些信息不能使用附件架构来表示。 对于此类情况，请按通道文档中的定义将此属性设置为特定于通道的内容。 有关详细信息，请参阅[实现特定于通道的功能](bot-framework-rest-connector-channeldata.md)。 |
| **channelId** | 字符串 | 用于唯一标识通道的 ID。 由通道设置。 | 
| 对话 | [ConversationAccount](#conversationaccount-object) | 一个 **ConversationAccount** 对象，用于定义活动所属的聊天。 |
| **code** | 字符串 | 指示聊天结束原因的代码。 |
| **entities** | 对象[] | 一个对象数组，表示消息中提到的实体。 此数组中的对象可以是任何 <a href="http://schema.org/" target="_blank">Schema.org</a> 对象。 例如，此数组可能包含 [Mention](#mention-object) 对象，用于标识聊天中提到的某人，以及 [Place](#place-object) 对象，用于标识聊天中提到的某个位置。 |
| **from** | [ChannelAccount](#channelaccount-object) | 一个 **ChannelAccount** 对象，用于指定消息的发送者。 |
| **historyDisclosed** | 布尔值 | 一个标志，指示是否公开历史记录。 默认值为“false”。 |
| **id** | 字符串 | 一个 ID，用于唯一标识通道上的活动。 | 
| **inputHint** | 字符串 | 一个值，指示在将消息传送到客户端后，机器人是接受、预期还是忽略用户输入。 下列值之一：**acceptingInput**、**expectingInput**、**ignoringInput**。 |
| **locale** | 字符串 | 应该用于在消息中显示文本的语言的区域设置，格式为 `<language>-<country>`。 通道使用此属性指示用户的语言，以便机器人可以指定该语言的显示字符串。 默认值为 **en-US**。 |
| **localTimestamp** | 字符串 | 在本地时区发送消息的日期和时间，以 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 格式表示。 |
| **membersAdded** | [ChannelAccount](#channelaccount-object)[] | 一个 **ChannelAccount** 对象数组，表示已加入聊天的用户的列表。 仅当活动 **type** 为“conversationUpdate”且用户已加入聊天时才会显示。 | 
| **membersRemoved** | [ChannelAccount](#channelaccount-object)[] | 一个 **ChannelAccount** 对象数组，表示已离开聊天的用户的列表。 仅当活动 **type** 为“conversationUpdate”且用户已离开聊天时才会显示。 | 
| name | 字符串 | 要调用的操作的名称或事件的名称。 |
| **recipient** | [ChannelAccount](#channelaccount-object) | 一个 **ChannelAccount** 对象，用于指定消息的接收者。 |
| **relatesTo** | [ConversationReference](#conversationreference-object) | 一个 **ConversationReference** 对象，用于定义聊天中的特定点。 |
| **replyToId** | 字符串 | 此消息回复的消息的 ID。 若要回复用户发送的消息，请将此属性设置为用户消息的 ID。 并非所有通道都支持线程回复。 在这些情况下，通道将忽略此属性并使用时间时间排序语义（时间戳）将消息追加到聊天。 | 
| **serviceUrl** | 字符串 | 用于指定通道服务终结点的 URL。 由通道设置。 | 
| **speak** | 字符串 | 机器人要在支持语音的通道上朗读的文本。 若要控制机器人语音的各种特性，如声音、语速、音量、发音和音调，请以<a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">语音合成标记语言 (SSML)</a> 格式指定此属性。 |
| **suggestedActions** | [SuggestedActions](#suggestedactions-object) | 一个 **SuggestedActions** 对象，用于定义用户可以选择的选项。 |
| **summary** | 字符串 | 消息包含的信息摘要。 例如，对于在电子邮件通道上发送的邮件，此属性可以指定电子邮件的前 50 个字符。 |
| **text** | 字符串 | 从用户发送给机器人或从机器人发送给用户的消息文本。 请参阅通道的相关文档，了解此属性内容的限制。 |
| **textFormat** | 字符串 | 消息**文本**的格式。 下列值之一：**markdown**、**plain**、**xml**。 有关文本格式的详细信息，请参阅[创建消息](bot-framework-rest-connector-create-messages.md)。 |
| **timestamp** | 字符串 | 在 UTC 时区发送消息的日期和时间，以 <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a> 格式表示。 |
| **topicName** | 字符串 | 活动所属聊天的主题。 |
| type | 字符串 | 活动的类型。 下列值之一：**contactRelationUpdate**、**conversationUpdate**、**deleteUserData**、**message**、**typing**、**endOfConversation**。 有关活动类型的详细信息，请参阅[活动概述](bot-framework-rest-connector-activities.md)。 |
| **值** | 对象 | 开放式值。 |
| **semanticAction** |[SemanticAction](#semanticaction-object) | 一个 **SemanticAction** 对象，表示对编程操作的引用。 |

<a href="#objects">返回到架构表</a>

### <a name="animationcard-object"></a>AnimationCard 对象
定义可播放动态 GIF 或短视频的卡。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **autoloop** | 布尔值 | 一个标志，指示在最后一个动态 GIF 结束后是否重播动态 GIF 列表。 若要自动重播动画，请将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **autostart** | 布尔值 | 一个标志，指示是否在显示卡时自动播放动画。 若要自动播放动画，请将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户执行一项或多项操作。 通道决定了可指定的按钮数。 |
| **duration** | 字符串 | 媒体内容的长度，采用 [ISO 8601 持续时间格式](https://www.iso.org/iso-8601-date-and-time-format.html)。 |
| **图片** | [ThumbnailUrl](#thumbnailurl-object) | 一个 **ThumbnailUrl** 对象，用于指定要在卡上显示的图像。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 一个 **MediaUrl** 对象数组，用于指定要播放的动态 GIF 的列表。 |
| **shareable** | 布尔值 | 一个标志，指示是否可以与其他人共享动画。 如果可以共享动画，则将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **subtitle** | 字符串 | 要在卡标题下显示的副标题。 |
| **text** | 字符串 | 要在卡的标题或副标题下显示的描述或提示。 |
| **title** | 字符串 | 卡的标题。 |
| **值** | 对象 | 此卡的补充参数 |

<a href="#objects">返回到架构表</a>

### <a name="attachment-object"></a>Attachment 对象
定义要包含在消息中的其他信息。 附件可以是媒体文件（例如，音频、视频、图像、文件）或资讯卡。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **contentType** | 字符串 | 附件中内容的媒体类型。 对于媒体文件，请将此属性设置为已知媒体类型，例如 **image/png**、**audio/wav** 和 **video/mp4**。 对于资讯卡，请将此属性设置为以下特定于供应商的类型之一：<ul><li>**application/vnd.microsoft.card.adaptive**：一种可以包含文本、语音、图像、按钮和输入字段的任意组合的资讯卡。 将 **content** 属性设置为 <a href="http://adaptivecards.io/documentation/#create-cardschema" target="_blank">AdaptiveCard</a> 对象。</li><li>**application/vnd.microsoft.card.animation**：一种播放动画的资讯卡。 将 **content** 属性设置为 [AnimationCard](#animationcard-object) 对象。</li><li>**application/vnd.microsoft.card.audio**：一种播放音频文件的资讯卡。 将 **content** 属性设置为 [AudioCard](#audiocard-object) 对象。</li><li>**application/vnd.microsoft.card.video**：一种播放视频的资讯卡。 将 **content** 属性设置为 [VideoCard](#videocard-object) 对象。</li><li>**application/vnd.microsoft.card.hero**：英雄卡。 将 **content** 属性设置为 [HeroCard](#herocard-object) 对象。</li><li>**application/vnd.microsoft.card.thumbnail**：缩略图卡。 将 **content** 属性设置为 [ThumbnailCard](#thumbnailcard-object) 对象。</li><li>**application/vnd.microsoft.com.card.receipt**：收据卡。 将 **content** 属性设置为 [ReceiptCard](#receiptcard-object) 对象。</li><li>**application/vnd.microsoft.com.card.signin**：用户登录卡。 将 **content** 属性设置为 [SignInCard](#signincard-object) 对象。</li></ul> |
| **contentUrl** | 字符串 | 附件内容的 URL。 例如，如果附件是图像，请将 **contentUrl** 设置为表示图像位置的 URL。 支持的协议包括：HTTP、HTTPS、文件和数据。 |
| **content** | 对象 | 附件内容。 如果附件是资讯卡，请将此属性设置为资讯卡对象。 此属性与 **contentUrl** 属性互相排斥。 |
| name | 字符串 | 附件的名称。 |
| **thumbnailUrl** | 字符串 | 通道可以使用的缩略图的 URL，但前提是它支持使用 **content** 或 **contentUrl** 的较小替代形式。 例如，如果将 **contentType** 设置为 **application/word**，将 **contentUrl** 设置为 Word 文档的位置，则可以包含表示文档的缩略图。 通道可以显示缩略图而不是文档。 当用户单击图像时，通道将打开文档。 |

<a href="#objects">返回到架构表</a>

### <a name="attachmentdata-object"></a>AttachmentData 对象 
描述附件数据。

| 属性 | 类型 | Description |
|----|----|----|
| name | 字符串 | 附件的名称。 |
| **originalBase64** | 字符串 | 附件内容。 |
| **thumbnailBase64** | 字符串 | 附件缩略图的内容。 |
| type | 字符串 | 附件的 Content-type。 |

### <a name="attachmentinfo-object"></a>AttachmentInfo 对象
描述附件。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| name | 字符串 | 附件的名称。 |
| type | 字符串 | 附件的 ContentType。 |
| **视图** | [AttachmentView](#attachmentview-object)[] | 一个 **AttachmentView** 对象数组，表示附件的可用视图。 |

<a href="#objects">返回到架构表</a>

### <a name="attachmentview-object"></a>AttachmentView 对象
定义附件视图。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **viewId** | 字符串 | 视图 ID。 |
| **大小** | 数字 | 文件大小。 |

<a href="#objects">返回到架构表</a>

<!-- TODO - can't find in swagger file -->
### <a name="attachmentupload-object"></a>AttachmentUpload 对象
定义要上传的附件。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| type | 字符串 | 附件的 ContentType。 | 
| name | 字符串 | 附件的名称。 | 
| **originalBase64** | 字符串 | 二进制数据，表示文件原始版本的内容。 |
| **thumbnailBase64** | 字符串 | 二进制数据，表示文件缩略图版本的内容。 |

<a href="#objects">返回到架构表</a>

### <a name="audiocard-object"></a>AudioCard 对象
定义可播放音频文件的卡。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **aspect** | 字符串 | 在 **image** 属性中指定的缩略图的纵横比。 有效值为 **16:9** 和 **9:16**。 |
| **autoloop** | 布尔值 | 一个标志，指示在最后一个音频文件结束后是否重播音频文件列表。 若要自动重播音频文件，请将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **autostart** | 布尔值 | 一个标志，指示是否在显示卡时自动播放音频。 若要自动播放音频，请将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户执行一项或多项操作。 通道决定了可指定的按钮数。 |
| **duration** | 字符串 | 媒体内容的长度，采用 [ISO 8601 持续时间格式](https://www.iso.org/iso-8601-date-and-time-format.html)。 |
| **图片** | [ThumbnailUrl](#thumbnailurl-object) | 一个 **ThumbnailUrl** 对象，用于指定要在卡上显示的图像。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 一个 **MediaUrl** 对象数组，用于指定要播放的音频文件的列表。 |
| **shareable** | 布尔值 | 一个标志，指示是否可以与其他人共享音频文件。 如果可以共享音频，则将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **subtitle** | 字符串 | 要在卡标题下显示的副标题。 |
| **text** | 字符串 | 要在卡的标题或副标题下显示的描述或提示。 |
| **title** | 字符串 | 卡的标题。 |
| **值** | 对象 | 此卡的补充参数 |

<a href="#objects">返回到架构表</a>

<!-- TODO - can't find in swagger file -->
### <a name="botdata-object"></a>BotData 对象
定义使用 Bot State 服务为用户、聊天或特定聊天上下文中的用户存储的状态数据。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **数据** | 对象 | 在请求中，它为指定要使用 Bot State 服务存储的属性和值的 JSON 对象。 在响应中，它为指定已使用 Bot State 服务存储的属性和值的 JSON 对象。 | 
| **eTag** | 字符串 | 一个实体标记值，可用于控制使用 Bot State 服务存储的数据的数据并发性。 有关详细信息，请参阅[管理状态数据](bot-framework-rest-state.md)。 | 

<a href="#objects">返回到架构表</a>

### <a name="cardaction-object"></a>CardAction 对象
定义要执行的操作。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **图片** | 字符串 | 要显示的图像的 URL | 
| **text** | 字符串 | 操作的文本 |
| **title** | 字符串 | 按钮的文本。 仅适用于按钮的操作。 |
按钮。 仅适用于按钮的操作。 |
| type | 字符串 | 要执行的操作类型。 有关有效值的列表，请参阅[向消息添加资讯卡附件](bot-framework-rest-connector-add-rich-cards.md)。 |
| **值** | 对象 | 操作的补充参数。 此属性的值因操作 **type** 而异。 有关详细信息，请参阅[向消息添加资讯卡附件](bot-framework-rest-connector-add-rich-cards.md)。 |

<a href="#objects">返回到架构表</a>

### <a name="cardimage-object"></a>CardImage 对象
定义要在卡上显示的图像。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **alt** | 字符串 | 图像描述。 应包含描述以支持可访问性。 |
| **tap** | [CardAction](#cardaction-object) | 一个 **CardAction** 对象，用于指定用户点击或单击图像时要执行的操作。 |
| **url** | 字符串 | 图像源或图像的 base64 二进制文件的 URL（例如，`data:image/png;base64,iVBORw0KGgo...`）。 |

<a href="#objects">返回到架构表</a>

### <a name="channelaccount-object"></a>ChannelAccount 对象
定义通道上的机器人或用户帐户。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **id** | 字符串 | 一个 ID，用于唯一标识通道上的机器人或用户。 |
| name | 字符串 | 机器人或用户的名称。 |

<a href="#objects">返回到架构表</a>

<!--TODO can't find-->
### <a name="conversation-object"></a>Conversation 对象
定义聊天，包括聊天中包含的机器人和用户。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **bot** | [ChannelAccount](#channelaccount-object) | 一个标识机器人的 **ChannelAccount** 对象。 |
| **isGroup** | 布尔值 | 一个标志，指示这是否是群组聊天。 如果这是群组聊天，则设置为 **true**；否则设置为 **false**。 默认值为 **false**。 若要启动群组聊天，通道必须支持群组聊天。 |
| **members** | [ChannelAccount](#channelaccount-object)[] | 一个 **ChannelAccount** 对象数组，用于标识聊天成员。 除非 **isGroup** 设置为 **true**，否则此列表必须包含单个用户。 此列表可能包含其他机器人。 |
| **topicName** | 字符串 | 聊天的标题。 |
| **activity** | [活动](#activity-object) | 在[创建聊天](#create-conversation)请求中，它为 **Activity** 对象，用于定义要发布到新聊天的第一条消息。 |

<a href="#objects">返回到架构表</a>

### <a name="conversationaccount-object"></a>ConversationAccount 对象
定义通道中的聊天。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **id** | 字符串 | 用于标识聊天的 ID。 每个通道的 ID 都是唯一的。 如果通道启动聊天，它会设置此 ID；否则，机器人会将此属性设置为启动聊天时在响应中返回的 ID（请参阅“启动聊天”）。 |
| **isGroup** | 布尔值 | 一个标志，指示聊天在活动生成时是否包含两个以上的参与者。 如果这是群组聊天，则设置为 **true**；否则设置为 **false**。 默认值为 **false**。 |
| name | 字符串 | 可用于标识聊天的显示名称。 |
| **conversationType** | 字符串 | 指示区分聊天类型（例如：群组、个人）的通道中的聊天类型。 |

<a href="#objects">返回到架构表</a>

### <a name="conversationparameters-object"></a>ConversationParameters 对象
定义用于创建新聊天的参数

| 属性 | 类型 | Description |
|----|----|----|
| **isGroup** | 布尔值 | 指示这是否是群组聊天。 |
| **bot** | [ChannelAccount](#channelaccount-object) | 聊天中机器人的地址。 |
| **members** | 数组 | 要添加到聊天的成员的列表。 |
| **topicName** | 字符串 | 聊天的主题标题。 仅在通道支持时才使用此属性。 |
| **activity** | [活动](#activity-object) | （可选）在创建新聊天时，将此活动用作聊天的初始消息。 |
| **channelData** | 对象 | 用于创建聊天的通道特定有效负载。 |

### <a name="conversationreference-object"></a>ConversationReference 对象
定义聊天中的特定点。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **activityId** | 字符串 | 一个 ID，用于唯一标识此对象引用的活动。 | 
| **bot** | [ChannelAccount](#channelaccount-object) | 一个 **ChannelAccount** 对象，用于标识此对象引用的聊天中的机器人。 |
| **channelId** | 字符串 | 一个 ID，用于唯一标识此对象引用的聊天中的通道。 | 
| 对话 | [ConversationAccount](#conversationaccount-object) | 一个 **ConversationAccount** 对象，用于定义此对象引用的聊天。 |
| **serviceUrl** | 字符串 | 一个 URL，用于指定此对象引用的聊天中的通道服务终结点。 | 
| **user** | [ChannelAccount](#channelaccount-object) | 一个 **ChannelAccount** 对象，用于标识此对象引用的聊天中的用户。 |

<a href="#objects">返回到架构表</a>

### <a name="conversationresourceresponse-object"></a>ConversationResourceResponse 对象
定义包含资源的响应。

| 属性 | 类型 | Description |
|----|----|----|
| **activityId** | 字符串 | 活动的 ID。 |
| **id** | 字符串 | 资源的 ID。 |
| **serviceUrl** | 字符串 | 服务终结点。 |

### <a name="error-object"></a>Error 对象
定义错误。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **code** | 字符串 | 错误代码。 |
| **message** | 字符串 | 对错误的说明。 |

<a href="#objects">返回到架构表</a>

### <a name="entity-object"></a>Entity 对象
定义实体对象。

| 属性 | 类型 | Description |
|----|----|----|
| type | 字符串 | 实体类型。 通常包含 schema.org 中的类型。 |


### <a name="errorresponse-object"></a>ErrorResponse 对象
定义 HTTP API 响应。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **error** | [错误](#error-object) | 一个包含错误相关信息的 **Error** 对象。 |

<a href="#objects">返回到架构表</a>

### <a name="fact-object"></a>Fact 对象
定义包含事实的键值对。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **键** | 字符串 | 事实的名称。 例如，**Check-in**。 在显示事实值时，该键用作标签。 |
| **值** | 字符串 | 事实的值。 例如，**10 October 2016**。 |

<a href="#objects">返回到架构表</a>

### <a name="geocoordinates-object"></a>Geocoordinates 对象
使用世界测地系统 (WSG84) 坐标定义地理位置。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **elevation** | 数字 | 位置的海拔高度。 |
| name | 字符串 | 位置的名称。 |
| **latitude** | 数字 | 位置的纬度。 |
| **longitude** | 数字 | 位置的经度。 |
| type | 字符串 | 此对象的类型。 始终设置为 **GeoCoordinates**。 |

<a href="#objects">返回到架构表</a>

### <a name="herocard-object"></a>HeroCard 对象
定义具有大图像、标题、文本和操作按钮的卡。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户执行一项或多项操作。 通道决定了可指定的按钮数。 |
| **images** | [CardImage](#cardimage-object)[] | 一个 **CardImage** 对象数组，用于指定要在卡上显示的图像。 英雄卡仅包含一个图像。 |
| **subtitle** | 字符串 | 要在卡标题下显示的副标题。 |
| **tap** | [CardAction](#cardaction-object) | 一个 **CardAction** 对象，用于指定用户点击或单击卡时要执行的操作。 它可以是与其中一个按钮相同的操作，也可以是不同的操作。 |
| **text** | 字符串 | 要在卡的标题或副标题下显示的描述或提示。 |
| **title** | 字符串 | 卡的标题。 |

<a href="#objects">返回到架构表</a>

<!--TODO can't find-->
### <a name="identification-object"></a>Identification 对象
标识资源。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **id** | 字符串 | 用于唯一标识资源的 ID。 |

<a href="#objects">返回到架构表</a>

### <a name="mediaeventvalue-object"></a>MediaEventValue 对象 
媒体事件的补充参数。

| 属性 | 类型 | Description |
|----|----|----|
| **cardValue** | 对象 | 在发出此事件的媒体卡的“值”字段中指定的回调参数。 |

### <a name="mediaurl-object"></a>MediaUrl 对象
定义媒体文件源的 URL。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **profile** | 字符串 | 用于描述媒体内容的提示。 |
| **url** | 字符串 | 媒体文件源的 URL。 |

<a href="#objects">返回到架构表</a>

<!--TODO can't find-->
### <a name="mention-object"></a>Mention 对象
定义聊天中提到的用户或机器人。<br/><br/> 


|          属性          |                   类型                   |                                                                                                                                                                                                                           Description                                                                                                                                                                                                                            |
|----------------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <strong>mentioned</strong> | [ChannelAccount](#channelaccount-object) | 一个 <strong>ChannelAccount</strong> 对象，用于指定所提到的用户或机器人。 请注意，某些通道（如 Slack）按聊天分配名称，因此，所提到的机器人名称（在消息的 <strong>recipient</strong> 属性中）可能与你在[注册](../bot-service-quickstart-registration.md)机器人时指定的句柄不同。 但是，两者的帐户 ID 相同。 |
|   <strong>text</strong>    |                  字符串                  |                                                                                                                         聊天中提到的用户或机器人。 例如，如果消息为“@ColorBot pick me a new color”，则此属性将设置为 <strong>@ColorBot</strong>。 并非所有通道都设置此属性。                                                                                                                          |
|   type    |                  字符串                  |                                                                                                                                                                                                   此对象的类型。 始终设置为 <strong>Mention</strong>。                                                                                                                                                                                                    |

<a href="#objects">返回到架构表</a>

### <a name="messagereaction-object"></a>MessageReaction 对象
定义对消息的回应。

| 属性 | 类型 | Description |
|----|----|----|
| type | 字符串 | 回应的类型。 |

### <a name="place-object"></a>Place 对象
定义聊天中提到的位置。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **address** | 对象 |  某个位置的地址。 此属性可以是 `string` 或 `PostalAddress` 类型的复杂对象。 |
| **geo** | [GeoCoordinates](#geocoordinates-object) | 一个 **GeoCoordinates** 对象，用于指定该位置的地理坐标。 |
| **hasMap** | 对象 | 该位置的地图。 此属性可以是 `string` (URL) 或 `Map` 类型的复杂对象。 |
| name | 字符串 | 该位置的名称。 |
| type | 字符串 | 此对象的类型。 始终设置为 **Place**。 |

<a href="#objects">返回到架构表</a>

### <a name="receiptcard-object"></a>ReceiptCard 对象
定义包含购买收据的卡。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户执行一项或多项操作。 通道决定了可指定的按钮数。 |
| **facts** | [Fact](#fact-object)[] | 一个 **Fact** 对象数组，用于指定有关购买的信息。 例如，酒店住宿收据的事实列表可能包括登记入住日期和退房日期。 通道决定了可指定的事实数量。 |
| **items** | [ReceiptItem](#receiptitem-object)[] | 一个 **ReceiptItem** 对象数组，用于指定已购买的商品 |
| **tap** | [CardAction](#cardaction-object) | 一个 **CardAction** 对象，用于指定用户点击或单击卡时要执行的操作。 它可以是与其中一个按钮相同的操作，也可以是不同的操作。 |
| **tax** | 字符串 | 一个货币格式的字符串，用于指定应用于所购商品的税额。 |
| **title** | 字符串 | 显示在收据顶部的标题。 |
| **total** | 字符串 | 一个货币格式的字符串，用于指定总购买价格，包括所有适用的税费。 |
| **vat** | 字符串 | 一个货币格式的字符串，用于指定应用于购买价格的增值税 (VAT) 金额。 |

<a href="#objects">返回到架构表</a>

### <a name="receiptitem-object"></a>ReceiptItem 对象
定义收据中的订单项。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **图片** | [CardImage](#cardimage-object) | 一个 **CardImage** 对象，用于指定要在订单项旁边显示的缩略图。  |
| **price** | 字符串 | 一个货币格式的字符串，用于指定所有已购单位的总价。 |
| **quantity** | 字符串 | 一个数字字符串，用于指定已购单位的数量。 |
| **subtitle** | 字符串 | 要在订单项标题下显示的副标题。 |
| **tap** | [CardAction](#cardaction-object) | 一个 **CardAction** 对象，用于指定用户点击或单击订单项时要执行的操作。 |
| **text** | 字符串 | 订单项描述。 |
| **title** | 字符串 | 订单项的标题。 |

<a href="#objects">返回到架构表</a>

### <a name="resourceresponse-object"></a>ResourceResponse 对象
定义包含资源 ID 的响应。<br/><br/>


|      属性       |  类型  |                Description                |
|---------------------|--------|-------------------------------------------|
| <strong>id</strong> | 字符串 | 用于唯一标识资源的 ID。 |

<a href="#objects">返回到架构表</a>

### <a name="signincard-object"></a>SignInCard 对象
定义允许用户登录服务的卡。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户登录到服务。 通道决定了可指定的按钮数。 |
| **text** | 字符串 | 要在登录卡上包含的描述或提示。 |

<a href="#objects">返回到架构表</a>

### <a name="suggestedactions-object"></a>SuggestedActions 对象
定义用户可以选择的选项。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **actions** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，用于定义建议的操作。 |
| **to** | string[] | 一个字符串数组，包含应向其显示建议操作的接收者的 ID。 |

<a href="#objects">返回到架构表</a>

### <a name="thumbnailcard-object"></a>ThumbnailCard 对象
定义具有缩略图、标题、文本和操作按钮的卡。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户执行一项或多项操作。 通道决定了可指定的按钮数。 |
| **images** | [CardImage](#cardimage-object)[] | 一个 **CardImage** 对象数组，用于指定要在卡上显示的缩略图。 通道决定了可指定的缩略图数量。 |
| **subtitle** | 字符串 | 要在卡标题下显示的副标题。 |
| **tap** | [CardAction](#cardaction-object) | 一个 **CardAction** 对象，用于指定用户点击或单击卡时要执行的操作。 它可以是与其中一个按钮相同的操作，也可以是不同的操作。 |
| **text** | 字符串 | 要在卡的标题或副标题下显示的描述或提示。 |
| **title** | 字符串 | 卡的标题。 |

<a href="#objects">返回到架构表</a>

### <a name="thumbnailurl-object"></a>ThumbnailUrl 对象
定义图像源的 URL。<br/><br/> 

| 属性 | 类型 | Description |
|----|----|----|
| **alt** | 字符串 | 图像描述。 应包含描述以支持可访问性。 |
| **url** | 字符串 | 图像源或图像的 base64 二进制文件的 URL（例如，`data:image/png;base64,iVBORw0KGgo...`）。 |

<a href="#objects">返回到架构表</a>

### <a name="videocard-object"></a>VideoCard 对象
定义可播放视频的卡。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **aspect** | 字符串 | 视频的纵横比（例如：16:9、4:3）。|
| **autoloop** | 布尔值 | 一个标志，指示在最后一个视频结束后是否重播视频列表。 若要自动重播视频，请将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **autostart** | 布尔值 | 一个标志，指示是否在显示卡时自动播放视频。 若要自动播放视频，请将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **buttons** | [CardAction](#cardaction-object)[] | 一个 **CardAction** 对象数组，允许用户执行一项或多项操作。 通道决定了可指定的按钮数。 |
| **duration** | 字符串 | 媒体内容的长度，采用 [ISO 8601 持续时间格式](https://www.iso.org/iso-8601-date-and-time-format.html)。 |
| **图片** | [ThumbnailUrl](#thumbnailurl-object) | 一个 **ThumbnailUrl** 对象，用于指定要在卡上显示的图像。 |
| **media** | [MediaUrl](#mediaurl-object)[] | 一个 **MediaUrl** 对象数组，用于指定要播放的视频的列表。 |
| **shareable** | 布尔值 | 一个标志，指示是否可以与其他人共享视频。 如果可以共享视频，则将此属性设置为 **true**；否则设置为 **false**。 默认值为 **true**。 |
| **subtitle** | 字符串 | 要在卡标题下显示的副标题。 |
| **text** | 字符串 | 要在卡的标题或副标题下显示的描述或提示。 |
| **title** | 字符串 | 卡的标题。 |
| **值** | 对象 | 此卡的补充参数|

<a href="#objects">返回到架构表</a>

### <a name="semanticaction-object"></a>SemanticAction 对象
定义对编程操作的引用。<br/><br/>

| 属性 | 类型 | Description |
|----|----|----|
| **id** | 字符串 | 此操作的 ID |
| **entities** | [实体](#entity-object) | 与此操作关联的实体 |

<a href="#objects">返回到架构表</a>
