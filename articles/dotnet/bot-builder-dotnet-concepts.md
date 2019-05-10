---
title: Bot Framework SDK for .NET 中的关键概念 | Microsoft Docs
description: 了解用于构建和部署 Bot Framework SDK for .NET 中可用的聊天机器人的关键概念和工具。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e01473a06a0cdbef635de33e5734b02351e36cea
ms.sourcegitcommit: 980612a922b8290b2faadaca193496c4117e415a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/26/2019
ms.locfileid: "64563431"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-net"></a>Bot Framework SDK for .NET 中的关键概念

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

本文介绍 Bot Framework SDK for .NET 中的关键概念。

## <a name="connector"></a>连接器

[Bot Framework Connector](bot-builder-dotnet-connector.md) 提供单个 REST API，使机器人能够跨多个通道（如 Skype、电子邮件、Slack 等）进行通信。 它通过从机器人到通道以及从通道到机器人的消息中继来促进机器人与用户之间的通信。 

在 Bot Framework SDK for .NET 中，可以使用[连接器][connectorLibrary]库访问连接器。 

## <a name="activity"></a>活动

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

有关 Bot Framework SDK for .NET 中的活动的详细信息，请参阅[活动概述](bot-builder-dotnet-activities.md)。

## <a name="dialog"></a>对话框

使用 Bot Framework SDK for .NET 创建机器人时，可以使用[对话框](bot-builder-dotnet-dialogs.md)为聊天建模并管理[聊天流](../bot-service-design-conversation-flow.md#dialog-stack)。 对话可以由其他对话组成以最大限度地重用，并且对话上下文可以维护在任何时间点都在聊天中处于活动状态的[对话堆栈](../bot-service-design-conversation-flow.md)。 包含对话框的会话可以跨计算机移植，这使得机器人实现可以扩展。 

在 Bot Framework SDK for .NET 中，可以使用[生成器][builderLibrary]库管理对话框。

## <a name="formflow"></a>FormFlow

可以在 Bot Framework SDK for .NET 中使用 [FormFlow](bot-builder-dotnet-formflow.md) 来简化从用户收集信息的机器人的构建操作。 例如，接受三明治订单的机器人必须从用户收集几条信息，例如面包类型、配料选择、尺寸等。 根据基本准则，FormFlow 可以自动生成管理这样的引导式聊天所必需的对话。

## <a name="state"></a>状态

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

若要详细了解如何使用 Bot Framework SDK for .NET 来管理状态，请参阅[管理状态数据](bot-builder-dotnet-state.md)。

## <a name="naming-conventions"></a>命名约定

Bot Framework SDK for .NET 库使用强类型化的采用 Pascal 大小写格式的命名约定。 但是，通过线路来回传输的 JSON 消息采用 camel 大小写格式的命名约定。 例如，C# 属性 **ReplyToId** 在通过线络传输的 JSON 消息中被序列化为 **replyToId**。

## <a name="security"></a>安全

你应确保只能通过 Bot Framework Connector 服务调用机器人的终结点。 有关本主题的详细信息，请参阅[保护机器人](bot-builder-dotnet-security.md)。

## <a name="next-steps"></a>后续步骤

现在你已了解每个机器人背后的概念。 可以使用模板快速[通过 Visual Studio 生成机器人](bot-builder-dotnet-quickstart.md)。 接下来，从对话开始，更详细地研究每个重要概念。

> [!div class="nextstepaction"]
> [Bot Framework SDK for .NET 中的对话框](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs
