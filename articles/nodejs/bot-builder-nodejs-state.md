---
title: 管理状态数据 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 来保存和检索状态数据。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 779411235bfef24719044b0fbad26574a373a34f
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404299"
---
# <a name="manage-state-data"></a>管理状态数据

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>内存中数据存储

内存中数据存储仅用于测试。 此存储是易失性的临时存储。 每当重新启动机器人便会清除数据。 若要将内存中存储用于测试目的，需要执行两项操作。 首先创建内存中存储的新实例：

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

然后，在创建 UniversalBot  时将其设置到机器人：

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

可以使用此方法设置你自己的自定义数据存储或使用任一 Azure 扩展  。

## <a name="manage-custom-data-storage"></a>管理自定义数据存储

出于生产环境中的性能和安全考虑，你可以实施自己的数据存储，或考虑实施以下数据存储选项之一：

1. [使用 Cosmos DB 管理状态数据](bot-builder-nodejs-state-azure-cosmosdb.md)

2. [使用表存储管理状态数据](bot-builder-nodejs-state-azure-table-storage.md)

使用任意一个 [Azure 扩展](https://www.npmjs.com/package/botbuilder-azure)选项，用于通过 Bot Framework SDK for Node.js 设置和持久保存数据的机制与内存中数据存储都将保持一致。

## <a name="storage-containers"></a>存储容器

在 Bot Framework SDK for Node.js 中，`session` 对象公开以下用于存储状态数据的属性。

| 属性 | 范围 | 说明 |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | 用户 | 包含在指定通道上为用户保存的数据。 此数据将在多个会话中保持原样。 |
| [`privateConversationData`][privateConversationDataURL] | 对话 | 包含指定通道上在特定会话上下文中为用户保存的数据。 这些数据对当前用户来说是私有的，并且仅在当前会话中保留。 当会话结束或显式调用 `endConversation` 时，属性将被清除。 |
| [`conversationData`][conversationDataURL] | 对话 | 包含指定通道上在特定会话上下文中保存的数据。 此数据与参与会话的所有用户共享，并且仅在当前会话中保留。 当会话结束或显式调用 `endConversation` 时，属性将被清除。 |
| [`dialogData`][dialogDataURL] | 对话框 | 包含仅为当前对话框保存的数据。 每个对话框都保留了一份此属性的副本。 从对话框堆栈中移除对话框时，将清除该属性。 |

这四个属性对应于可以用来存储数据的四个数据存储容器。 用于存储数据的属性将取决于所存储的数据的适当范围、储存的数据的性质，以及你希望数据保留的时间。 例如，如果需要存储可跨多个会话使用的用户数据，请考虑使用 `userData` 属性。 如果需要在会话范围内临时存储一个本地变量值，请考虑使用 `dialogData` 属性。 如果需要临时存储必须可在多个对话框中访问的数据，请考虑使用 `conversationData` 属性。

## <a name="data-persistence"></a>数据暂留

默认情况下，使用 `userData`、`privateConversationData` 和 `conversationData` 属性存储的数据设置为保留到会话结束。 如果不希望数据保留在 `userData` 容器中，请将 `persistUserData` 标记设置为 false  。 如果不希望数据保留在 `conversationData` 容器中，请将 `persistConversationData` 标记设置为 false  。 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> 不能禁用 `privateConversationData` 容器的数据暂留；它始终保持暂留状态。

## <a name="set-data"></a>设置数据

可以将简单的 JavaScript 对象直接保存到存储容器中进行存储。 对于 `Date` 等复杂对象，请考虑将其转换为 `string`。 这是因为状态数据被序列化并存储为 JSON。 下面的代码示例展示了如何存储基元数据、数组、对象映射和复杂的 `Date` 对象。 

**存储基元数据**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**存储数组**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**存储对象映射**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**存储日期和时间** 

对于复杂的 JavaScript 对象，在保存到存储容器之前请将其转换为字符串。 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>保存数据

保存容器之前，在每个存储容器中创建的数据都将保留在内存中。 Bot Framework SDK for Node.js 向 `ChatConnector` 服务分批发送数据，以便在发送消息时保存数据。 若要保存存储容器中的现有数据而不发送任何消息，可以手动调用 [`save`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save) 方法。 如果不调用 `save` 方法，存储容器中的现有数据将保留为批处理的一部分。

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>获取数据

若要访问保存在特定存储容器中的数据，只需引用相应的属性。 下面的代码示例展示了如何访问之前存储为基元数据、数组、对象映射和复杂 Date 对象的数据。

**访问基元数据**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**访问数组**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**访问对象映射**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**访问 Date 对象** 

检索字符串形式的日期数据，然后将它转换为 JavaScript 的 Date 对象。 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>删除数据

默认情况下，从对话框堆栈中移除对话框时，存储在 `dialogData` 容器中的数据将被清除。 同样，当调用 `endConversation` 方法，存储在 `conversationData` 容器和 `privateConversationData` 容器中的数据也将被清除。 但是，若要删除存储在 `userData` 容器中的数据，必须显式清除它。

若要显式清除存储在任何容器中的数据，只需重置容器，如以下代码示例所示。 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

请勿设置数据容器 `null` 或从 `session` 对象中将其删除，这样做会导致下次尝试访问容器时出错。 此外，手动清除内存中的容器后，你可能想要手动调用 `session.save();`，以清除以前保存的任何相应数据。

## <a name="next-steps"></a>后续步骤

现在你已了解如何管理用户状态数据，接下来让我们看一下如何使用它来更好地管理会话流。

> [!div class="nextstepaction"]
> [管理会话流](bot-builder-nodejs-dialog-manage-conversation-flow.md)

## <a name="additional-resources"></a>其他资源
- [提示用户输入](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
