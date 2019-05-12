---
title: Bot Framework 中的 ID 指南 | Microsoft Docs
description: 本指南介绍 Bot Framework v3 协议中存在的 ID 字段的特征。
keywords: id, 机器人, 协议
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/30/2019
ms.openlocfilehash: 28932ca45c6faaad2f17ecc03f026ba04352a5a1
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033070"
---
# <a name="id-fields-in-the-bot-framework"></a>Bot Framework 中的 ID 字段

本指南介绍 Bot Framework 中 ID 字段的特征。

## <a name="channel-id"></a>通道 ID

每个 Bot Framework 通道都由唯一 ID 标识。

示例： `"channelId": "skype"`

通道 ID 用作其他 ID 的命名空间。 Bot Framework 协议中的运行时调用必须在通道的上下文中进行；通道为通信时使用的会话和帐户 ID 赋予意义。

根据约定，所有通道 ID 都是小写形式。 通道保证它们发出的通道 ID 具有一致的外壳，因此，机器人可以使用序号比较来建立等效性。

### <a name="rules-for-channel-ids"></a>通道 ID 的规则

- 通道 ID 区分大小写。

## <a name="bot-handle"></a>机器人句柄

已注册到 Azure 机器人服务的每个机器人都有一个机器人句柄。

示例： `FooBot`

机器人句柄表示机器人已注册到在线 Azure 机器人服务。 此注册与 HTTP Webhook 终结点和通道注册相关联。

Azure 机器人服务确保机器人句柄的唯一性。 Azure 门户执行不区分大小写的唯一性检查（意味着将机器人句柄的大小写变体视为单个句柄），尽管这是 Azure 门户的特点，但不一定是机器人句柄本身的特点。

### <a name="rules-for-bot-handles"></a>机器人句柄的规则

* 机器人句柄在 Bot Framework 中具有唯一性（不区分大小写）。

## <a name="app-id"></a>应用 ID

已注册到 Azure 机器人服务的每个机器人都有一个应用 ID。

> [!NOTE]
> 以前，应用通常被称为“MSA 应用”或“MSA/AAD 应用”。 现在，应用通常简称为“应用”，但某些协议元素可能会永久性地将应用称为“MSA 应用”。

示例： `"msaAppId": "353826a6-4557-45f8-8d88-6aa0526b8f77"`

应用表示向标识团队的应用门户注册，并充当 Bot Framework 运行时协议中的服务到服务标识机制。 应用可能具有其他非机器人关联，例如网站和移动/桌面应用程序。

每个已注册的机器人都有且只有一个应用。 尽管机器人所有者无法独立更改与其机器人相关联的应用，但 Bot Framework 团队可以在少数特殊情况下执行此操作。

机器人和通道可以使用应用 ID 唯一标识已注册的机器人。

应用 ID 必须是 GUID。 比较应用 ID 时不应考虑大小写。

### <a name="rules-for-app-ids"></a>应用 ID 的规则

* 应用 ID 在 Microsoft 应用平台中具有唯一性（GUID 比较）。
* 每个机器人都有且只有一个相应的应用。
* 需要 Bot Framework 团队的帮助才能更改与机器人相关联的应用。

## <a name="channel-account"></a>通道帐户

每个机器人和用户在每个通道中都有一个帐户。 该帐户包含一个标识符 (`id`) 及其他信息性机器人非结构化数据，如可选名称。

示例： `"from": { "id": "john.doe@contoso.com", "name": "John Doe" }`

此帐户描述了可以发送和接收消息的通道内的地址。 在某些情况下，这些注册存在于单个服务中（例如 Skype、Facebook）。 其他情况下，它们注册到多个系统（电子邮件地址、电话号码）。 在更匿名的通道（例如，网上聊天）中，注册可能是短暂的。

通道帐户嵌套在通道内。 例如，Facebook 帐户只是一个数字。 此数字可能在其他通道中具有不同的含义，并且它在所有通道外部无意义。

通道帐户与用户（实际人员）之间的关系取决于与每个通道关联的约定。 例如，短信号码通常在一段时间内指代一个人，之后该号码可以转移给其他人。 相反，Facebook 帐户通常指代具有永久性的一个人，尽管两个人共享 Facebook 帐户的情况并不少见。

在大多数通道中，可以适当地将通道帐户视为一种可以传递邮件的邮箱。 大多数通道通常允许多个地址映射到单个邮箱；例如，“jdoe@contoso.com”和“john.doe@service.contoso.com”可能会解析为同一个收件箱。 一些通道更进一步，根据正在访问它的机器人来相应更改帐户的地址；例如，Skype 和 Facebook 都会更改用户 ID，使每个机器人都有不同的地址来发送和接收消息。

虽然在某些情况下可以在地址之间建立等效性，但建立邮箱之间的等效性和人与人之间的等效性需要了解通道内的约定，并且在许多情况下是不可能实现的。

通过发送到机器人的活动的 `recipient` 字段向机器人通知其通道帐户地址。

### <a name="rules-for-channel-accounts"></a>通道帐户的规则

* 通道帐户只有在其关联的通道内才有意义。
* 多个 ID 可能会解析为同一帐户。
* 可以使用序号比较来确定两个 ID 是相同的。
* 通常无法使用比较来确认两个不同的 ID 是否会解析为相同的帐户、机器人或人员。
* ID、帐户、邮箱和人员之间的关联的稳定性取决于通道。

## <a name="conversation-id"></a>会话 ID

在会话上下文中发送和接收消息，会话可通过 ID 识别。

示例： `"conversation": { "id": "1234" }`

会话包含消息交换和其他活动。 每个会话拥有零个或多个活动，每个活动都正好出现在一个会话中。 会话可能是永久性的，也可能具有不同的开始和结束内容。 创建、修改或结束会话的过程发生在通道内（即，当通道感知到会话时，即存在会话），并且这些过程的特征由通道确立。

会话内的活动由用户和机器人发送。 用户“参与”会话的定义因通道而异，理论上可以包含当前用户、曾收到消息的用户、发送消息的用户。

一些通道（例如，短信、Skype 和可能存在的其他通道）具有这样一种奇怪的现象：分配给 1:1 会话的会话 ID 是远程通道帐户 ID。 这种现象有两个副作用：
1. 会话 ID 根据会话 ID 的查看者而定。 如果参与者 A 和 B 正在对话，参与者 A 看到的会话 ID 是“B”，参与者 B 看到的会话 ID 是“A”。
2. 如果机器人在此通道中有多个通道帐户（例如，如果机器人有两个短信号码），那么会话 ID 不足以唯一地识别机器人视野内的会话。

因此，会话 ID 不一定唯一识别通道（甚至是单个机器人）中的单个会话。

### <a name="rules-for-conversation-ids"></a>会话 ID 的规则

* 会话只有在其关联的通道内才有意义。
* 多个 ID 可能会解析为同一会话。
* 序号相等不一定能确定两个会话 ID 是相同的会话（尽管在大多数情况下它确实如此）。

## <a name="activity-id"></a>活动 ID

活动在 Bot Framework 协议内进行发送和接收，这些活动有时是可识别的。

示例： `"id": "5678"`

活动 ID 是可选的，供通道用于支持机器人在后续 API 调用中引用 ID（如果可用）：
* 回复特定活动
* 活动级别的参与者列表的查询

由于没有构建其他用例，因此，没有其他规则来处理活动 ID。
