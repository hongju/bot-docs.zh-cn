---
title: 管理状态数据 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 来保存和检索状态数据。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b218ae4ffd2ffbfe9144b4143f2600be15d688dd
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297838"
---
# <a name="manage-state-data"></a>管理状态数据
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>内存中数据存储

内存中数据存储仅用于测试。 此存储是易失性和临时性的。 每次重启机器人都会清除数据。 若要将内存中存储用于测试目的，需要： 

安装以下 NuGet 包： 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

在 **Application_Start** 方法中创建内存中存储的新实例，并注册新的数据存储：

```cs
// Global.asax file

var store = new InMemoryDataStore();

Conversation.UpdateContainer(
           builder =>
           {
               builder.Register(c => store)
                         .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                         .AsSelf()
                         .SingleInstance();

               builder.Register(c => new CachingBotDataStore(store,
                          CachingBotDataStoreConsistencyPolicy
                          .ETagBasedConsistency))
                          .As<IBotDataStore<BotData>>()
                          .AsSelf()
                          .InstancePerLifetimeScope();


           });
GlobalConfiguration.Configure(WebApiConfig.Register);

```

可以使用此方法设置自己的自定义数据存储，或使用任一 Azure 扩展。

## <a name="manage-custom-data-storage"></a>管理自定义数据存储

出于生产环境中的性能和安全考虑，你可以实施自己的数据存储，或考虑实施以下数据存储选项之一：

1. [使用 Cosmos DB 管理状态数据](bot-builder-dotnet-state-azure-cosmosdb.md)

2. [使用表存储管理状态数据](bot-builder-dotnet-state-azure-table-storage.md)

使用上述任何一个 [Azure 扩展](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)选项时，用于通过 Bot Framework SDK for .NET 设置和持久保存数据的机制将与内存中数据存储保持一致。

## <a name="bot-state-methods"></a>机器人状态方法

下表列出了可用于管理状态数据的方法。

| 方法 | 范围 | 目标 |                                                
|----|----|----|
| `GetUserData` | 用户 | 获取以前在指定的通道上为用户保存的状态数据 |
| `GetConversationData` | 对话 | 获取以前在指定的通道上为聊天保存的状态数据 |
| `GetPrivateConversationData` | 用户和聊天 | 获取以前在指定的通道上为聊天中的用户保存的状态数据 |
| `SetUserData` | 用户 | 在指定的通道上保存用户的状态数据 |
| `SetConversationData` | 对话 | 在指定的通道上保存聊天的状态数据。 <br/><br/>**注意**：由于 `DeleteStateForUser` 方法不会删除使用 `SetConversationData` 方法存储的数据，因此，不得使用此方法来存储用户的个人身份信息 (PII)。 |
| `SetPrivateConversationData` | 用户和聊天 | 在指定的通道上保存聊天中用户的状态数据 |
| `DeleteStateForUser` | 用户 | 删除以前使用 `SetUserData` 方法或 `SetPrivateConversationData` 方法存储的用户状态数据。 <br/><br/>**注意**：当机器人收到 [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) 类型的活动或 [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate) 类型的活动（指示已从用户的联系人列表中删除机器人）时，应调用此方法。 |

如果机器人使用某个“**Set...Data**”方法保存状态数据，则机器人将来在同一上下文中收到的消息将包含该数据，机器人可以使用“**Get...Data**”方法访问该数据。

## <a name="useful-properties-for-managing-state-data"></a>可管理状态数据的有用属性

每个 [Activity][Activity] 对象包含用于管理状态数据的属性。

| 属性 | Description | 使用案例 |
|----|----|----|
| `From` | 唯一标识通道上的用户 | 存储和检索与用户关联的状态数据 |
| `Conversation` | 唯一标识聊天 | 存储和检索与聊天关联的状态数据 |
| `From` 和 `Conversation` | 唯一标识用户和聊天 | 存储和检索与特定聊天上下文中特定用户相关联的状态数据 |

> [!NOTE]
> 即使你选择在自己的数据库中存储状态数据而不使用 Bot Framework 状态数据存储，也可以使用这些属性值作为键。

## <a id="state-client"></a> 创建状态客户端

使用 `StateClient` 对象可以通过 Bot Builder SDK for .NET 来管理状态数据。 如果你有权访问属于管理状态数据时所在的同一上下文的消息，则可以通过对 `Activity` 对象调用 `GetStateClient` 方法来创建状态客户端。

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient1)]

如果你无权访问属于管理状态数据时所在的同一上下文的消息，只需创建 `StateClient` 类的新实例即可创建状态客户端。 在此示例中，`microsoftAppId` 和 `microsoftAppPassword` 是在[创建机器人](../bot-service-quickstart.md)期间为机器人获取的 Bot Framework 身份验证凭据。

> [!NOTE]
> 若要查找机器人的 **AppID** 和 **AppPassword**，请参阅 [ MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

[!code-csharp[Get State client](../includes/code/dotnet-state.cs#getStateClient2)]

> [!NOTE]
> 默认的状态客户端存储在中心服务中。 对于某些通道，可能需要使用通道本身托管的状态 API，以便可以在通道提供的合规存储中存储状态数据。

## <a name="get-state-data"></a>获取状态数据

每个“**Get...Data**”方法返回 `BotData` 对象，其中包含指定的用户和/或聊天的状态数据。 若要从 `BotData` 对象获取特定的属性值，请调用 `GetProperty` 方法。 

以下代码示例演示如何从用户数据中获取某个类型化属性。 

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty1)]

以下代码示例演示如何从用户数据中的复杂类型获取某个属性。

[!code-csharp[Get state property](../includes/code/dotnet-state.cs#getProperty2)]

如果为“**Get...Data**”方法调用指定的用户和/或聊天不存在状态数据，返回的 `BotData` 对象将包含以下属性值： 
- `BotData.Data` = null
- `BotData.ETag` = "*"

## <a name="save-state-data"></a>保存状态数据

若要保存状态数据，请先调用相应的“**Get...Data**”方法获取 `BotData` 对象，再调用 `SetProperty` 方法为要更新的每个属性更新该对象，然后调用相应的“**Set...Data**”方法保存该对象。 

> [!NOTE]
> 对于通道上的每个用户、通道上的每个聊天，以及通道上聊天上下文中的每个用户，最多可以存储 32 KB 的数据。 

以下代码示例演示如何保存用户数据中的类型化属性。

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty1)]

以下代码示例演示如何保存用户数据内的复杂类型中的属性。 

[!code-csharp[Set state property](../includes/code/dotnet-state.cs#setProperty2)]

## <a name="handle-concurrency-issues"></a>处理并发性问题

当机器人尝试保存状态数据时，如果机器人的另一个实例已更改该数据，机器人可能会收到 HTTP 状态代码为“412 不满足前提条件”的错误响应。 可按以下代码示例中所示设计机器人，以应对这种情况。

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>其他资源

- [Bot Framework 故障排除指南](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
