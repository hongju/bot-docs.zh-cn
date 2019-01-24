---
title: 直接写入存储 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for .NET 将数据直接写入到存储。
keywords: 存储、读取和写入、内存存储、eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f5f566c48eb62bd6b60c869c28904bc61d74eda4
ms.sourcegitcommit: fc75b206e276ce66e9848e97d75f562bf9401f04
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/24/2019
ms.locfileid: "54837974"
---
# <a name="write-directly-to-storage"></a>直接写入存储

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

可以对存储对象进行直接读写，不需使用中间件或上下文对象。 这适用于机器人所用数据来源于机器人聊天流外部的情况。 例如，假设机器人允许用户询问天气预报，而它通过读取外部数据库中的数据来检索指定日期的天气预报。 天气数据库的内容不依赖于用户信息或聊天上下文，因此可以直接从存储中读取，无需使用状态管理器。 本文中的代码示例演示如何使用**内存存储**、**Cosmos DB**、**Blob 存储**和 **Azure Blob 脚本存储**对存储进行数据读写。 

## <a name="prerequisites"></a>先决条件
- 如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/en-us/free/)帐户。
- 安装 Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)

## <a name="memory-storage"></a>内存存储

我们将先创建一个可以对内存存储进行数据读写的机器人。 内存存储仅用于测试，不用于生产。 请确保在发布机器人之前，将存储设置为 Cosmos DB 或 Blob 存储。

#### <a name="build-a-basic-bot"></a>生成基础机器人

本主题的其余部分在 Echo 机器人的基础上进行构建。 可以用 [C#](../dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [JS](../javascript/bot-builder-javascript-quickstart.md) 创建一个。 可以使用 Bot Framework Emulator 连接到机器人，与之聊天并对其进行测试。 下面的示例将来自用户的每个消息添加到列表。 包含列表的数据结构保存到存储中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Create local Memory Storage.
private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Create cancellation token (used by Async Write operation).
public CancellationToken cancellationToken { get; private set; }

// Class for storing a log of utterances (text of messages) as a list.
public class UtteranceLog : IStoreItem
{
     // A list of things that users have said to the bot
     public List<string> UtteranceList { get; } = new List<string>();

     // The number of conversational turns that have occurred        
     public int TurnNumber { get; set; } = 0;

     // Create concurrency control where this is used.
     public string ETag { get; set; } = "*";
}

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext context)
{
     
     var activityType = context.Activity.Type;
     // See if activity type for this turn is a message from the user.
     if (activityType == ActivityTypes.Message)
     {
         var utterance = context.Activity.Text;
         UtteranceLog logItems = null;
          
         // see if there are previous messages saved in sstorage.
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;

         // If no stored messages were found, create and store a new entry.
         if (logItems is null)
         {
             logItems = new UtteranceLog();
         }
         
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await context.SendActivityAsync($"The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
             changes.Add("UtteranceLog", logItems);
          };
          
          // Save new list to your Storage.
          await _myStorage.WriteAsync(changes,cancellationToken);
     }
     return;
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const { BotFrameworkAdapter, ConversationState, BotStateSet, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Create server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add memory storage.
var storage = new MemoryStorage();

const conversationState = new ConversationState(storage);
adapter.use(conversationState);

// Listen for incoming activity .
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            await logMessageText(storage, context);

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});

async function logMessageText(storage, context) {
    let utterance = context.activity.text;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLog"])
        // Check the result.
        var utteranceLog = storeItems["UtteranceLog"];

        if (typeof (utteranceLog) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLog"].UtteranceList.push(utterance);

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to utterance log.');
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLog: ${err}`);
            }

         } else {
            context.sendActivity(`need to create new utterance log`);
            storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }

            try {
                await storage.write(storeItems)
                context.sendActivity('Successful write to log.');
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}
```

---

### <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。 
2. 选择创建项目时所在目录中的 .bot 文件。

### <a name="interact-with-your-bot"></a>与机器人交互
向机器人发送消息，机器人会列出所收到的消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>使用 Cosmos DB
使用内存存储以后，我们现在要更新代码，以便使用 Azure Cosmos DB。 Cosmos DB 由 Microsoft 推出的全球分布式多模型数据库。 使用 Azure Cosmos DB 可跨任意数量的 Azure 地理区域弹性且独立地缩放吞吐量和存储。 它通过综合服务级别协议 (SLA) 提供吞吐量、延迟、可用性和一致性保证。 

### <a name="set-up"></a>设置
若要在机器人中使用 Cosmos DB，需在编写代码之前进行一些设置。

#### <a name="create-your-database-account"></a>创建数据库帐户
1. 在新浏览器窗口中，登录到 [Azure 门户](http://portal.azure.com)。
2. 单击“创建资源”>“数据库”>“Azure Cosmos DB”
3. 在“新建帐户”页的“ID”字段中提供唯一的名称。 至于 **API**，请选择“SQL”，然后提供“订阅”、“位置”、“资源组”信息。
4. 然后单击“创建”。

创建帐户需要几分钟时间。 等待门户中显示“祝贺你! 已创建 Azure Cosmos DB 帐户”页。

##### <a name="add-a-collection"></a>添加集合
1. 单击“设置”>“新建集合”。 “添加集合”区域显示在最右侧，可能需要向右滚动才能看到它。 由于最近对 Cosmos DB 进行的更新，请务必添加单个分区键：_/id_。此键将避免跨分区查询错误。

![添加 Cosmos DB 集合](./media/add_database_collection.png)

2. 新数据库集合为“bot-cosmos-sql-db”，集合 ID 为“bot-storage”。 在随后的编码示例（见下）中，我们会使用这些值。
 -
![Cosmos DB](./media/cosmos-db-sql-database.png)

3. 可以在数据库设置的“密钥”选项卡中找到终结点 URI 和密钥。 需要在本文后面使用这些值来配置代码。 

![Cosmos DB 密钥](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>添加配置信息
用于添加 Cosmos DB 存储的配置数据很简短。若要构建更复杂的机器人，可以使用相同方法添加其他配置设置。 以下示例使用上一示例中 Cosmos DB 数据库和集合的名称。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 `.env` 文件中添加以下信息。 
 
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=Tasks
COLLECTION=Items
```
---

#### <a name="installing-packages"></a>安装包
确保有 Cosmos DB 所需的包

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

可以通过 npm 在项目中添加对 botbuilder-azure 的引用：-->


```powershell
npm install --save botbuilder-azure 
```

若要使用 .env 配置文件，需在机器人中包含一个附加的包。 首先，从 npm 获取 dotnet 包：

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>实现 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

以下示例代码运行时，使用的是与上面提供的[内存存储](#memory-storage)示例相同的机器人代码。
以下代码片段演示如何为“_myStorage_”实现 Cosmos DB 存储，替换本地内存存储。 

```csharp
using Microsoft.Bot.Builder.Azure;

// Create access to Cosmos DB storage.
// Replaces Memory Storage with reference to Cosmos DB.
private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
{
   AuthKey = CosmosDBKey,
   CollectionId = CosmosDBCollectionName,
   CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
   DatabaseId = CosmosDBDatabaseName,
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

以下示例代码类似于[内存存储](#memory-storage)，但有一些小的变化。

需要来自 botbuilder-azure 的 `CosmosDbStorage`，并将 dotenv 配置为读取 `.env` 文件。

**app.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
require('dotenv').config()
```

将内存存储替换为对 Cosmos DB 的引用。

```javascript
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

const conversationState = new ConversationState(storage);
adapter.use(conversationState);
```

---

## <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。 
2. 选择创建项目时所在目录中的 .bot 文件。

## <a name="interact-with-your-bot"></a>与机器人交互
向机器人发送消息，机器人会列出所收到的消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>查看数据
运行机器人并保存信息以后，即可在 Azure 门户的“数据资源管理器”选项卡下查看。 

![数据资源管理器示例](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>使用 eTag 管理并发
在机器人代码示例中，我们将每个 `IStoreItem` 的 `eTag` 属性设置为 `*`。 存储对象的 `eTag`（实体标记）成员在 Cosmos DB 中用于管理并发。 如果在机器人向存储中写入内容时，该机器人的另一实例更改了同一存储中的对象，`eTag` 会指示数据库应该如何处理。 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>最后一次写入才算 - 允许覆盖
星号 (`*`) 的 `eTag` 属性值指示最后一次写入才算数。 在创建新的数据存储时，可以将属性的 `eTag` 设置为 `*`，以表明以前未保存正在编写的数据，或者想要最后一次写入覆盖任何以前保存的属性。 如果并发性对于机器人来说不是问题，则对于任何要写入的数据，将 `eTag` 属性设置为 `*` 就可以允许覆盖。

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>保持并发并防止覆盖
将数据存储到 Cosmos DB 中时，若要防止对属性进行并发访问并避免覆盖另一个机器人实例的更改，可以对 `eTag` 使用 `*` 之外的值。 当机器人尝试保存状态数据但 `eTag` 的值与存储中 `eTag` 的值不同时，它会收到包含 `etag conflict key=` 消息的错误响应。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

默认情况下，每当机器人写入到该项时，Cosmos DB 存储都会检查存储对象的 `eTag` 属性是否等同，然后在每次写入之后将其更新为新的惟一值。 如果写入的 `eTag` 属性与存储中的 `eTag` 不匹配，这意味着另一个机器人或线程已更改数据。 

例如，假设你想让机器人编辑已保存的注释，但不希望机器人覆盖另一个机器人实例所做的更改。 如果另一个机器人实例已进行了编辑，你希望用户使用最新更新编辑版本。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

首先，创建实现 `IStoreItem` 的类。

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

接下来，通过创建存储对象创建初始注释，并将该对象添加到存储。

```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

然后访问和更新注释，保留从存储中读取的 `eTag`。

```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

如果在写入变更之前在存储中更新了注释，调用 `Write` 将引发异常。


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

请将一个帮助程序函数添加到机器人末尾，该机器人会将示例注释写入到数据存储中。
首先创建 `myNoteData` 对象。

```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

在 `createSampleNote` 帮助程序函数中初始化 `changes` 对象并向其添加注释，然后将结果写入存储。

```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
    await storage.write(changes);
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

然后，为了在以后访问并更新此注释，我们创建另一个帮助程序函数。当用户键入“更新注释”时，就可以访问该函数。

```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
            await storage.write(note); // Write the changes back to storage
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

如果在写入变更之前在存储中更新了注释，调用 `write` 将引发异常。

---

若要保持并发，始终从存储读取属性，然后修改读取的属性，以便保持 `eTag`。 如果从存储中读取用户数据，响应将包含 eTag 属性。 如果更改数据并将更新后的数据写入到存储中，请求应包含指定与之前读取值相同的值的 eTag 属性。 但是，编写将其 `eTag` 设置为 `*` 的对象将允许写入覆盖任何其他更改。

## <a name="using-blob-storage"></a>使用 Blob 存储 
Azure Blob 存储是 Microsoft 提供的适用于云的对象存储解决方案。 Blob 存储最适合存储巨量的非结构化数据，例如文本或二进制数据。

### <a name="create-your-blob-storage-account"></a>创建 Blob 存储帐户
若要在机器人中使用 Blob 存储，需在编写代码之前进行一些设置。
1. 在新浏览器窗口中，登录到 [Azure 门户](http://portal.azure.com)。
2. 单击“创建资源”>“存储”>“存储帐户 - Blob、文件、表、队列”
3. 在“新建帐户”页中输入存储帐户的“名称”，选择“Blob 存储”作为“帐户类型”，提供“位置”、“资源组”和“订阅”信息。  
4. 然后单击“创建”。

![创建 Blob 存储](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>添加配置信息

找到所需的 Blob 存储密钥，以便为机器人配置 Blob 存储，如上所示：
1. 在 Azure 门户中打开 Blob 存储帐户，然后选择“设置”>“访问密钥”。

![找到 Blob 存储密钥](./media/find-blob-storage-keys.png)

我们现在将使用这其中的两个密钥，以便为代码提供访问 Blob 存储帐户的权限。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
更新将“_myStorage_”指向现有 Blob 存储帐户的代码行。

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

将“_myStorage_”设置为指向 Blob 存储帐户以后，机器人代码现在就可以通过 Blob 存储来存储和检索数据。

## <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。 
2. 选择创建项目时所在目录中的 .bot 文件。

## <a name="interact-with-your-bot"></a>与机器人交互
向机器人发送消息，机器人会列出所收到的消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>查看数据
运行机器人并保存信息以后，即可在 Azure 门户的“存储资源管理器”选项卡下查看。

## <a name="blob-transcript-storage"></a>Blob 脚本存储
Azure Blob 脚本存储提供专门的存储选项，可以轻松地以记录脚本形式保存和检索用户聊天内容。 Azure Blob 脚本存储尤其适合在调试机器人性能时自动捕获要检查的用户输入。

### <a name="set-up"></a>设置
Azure Blob 脚本存储可以使用通过上面的“创建 Blob 存储帐户”和“添加配置信息”部分详述的步骤创建的 Blob 存储帐户。 为了进行本次讨论，我们添加了新的 Blob 容器“_mybottranscripts_”。 

### <a name="implementation"></a>实现 
以下代码将脚本存储指针“_transcriptStore_”连接到新的 Azure Blob 脚本存储帐户。 在此处显示的用于存储用户聊天内容的源代码基于[聊天历史记录](https://aka.ms/bot-history-sample-code)示例。 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>在 Azure Blob 脚本中存储用户聊天内容
可以使用 Blob 容器来存储脚本以后，即可保留用户与机器人的聊天内容。 这些聊天内容可以在以后用作调试工具，以便了解用户与机器人的交互情况。 以下代码保留当 activity.text 收到输入消息 _!history_ 时的用户聊天输入。


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id, continuationToken);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>找到为通道存储的所有脚本
若要查看自己存储了哪些数据，可以使用以下代码以编程方式查找所有已存储脚本的“_ConversationID_”。 使用机器人模拟器来测试代码时，选择“重新开始”即可启动一个使用新“_ConversationID_”的新脚本。

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>从 Azure Blob 脚本存储中检索用户聊天内容
将机器人交互脚本存储到 Azure Blob 脚本存储中以后，即可以编程方式检索它们，以便使用 AzureBlobTranscriptStorage 方法“_GetTranscriptActivities_”进行测试或调试。 以下代码片段从每个存储的脚本中检索在过去 24 小时内收到并存储的所有用户输入脚本。


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>从 Azure Blob 脚本存储中删除存储的脚本
使用用户输入数据对机器人进行测试或调试以后，即可以编程方式从 Azure Blob 脚本存储中删除存储的脚本。 以下代码片段从机器人脚本存储中删除所有存储的脚本。


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


以下链接提供有关 [Azure Blob 脚本存储](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)的详细信息  

## <a name="next-steps"></a>后续步骤
现在你已经了解如何直接从存储中进行读写，接下来让我们来看看如何使用状态管理器来执行这些操作。

> [!div class="nextstepaction"]
> [使用会话和用户属性保存状态](bot-builder-howto-v4-state.md)

