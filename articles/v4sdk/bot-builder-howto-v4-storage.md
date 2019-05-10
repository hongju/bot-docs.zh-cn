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
ms.date: 4/13/19
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 417833b380e80788e26682ce3abd9cc98199eee5
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032825"
---
# <a name="write-directly-to-storage"></a>直接写入存储

[!INCLUDE[applies-to](../includes/applies-to.md)]

可以对存储对象进行直接读写，不需使用中间件或上下文对象。 这可能适用于机器人用来保留聊天的数据，或者来源于机器人聊天流外部的数据。 在此数据存储模型中，数据是直接从存储读取的，而不是使用状态管理器读取的。 本文中的代码示例演示如何使用**内存存储**、**Cosmos DB**、**Blob 存储**和 **Azure Blob 脚本存储**对存储进行数据读写。 

## <a name="prerequisites"></a>先决条件
- 如果没有 Azure 订阅，请在开始之前创建一个[免费](https://azure.microsoft.com/free/)帐户。
- 熟悉以下操作：在本地创建适用于 [dotnet](https://aka.ms/bot-framework-www-c-sharp-quickstart) 或 [nodeJS](https://aka.ms/bot-framework-www-node-js-quickstart) 的机器人。
- 适用于 [C#](https://aka.ms/bot-vsix) 或 [nodeJS](https://nodejs.org) 和 [yeoman](http://yeoman.io) 的 Bot Framework SDK v4 模板。

## <a name="about-this-sample"></a>关于此示例
本文中的示例代码首先演示基本聊天机器人的结构，然后通过添加更多的代码（下面会提供）来扩展该机器人的功能。 此扩展代码会创建一个列表，用于在收到用户输入时保留这些输入。 在每个轮次，会将完整的用户输入列表回显给用户。 在该轮次结束时，包含此输入列表的数据结构将保存到存储中。 作为附加功能探讨的各种存储将添加到此示例代码。

## <a name="memory-storage"></a>内存存储

Bot Framework SDK 允许使用内存中存储来存储用户输入。 内存存储仅用于测试，不用于生产。 持久性存储类型（例如数据库存储）最适合生产用机器人。 请确保在发布机器人之前，将存储设置为 Cosmos DB 或 Blob 存储。

#### <a name="build-a-basic-bot"></a>生成基础机器人

本主题的其余部分在 Echo 机器人的基础上进行构建。 可以遵照有关生成 [C# EchoBot](https://aka.ms/bot-framework-www-c-sharp-quickstart) 或 [JS EchoBot](https://aka.ms/bot-framework-www-node-js-quickstart) 的快速入门说明，在本地生成聊天机器人示例代码。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Represents a bot saves and echoes back user input.
public class EchoBot : ActivityHandler
{
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
     
   // Echo back user input.
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      // preserve user input.
      var utterance = turnContext.Activity.Text;  
      // make empty local logitems list.
      UtteranceLog logItems = null;
          
      // see if there are previous messages saved in storage.
      try
      {
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;
      }
      catch
      {
         // Inform the user an error occured.
         await turnContext.SendActivityAsync("Sorry, something went wrong reading your stored messages!");
      }
         
      // If no stored messages were found, create and store a new entry.
      if (logItems is null)
      {
         // add the current utterance to a new object.
         logItems = new UtteranceLog();
         logItems.UtteranceList.Add(utterance);
         // set initial turn counter to 1.
         logItems.TurnNumber++;

         // Show user new user message.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");

         // Create Dictionary object to hold received user messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         }
         try
         {
            // Save the user message to your Storage.
            await _myStorage.WriteAsync(changes, cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      // Else, our Storage already contained saved user messages, add new one to the list.
      else
      {
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         };
         
         try
         {
            // Save new list to your Storage.
            await _myStorage.WriteAsync(changes,cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      ...  // OnMessageActivityAsync( )
   }
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用 .env 配置文件，需在机器人中包含一个附加的包。 如果尚未安装 dotnet 包，请从 npm 获取它：

```powershell
npm install --save dotenv
```

**bot.js**
```javascript
const { ActivityHandler, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// Process incoming requests - adds storage for messages.
class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async turnContext => { console.log('this gets called (message)'); 
        await turnContext.sendActivity(`You said '${ turnContext.activity.text }'`); 
        // Save updated utterance inputs.
        await logMessageText(storage, turnContext);
    });
        this.onConversationUpdate(async turnContext => { console.log('this gets called (conversation update)'); 
        await turnContext.sendActivity('[conversationUpdate event detected]'); });
    }
}

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, turnContext) {
    let utterance = turnContext.activity.text;
    // debugger;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLogJS"])
        // Check the result.
        var UtteranceLogJS = storeItems["UtteranceLogJS"];
        if (typeof (UtteranceLogJS) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLogJS"].turnNumber++;
            storeItems["UtteranceLogJS"].UtteranceList.push(utterance);
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```
---

### <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
- 安装 Bot Framework [Emulator](https://aka.ms/bot-framework-emulator-readme)。接下来，启动模拟器，然后在仿真器中连接到机器人：

1. 单击仿真器“欢迎使用”选项卡中的“新建机器人配置”链接。 
2. 填写用于连接到机器人的字段，并指定启动机器人时要在网页上显示的信息。

### <a name="interact-with-your-bot"></a>与机器人交互
向机器人发送消息 机器人会列出它所收到的消息。

![在仿真器中测试存储机器人](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>使用 Cosmos DB
使用内存存储以后，我们现在要更新代码，以便使用 Azure Cosmos DB。 Cosmos DB 由 Microsoft 推出的全球分布式多模型数据库。 使用 Azure Cosmos DB 可跨任意数量的 Azure 地理区域弹性且独立地缩放吞吐量和存储。 它通过综合服务级别协议 (SLA) 提供吞吐量、延迟、可用性和一致性保证。 

### <a name="set-up"></a>设置
若要在机器人中使用 Cosmos DB，需在编写代码之前进行一些设置。 有关 Cosmos DB 数据库和应用创建过程的深入介绍，请访问适用于 [Cosmos DB dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) 或 [Cosmos DB nodejs](https://aka.ms/Bot-framework-create-nodejs-cosmosdb) 的文档。

### <a name="create-your-database-account"></a>创建数据库帐户

1. 在新浏览器窗口中，登录到 [Azure 门户](http://portal.azure.com)。

![创建 Cosmos DB 数据库](./media/create-cosmosdb-database.png)

2. 单击“创建资源”>“数据库”>“Azure Cosmos DB”

![Cosmos DB -“新建帐户”页](./media/cosmosdb-new-account-page.png)

3. 在“新建帐户”页上，提供“订阅”和“资源组”信息。 在“帐户名”字段中创建唯一的名称 - 此名称最终会成为数据访问 URL 名称的一部分。 对于“API”，请选择“Core(SQL)”，并提供一个附近的**位置**以加快数据访问速度。
4. 然后单击“查看 + 创建”。
5. 验证成功后，单击“创建”。

创建帐户需要几分钟时间。 等待门户中显示“祝贺你! 已创建 Azure Cosmos DB 帐户”页。

### <a name="add-a-collection"></a>添加集合

![添加 Cosmos DB 集合](./media/add_database_collection.png)

1. 单击“设置”>“新建集合”。 “添加集合”区域显示在最右侧，可能需要向右滚动才能看到它。 由于最近对 Cosmos DB 进行的更新，请务必添加单个分区键：_/id_。此键将避免跨分区查询错误。

![Cosmos DB](./media/cosmos-db-sql-database.png)

2. 新数据库集合为“bot-cosmos-sql-db”，集合 ID 为“bot-storage”。 在随后的编码示例（见下）中，我们会使用这些值。

![Cosmos DB 密钥](./media/comos-db-keys.png)

3. 可以在数据库设置的“密钥”选项卡中找到终结点 URI 和密钥。 需要在本文后面使用这些值来配置代码。 

### <a name="add-configuration-information"></a>添加配置信息
用于添加 Cosmos DB 存储的配置数据很简短。若要构建更复杂的机器人，可以使用相同方法添加其他配置设置。 以下示例使用上一示例中 Cosmos DB 数据库和集合的名称。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
public class EchoBot : ActivityHandler
{
   private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
   private const string CosmosDBKey = "<your-cosmos-db-account-key>";
   private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
   private const string CosmosDBCollectionName = "bot-storage";
   ...
   
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 `.env` 文件中添加以下信息。

**.env**
```javascript
DB_SERVICE_ENDPOINT="<your-Cosmos-db-URI>"
AUTH_KEY="<your-cosmos-db-account-key>"
DATABASE="<bot-cosmos-sql-db>"
COLLECTION="<bot-storage>"
```
---

#### <a name="installing-packages"></a>安装包
确保有 Cosmos DB 所需的包

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

可以通过 npm 在项目中添加对 botbuilder-azure 的引用。
>**注意** - 此 npm 包依赖于开发计算机上的现有 Python 安装。 如果你事先尚未安装 Python，可在此处找到适用于你的计算机的安装资源：[Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

如果尚未安装 dotnet 包，请从 npm 获取该包以访问 `.env` 文件设置。

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>实现 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

以下示例代码运行时，使用的是与上面提供的[内存存储](#memory-storage)示例相同的机器人代码。
以下代码片段演示如何为“_myStorage_”实现 Cosmos DB 存储，替换本地内存存储。 内存存储已注释掉，并已替换为对 Cosmos DB 的引用。

**EchoBot.cs**
```csharp

using System;
...
using Microsoft.Bot.Builder.Azure;
...
public class EchoBot : ActivityHandler
{
   // Create local Memory Storage - commented out.
   // private static readonly MemoryStorage _myStorage = new MemoryStorage();

   // Replaces Memory Storage with reference to Cosmos DB.
   private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
   {
      AuthKey = CosmosDBKey,
      CollectionId = CosmosDBCollectionName,
      CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
      DatabaseId = CosmosDBDatabaseName,
   });
   
   ...
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

以下示例代码类似于[内存存储](#memory-storage)，但有一些小的变化。

需要 `botbuilder-azure` 中的 `CosmosDbStorage`，并将 dotenv 配置为读取 `.env` 文件。

**bot.js**

```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
注释掉内存存储，将其替换为对 Cosmos DB 的引用。

**bot.js**
```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();

// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to CosmosDb Storage - this replaces local Memory Storage.
var storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT, 
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

```
---

## <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

## <a name="test-your-bot-with-bot-framework-emulator"></a>使用 Bot Framework Emulator 测试机器人
现在请启动 Bot Framework Emulator 并连接到机器人：

1. 单击仿真器“欢迎使用”选项卡中的“新建机器人配置”链接。 
2. 填写用于连接到机器人的字段，并指定启动机器人时要在网页上显示的信息。

## <a name="interact-with-your-bot"></a>与机器人交互
向机器人发送消息，机器人会列出所收到的消息。
![模拟器运行](./media/emulator-direct-storage-test.png)


### <a name="view-your-data"></a>查看数据
运行机器人并保存信息以后，即可在 Azure 门户的“数据资源管理器”选项卡下查看存储的数据。 

![数据资源管理器示例](./media/data_explorer.PNG)


## <a name="using-blob-storage"></a>使用 Blob 存储 
Azure Blob 存储是 Microsoft 提供的适用于云的对象存储解决方案。 Blob 存储最适合存储巨量的非结构化数据，例如文本或二进制数据。

### <a name="create-your-blob-storage-account"></a>创建 Blob 存储帐户
若要在机器人中使用 Blob 存储，需在编写代码之前进行一些设置。
1. 在新浏览器窗口中，登录到 [Azure 门户](http://portal.azure.com)。

![创建 Blob 存储](./media/create-blob-storage.png)

2. 单击“创建资源”>“存储”>“存储帐户 - Blob、文件、表、队列”

![Blob 存储 -“新建帐户”页](./media/blob-storage-new-account.png)

3. 在“新建帐户”页中输入存储帐户的“名称”，选择“Blob 存储”作为“帐户类型”，提供“位置”、“资源组”和“订阅”信息。  
4. 然后单击“查看 + 创建”。
5. 验证成功后，单击“创建”。

### <a name="create-blob-storage-container"></a>创建 Blob 存储容器
创建 Blob 存储帐户后，通过以下方式打开此帐户 
1. 选择资源。
2. 在存储资源管理器（预览版）中选择“打开”

![创建 Blob 存储容器](./media/create-blob-container.png)

3. 右键单击“BLOB 容器”，并选择“创建 Blob 容器”。
4. 添加一个名称。 稍后将使用此名称作为“your-blob-storage-container-name”值来提供对 Blob 存储帐户的访问。

#### <a name="add-configuration-information"></a>添加配置信息
找到所需的 Blob 存储密钥，以便为机器人配置 Blob 存储，如上所示：
1. 在 Azure 门户中打开 Blob 存储帐户，然后选择“设置”>“访问密钥”。

![找到 Blob 存储密钥](./media/find-blob-storage-keys.png)

我们将使用密钥 1“连接字符串”作为“your-blob-storage-container-name”值来提供对 Blob 存储帐户的访问。

#### <a name="installing-packages"></a>安装包
如果事先尚未安装以下包，请安装这些包以使用 Cosmos DB。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

通过 npm 在项目中添加对 botbuilder-azure 的引用。
>**注意** - 此 npm 包依赖于开发计算机上的现有 Python 安装。 如果你事先尚未安装 Python，可在此处找到适用于你的计算机的安装资源：[Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

如果尚未安装 dotnet 包，请从 npm 获取该包以访问 `.env` 文件设置。

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>实现 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;
```
更新将“_myStorage_”指向现有 Blob 存储帐户的代码行。

**EchoBot.cs**
```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 `.env` 文件中添加以下信息。

**.env**
```javascript
BLOB_NAME="<your-blob-storage-container-name>"
BLOB_STRING="<your-blob-storage-account-string>"
```

按如下所示更新 `bot.js` 文件。 需要 `botbuilder-azure` 中的 `BlobStorage`

**bot.js**
```javascript
const { BlobStorage } = require("botbuilder-azure");
```

如果尚未添加所需的代码用于加载 `.env` 文件来实现 Cosmos DB 存储，请在此处添加。

```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();
```
现在，请通过注释掉以前的存储定义并添加以下代码来更新代码，以将“存储”指向现有的 Blob 存储帐户。

**bot.js**
```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```
---

将存储设置为指向 Blob 存储帐户之后，机器人代码可以通过 Blob 存储来存储和检索数据。

## <a name="start-your-bot"></a>启动机器人
在本地运行机器人。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击仿真器“欢迎使用”选项卡中的“新建机器人配置”链接。 
2. 填写用于连接到机器人的字段，并指定启动机器人时要在网页上显示的信息。

## <a name="interact-with-your-bot"></a>与机器人交互
向机器人发送消息，机器人会列出所收到的消息。

![在仿真器中测试存储机器人](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>查看数据
运行机器人并保存信息以后，即可在 Azure 门户的“存储资源管理器”选项卡下查看。

## <a name="blob-transcript-storage"></a>Blob 脚本存储
Azure Blob 脚本存储提供专门的存储选项，可以轻松地以记录脚本形式保存和检索用户聊天内容。 Azure Blob 脚本存储尤其适合在调试机器人性能时自动捕获要检查的用户输入。

### <a name="set-up"></a>设置
Azure Blob 脚本存储可以使用通过上面的“创建 Blob 存储帐户”和“添加配置信息”部分详述的步骤创建的 Blob 存储帐户。 现在添加一个容器用于保存脚本

![创建脚本容器](./media/create-blob-transcript-container.png)

1. 打开 Azure Blob 存储帐户。
1. 单击“存储资源管理器”。
1. 右键单击“BLOB 容器”，并选择“创建 Blob 容器”。
1. 输入脚本容器的名称，然后选择“确定”。 （此处输入了 mybottranscripts）

### <a name="implementation"></a>实现 
以下代码将脚本存储指针 `_myTranscripts` 连接到新的 Azure Blob 脚本存储帐户。 若要使用新的容器名称 <your-blob-transcript-container-name> 创建此链接，请在 Blob 存储中创建一个新容器用于保存脚本文件。

**echoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

public class EchoBot : ActivityHandler
{
   ...
   
   private readonly AzureBlobTranscriptStore _myTranscripts = new AzureBlobTranscriptStore("<your-blob-transcript-storage-account-string>", "<your-blob-transcript-container-name>");
   
   ...
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>在 Azure Blob 脚本中存储用户聊天内容
可以使用 Blob 容器来存储脚本以后，即可保留用户与机器人的聊天内容。 这些聊天内容可以在以后用作调试工具，以便了解用户与机器人的交互情况。 每次在仿真器中选择“重启聊天”都会启动新脚本聊天列表的创建。 以下代码将在存储的脚本文件中保留用户聊天输入。
- 当前脚本是使用 `LogActivityAsync` 保存的。
- 可以使用 `ListTranscriptsAsync` 检索已保存的脚本。
在此示例代码中，存储的每个脚本的 ID 将保存到名为“storedTranscripts”的列表。 稍后将使用此列表来管理保留的已存储 Blob 脚本数。

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    ...
}

```

### <a name="manage-stored-blob-transcripts"></a>管理已存储的 Blob 脚本
尽管存储的脚本可用作调试工具，但一段时间后，存储的脚本数可能会大大增加，以致难以照料。 下面提供的附加代码使用 `DeleteTranscriptAsync` 从 Blob 脚本存储中删除除最后三个检索到的脚本项以外的其他所有脚本。

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    // Manage the size of your transcript storage.
    for (int i = 0; i < pageSize; i++)
    {
       // Remove older stored transcripts, save just the last three.
       if (i < pageSize - 3)
       {
          string thisTranscriptId = storedTranscripts[i];
          try
          {
             await _myTranscripts.DeleteTranscriptAsync("emulator", thisTranscriptId);
           }
           catch (System.Exception ex)
           {
              await turnContext.SendActivityAsync("Debug Out: DeleteTranscriptAsync had a problem!");
              await turnContext.SendActivityAsync("exception: " + ex.Message);
           }
       }
    }
    ...
}

```

以下链接提供了有关 [Azure Blob 脚本存储](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore)的详细信息 

## <a name="additional-information"></a>其他信息

### <a name="manage-concurrency-using-etags"></a>使用 eTag 管理并发
在机器人代码示例中，我们将每个 `IStoreItem` 的 `eTag` 属性设置为 `*`。 存储对象的 `eTag`（实体标记）成员在 Cosmos DB 中用于管理并发。 如果在机器人向存储中写入内容时，该机器人的另一实例更改了同一存储中的对象，`eTag` 会指示数据库应该如何处理。 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>最后一次写入才算 - 允许覆盖
星号 (`*`) 的 `eTag` 属性值指示最后一次写入才算数。 在创建新的数据存储时，可以将属性的 `eTag` 设置为 `*`，以表明以前未保存正在编写的数据，或者想要最后一次写入覆盖任何以前保存的属性。 如果并发性对于机器人来说不是问题，则对于任何要写入的数据，将 `eTag` 属性设置为 `*` 就可以允许覆盖。

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>保持并发并防止覆盖
将数据存储到 Cosmos DB 中时，若要防止对属性进行并发访问并避免覆盖另一个机器人实例的更改，可以对 `eTag` 使用 `*` 之外的值。 当机器人尝试保存状态数据但 `eTag` 的值与存储中 `eTag` 的值不同时，它会收到包含 `etag conflict key=` 消息的错误响应。 <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

默认情况下，每当机器人写入到该项时，Cosmos DB 存储都会检查存储对象的 `eTag` 属性是否等同，然后在每次写入之后将其更新为新的惟一值。 如果写入的 `eTag` 属性与存储中的 `eTag` 不匹配，这意味着另一个机器人或线程已更改数据。 

例如，假设你想让机器人编辑已保存的注释，但不希望机器人覆盖另一个机器人实例所做的更改。 如果另一个机器人实例已进行了编辑，你希望用户使用最新更新编辑版本。

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

首先，创建实现 `IStoreItem` 的类。

**EchoBot.cs**
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

接下来，通过创建存储对象创建初始注释，并将该对象添加到存储。

**EchoBot.cs**
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

**EchoBot.cs**
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


### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

请将一个帮助程序函数添加到机器人末尾，该机器人会将示例注释写入到数据存储中。
首先创建 `myNoteData` 对象。

**bot.js**
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

**bot.js**
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
     var list = changes["Note"].contents;
     await context.sendActivity(`Successful created a note: ${list}`);
} catch (err) {
     await context.sendActivity(`Could not create note: ${err}`);
}
```

可以通过添加以下调用，从机器人逻辑内部访问该帮助器函数：

**bot.js**
```javascript
// Save a note with etag.
await createSampleNote(storage, turnContext);
```

创建后，为了在以后检索和更新此注释，我们创建了另一个帮助器函数。当用户键入“update note”时，就可以访问该函数。

**bot.js**
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
             var list = note["Note"].contents;
             await context.sendActivity(`Successfully updated note: ${list}`);
        } catch (err) {            
             console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

可以通过添加以下调用，从机器人逻辑内部访问此帮助器函数：

**bot.js**
```javascript
// Update a note with etag.
await updateSampleNote(storage, turnContext);
```

如果在你尝试写回更改之前，另一个用户更新了存储中的注释，则 `eTag` 值将不再匹配，并且调用 `write` 将引发异常。

---

若要保持并发，始终从存储读取属性，然后修改读取的属性，以便保持 `eTag`。 如果从存储中读取用户数据，响应将包含 eTag 属性。 如果更改数据并将更新后的数据写入到存储中，请求应包含指定与之前读取值相同的值的 eTag 属性。 但是，编写将其 `eTag` 设置为 `*` 的对象将允许写入覆盖任何其他更改。

## <a name="next-steps"></a>后续步骤
现在你已经了解如何直接从存储中进行读写，接下来让我们来看看如何使用状态管理器来执行这些操作。

> [!div class="nextstepaction"]
> [使用会话和用户属性保存状态](bot-builder-howto-v4-state.md)

