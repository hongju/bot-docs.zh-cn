---
title: 保存用户数据 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK 将用户状态数据保存到存储区。
keywords: 保存用户数据, 存储, 聊天数据
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 539e9e1cd772495d849ce106ee7d6a157fc1a9c0
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/04/2018
ms.locfileid: "39515077"
---
# <a name="persist-user-data"></a>保存用户数据

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

当机器人要求用户进行输入时，你可能希望将某些信息保存到某种形式的存储中。 借助 Bot Builder SDK，你可使用“内存中存储”、“文件存储”或“数据库存储”（如 CosmosDB 或 SQL）来存储用户的输入项，其中本地存储类型主要用于测试或原型制作，而后期存储类型最适合生产机器人。

本教程将展示如何定义存储对象，以及如何将用户的输入项保存到存储对象中，使其可持久保存。 

> [!NOTE]
> 无论你选择使用哪种存储类型，连接和保存数据的过程都是相同的。 本教程使用 `FileStorage` 作为存储来保存数据。
> 有关状态和其他存储类型的详细信息，请参阅[管理聊天和用户状态](bot-builder-howto-v4-state.md)。

## <a name="prequisite"></a>先决条件 

本教程是基于[预订餐位](bot-builder-tutorial-waterfall.md)教程进行编写的。 在“预订餐位”教程中，你构建了一个机器人，它要求用户提供 3 条有关在你餐厅订座的信息。 但是，用户输入的内容未保留。 本教程将让该机器人能够持久存储数据。

## <a name="add-storage-to-middleware-layer"></a>将存储添加到中间件层

Bot Builder V4 SDK 通过状态管理器中间件处理状态和存储。 中间件提供一个抽象层，它让你可使用简单的键值存储访问属性（不考虑基础存储的类型）。 无论基础存储是内存中、文件存储还是 Azure 表存储类型，状态管理器都会将数据写入存储并管理并发。 在本教程中，我们将使用 `FileStorage` 来保留用户的输入项。

`FileStorage` 提供程序附带 `bot-builder` 包。 最初使用的示例采用的是 `MemoryStorage` 提供程序。 此存储类型使用机器人的易失性内存，它在机器人重启时被释放。 而另一方面，`FileStorage` 库的行为与数据库类似。 也就是说，此库会将存储信息写入到本地计算机上的文件。 你可指定放置此存储文件的位置，以便日后进行检查。

要使用 `FileStorage`，请在机器人中找到定义了 `conversationState` 对象的语句，并将其更新以创建 `new botbuilder.FileStorage("c:/temp")`。 此外，你可以定义应将该存储文件写出的位置。 这样一来，你即可轻松找到它来检查所保存的内容。

# <a name="ctabcstab"></a>[C#](#tab/cstab)
```cs
var storage = new FileStorage("c:/temp");

// These two classes are simply Dictionaries to store state
options.Middleware.Add(new ConversationState<MyBot.convoState>(storage));
options.Middleware.Add(new UserState<MyBot.userState>(storage));
```

Bot Builder SDK 提供 3 个具有不同范围的状态对象供你选择。

| 省/直辖市/自治区 | 范围 | Description |
| ---- | ---- | ---- |
| `dc.ActiveDialog.State` | 对话 | 可供瀑布式对话中的步骤使用的状态。 |
| `ConversationState` | 聊天 | 可供当前聊天使用的状态。 |
| `UserState` | user | 可供多个聊天使用的状态。 |

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js**
```javascript
// Storage
const storage = new FileStorage("c:/temp");
const convoState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(convoState, userState));
```

`BotStateSet` 可同时管理 `ConversationState` 和 `UserState`。 在保存用户数据时，可进行选择。 Bot Builder SDK 提供 3 个具有不同范围的状态对象供你选择。

| 省/直辖市/自治区 | 范围 | Description |
| ---- | ---- | ---- |
| `dc.activeDialog.state` | 对话 | 可供瀑布式对话中的步骤使用的状态。 |
| `ConversationState` | 聊天 | 可供当前聊天使用的状态。 |
| `UserState` | user | 可供多个聊天使用的状态。 |

---
 

## <a name="persist-state"></a>保存状态

对于机器人，可写入上述三个状态位置中的任何一个，这具体取决于要保存的内容和需要保存的时长。  

# <a name="ctabcstab"></a>[C#](#tab/cstab)

状态管理器中间件在每个回合结束时替你写入状态。 因此，你只需在机器人中将数据分配给所选的状态对象即可。 在本例中，我们使用 `dc.ActiveDialog.State` 来跟踪用户在预约中输入的内容。 这样，就可将用户输入的内容存储在对话的临时状态对象范围中，而不是将其保存在全局变量中。 只有该对话处于活动状态时，才存在此对象；如果你想要将其保留更长时间，必须先将它传输到其他某个状态对象。 如果这样，我们将在瀑布式聊天的最后一步向聊天状态分配预约 `msg`。

```cs
dialogs.Add("reserveTable", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt for the guest's name.
        await dc.Context.SendActivity("Welcome to the reservation service.");

        dc.ActiveDialog.State = new Dictionary<string, object>();

        await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
    },
    async(dc, args, next) =>
    {
        var dateTimeResult = ((DateTimeResult)args).Resolution.First();

        dc.ActiveDialog.State["date"] = Convert.ToDateTime(dateTimeResult.Value);
        
        // Ask for next info
        await dc.Prompt("partySizePrompt", "How many people are in your party?");

    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["partySize"] = (int)args["Value"];

        // Ask for next info
        await dc.Prompt("textPrompt", "Who's name will this be under?");
    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["name"] = args["Text"];
        string msg = "Reservation confirmed. Reservation details - " +
        $"\nDate/Time: {dc.ActiveDialog.State["date"].ToString()} " +
        $"\nParty size: {dc.ActiveDialog.State["partySize"].ToString()} " +
        $"\nReservation name: {dc.ActiveDialog.State["name"]}";

        var convo = ConversationState<convoState>.Get(dc.Context);

        // In production, you may want to store something more helpful
        convo[$"{dc.ActiveDialog.State["name"]} reservation"] = msg;

        await dc.Context.SendActivity(msg);
        await dc.End();
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

状态管理器中间件在每个回合结束时替你将状态写入到文件中。 因此，你只需在机器人中将数据分配给所选的状态对象即可。 在本例中，我们使用 `dc.activeDialog.state` 跟踪用户在 `reservervationInfo` 对象中输入的内容。 这样，就可将用户输入的内容存储在对话的临时状态对象范围中，而不是将其保存在全局变量中。 因为只有在对话处于活动状态时此对象才存在，因此若要保留它，必须将其传输到其他某个状态对象。 如果这样，我们将在瀑布式对话的最后一步向 `convo` 状态分配 `reservationInfo`。

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        dc.activeDialog.state.reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.reserveName = result;
        
        // Persist data
        var convo = conversationState.get(dc.context);; // conversationState.get(dc.context);
        convo.reservationInfo = dc.activeDialog.state.reservationInfo;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${dc.activeDialog.state.reservationInfo.dateTime} 
            <br/>Party size: ${dc.activeDialog.state.reservationInfo.partySize} 
            <br/>Reservation name: ${dc.activeDialog.state.reservationInfo.reserveName}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

现在，你已准备好将此消息连接到机器人逻辑中。

## <a name="start-the-dialog"></a>启动对话

无需在此处进行任何代码更改。 只需运行机器人并发送消息即可开始 `reserveTable` 聊天。

## <a name="check-file-storage-content"></a>检查文件存储内容

运行机器人并完成 `reserveTable` 聊天后，找到保存到指定位置（例如“C:/temp”）的文件中的信息。 文件名称前面附有“conversation!” 或者“user!”一词。 按日期对文件进行排序可能会有帮助，这样可让你更轻松地找到最新的文件。

## <a name="next-steps"></a>后续步骤

现在你已了解如何保存用户输入的内容，接下来让我们看看你可使用提示库要求用户输入哪些类型的内容。

> [!div class="nextstepaction"]
> [提示用户输入](~/v4sdk/bot-builder-prompts.md)
