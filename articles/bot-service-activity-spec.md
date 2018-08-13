---
title: Bot Framework 规范 | Microsoft Docs
description: Bot Framework 规范
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/07/2018
ms.openlocfilehash: 0406d489f7d1e27131b4b01411e86850ca4a17b8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297063"
---
# <a name="bot-framework----activity"></a>Bot Framework -- 活动

## <a name="abstract"></a>摘要

Bot Framework 活动架构是人类和自动化软件执行的聊天操作的应用程序级表示。 此架构包括用于传输文本、多媒体和非内容操作（例如社交互动和键入指示器）的预配。

此架构用于 Bot Framework 协议内并由 Microsoft 聊天系统和交互机器人以及来自各种源的客户端实现。

## <a name="table-of-contents"></a>目录

1. [介绍](#introduction)
2. [基本活动结构](#basic-activity-structure)
3. [消息活动](#message-activity)
4. [联系人关系更新活动](#contact-relation-update-activity)
5. [聊天更新活动](#conversation-update-activity)
6. [聊天结束活动](#end-of-conversation-activity)
7. [事件活动](#event-activity)
8. [调用活动](#invoke-activity)
9. [安装更新活动](#installation-update-activity)
10. [消息删除活动](#message-delete-activity)
11. [消息更新活动](#message-update-activity)
12. [消息回应活动](#message-reaction-activity)
13. [键入活动](#typing-activity)
14. [复杂类型](#complex-types)
15. [参考](#references)
16. [附录 I - 更改](#appendix-i---changes)
17. [附录 II - 非 IRI 实体类型](#appendix-ii---non-iri-entity-types)
18. [附录 III - 使用调用活动的协议](#appendix-iii---protocols-using-the-invoke-activity)

## <a name="introduction"></a>介绍

### <a name="overview"></a>概述

Bot Framework Activity 架构表示由人类和自动化软件在聊天应用程序、电子邮件和其他文本交互程序中所做的聊天行为。 每个活动对象都包括一个类型字段并表示单个操作：通常是发送文本内容，但也包括多媒体附件和非内容行为，例如“赞”按钮或键入指示器。

本文档提供了每种活动类型的含义，并介绍了可能包括的必需和可选字段。 它还定义了客户端和服务器的角色，并提供了有关每个参与者必须掌握哪些字段可以忽略哪些字段的指导。

此规范中有三个重要角色：客户端，它们代表用户发送和接收活动；机器人，它们发送和接收活动并且通常是自动化的；通道，它存储活动并在客户端与机器人之间转发活动。

虽然此规范要求在角色之间传输活动，但此处未介绍该传输的确切性质。

为了实现紧凑性，此规范中未定义可视化交互卡。 对于这方面的内容，请参阅[自适应卡](https://adaptivecards.io) [[11](#references)]规范。 此卡和其他未定义的卡类型可以作为附件包括在 Bot Framework 活动中。

### <a name="requirements"></a>要求

本文档中的关键字“必须”、“不得”、“必需”、“应”、“不应”、“应当”、“不应当”、“建议”、“可以”和“可选”根据 [RFC 2119](https://tools.ietf.org/html/rfc2119) [[1](#references)] 中所述予以解释。

如何某个实现不满足它实现的协议的“必须”或“必需”级别的一个或多个要求，则它不符合规范。 满足其协议的所有“必须”或“必需”级别和所有“应当”级别要求的实现称为“无条件地符合”；满足其协议的所有“必须”级别要求但并非满足所有“应当”级别要求的实现称为“有条件地符合”。

### <a name="numbered-requirements"></a>带编号的要求

以 `RXXXX` 形式的标记开头的行是设计为按本文档之外的讨论中的编号引用的特定要求。 它们的重要性不比在 `RXXXX` 行之外所做的规范陈述更高或更低。

`R1000`：此规范的编辑者“可以”添加新的 `RXXXX` 要求。 他们“应当”查找保留了文档的流的数字 `RXXXX` 值。

`R1001`：编辑者“不得”对现有 `RXXXX` 要求进行重新编号。

`R1002`：编辑者“可以”删除或修订 `RXXXX` 要求。 如果修订，如果要求的主题基本未变，编辑者“应当”保留现有 `RXXXX` 值。

`R1003`：编辑者“不应当”重复使用已停用的 `RXXXX` 值。 “可以”将已删除的值的列表保留在本文档的末尾。

### <a name="terminology"></a>术语

活动
> 由机器人、通道或客户端表达的符合活动架构的操作。

通道
> 用来发送和接收活动，并将其与聊天或应用程序行为进行相互转换的软件。 通道是活动数据的权威存储。

机器人
> 用来发送和接收活动，并生成自动化、半自动化或完全手动响应的软件。 机器人具有向通道进行了注册的终结点。

客户端
> 通常代表人类用户来发送和接收活动的软件。 客户端没有终结点。

发送方
> 传输活动的软件。

接收方
> 接受活动的软件。

终结点
> 机器人或通道可以在其中接收活动的可编程寻址位置。

地址
> 可以从其中联系用户或机器人的标识符或地址。

字段
> 活动或嵌套对象内的已命名值。

### <a name="overall-organization"></a>整个组织

活动对象是名称/值对的简单列表，其中一些是基元对象，另一些是复杂对象（嵌套对象）。 活动对象通常以 JSON 格式表示，但是也可以映射为 .Net 或 JavaScript 中的内存中数据结构。

活动 `type` 字段控制活动和其中包含的字段的含义。 每个字段是必需的、可选的或被忽略的，具体取决于参与者所扮演的角色（客户端、机器人或通道）。 例如，`id` 字段归通道拥有，在某些情况下是必需的，但如果是由机器人发送的，则会被忽略。

描述活动标识和任何参与者的字段（例如 `type` 和 `from` 字段）是在所有活动之间共享的。 在许多编程语言中，可以方便地基于一个核心基类型对这些字段进行组织，可以从该基类型派生其他更具体的活动类型。

当存储或传输活动时，某些字段在传输机制内可能会重复。 例如，如果活动通过 HTTP POST 传输到包括聊天 ID 的 URL，则接收方可以推断其值，不要求在活动正文内提供该值。 本文档只是描述了这些字段的抽象要求，是必须显式声明这些值还是允许使用隐式值或推断值由控制协议确定。

当机器人或客户端向通道发送活动时，通常会请求记录该活动。 当通道向机器人或客户端发送活动时，通常会通知该活动已记录。

## <a name="basic-activity-structure"></a>基本活动结构

本部分定义了对活动对象的基本结构的要求。

活动对象包括名称/值对的简单列表，称为字段。 字段可以是基元类型。 JSON 用作通用交换格式，虽然并非所有活动都必须始终序列化为 JSON，但它们必须可以序列化为 JSON。 这使得各个实现能够依靠一组简单的约定来处理已知的和未知的活动字段。

`R2001`：活动“必须”能够序列化为 JSON 格式，包括符合字段唯一性约束等约束。

`R2002`：接收方“可以”允许大小写不正确的字段名称，但这不是必需的。 接收方“可以”拒绝未包含大小写正确的字段的活动。

`R2003`：接收方“可以”拒绝包含了其类型与本规范中描述的值类型不匹配的字段值的活动。

`R2004`：除非另行说明，发送方“不应当”为字符串字段包括空的字符串值。

`R2005`：除非另行说明，发送方“可以”在活动或任何嵌套的复杂对象中包括其他字段。 接收方“必须”接受它们不理解的字段。

`R2006`：接收方“应当”接受它们不理解的类型的事件。

### <a name="type"></a>类型

`type` 字段控制每个活动的含义，并且根据约定应当是短字符串（例如 "`message`"）。 发送方可以定义其自己的应用程序层类型，但是建议它们选择不可能与将来明确定义的值冲突的值。 如果发送方使用 URI 作为类型值，则它们“不应当”实现 URI 阶梯比较来建立等效性。

`R2010`：活动“必须”包括一个具有字符串值类型的 `type` 字段。

`R2011`：如果两个 `type` 值的序号完全相同，则它们是等效的。

`R2012`：发送方“可以”生成本文档中未定义的活动 `type` 值。

`R2013`：通道“应当”拒绝它不理解的类型的活动。

`R2014`：机器人或客户端“应当”忽略它不理解的类型的活动。

### <a name="channel-id"></a>通道 ID

`channelId` 字段为活动建立通道和权威存储。 `channelId` 字段的值是字符串类型的。

`R2020`：活动“必须”包括一个具有字符串值类型的 `channelId` 字段。

`R2021`：如果两个 `channelId` 值的序号完全相同，则它们是等效的。

`R2022`：通道“可以”忽略或拒绝它收到的没有预期的 `channelId` 值的任何活动。

### <a name="id"></a>ID

`id` 字段在活动已记录到通道中后为活动建立标识。 尚未记录的正在进行的活动没有标识。 并非所有活动都分配有标识（例如，[键入活动](#typing-activity)可能从来不会分配有 `id`。）`id` 字段的值是字符串类型的。

`R2030`：通道“应当”包括一个 `id` 字段，前提是此字段适用于该活动。

`R2031`：客户端和机器人“不应当”在它们生成的活动中包括 `id` 字段。

为便于实现，应当假设其他参与者没有掌握关于活动 ID 的丰富知识，并且它们将仅使用序号比较来建立等效性。

例如，通道可以为每个活动 ID 使用十六进制编码的 GUID。 尽管以大写字母编码的 GUID 在逻辑上等效于以小写字母编码的 GUID，但发送方只要有可能就“不应当”使用这些替代编码。 每个 ID 的规范化版本是由权威存储（即通道）建立的。

`R2032`：生成 `id` 值时，发送方“应当”选择可以通过序号比较建立其等效性的值。 不过，如果发送方和接收方特别了解情况，则它们“可以”允许序号不等效的两个值的逻辑等效性。

`id` 字段设计为允许重复数据删除，但这在大多数应用程序中是禁止的。

`R2033`：接收方“可以”按 ID 删除重复的活动，不过，发送方“不应当”指望接收方执行此重复数据删除。

### <a name="timestamp"></a>时间戳

`timestamp` 字段记录活动发生时的确切 UTC 时间。 由于计算系统的分布式性质，重要的时间是通道（权威存储）记录活动的时间。 客户端或机器人启动活动的时间可以单独在 `localTimestamp` 字段中传输。 `timestamp` 字段的值是字符串内的 [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] 编码的日期时间。

`R2040`：通道“应当”包括一个 `timestamp` 字段，前提是此字段适用于该活动。

`R2041`：客户端和机器人“不应当”在它们生成的活动中包括 `timestamp` 字段。

`R2042`：客户端和机器人“不应当”使用 `timestamp` 来拒绝活动，因为活动可能乱序出现。 不过，它们“可以”使用 `timestamp` 在 UI 内对活动进行排序或者将其用于下游处理。

`R2043`：发送方“应当”始终对 `timestamp` 字段使用 UTC 编码格式的值，并且它们“应当”始终在该值中使用 Z 作为显式 UTC 标记。

### <a name="local-timestamp"></a>本地时间戳

`localTimestamp` 字段表示生成活动的日期时间和时区偏移量。 这可能不同于记录活动的 UTC `timestamp`。 `localTimestamp` 字段的值是字符串内的 ISO 8601 [[3](#references)] 编码的日期时间。

`R2050`：客户端和机器人“可以”在其活动中包括 `localTimestamp` 字段。 它们“应当”在编码值中显式列出时区偏移量。

`R2051`：将来自发送方的活动转发到接收者时，通道“应当”保留 `localTimestamp`。

### <a name="from"></a>源

`from` 字段描述哪个客户端、机器人或通道生成了活动。 `from` 字段的值是[通道帐户](#channel-account)类型的复杂对象。

`from.id` 字段标识谁生成了活动。 大多数情况下，这是系统中的另一个用户或机器人。 某些情况下，`from` 字段标识通道自身。

`R2060`：通道在生成活动时“必须”包括 `from` 和 `from.id` 字段。

`R2061`：机器人和客户端在生成活动时“应当”包括 `from` 和 `from.id` 字段。 通道“可以”由于缺少 `from` 和 `from.id` 字段而拒绝活动。

`from.name` 字段是可选的，表示帐户在通道内的显示名称。 通道“应当”包括此值，以便客户端和机器人可以填充其 UI 和后端系统。 机器人和客户端“不应当”将此值发送到具有此存储的中央记录的通道，但是它们“可以”将此值发送到在每个活动上填充此值的通道（例如电子邮件通道）。

`R2062`：如果 `from` 字段存在并且 `from.name` 可用，则通道“应当”包括 `from.name` 字段。

`R2063`：机器人和客户端“不应当”包括 `from.name` 字段，除非它在通道内在语义上有价值。

### <a name="recipient"></a>接收者

`recipient` 字段描述哪个客户端或机器人接收此活动。 只有当活动仅传输给一个接收者时此字段才有意义；当活动广播到多个接收者时（像活动发送到通道时发生的情况一样），此字段没有意义。 此字段的用途是使接收者可以识别它们自己。 当客户端或机器人在通道内有多个标识时，此字段比较有用。 `recipient` 字段的值是[通道帐户](#channel-account)类型的复杂对象。

`R2070`：将活动传输到单个接收者时，通道“必须”包括 `recipient` 和 `recipient.id` 字段。

`R2071`：机器人和客户端在生成活动时“不应当”包括 `recipient` 字段。

`recipient.name` 字段是可选的，表示帐户在通道内的显示名称。 通道“应当”包括此值，以便客户端和机器人可以填充其 UI 和后端系统。

`R2072`：如果 `recipient` 字段存在并且 `recipient.name` 可用，则通道“应当”包括 `recipient.name` 字段。

### <a name="conversation"></a>聊天

`conversation` 字段描述了活动所在的聊天。 `conversation` 字段的值是[聊天帐户](#conversation-account)类型的复杂对象。

`R2080`：通道、机器人和客户端在生成活动时“必须”包括 `conversation` 和 `conversation.id` 字段。

`conversation.name` 字段是可选的，如果它存在并可用，则它表示聊天的显示名称。

`R2081`：通道“应当”包括 `conversation.name` 和 `conversation.isGroup` 字段，前提是这些字段可用。

`R2082`：机器人和客户端“不应当”包括 `conversation.name` 字段，除非它在通道内在语义上有价值。

`R2083`：机器人和客户端“不应当”在它们生成的活动中包括 `conversation.isGroup` 字段。

### <a name="reply-to-id"></a>答复对象 ID

`replyToId` 字段标识当前活动是其答复的上一个活动。 此字段允许在参与者之间传达链式聊天和评论嵌套。 `replyToId` 仅在当前聊天内有效。 （有关对其他聊天的引用，请参阅 [relatesTo](#relates-to)。）`replyToId` 字段的值是字符串。

`R2090`：当活动是对另一个活动的答复时，发送方“应当”在活动上包括 `replyToId`。

`R2091`：如果某个活动的 `replyToId` 未引用聊天内的有效活动，则通道“可以”拒绝该活动。

`R2092`：如果知道通道不使用 `replyToId` 字段，则机器人和客户端“可以”省略该字段，即使要发送的活动是对另一个活动的答复。

### <a name="entities"></a>实体

`entities` 字段包含与此活动相关的元数据对象的简单列表。 不同于附件（请参阅 [attachments](#attachments) 字段），实体并非必须在清单中列为用户可交互内容元素，并且如果不被理解将被忽略。 发送方可以包括它们认为对于接收方可能有用的实体，即使它们不确定接收方是否可以接受这些实体。 每个 `entities` 列表元素的值都是[实体](#entity)类型的一个复杂对象。

`R2100`：如果 `entities` 字段未包含任何元素，则发送方“应当”省略该字段。

`R2101`：发送方“可以”发送同一类型的多个实体，前提是这些实体具有不同的含义。

`R2102`：发送方“不得”具有类型和内容完全相同的两个或更多实体。

`R2103`：发送方和接收方“不应当”依赖于活动中包括的实体的特定顺序。

### <a name="channel-data"></a>通道数据

活动架构中的扩展性数据大部分组织在 `channelData` 字段中。 这简化了实现了协议的 SDK 中的管道工程。 `channelData` 对象的格式是由发送或接收活动的通道定义的。

`R2200`：通道“不应当”定义是 JSON 基元（例如字符串、整数）的 `channelData` 格式。 相反，它们“应当”将 `channelData` 定义为复杂类型，或者将其保留为未定义的。

`R2201`：如果没有为当前通道定义 `channelData` 格式，则接收方“应当”忽略 `channelData` 的内容。

### <a name="service-url"></a>服务 URL

活动通常是异步发送的，使用不同的传输连接来发送和接收流量。 `serviceUrl` 字段由通道用来指示必须将对当前活动的答复发送到的 URL。 `serviceUrl` 字段的值是字符串类型的。

`R2300`：通道“必须”在它们发送到机器人的所有活动中包括 `serviceUrl` 字段。

`R2301`：对于展示了它们已知道通道终结点的客户端，通道“不应当”为其包括 `serviceUrl` 字段。

`R2302`：机器人和客户端“不应当”在它们生成的活动中填充 `serviceUrl` 字段。

`R2302`：通道“必须”忽略由机器人和客户端发送的活动中的 `serviceUrl` 的值。

`R2304`：通道“应当”为 `serviceUrl` 字段使用稳定的值，因为机器人可以长期保留这些值。

## <a name="message-activity"></a>消息活动

消息活动提供要在聊天界面中显示的内容。 消息活动可以包含文本、语音、交互卡以及二进制文件或未知附件；通常，通道最多需要这些项之一，消息活动便可正确设置格式。

消息活动由 `type` 值 `message` 予以标识。

### <a name="text"></a>文本

`text` 字段包含文本内容，采用 Markdown 格式、XML 或纯文本。 格式由 [`textFormat`](#text-format) 字段控制，如果未指定或不明确，则视为纯文本。 `text` 字段的值是字符串类型的。

`R3000`：`text` 字段“可以”包含空字符串来指示发送不含内容的文本。

`R3001`：通道“应当”以针对该通道从容降级的方式处理 `markdown` 格式的文本。

### <a name="text-format"></a>文本格式

`textFormat` 字段指示 [`text`](#text) 字段应当解释为 [Markdown](https://daringfireball.net/projects/markdown/) [[4](#references)]、纯文本还是 XML。 `textFormat` 字段的值是字符串类型的，具有定义的值 `markdown`、`plain` 和 `xml`。 默认值为 `plain`。 此字段未设计为使用任意值进行扩展。

`textFormat` 字段控制附件等项目中的其他字段。此关系在本文档中的其他位置在那些字段中进行了描述。

`R3010`：如果发送方包括 `textFormat` 字段，则它“应当”仅发送已定义的值。

`R3011`：如果值为 `plain`，则发送方“应当”省略 `textFormat`。

`R3012`：接收方“应当”将未定义的值解释为 `plain`。

`R3013`：机器人和客户端“不应当”发送值 `xml`，除非它们事先知道通道支持该值，并且知道受支持的 XML 方言的特征。

`R3014`：通道“不应当”向机器人发送 `markdown` 或 `xml` 内容。

`R3015`：通道“应当”接受至少为 `plain` 和 `markdown` 的 `textformat` 值。

`R3016`：通道“可以”拒绝 `textformat` 值 `xml`。

### <a name="locale"></a>区域设置

`locale` 字段传达 [`text`](#text) 字段的语言代码。 `locale` 字段的值是字符串内的一个 [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] 代码。

`R3020`：接收方“应当”将 `locale` 字段的缺少值和未知值视为未知的。

`R3021`：接收方“不应当”拒绝具有未知区域设置的活动。

### <a name="speak"></a>讲述

`speak` 字段指示应当如何通过“文本转语音”系统来讲述活动。 此字段仅用来自定义当默认值被认为不合适时的语音呈现。 它替代了针对活动内任何内容（包括文本、附件和摘要）的语音合成。 `speak` 字段的值在字符串内进行 [SSML](https://www.w3.org/TR/speech-synthesis/) [[6](#references)] 编码。 允许使用未包装的文本，并且会自动将其升级为裸 SSML。

`R3030`：`speak` 字段“可以”包含空字符串来指示不应当生成语音。

`R3031`：无法生成语音的接收方“应当”忽略 `speak` 字段。

`R3032`：如果未找到根 SSML 元素，则接收方“必须”考虑将包括的文本作为 SSML `<speak>` 标记的内部内容（前面有有效的 [XML Prolog](https://www.w3.org/TR/xml/#sec-prolog-dtd) [[7](#references)]，否则会升级为有效的 SSML 文档）。

`R3033`：接收方“不应当”使用 XML DTD 或架构解决方案包含来自所传送 XML 片段之外的远程资源。

`R3034`：通道“不应当”向机器人发送 `speak` 字段。

### <a name="input-hint"></a>输入提示

`inputHint` 字段指示活动的生成器是否期待响应。 此字段主要用于具有模式用户界面的通道内，并且通常不用于具有连续聊天馈送的通道中。 `inputHint` 字段的值是字符串类型的，具有定义的值 `accepting`、`expecting` 和 `ignoring`。 默认值为 `accepting`。

`R3040`：如果发送方包括 `inputHint` 字段，则它“应当”仅发送已定义的值。

`R3041`：如果向通道发送其中使用了 `inputHint` 的活动，则机器人“应当”包括此字段，即使当值为 `accepting` 时也是如此。

`R3042`：接收方“应当”将未定义的值解释为 `accepting`。

### <a name="attachments"></a>附件

`attachments` 字段包含要显示为此活动的一部分的对象的简单列表。 每个 `attachments` 列表元素的值都是[附件](#attachment)类型的一个复杂对象。

`R3050`：如果 `attachments` 字段未包含任何元素，则发送方“应当”省略该字段。

`R3051`：发送方“可以”发送同一类型的多个实体。

`R3052`：接收方“可以”将未知类型的附件视为可下载的文档。

`R3053`：接收方在处理内容时“应当”保留附件的顺序，除非呈现限制强制进行更改，例如，将文档分组到图像之后。

### <a name="attachment-layout"></a>附件布局

`attachmentLayout` 字段指示用户界面呈现器如何呈现 [`attachments`](#attachments) 字段中包括的内容。 `attachmentLayout` 字段的值是字符串类型的，具有定义的值 `list` 和 `carousel`。 默认值为 `list`。

`R3060`：如果发送方包括 `attachmentLayout` 字段，则它“应当”仅发送已定义的值。

`R3061`：接收方“应当”将未定义的值解释为 `list`。

### <a name="summary"></a>摘要

`summary` 字段包含用来在不支持 [`attachments`](#attachments) 的通道上对其进行替换的文本。 `summary` 字段的值是字符串类型的。

`R3070`：接收方“应当”考虑 `summary` 字段在逻辑上跟在 `text` 字段之后。

`R3071`：通道“不应当”向机器人发送 `summary` 字段。

`R3072`：能够处理活动内的所有附件的通道“应当”忽略 `summary` 字段。

### <a name="suggested-actions"></a>建议的操作

`suggestedActions` 字段包含可以显示给用户的交互操作的有效负载。 是否支持 `suggestedActions` 及其显示很大程度上取决于通道。 `suggestedActions` 字段的值是[建议的操作](#suggested-actions-2)类型的复杂对象。

### <a name="value"></a>值

`value` 字段包含特定于要发送的活动的编程有效负载。 本文档的其他部分中描述了其用法并定义了其含义和格式。

`R3080`：发送方“不应当”包括是基元类型（例如字符串、整数）的 `value` 字段。 `value` 字段“应当”是复杂类型或省略。

### <a name="expiration"></a>过期时间

`expiration` 字段包含一个时间，活动在该时间应当被视为“过期”并且不应发送给接收者。 `expiration` 字段的值是字符串内的 [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html) [[2](#references)] 编码的日期时间。

`R3090`：发送方“应当”始终对 `expiration` 字段使用 UTC 编码格式的值，并且它们“应当”始终在该值中使用 Z 作为显式 UTC 标记。

### <a name="importance"></a>重要性

`importance` 字段包含一组枚举的值，用以向接收者表明活动的相对重要性。  接收方负责将这些重要性提示映射到用户体验。 `importance` 字段的值是字符串类型的，具有定义的值 `low`、`normal` 和 `high`。 默认值为 `normal`。

`R3100`：如果发送方包括 `importance` 字段，则它“应当”仅发送已定义的值。

`R3101`：接收方“应当”将未定义的值解释为 `normal`。

### <a name="deliverymode"></a>DeliveryMode

`deliveryMode` 字段包含一组枚举的值中的一个，用以向接收者表明活动的替代传递路径。  `DeliveryMode` 字段的值是字符串类型的，具有定义的值 `normal` 和 `notification`。 默认值为 `normal`。

`R3110`：如果发送方包括 `deliveryMode` 字段，则它“应当”仅发送已定义的值。

`R3111`：接收方“应当”将未定义的值解释为 `normal`。

## <a name="contact-relation-update-activity"></a>联系人关系更新活动

联系人关系更新活动表明接收者与通道内的某个用户之间的关系发生了变化。 联系人关系更新活动通常不包含用户生成的内容。 联系人关系更新活动描述的关系更新存在于 `from` 字段中的用户（通常但并非始终是发起更新的用户）与 `recipient` 字段中的用户或机器人之间。

联系人关系更新活动由 `type` 值 `contactRelationUpdate` 予以标识。

### <a name="action"></a>操作

`action` 字段描述了联系人关系更新活动的含义。 `action` 字段的值是字符串。 仅定义了 `add` 和 `remove` 值，用以指示 `from` 与 `recipient` 字段中的用户/机器人之间的关系。

## <a name="conversation-update-activity"></a>聊天更新活动

聊天更新活动描述在聊天的成员、说明、存在或其他方面发生的更改。 聊天更新活动通常不包含用户生成的内容。 `conversation` 字段描述了要更新的聊天。

聊天更新活动由 `type` 值 `conversationUpdate` 予以标识。

`R4100`：发送方“可以”在聊天更新活动中包括 `membersAdded`、`membersRemoved`、`topicName` 和 `historyDisclosed` 字段中的零个或多个。

`R4101`：每个 `channelAccount`（由 `id` 字段予以标识）“应当”最多在 `membersAdded` 和 `membersRemoved` 字段中出现一次。 ID“不应当”同时出现在这两个字段中。 ID 在任一字段中都“不应当”重复。

`R4102`：如果没有在聊天中添加或删除某个通道帐户，则通道“不应当”使用聊天更新活动来指示对该通道帐户的字段（例如 `name`）的更改。

`R4103`：如果活动未表明 `topicName` 或 `historyDisclosed` 字段的值发生更改，则通道“不应当”发送这些字段。

### <a name="members-added"></a>添加的成员

`membersAdded` 字段包含添加到聊天的通道参与者（机器人或用户）的列表。 `membersAdded` 字段的值是 [`channelAccount`](#channel-account) 类型的数组。

### <a name="members-removed"></a>删除的成员

`membersRemoved` 字段包含从聊天中删除的通道参与者（机器人或用户）的列表。 `membersRemoved` 字段的值是 [`channelAccount`](#channel-account) 类型的数组。

### <a name="topic-name"></a>主题名称

`topicName` 字段包含聊天的文本主题或说明。 `topicName` 字段的值是字符串类型的。

### <a name="history-disclosed"></a>公开的历史记录

`historyDisclosed` 字段已弃用。

`R4110`：发送方“不应当”包括 `historyDisclosed` 字段。

## <a name="end-of-conversation-activity"></a>聊天结束活动

聊天结束活动从接收者的角度表明聊天结束。 这可能是因为聊天已完全结束，也可能是因为接收者已被采用无法与聊天结束区分的方式从聊天中删除。 `conversation` 字段描述了要结束的聊天。

聊天结束活动由 `type` 值 `endOfConversation` 予以标识。

`code` 和 `text` 字段都是可选的。

### <a name="code"></a>代码

`code` 字段包含描述聊天为何或如何结束的编程值。 `code` 字段的值是字符串类型的，其含义是由发送活动的通道定义的。

### <a name="text"></a>文本

`text` 字段包含要传达给用户的可选文本内容。 `text` 字段的值是字符串类型的，其格式为纯文本。

## <a name="event-activity"></a>事件活动

事件活动将编程信息从客户端或通道传达给机器人。 事件活动的含义是由 `name` 字段定义的，该字段在通道作用域内有意义。 事件活动设计用来输送交互信息（例如，按钮单击）和非交互信息（例如，关于客户端自动更新嵌入的语音模型的通知）。

事件活动是[调用活动](#invoke-activity)的异步对等活动。 不同于调用，事件设计为由客户端应用程序扩展进行扩展。

事件活动由 `type` 值 `event` 和 `name` 字段的具体值予以标识。

`R5000`：如果客户端允许应用程序自定义，则通道“可以”允许在客户端与机器人之间使用应用程序定义的事件消息。

### <a name="name"></a>名称

`name` 字段控制事件的含义和 `value` 字段的架构。 `name` 字段的值是字符串类型的。

`R5001`：事件活动“必须”包含 `name` 字段。

`R5002`：接收方“必须”忽略包含它们不理解的 `name` 字段的活动。

### <a name="value"></a>值

`value` 字段包含特定于此事件的参数，如事件名称所定义。 `value` 字段的值是复杂类型。

`R5100`：如果是由事件名称定义的，则 `value` 字段“可以”缺少或为空。

`R5101`：事件活动的扩展“不应当”要求接收方使用除了活动 `type` 和 `name` 字段之外的任何信息来理解 `value` 字段的架构。

### <a name="relates-to"></a>关联到

`relatesTo` 字段在该活动内引用另一个聊天，以及特定的活动（可选）。 `relatesTo` 字段的值是聊天引用类型的复杂对象。

`R5200`：`relatesTo` 不应当引用由 `conversation` 字段标识的聊天内的活动。

## <a name="invoke-activity"></a>调用活动

调用活动将编程信息从客户端或通道传达给机器人，并且具有要在通道内使用的对应返回有效负载。 调用活动的含义是由 `name` 字段定义的，该字段在通道作用域内有意义。 

调用活动是[事件活动](#event-activity)的同步对等活动。 事件活动设计为可扩展。 调用活动的不同仅体现在将响应有效负载返回到通道的能力；因为通道必须决定在何处以及如何处理这些响应有效负载，所以，只有向通道添加了对每个调用名称的显式支持时，调用才有用。 因此，调用不是作为通用应用程序扩展机制设计的。

调用活动由 `type` 值 `invoke` 和 `name` 字段的具体值予以标识。

[附录 III](#appendix-iii---protocols-using-the-invoke-activity) 中包括了已定义的调用活动的列表。

`R5301`：通道“不应当”允许在客户端与机器人之间使用应用程序定义的调用消息。

### <a name="name"></a>名称

`name` 字段控制调用的含义和 `value` 字段的架构。 `name` 字段的值是字符串类型的。

`R5401`：调用活动“必须”包含 `name` 字段。

`R5402`：接收方“必须”忽略包含它们不理解的 `name` 字段的活动。

### <a name="value"></a>值

`value` 字段包含特定于此事件的参数，如事件名称所定义。 `value` 字段的值是复杂类型。

`R5500`：如果是由事件名称定义的，则 `value` 字段“可以”缺少或为空。

`R5501`：事件活动的扩展“不应当”要求接收方使用除了活动 `type` 和 `name` 字段之外的任何信息来理解 `value` 字段的架构。

### <a name="relates-to"></a>关联到

`relatesTo` 字段在该活动内引用另一个聊天，以及特定的活动（可选）。 `relatesTo` 字段的值是[聊天引用](#conversation-reference)类型的复杂对象。

`R5600`：`relatesTo` 不应当引用由 `conversation` 字段标识的聊天内的活动。

## <a name="installation-update-activity"></a>安装更新活动

安装更新活动表示在通道的组织单位（例如客户租户或“团队”）内安装或卸载机器人。 安装更新活动通常不表示添加或删除通道。

安装更新活动由 `type` 值 `installationUpdate` 予以标识。

`R5700`：当在通道内的租户、团队或其他组织单位中添加或删除机器人时，通道“可以”发送安装活动。

`R5701`：当在通道内安装或删除机器人时，通道“不应当”发送安装活动。

### <a name="action"></a>操作

`action` 字段描述了安装更新活动的含义。 `action` 字段的值是字符串。 仅定义了 `add` 和 `remove` 值。

## <a name="message-delete-activity"></a>消息删除活动

消息删除活动表示删除聊天内的现有消息活动。 删除的活动是通过活动内的 `id` 和 `conversation` 字段引用的。

消息删除活动由 `type` 值 `messageDelete` 予以标识。

`R5800`：通道“可以”选择针对聊天内的所有删除发送消息删除活动，针对聊天内的部分删除（例如仅限某些用户执行的删除）发送消息删除活动，或者不针对聊天内的任何删除发送消息删除活动。

`R5801`：对于机器人未观察到的聊天或活动，通道“不应当”发送消息删除活动。

`R5802`：如果机器人触发了删除，则通道“不应当”将消息删除活动发送回该机器人。

`R5803`：通道“不应当”发送与其类型不是 `message` 的活动对应的消息删除活动。

## <a name="message-update-activity"></a>消息更新活动

消息更新活动表示更新聊天内的现有消息活动。 更新的活动是通过活动内的 `id` 和 `conversation` 字段引用的，并且消息更新活动包含修订的消息活动中的所有字段。

消息更新活动由 `type` 值 `messageUpdate` 予以标识。

`R5900`：通道“可以”选择针对聊天内的所有更新发送消息更新活动，针对聊天内的部分更新（例如仅限某些用户执行的更新）发送消息更新活动，或者不针对聊天内的任何更新发送消息更新活动。

`R5901`：如果机器人触发了更新，则通道“不应当”将消息更新活动发送回该机器人。

`R5902`：通道“不应当”发送与其类型不是 `message` 的活动对应的消息更新活动。

## <a name="message-reaction-activity"></a>消息回应活动

消息回应活动表示在聊天内的现有消息活动上进行社交互动。 原始活动是通过活动内的 `id` 和 `conversation` 字段引用的。 `from` 字段表示回应的来源（即对消息进行了回应的用户）。

消息回应活动由 `type` 值 `messageReaction` 予以标识。

### <a name="reactions-added"></a>添加的回应

`reactionsAdded` 字段包含添加到此活动的回应的列表。 `reactionsAdded` 字段的值是 [`messageReaction`](#message-reaction) 类型的数组。

### <a name="reactions-removed"></a>删除的回应

`reactionsRemoved` 字段包含从此活动中删除的回应的列表。 `reactionsRemoved` 字段的值是 [`messageReaction`](#message-reaction) 类型的数组。

## <a name="typing-activity"></a>键入活动

键入活动表示用户或机器人正在进行的输入。 此活动通常在用户输入了按键时发送，此活动还可以由机器人用来指示它们“正在思考”，并且还可以用来指示正在收集来自用户的音频等等。

键入活动设计为在 UI 中保留三秒。

键入活动由 `type` 值 `typing` 予以标识。

`R6000`：如果可以，客户端在收到键入活动时“应当”将键入指示器显示三秒。

`R6001`：除非通道已经知道，否则发送方“不应当”以高于每三秒一次的频率发送键入操作。 （发送方“可以”每两秒发送一次键入活动以防止出现间隙。）

`R6002`：如果通道为键入活动分配了 [`id`](#id)，则它“可以”允许机器人和客户端在键入活动过期之前将其删除。

`R6003`：如果可以，通道“应当”将键入活动发送到机器人。

## <a name="complex-types"></a>复杂类型

本部分定义在上述的活动架构内使用的复杂类型。

### <a name="attachment"></a>附件

附件是[消息活动](#message-activity)内包括的内容：卡、二进制文档和其他交互内容。 它们将与文本内容一起显示。 可以在 `contentUrl` 字段内通过 URL 数据 URI 发送内容，也可以在 `content` 字段中以嵌入方式发送。

`R7100`：发送方“不应当”在单个附件中同时包括 `content` 和 `contentUrl` 字段。

#### <a name="content-type"></a>内容类型

`contentType` 字段描述附件内容的 [MIME 媒体类型](https://www.iana.org/assignments/media-types/media-types.xhtml) [[8](#references)]。 `contentType` 字段的值是字符串类型的。

#### <a name="content"></a>内容

当存在时，`content` 字段包含结构化的 JSON 对象附件。 `content` 字段的值是由 `contentType` 字段定义的复杂类型。

`R7110`：发送方“不应当”在附件的 `content` 字段中包括 JSON 基元。

#### <a name="content-url"></a>内容 URL

如果存在，`contentUrl` 字段包含附件内容的 URL。 通道通常情况下支持数据 URI（如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义）。 `contentUrl` 字段的值是字符串类型的。

`R7120`：接收方“应当”接受 HTTPS URL。

`R7121`：接收方“可以”接受 HTTP URL。

`R7122`：通道“应当”接受数据 URI。

`R7123`：通道“不应当”将数据 URI 发送到客户端或机器人。

#### <a name="name"></a>名称

`name` 字段包含附件的可选名称或文件名。 `name` 字段的值是字符串类型的。

#### <a name="thumbnail-url"></a>缩略图 URL

某些客户端能够将自定义缩略图显示为非交互附件，或者显示为交互附件的占位符。 `thumbnailUrl` 字段标识此缩略图的来源。 通常情况下还允许使用数据 URI（如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义）。

`R7140`：接收方“应当”接受 HTTPS URL。

`R7141`：接收方“可以”接受 HTTP URL。

`R7142`：通道“应当”接受数据 URI。

`R7143`：通道“不应当”向机器人发送 `thumbnailUrl` 字段。

### <a name="card-action"></a>卡操作

卡操作表示在卡中使用的或作为[建议的操作](#suggested-actions)提供的可单击或交互式按钮。 它们用于要求用户提供输入。 尽管其名称如此，但是卡操作不仅限于在卡上使用。

卡操作只有当发送到通道时才有意义。

通道决定每个操作在其用户体验中如何显示。 大多数情况下，卡是可单击的。 其他情况下，可以通过语音输入选择它们。 当通道未提供交互式激活体验（例如，通过短信进行交互）时，通道可能根本不支持激活。 有关如何呈现操作的决策是由本文档中其他位置的规范要求控制的（例如，在卡格式中，或者在[建议的操作](#suggested-actions)定义中）。

#### <a name="type"></a>类型

`type` 字段描述按钮的含义以及激活了按钮时的行为。 根据约定，`type` 字段的值是短字符串（例如 "`openUrl`"）。 有关特定于每个操作类型的要求，请参阅后续部分。

* 类型为 `messageBack` 的操作表示将通过聊天系统发送的文本响应。
* 类型为 `imBack` 的操作表示添加到聊天馈送的文本响应。
* 类型为 `postBack` 的操作表示不添加到聊天馈送的文本响应。
* 类型为 `openUrl` 的操作表示要由客户端处理的超链接。
* 类型为 `downloadFile` 的操作表示要下载的超链接。
* 类型为 `showImage` 的操作表示可以显示的图像。
* 类型为 `signin` 的操作表示要由客户端的登录系统处理的超链接。
* 类型为 `playAudio` 的操作表示可以播放的音频媒体。
* 类型为 `playVideo` 的操作表示可以播放的视频媒体。
* 类型为 `call` 的操作表示可以呼叫的电话号码。
* 类型为 `payment` 的操作表示请求提供付款。

#### <a name="title"></a>标题

`title` 字段包括要在按钮的表面上显示的文本。 `title` 字段的值是字符串类型的，并且不包含标记。

此字段适用于所有类型的操作。

`R7210`：通道“不应当”处理 `title` 字段内的标记（例如 Markdown）。

#### <a name="image"></a>图像

`image` 字段包含引用了要在按钮的表面上显示的图像的 URL。 通道通常情况下支持数据 URI（如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义）。 `image` 字段的值是字符串类型的。

此字段适用于所有类型的操作。

`R7220`：通道“应当”接受 HTTPS URL。

`R7221`：通道“可以”接受 HTTP URL。

`R7222`：通道“应当”接受数据 URI。

#### <a name="text"></a>文本

`text` 字段包含当单击按钮时要发送到机器人的以及要在聊天馈送中包括的文本内容。 `text` 字段的内容可能会显示，也可能不会显示，具体取决于按钮类型。 `text` 字段可以包含由活动根中的 [`textFormat`](#text-format) 字段控制的标记。 `text` 字段的值是字符串类型的。

此字段仅在选择类型的操作上使用。 本文档下文中包括了有关每种类型的操作的详细信息。

`R7230`：`text` 字段“可以”包含空字符串来指示发送不含内容的文本。

`R7231`：通道“应当”根据活动根中的 [`textFormat`](#text-format) 字段处理 `text` 字段的内容。

#### <a name="display-text"></a>显示文本

`displayText` 字段包含当单击按钮时要在聊天馈送中包括的文本内容。 当在通道内技术上可行时，`displayText` 字段“应当”始终显示。 `displayText` 字段可以包含由活动根中的 [`textFormat`](#text-format) 字段控制的标记。 `displayText` 字段的值是字符串类型的。

此字段仅在选择类型的操作上使用。 本文档下文中包括了有关每种类型的操作的详细信息。

`R7240`：`displayText` 字段“可以”包含空字符串来指示发送不含内容的文本。

`R7241`：通道“应当”根据活动根中的 [`textFormat`](#text-format) 字段处理 `displayText` 字段的内容。

#### <a name="value"></a>值

`value` 字段包含当单击按钮时要发送到机器人的编程内容。 `value` 字段的内容是任意基元或复杂类型，但是某些活动类型会对此字段进行约束。

此字段仅在选择类型的操作上使用。 本文档下文中包括了有关每种类型的操作的详细信息。

#### <a name="message-back"></a>回复消息

`messageBack` 操作表示将通过聊天系统发送的文本响应。 回复消息使用以下字段：
* `type` ("`messageBack`")
* `title`
* `image`
* `text`
* `displayText`
* `value`（任意类型）

`R7350`：发送方“不应当”包括是基元类型（例如字符串、整数）的 `value` 字段。 `value` 字段“应当”是复杂类型或省略。

`R7351`：通道“可以”拒绝或丢弃非复杂类型的 `value` 字段。

`R7352`：当激活后，通道“必须”向所有相关接收者发送 `message` 类型的活动。

`R7353`：如果通道支持存储和传输文本，则必须在生成的消息活动的 `text` 字段中保留和传输操作的 `text` 字段的内容。

`R7352`：如果通道支持存储和传输附加编程值，则必须在生成的消息活动的 `value` 字段中保留和传输 `value` 字段的内容。

`R7353`：如果通道支持在聊天馈送中保留与发送到机器人的值不同的值，则它“必须”在聊天历史记录中包括 `displayText` 字段。

`R7354`：如果通道不支持 `R7353`，但支持在聊天馈送中记录文本，则它“必须”在聊天历史记录中包括 `text` 字段。

#### <a name="im-back"></a>回复 IM

`imBack` 操作表示添加到聊天馈送的文本响应。 回复消息使用以下字段：
* `type` ("`imBack`")
* `title`
* `image`
* `value`（字符串类型）

`R7360`：当激活后，通道“必须”向所有相关接收者发送 `message` 类型的活动。

`R7361`：如果通道支持存储和传输文本，则必须在生成的消息活动的 `text` 字段中保留和传输 `title` 字段的内容。

`R7362`：如果活动上缺少 `title` 字段并且 `value` 字段是字符串类型的，则通道“可以”在生成的消息活动的 `text` 字段中传输 `value` 字段的内容。

`R7363`：如果通道支持在聊天馈送中记录文本，则它“必须”在聊天历史记录中包括 `title` 字段的内容。

#### <a name="post-back"></a>回复帖子

`postBack` 操作表示不添加到聊天馈送的文本响应。 回复帖子使用以下字段：
* `type` ("`postBack`")
* `title`
* `image`
* `value`（字符串类型）

`R7370`：当激活后，通道“必须”向所有相关接收者发送 `message` 类型的活动。

`R7371`：当激活“回复帖子”操作时，通道“不应当”在聊天历史记录中包括文本。

`R7372`：通道“必须”拒绝或丢弃非字符串类型的 `value` 字段。

`R7373`：如果通道支持存储和传输文本，则必须在生成的消息活动的 `text` 字段中保留和传输 `value` 字段的内容。

`R7374`：如果通道在聊天馈送中不包括历史记录的情况下无法支持传输到机器人，则它“应当”使用 `title` 字段作为显示文本。

#### <a name="open-url-actions"></a>打开 URL 操作

`openUrl` 操作表示要由客户端处理的超链接。 打开 URL 使用以下字段：
* `type` ("`openUrl`")
* `title`
* `image`
* `value`（字符串类型）

`R7380`：发送方“必须”在 `openUrl` 操作的 `value` 字段中包括一个 URL。

`R7381`：接收方“可以”拒绝其 `value` 字段缺少或不是字符串的 `openUrl` 操作。

`R7382`：接收方“应当”拒绝或丢弃其 `value` 字段包含数据 URI 的 `openUrl` 操作，如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义。

`R7383`：接收方“不应当”拒绝其 `value` URI 是其他意外 URI 方案或值的 `openUrl` 操作。

`R7384`：了解特定 URI 方案（例如 HTTP）的客户端“可以”在嵌入的呈现器（例如浏览器控件）中处理 `openUrl` 操作。

`R7385`：当可用时，客户端“应当”将 `R7354` 未处理的 `openUrl` 操作的处理委托给操作系统级或 shell 级 URI 处理程序。

#### <a name="download-file-actions"></a>下载文件操作

`downloadFile` 操作表示要下载的超链接。 下载文件使用以下字段：
* `type` ("`downloadFile`")
* `title`
* `image`
* `value`（字符串类型）

`R7390`：发送方“必须”在 `downloadFile` 操作的 `value` 字段中包括一个 URL。

`R7391`：接收方“可以”拒绝其 `value` 字段缺少或不是字符串的 `downloadFile` 操作。

`R7392`：接收方“应当”拒绝或丢弃其 `value` 字段包含数据 URI 的 `downloadFile` 操作，如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义。

#### <a name="show-image-file-actions"></a>显示图像文件操作

`showImage` 操作表示可以显示的图像。 显示图像使用以下字段：
* `type` ("`showImage`")
* `title`
* `image`
* `value`（字符串类型）

`R7400`：发送方“必须”在 `showImage` 操作的 `value` 字段中包括一个 URL。

`R7401`：接收方“可以”拒绝其 `value` 字段缺少或不是字符串的 `showImage` 操作。

`R7402`：接收方“可以”拒绝其 `value` 字段是数据 URI 的 `showImage` 操作，如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义。

#### <a name="signin"></a>登录

`signin` 操作表示要由客户端的登录系统处理的超链接。 登录使用以下字段：
* `type` ("`signin`")
* `title`
* `image`
* `value`（字符串类型）

`R7410`：发送方“必须”在 `signin` 操作的 `value` 字段中包括一个 URL。

`R7411`：接收方“可以”拒绝其 `value` 字段缺少或不是字符串的 `signin` 操作。

`R7412`：接收方“必须”拒绝或丢弃其 `value` 字段包含数据 URI 的 `signin` 操作，如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义。

#### <a name="play-audio"></a>播放音频

`playAudio` 操作表示可以播放的音频媒体。 播放音频使用以下字段：
* `type` ("`playAudio`")
* `title`
* `image`
* `value`（字符串类型）

`R7420`：当激活后，通道“可以”发送媒体事件。

`R7421`：通道“必须”拒绝或丢弃非字符串类型的 `value` 字段。

`R7422`：在事先不知道通道是否支持数据 URI 的情况下，发送方“不应当”发送数据 URI，如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义。

#### <a name="play-video"></a>播放视频

`playAudio` 操作表示可以播放的视频媒体。 播放视频使用以下字段：
* `type` ("`playVideo`")
* `title`
* `image`
* `value`（字符串类型）

`R7430`：当激活后，通道“可以”发送媒体事件。

`R7431`：通道“必须”拒绝或丢弃非字符串类型的 `value` 字段。

`R7432`：在事先不知道通道是否支持数据 URI 的情况下，发送方“不应当”发送数据 URI，如 [RFC 2397](https://tools.ietf.org/html/rfc2397) [[9](#references)] 中所定义。

#### <a name="call"></a>呼叫

`call` 操作表示可以呼叫的电话号码。 呼叫使用以下字段：
* `type` ("`call`")
* `title`
* `image`
* `value`（字符串类型）

`R7440`：发送方“必须”在 `signin` 操作的 `value` 字段中包括方案 `tel` 的 URL。

`R7441`：接收方“必须”拒绝其 `value` 字段缺少或不是 `tel` 方案的字符串 URI 的 `signin` 操作。

#### <a name="payment"></a>付款

`payment` 操作表示请求提供付款。 付款使用以下字段：
* `type` ("`payment`")
* `title`
* `image`
* `value`（复杂类型 [Payment Request](#payment-request)）

`R7450`：发送方“必须”在 `payment` 操作的 `value` 字段中包括[付款请求](#payment-request)类型的复杂对象。

`R7451`：通道“必须”拒绝其 `value` 字段缺少或不是[付款请求](#payment-request)类型的复杂对象的 `payment` 操作。

### <a name="channel-account"></a>通道帐户

通道帐户表示通道内的标识。 通道帐户包括一个 ID，可以使用它来标识和联系该通道内的帐户。 有时候，这些 ID 存在于单个命名空间内（例如 Skype ID）；有时候，它们以联盟形式分布在许多服务器中（例如电子邮件地址）。 除了 ID 之外，通道帐户还包括显示名称和 Azure Active Directory (AAD) 对象 ID。

#### <a name="channel-account-id"></a>通道帐户 ID

`id` 字段是在通道内的标识符和地址。 `id` 字段的值是字符串。 通道内使用电子邮件地址的一个示例 `id` 为“name@example.com”。

`R7510`：通道“应当”为帐户 ID 使用相同的值和约定，无论它们在架构内处于什么位置（`from.id`、`recipient.id`、`membersAdded`，等等）。 这允许机器人和客户端使用序号字符串比较来了解诸如何时在 `conversationUpdate` 活动的 `membersAdded` 字段中描述了它们。

#### <a name="channel-account-name"></a>通道帐户名称

`name` 字段是在通道内的可选友好名称。 `name` 字段的值是字符串。 通道内的一个示例 `name` 为“John Doe”

#### <a name="channel-account-aad-object-id"></a>通道帐户 AAD 对象 ID

`aadObjectId` 字段是与帐户在 Azure Active Directory (AAD) 内的对象 ID 对应的一个可选 ID。 `aadObjectId` 字段的值是字符串。

### <a name="conversation-account"></a>聊天帐户

聊天帐户表示聊天在通道内的标识。 在仅支持两个帐户之间的单一聊天（例如短信）的通道中，聊天帐户一直存在，没有预先确定的开始或结束。 在支持多个并行聊天（例如电子邮件）的通道中，每个聊天都可能有唯一的 ID。

#### <a name="conversation-account-id"></a>聊天帐户 ID

`id` 字段是在通道内的标识符。 此 ID 的格式是由通道定义的，并且在整个协议中作为不透明字符串使用。

通道“应当”选择对于聊天内的所有参与者都很稳定的 `id` 值。 （例如，用于 1:1 聊天的 `id` 字段的一个不佳示例是使用另一参与者的 ID 作为 `id` 值。 这将得到从每个参与者的角度来看各不相同的 `id`。 更好的选择是对两个参与者的 ID 进行排序并将它们连接在一起，这样得到的结果对双方都是相同的。）

#### <a name="conversation-account-name"></a>聊天帐户名称

`name` 字段是聊天在通道内的可选友好名称。 `name` 字段的值是字符串。

#### <a name="conversation-account-aad-object-id"></a>聊天帐户 AAD 对象 ID

`aadObjectId` 字段是与聊天在 Azure Active Directory (AAD) 内的对象 ID 对应的一个可选 ID。 `aadObjectId` 字段的值是字符串。

#### <a name="conversation-account-is-group"></a>聊天帐户是组

`isGroup` 字段指示聊天在活动生成时是否包含两个以上参与者。 `isGroup` 字段的值是布尔值，如果省略，则默认值为 `false`。 此字段通常控制通道中的参与者的即刻行为，并且当且仅当两个以上参与者能够在聊天内既发送又接收活动时，它才“应当”设置为 `true`。

### <a name="entity"></a>实体

实体输送关于活动或聊天的元数据。 每个实体的含义和形态是由 `type` 字段定义的，并且有其他特定于类型的字段作为 `type` 字段的对等方存在。

`R7600`：接收方“必须”忽略它们不理解其类型的实体。

`R7601`：接收方“应当”忽略它们理解其类型但由于语法错误等问题而无法处理的实体。

对于将现有实体类型映射为活动实体格式的各方，建议在 IRI 与实体架构之间的绑定中解决与 `type` 字段名称的冲突以及与序列化要求 `R2001` 的不兼容。

#### <a name="type"></a>类型

`type` 字段是必需的，它定义实体的含义和形态。 虽然[附录 II](#appendix-ii---non-iri-entity-types) 中定义了少量非 IRI 实体类型，但 `type` 用来包含 [IRI](https://tools.ietf.org/html/rfc3987) [[3](#references)]。

`R7610`：发送方仅“应当”为[附录 II](#appendix-ii---non-iri-entity-types) 中描述的类型使用非 IRI 类型名称。

`R7611`：发送方“可以”为[附录 II](#appendix-ii---non-iri-entity-types) 中描述的类型发送 IRI 类型，前提是它们已知道接收方理解这些类型。

`R7612`：发送方“应当”为[附录 II](#appendix-ii---non-iri-entity-types) 中未定义的实体类型使用或建立 IRI。

### <a name="suggested-actions"></a>建议的操作

可以在消息内容中发送建议的操作以在客户端 UI 中创建交互式操作元素。

`R7700`：不支持能够呈现建议操作的 UI 的客户端“应当”忽略 `suggestedActions` 字段。

`R7701`：如果 `actions` 字段为空，则发送方“应当”省略 `suggestedActions` 字段。

#### <a name="to"></a>显示给

`to` 字段包含应当向其显示建议的操作的通道帐户 ID。 可以使用此字段对操作进行筛选，使其仅应用于聊天中的一部分参与者。

`R7710`：如果 `to` 字段缺少或为空，则客户端“应当”向所有聊天参与者显示建议的操作。

`R7711`：如果 `to` 字段包含无效的 ID，则“应当”忽略那些值。

#### <a name="actions"></a>操作

`actions` 字段包含要显示的操作的简单列表。 每个 `actions` 列表元素的值都是 `cardAction` 类型的一个复杂对象。

### <a name="message-reaction"></a>消息回应

消息回应表示社交互动（“赞”、“+1”，等等）。 消息回应当前仅输送单个字段：`type` 字段。

#### <a name="type"></a>类型

`type` 字段描述社交互动的类型。 `type` 字段的值是字符串，其含义是由在其中进行互动的通道定义的。 有一些常用值，例如 `like` 和 `+1`，但这些是按约定而不是按规则统一的。

## <a name="references"></a>参考

1. [RFC 2119](https://tools.ietf.org/html/rfc2119)
2. [ISO 8601](https://www.iso.org/iso-8601-date-and-time-format.html)
3. [RFC 3987](https://tools.ietf.org/html/rfc3987)
4. [Markdown](https://daringfireball.net/projects/markdown/)
5. [ISO 639](https://www.iso.org/iso-639-language-codes.html)
6. [SSML](https://www.w3.org/TR/speech-synthesis/)
7. [XML](https://www.w3.org/TR/xml/)
8. [MIME 媒体类型](https://www.iana.org/assignments/media-types/media-types.xhtml)
9. [RFC 2397](https://tools.ietf.org/html/rfc2397)
10. [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html)
11. [自适应卡](https://adaptivecards.io)

# <a name="appendix-i---changes"></a>附录 I - 更改

## <a name="2018-02-07"></a>2018-02-07

* 初始草稿

# <a name="appendix-ii---non-iri-entity-types"></a>附录 II - 非 IRI 实体类型

活动[实体](#entity)传达关于活动的额外元数据，例如用户的位置或他们使用的消息传送应用的版本。 活动类型应当是 IRI，但有少量非 IRI 名称也很常用。 本附录是支持的非 IRI 实体类型的详尽列表。

| 类型           | 等效 URI                          | 说明               |
| -------------- | --------------------------------------- | ------------------------- |
| GeoCoordinates | https://schema.org/GeoCoordinates       | Schema.org 地理坐标 |
| Mention        | https://botframework.com/schema/mention | @-mention                 |
| Place          | https://schema.org/Place                | Schema.org 位置          |
| Thing          | https://schema.org/Thing                | Schema.org 事物          |
| clientInfo     | 不适用                                     | Skype 客户端信息         |

### <a name="clientinfo"></a>clientInfo

clientInfo 实体包含有关用来发送用户的消息的客户端软件的扩展信息。 它包含三个属性，这些属性都是可选的。

`R9201`：机器人“不应当”发送 `clientInfo` 实体。

`R9202`：仅当填充了一个或多个字段时，发送方才“应当”包括 `clientInfo` 实体。

#### <a name="locale-deprecated"></a>区域设置（已弃用）

`locale` 字段包含用户的区域设置。 此字段与活动根中的 [`locale`](#locale) 字段完全相同。 `locale` 字段的值是字符串内的一个 [ISO 639](https://www.iso.org/iso-639-language-codes.html) [[5](#references)] 代码。

`clientInfo` 中的 `locale` 字段已弃用。

`R9211`：接收方“不应当”使用 `clientInfo` 对象中的 `locale` 字段。

`R9212`：出于兼容性原因，发送方“可以”填充 `clientInfo` 中的 `locale` 字段。 如果不需要兼容较旧的接收方，则发送方“不应当”发送 `locale` 属性。

#### <a name="country"></a>国家/地区

`country` 字段包含检测到的用户位置。 此值可能不同于任何 [`locale`](#locale) 数据，因为 `country` 是检测到的，而 `locale` 通常是用户或应用程序设置。 `country` 字段的值是 [ISO 3166-1](https://www.iso.org/iso-3166-country-codes.html) [[10](#references)] 2 字母或 3 字母国家/地区代码。

`R9220`：通道“不应当”允许客户端为 `country` 字段指定任意值。 通道“应当”使用诸如 GPS、位置 API 或 IP 地址检测之类的机制来确定生成请求的国家/地区。

#### <a name="platform"></a>平台

`platform` 字段描述用来生成活动的消息传送客户端平台。 `platform` 字段的值是字符串，可能值及其含义的列表是由发送它们的通道定义的。

注意，在具有持久性聊天馈送的通道上，`platform` 通常仅在决定要包括什么内容时有用，在决定该内容的格式时无用。 例如，如果移动设备上的用户请求产品支持帮助，则机器人可以生成特定于其移动设备的帮助。 但是，用户然后可以在其电脑上重新打开聊天馈送以便在该屏幕上阅读帮助，同时对其移动设备进行更改。 在这种情况下，`platform` 字段用来指示内容，但是该内容应当可以在其他设备上查看。

`R9230`：机器人“不应当”使用 `platform` 字段来控制如何对响应数据进行格式化，除非它们明确知道它们发送的内容只能在所考虑的设备上查看。

# <a name="appendix-iii---protocols-using-the-invoke-activity"></a>附录 III - 使用调用活动的协议

[调用活动](#invoke-activity)设计为仅在 Bot Framework 通道支持的协议内使用（也就是说，它不是一种通用扩展机制）。 本附录包含使用此活动的所有 Bot Framework 协议的列表。

## <a name="payments"></a>付款

Bot Framework 付款协议使用“调用”来计算运费和税率，并严格确认已完成的付款。

付款协议定义了三个操作（在调用活动的 `name` 字段中定义的）：
* `payment/shippingAddressChange`
* `payment/shppingOptionsChange`
* `payment/paymentResponse`

可以在[请求付款](https://docs.microsoft.com/en-us/bot-framework/dotnet/bot-builder-dotnet-request-payment)页面上找到更多详细信息。

## <a name="teams-compose-extension"></a>团队撰写扩展

Microsoft Teams 通道为[撰写扩展](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/messaging-extensions)使用调用。 调用的此用法特定于 Microsoft Teams。



