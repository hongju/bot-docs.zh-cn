---
title: 管理状态数据 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for .NET 来保存和检索状态数据。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3ee3af72d1c03faf485a64adb8d9fa2548f5d99d
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323663"
---
# <a name="manage-state-data"></a>管理状态数据

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>内存中数据存储

内存中数据存储仅用于测试。 此存储是易失性的临时存储。 每次重启机器人都会清除数据。 若要将内存中存储用于测试目的，需要： 

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

| 属性 | 说明 | 使用案例 |
|----|----|----|
| `From` | 唯一标识通道上的用户 | 存储和检索与用户关联的状态数据 |
| `Conversation` | 唯一标识聊天 | 存储和检索与聊天关联的状态数据 |
| `From` 和 `Conversation` | 唯一标识用户和聊天 | 存储和检索与特定聊天上下文中特定用户相关联的状态数据 |

> [!NOTE]
> 即使你选择在自己的数据库中存储状态数据而不使用 Bot Framework 状态数据存储，也可以使用这些属性值作为键。

## <a name="handle-concurrency-issues"></a>处理并发性问题

当机器人尝试保存状态数据时，如果机器人的另一个实例已更改该数据，机器人可能会收到 HTTP 状态代码为“412 不满足前提条件”的错误响应。 可按以下代码示例中所示设计机器人，以应对这种情况。

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>其他资源

- [Bot Framework 故障排除指南](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET 参考</a>

[Activity]: https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
