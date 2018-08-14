---
title: 使用聊天和用户属性保存状态 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 的 V4 版本保存和检索状态数据。
keywords: 聊天状态, 用户状态, 状态中间件, 聊天流, 文件存储, azure 表存储
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16df371b1cabb4b3eb47d1f491a5d45e26627d38
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352846"
---
# <a name="save-state-using-conversation-and-user-properties"></a>使用聊天和用户属性保存状态

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

要让机器人保存聊天和用户状态，请先初始化状态管理器中间件，然后再使用聊天和用户状态属性。
要详细了解如何使用该状态，请参阅[状态和存储](./bot-builder-storage-concept.md)。

## <a name="initialize-state-manager-middleware"></a>初始化状态管理器中间件

在 SDK 中，需要先初始化机器人适配器以使用状态管理器中间件，然后才能使用聊天或用户属性存储。 聊天状态用于聊天属性，用户状态用于用户属性。 （可以跨多个聊天访问用户状态属性。）状态管理器中间件提供一个抽象项，它可让你使用简单的键值或对象存储访问属性（不考虑基础存储的类型）。 无论基础存储是内存中、文件存储还是 Azure 表存储类型，状态管理器都会将数据写入存储并管理并发。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
要查看 `ConversationState` 的初始化方式，请参阅 Microsoft.Bot.Samples.EchoBot-AspNetCore 示例中的 `Startup.cs`。

```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

    IStorage dataStore = new MemoryStorage();
    options.Middleware.Add(new ConversationState<EchoState>(dataStore));
});
```

在行 `options.Middleware.Add(new ConversationState<EchoState>(dataStore));` 中，`ConversationState` 是聊天状态管理器对象，它作为中间件添加到机器人中。 `EchoState` 类型参数是表示要如何存储聊天状态信息的类型。 机器人可以使用聊天或用户状态数据的任何类类型。

`EchoState` 的实现位于 `EchoBot.cs` 中：
```csharp
public class EchoState
{
    public int TurnNumber { get; set; }
}
``` 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

定义存储提供程序并将其分配给要使用的状态管理器。
对于 `ConversationState` 和 `UserState` 管理中间件，可以使用相同的存储提供程序。
然后，使用 `BotStateSet` 库将其连接到要管理数据持久性的中间件层。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware.
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Alternatively, use both conversation and user state middleware.
// const storage = new MemoryStorage;
// const conversationState = new ConverstationState(storage);
// const userState = new UserState(storage);
// adapter.use(new BotStateSet(conversationState, userState));
```    
---

> [!NOTE] 
> 内存中数据存储仅用于测试。 此存储是易失性的临时存储。 每次重启机器人时都会清除数据。 要设置聊天状态和用户状态的其他基础存储媒体，请参阅本文稍后部分的[文件存储](#file-storage)和 [Azure 表存储](#azure-table-storage)。 

### <a name="configuring-state-manager-middleware"></a>配置状态管理器中间件

初始化状态中间件时，使用可选的状态设置参数可更改属性保存方式的默认行为。 默认设置为：

* 保留超过回合上下文生存期的属性。
* 如果将机器人的多个实例写入属性，请允许机器人的最后一个实例覆盖前一个实例。

## <a name="use-conversation-and-user-state-properties"></a>使用聊天和用户状态属性 
<!-- middleware and message context properties -->

配置状态管理器中间件后，可从上下文对象中获取聊天状态和用户状态属性。
<!-- Changes are written to storage before the `SendActivity()` pipeline completes. -->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

可通过 Bot Builder SDK 中的 `Microsoft.Bot.Samples.EchoBot` 示例了解其工作原理。 

在 `OnTurn` 处理程序中，`context.GetConversationState` 获取聊天状态以访问所定义的数据，而你可使用属性修改状态（在本例中是指递增 `TurnNumber`）。

```csharp
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

可通过基于 Yeoman 生成器示例中生成的 EchoBot 查看其工作原理。

此代码示例显示了如何将回合计数器存储到聊天状态。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="using-conversation-state-to-direct-conversation-flow"></a>使用聊天状态来指示聊天流

在设计聊天流时，通过定义状态标记来指示聊天流非常有用。 该标记可以是简单的布尔类型，也可以是包含当前主题名称的类型。 该标记可帮助跟踪在聊天中所处的位置。 例如，布尔类型标记可指出你是否参与了聊天。 而主题名称属性可指出你当前所参与的聊天。

以下示例使用布尔值“have asked name”属性标记机器人已询问过用户姓名的情况。 收到下一条消息时，机器人会检查该属性。 如果该属性设置为 `true`，则机器人知道刚才已询问过用户的姓名，此时将传入的消息解释为一个要保存为用户属性的名称。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```csharp
public class ConversationInfo
{
    public bool haveAskedNameFlag { get; set; }
    public bool haveAskedNumberFlag { get; set; }
}

public class UserInfo
{
    public string name { get; set; }
    public string telephoneNumber { get; set; }
    public bool done { get; set; }
}

public async Task OnTurn(ITurnContext context)
{
    // Get state objects. Default objects are created if they don't already exist.
    var convo = ConversationState<ConversationInfo>.Get(context);
    var user = UserState<UserInfo>.Get(context);

    if (context.Activity.Type is ActivityTypes.Message)
    {
        if (string.IsNullOrEmpty(user.name) && !convo.haveAskedNameFlag)
        {
            // Ask for the name.
            await context.SendActivity("Hello. What's your name?");

            // Set flag to show we've asked for the name. We save this out so the
            // context object for the next turn of the conversation can check haveAskedName
            convo.haveAskedNameFlag = true;
        }
        else if (!convo.haveAskedNumberFlag)
        {
            // Save the name.
            var name = context.Activity.AsMessageActivity().Text;
            user.name = name;
            convo.haveAskedNameFlag = false; // Reset flag

            // Ask for the phone number. You might want a flag to track this, too.
            await context.SendActivity($"Hello, {name}. What's your telephone number?");
            convo.haveAskedNumberFlag = true;
        }
        else if (convo.haveAskedNumberFlag)
        {
            // save the telephone number
            var telephonenumber = context.Activity.AsMessageActivity().Text;

            user.telephoneNumber = telephonenumber;
            convo.haveAskedNumberFlag = false; // Reset flag
            await context.SendActivity($"Got it. I'll call you later.");
        }
    }
}
```

要设置用户状态，使其可通过 `UserState<UserInfo>.Get(context)` 返回，请添加用户状态中间件。 例如，在 ASP.NET Core EchoBot 的 `Startup.cs` 中更改 ConfigureServices.cs 中的代码：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        
        IStorage dataStore = new MemoryStorage();
        options.Middleware.Add(new ConversationState<ConversationInfo>(dataStore));
        options.Middleware.Add(new UserState<UserInfo>(dataStore));
    });
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**app.js**

```js
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter (it's ok for MICROSOFT_APP_ID and MICROSOFT_APP_PASSWORD to be blank for now)  
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Storage
const storage = new MemoryStorage();
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        const convo = conversationState.get(context);
        const user = userState.get(context);

        if (isMessage) {
            if(!user.name && !convo.haveAskedNameFlag){
                // Ask for the name.
                await context.sendActivity("What is your name?")
                // Set flag to show we've asked for the name. We save this out so the
                // context object for the next turn of the conversation can check haveAskedNameFlag
                convo.haveAskedNameFlag = true;
            } else if(convo.haveAskedNameFlag){
                // Save the name.
                user.name = context.activity.text;
                convo.haveAskedNameFlag = false; // Reset flag

                await context.sendActivity(`Hello, ${user.name}. What's your telephone number?`);
                convo.haveAskedNumberFlag = true; // Set flag
            } else if(convo.haveAskedNumberFlag){
                // save the phone number
                user.telephonenumber = context.activity.text;
                convo.haveAskedNumberFlag = false; // Reset flag
                await context.sendActivity(`Got it. I'll call you later.`);
            }
        }

        // ...
    });
});

```

---

还可使用对话的“瀑布”模型。 该对话持续跟踪聊天状态，因此你无需创建标记进行跟踪。 有关详细信息，请参阅[使用对话管理聊天](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="file-storage"></a>文件存储

内存存储提供程序使用在重启机器人时释放的内存中存储。 最好仅出于测试目的使用它。 如果要保留数据但不想将机器人连接到数据库，就可使用文件存储提供程序。 虽然此提供程序也用于测试目的，但它会将状态数据保留在文件中，便于你进行检查。 数据按 JSON 格式写出到文件。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

转到 Microsoft.Bot.Samples.EchoBot-AspNetCore 示例中的 `Startup.cs`，然后编辑 `ConfigureServices` 方法中的代码。
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // Using file storage instead of in-memory storage.
        IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());
        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
``` 

运行代码，让 echobot 回显几次输入。

然后转到 `System.IO.Path.GetTempPath()` 指定的目录。 此时应会看到一个名称以“conversation”开头的文件。 打开该文件并查找 JSON。 该文件包含如下内容：
```json
{
  "$type": "Microsoft.Bot.Samples.Echo.EchoState, Microsoft.Bot.Samples.EchoBot",
  "TurnNumber": "3",
  "eTag": "ecfe2a23566b4b52b2fe697cffc59385"
}
```

`$type` 指定在机器人中用于存储聊天状态的数据结构类型。 `TurnNumber` 字段与 `EchoState` 类中的 `TurnNumber` 属性相对应。 `eTag` 字段继承自 `IStoreItem`，它是一个唯一值，每当机器人更新聊天状态时都会自动更新。  机器人可通过 eTag 字段实现乐观并发。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

要使用 `FileStorage`，请更新之前在[使用聊天和用户状态属性](#use-conversation-and-user-state-properties)部分中所述的回显机器人示例。 请确保将 `storage` 设置为 `FileStorage` 而不是 `MemoryStorage`，同时需具备来自 botbuilder 的 `FileStorage`。 仅需进行上述更改。 

```javascript
// Storage
const storage = new FileStorage("c:/temp");
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));
```

`FileStorage` 提供程序将“路径”用作参数。 通过指定路径，你可使用机器人中的持久化信息轻松查找文件。 系统将对每个聊天创建一个新的文件。 因此，你可能在路径中发现多个以 `conversation!` 开头的文件名。 你可以按日期排序，从而更轻松地查找最新聊天。 但只能找到一个用户状态文件。 文件名将以 `user!` 开头。 每当其中任一对象的状态发生变化时，状态管理器都会更新文件以反映更改的内容。

运行机器人并向其发送一些消息。 然后，找到存储文件并将其打开。 下面是跟踪回合计数器的回显机器人可能显示的 JSON 内容。

```json
{
  "turnNumber": "3",
  "eTag": "322"
}
```
---

## <a name="azure-table-storage"></a>Azure 表存储

你还可以将 Azure 表存储用作存储媒体。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 Microsoft.Bot.Samples.EchoBot-AspNetCore 示例中，添加对 `Microsoft.Bot.Builder.Azure` NuGet 包的引用。

然后，转到 `Startup.cs`，添加 `using Microsoft.Bot.Builder.Azure;` 语句，再编辑 `ConfigureServices` 方法中的代码。
```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
    // The parameters are the connection string and table name.
    // "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
    // Replace it with your own connection string if you're not using the emulator
    options.Middleware.Add(new ConversationState<EchoState>(new AzureTableStorage("UseDevelopmentStorage=true","conversationstatetable")));
    // you could also specify the cloud storage account instead of the connection string
    /* options.Middleware.Add(new ConversationState<EchoState>(
        new AzureTableStorage(WindowsAzure.Storage.CloudStorageAccount.DevelopmentStorageAccount, "conversationstatetable"))); */
    options.EnableProactiveMessages = true;
});
```
`UseDevelopmentStorage=true` 是可与 [Azure 存储模拟器][AzureStorageEmulator]一起使用的连接字符串。 如果不使用模拟器，请将其替换为你自己的连接字符串。

如果表的名称均不是你在 `AzureTableStorage` 构造函数中指定的名称，则创建带此名称的表。

<!-- 
TODO: step-by-step inspection of the stored table
-->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 echobot 示例的 `app.js` 中，可使用 `AzureTableStorage` 创建 ConversationState。 

```bash
npm install --save botbuilder-azure@preview
```

```javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState } = require('botbuilder');
const { TableStorage } = require('botbuilder-azure');

// ...

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
// The parameters are the connection string and table name.
// "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
// Replace it with your own connection string if you're not using the emulator
var azureStorage = new TableStorage({ tableName: "TestAzureTable1", storageAccountOrConnectionString: "UseDevelopmentStorage=true"})

// You can alternatively use your account name and table name
// var azureStorage = new TableStorage({tableName: "TestAzureTable2", storageAccessKey: "V3ah0go4DLkMtQKUPC6EbgFeXnE6GeA+veCwDNFNcdE6rqSVE/EQO/kjfemJaitPwtAkmR9lMKLtcvgPhzuxZg==", storageAccountOrConnectionString: "storageaccount"});

const conversationState = new ConversationState(azureStorage);
adapter.use(conversationState);

```

`UseDevelopmentStorage=true` 是可与 [Azure 存储模拟器][AzureStorageEmulator]一起使用的连接字符串。
如果表的名称均不是你在 `AzureTableStorage` 构造函数中指定的名称，则创建带此名称的表。

---

要查看保存的聊天状态数据，请运行该示例，然后使用 [Azure 存储资源管理器][AzureStorageExplorer]打开该表。

![Azure 存储资源管理器中的 echobot 聊天状态数据](media/how-to-state/echostate-azure-storage-explorer.png)


分区键是特定于当前聊天的唯一生成的键。 如果重启机器人或开始新的聊天，则新的聊天将使用自己的分区键获得其自己的行。 `EchoState` 数据结构序列化为 JSON 并保存在 Azure 表的 Json 列中。 

```json
{
    "$type":"Microsoft.Bot.Samples.Echo.AspNetCore.EchoState, Microsoft.Bot.Samples.EchoBot-AspNetCore",
    "TurnNumber":2,
    "LastMessage":"second message",
    "eTag":"*"
}
```

BotBuilder SDK 添加了 `$type` 和 `eTag` 字段。 有关 eTag 的详细信息，请参阅[管理乐观并发](bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)


## <a name="next-steps"></a>后续步骤

你已了解如何使用状态从存储中读取机器人数据或将其写入到存储中，接下来让我们了解如何直接从存储中读取或直接写入存储。

> [!div class="nextstepaction"]
> [如何直接写入存储](bot-builder-howto-v4-storage.md)。

## <a name="additional-resources"></a>其他资源
有关存储的更多背景知识，请参阅 [Bot Builder SDK 中的存储](bot-builder-storage-concept.md)

<!-- Links -->
[AzureStorageEmulator]: https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator
[AzureStorageExplorer]: https://azure.microsoft.com/en-us/features/storage-explorer/
