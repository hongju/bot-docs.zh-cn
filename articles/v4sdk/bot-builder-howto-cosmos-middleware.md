---
title: 使用 Cosmos DB 创建中间件 | Microsoft Docs
description: 了解如何创建登录 Cosmos DB 的自定义中间件。
keywords: 中间件, cosmos db, 数据库, azure, onTurn
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 29edff2db0e513a37b8ca8ac0f97e0647282567d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297941"
---
# <a name="create-middleware-that-logs-in-cosmos-db"></a>创建登录 Cosmos DB 的中间件

虽然 SDK 提供了有用的中间件，但在某些情况下，你需要实现自己的中间件才能实现所需目标。

在此示例中，我们将创建一个连接到 Cosmos DB 的中间件层，以记录所有收到的消息，以及我们发回的答复。 在此处看到的代码作为完整源代码随我们的[示例](../dotnet/bot-builder-dotnet-samples.md)一起提供。

## <a name="set-up"></a>设置

我们需要在编写代码之前进行一些设置。

### <a name="create-your-database"></a>创建数据库

首先，我们需要有地方存储数据库。  这可以通过 Azure 帐户完成，或者出于测试目的，可以使用 [Cosmos DB 模拟器](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)。 无论选择哪种方法，我们都需要数据库的终结点 URI 和密钥。

示例和此处提供的代码正在使用模拟器。 终结点和密钥是为 Cosmos DB 测试帐户提供的，它们是随模拟器文档提供的常用值，不能用于生产。

如果想使用实际的 Azure Cosmos DB，请按照 [Cosmos DB 入门](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started)主题的步骤 1 进行操作。 操作完成后，可以在数据库设置的“密钥”选项卡中找到终结点 URI 和密钥。 下面的配置文件将需要这些值。

### <a name="build-a-basic-bot"></a>生成基础机器人

本主题的其余部分构建了一个基础机器人，例如概述中构建的 HelloBot。 可以使用 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) 连接到机器人，与之会话并对其进行测试。

除了使用 Bot Framework 的标准机器人所需的引用之外，还需要添加对以下 NuGet 包的引用：

* Microsoft.Azure.Documentdb.Core
* System.Configuration.ConfigurationManager

这些库用于与数据库进行通信，并分别使用配置文件。

### <a name="add-configuration-file"></a>添加配置文件

出于多种原因，使用配置文件是一种很好的做法，因此我们将在此处进行使用。 我们的配置数据简短，但是，随着机器人变得更加复杂，你可以添加其他配置设置。

# <a name="ctabcs"></a>[C#](#tab/cs)
1. 将文件添加到名为 `app.config` 的项目中。
2. 将以下信息保存到其中。 终结点和密钥是为本地模拟器定义的，但可以针对 Cosmos DB 实例进行更新。
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="EmulatorDbUrl" value="https://localhost:8081"/>
            <add key="EmulatorDbKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="/>
        </appSettings>
    </configuration>
    ```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
1. 将文件添加到名为 `config.js` 的项目中。
2. 将以下信息保存到其中。 终结点和密钥是为本地模拟器定义的，但可以针对 Cosmos DB 实例进行更新。
    ```javascript
        var config = {}

        config.EmulatorServiceEndpoint = "https://localhost:8081";
        config.EmulatorAuthKey = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWE+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
        config.database = {
            "id": "Tasks"
        };
        config.collection = {
            "id": "Items"
        };

        module.exports = config;
    ```

---

如果使用的是实际的 Cosmos DB，则可以使用 Cosmos DB 设置中的值替换上述两个密钥的值。 或者，可以在配置中添加其他两个密钥，以便在用于测试的模拟器和实际的 Cosmos DB 之间切换，例如：

# <a name="ctabcs"></a>[C#](#tab/cs)
```
    <add key="ActualDbUrl" value="<your database URI>"/>
    <add key="ActualDbKey" value="<your database key>"/>
```
 如果确实添加了两个其他密钥并且想要使用实际数据库，请确保在下面的构造函数中指定正确的密钥，而不是 `EmulatorDbUrl` 和 `EmulatorDbKey`。

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```
    config.ActualServiceEndpoint = "your database URI;
    config.ActualAuthKey = "your database key";
```

如果确实添加了两个其他密钥并且想要使用实际数据库，请确保在下面的构造函数中指定正确的密钥，而不是 `EmulatorServiceEndpoint` 和 `EmulatorAuthKey`。

---



## <a name="creating-your-middleware"></a>创建中间件

# <a name="ctabcs"></a>[C#](#tab/cs)
在我们开始编写实际的中间件逻辑之前，请在机器人项目中为中间件创建一个新类。 该类创建完成后，将命名空间更改为用于其余机器人的 `Microsoft.Bot.Samples`。 在这里，你将看到我们添加了对一些新库的引用：

* `Newtonsoft.Json;`
* `Microsoft.Azure.Documents;`
* `Microsoft.Azure.Documents.Client;`
* `Microsoft.Azure.Documents.Linq;`

各种 `Microsoft.Azure.Documents.*` 用于与数据库通信的不同部件，`Newtonsoft.Json` 帮助我们解析在数据库来回传递的 JSON。

最后，每个中间件都实现 `IMiddleware`，要求我们为中间件定义适当的处理程序以使其正常工作。

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Configuration;
using Newtonsoft.Json;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

namespace Microsoft.Bot.Samples
{
    public class CosmosMiddleware : IMiddleware
    {
        ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
在开始编写实际的中间件逻辑之前，需要创建以 `onTurn()` 开头的自定义中间件。 

```javascript
adapter.use({onTurn: async (context, next) =>{
    // Middleware logic here...
}})
    
```

---

#### <a name="defining-local-variables"></a>定义局部变量

# <a name="ctabcs"></a>[C#](#tab/cs)
接下来，需要用于操纵数据库的局部变量和用于存储想要记录的信息的类。 该信息类（我们称之为 `Log`）定义将要命名的与每个成员关联的 JSON 属性。 我们将再进一步回过头来看看。

```cs
    private string Endpoint;
    private string Key;
    private static readonly string Database = "BotData";
    private static readonly string Collection = "BotCollection";
    public DocumentClient docClient;

    public class Log
    {
        [JsonProperty("Time")]
        public string Time;

        [JsonProperty("MessageReceived")]
        public string Message;

        [JsonProperty("ReplySent")]
        public string Reply;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
接下来，我们需要一个局部变量来存储想要记录的信息。 在中间件中，我们必须访问具有 Cosmos DB 作为存储提供程序的 conversationState。 变量 `info` 将是我们将要读取和写入的状态的对象。 

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(storage);
adapter.use(conversationState);

adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);
    // More middleware logic 
}})
```

--- 


#### <a name="initialize-database-connection"></a>初始化数据库连接

# <a name="ctabcs"></a>[C#](#tab/cs)
这个中间件的构造函数负责从上面定义的配置文件中提取必要的信息并创建 `DocumentClient`，这将允许我们从数据库进行读取和写入。 还要确保已设置数据库和集合。

正确使用新的 `DocumentClient` 需要对其进行处理，因此析构函数执行的就是该操作。

数据库的创建依赖于尝试先对数据库进行基本读取，然后读取该数据库中的集合。 如果遇到 `NotFound` 异常，我们会捕获并创建数据库或集合，而不是将其抛出堆栈。 在大多数情况下，这些已经创建，但是为了本示例方便，这使得我们不必担心在首次运行之前创建该数据库。 在示例代码中，为简单起见，数据库和集合分为两个函数，但它们已合并到下面的代码片段中。

有关 CosmosDB 的详细信息，请查看它们的[网站](https://docs.microsoft.com/en-us/azure/cosmos-db/)。 对于本主题的其余部分，我们将轻松了解 Cosmos DB 的详细信息，并专注于机器人使用它所需的内容。

`Key``Endpoint` 和 `docClient` 的定义包含在上面的代码片段中，为清楚起见，仅在此处进行复制。

```cs
    private string Endpoint;
    private string Key;
    // ...
    public DocumentClient docClient;
    // ...

    public CosmosMiddleware()
    {
        Endpoint = ConfigurationManager.AppSettings["EmulatorDbUrl"];
        Key = ConfigurationManager.AppSettings["EmulatorDbKey"];
        docClient = new DocumentClient(new Uri(Endpoint), Key);
        CreateDatabaseAndCollection().ConfigureAwait(false);
    }

    ~CosmosMiddleware()
    {
        docClient.Dispose();
    }

    private async Task CreateDatabaseAndCollection()
    {
        try
        {
            await docClient.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(Database));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDatabaseAsync(new Database { Id = Database });
            }
            else
            {
                throw;
            }
        }

        try
        {
            await docClient.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri
                (Database, Collection));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(Database),
                    new DocumentCollection { Id = Collection });
            }
            else
            {
                throw;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
数据库的创建依赖于尝试先对数据库进行基本读取，然后读取该数据库中的集合。 如果点击 `undefined`，我们捕获它并创建一个名为 `log` 的对象，该对象将被设置为空数组。 在大多数情况下，已经创建了 `log`，但是为了本示例方便，这使得我们不必担心要在首次运行之前创建该数据库。 在示例代码中，检查 `info.log` 是否未定义。 如果未定义，则将其设置为空数组。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    // More bot logic below
}})
```
---

#### <a name="read-from-database-logic"></a>从数据库逻辑中读取

最后一个帮助程序函数从数据库中读取并返回最新指定的记录数。 值得注意的是，有比我们此处使用的检索数据更好的数据库实践，特别是数据存储的规模相当大的时候。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task<string> ReadFromDatabase(int numberOfRecords)
    {
        var documents = docClient.CreateDocumentQuery<Log>(
                UriFactory.CreateDocumentCollectionUri(Database, Collection))
                .AsDocumentQuery();
        List<Log> messages = new List<Log>();
        while (documents.HasMoreResults)
        {
            messages.AddRange(await documents.ExecuteNextAsync<Log>());
        }

        // Create a sublist of messages containing the number of requested records.
        List<Log> messageSublist = messages.GetRange(messages.Count - numberOfRecords, numberOfRecords);

        string history = "";

        // Send the last 3 messages.
        foreach (Log logEntry in messageSublist)
        {
            history += ("Message was: " + logEntry.Message + " Reply was: " + logEntry.Reply + "  ");
        }

        return history;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

下面的代码仍在 `onTurn()` 函数中，如果用户输入字符串“history”，将调用该代码。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through the info.log array
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    }
    //...
}})

```

---

#### <a name="define-onturn"></a>定义 OnTurn()

然后，标准中间件方法 `OnTurn()` 处理剩下的工作。 我们只想在当前活动为消息时进行记录，为此，我们会在调用 `next()` 前后检查消息。

我们检查的第一件事是，这是否是用户要求提供最新历史记录的特殊案例消息。 如果是，我们从数据库调用读取来获取最新记录，并将其发送到会话。 由于这完全可处理当前活动，我们将管道短路而不是传递执行。

为了获得机器人发送的响应，我们会在每次收到消息时创建一个处理程序来获取这些响应，这可以在我们给 `OnSendActivity()` 的 lambda 中看到。 它构建一个字符串来收集通过 `SendActivity()` 发送给该上下文对象的所有消息。

一旦执行从 `next()` 返回管道，我们就会汇总日志数据并将其写入数据库。 

在向本机器人发送一些消息后查看 Cosmos DB 数据资源管理器，你应该在各个记录中看到数据。 在检查其中一条记录时，应该看到前三项是我们的日志数据的三个值，命名为我们为各自的 `JsonProperty` 指定的字符串。

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task OnTurn
        (ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        string botReply = "";

        if (context.Activity.Type == ActivityTypes.Message)
        {
            if (context.Activity.Text == "history")
            {
                // Read last 3 responses from the database, and short circuit future execution.
                await context.SendActivity(await ReadFromDatabase(3));
                return;
            }

            // Create a send activity handler to grab all response activities 
            // from the activity list.
            context.OnSendActivity(async (activityContext, activityList, activityNext) =>
            {
                foreach (Activity activity in activityList)
                {
                    botReply += (activity.Text + " ");
                }
                await activityNext();
            });
        }

        // Pass execution on to the next layer in the pipeline.
        await next();

        // Save logs for each conversational exchange only.
        if (context.Activity.Type == ActivityTypes.Message)
        {
            // Build a log object to write to the database.
            var logData = new Log
            {
                Time = DateTime.Now.ToString(),
                Message = context.Activity.Text,
                Reply = botReply
            };

            // Write our log to the database.
            try
            {
                var document = await docClient.CreateDocumentAsync(UriFactory.
                    CreateDocumentCollectionUri(Database, Collection), logData);
            }
            catch (Exception ex)
            {
                // More logic for what to do on a failed write can be added here
                throw ex;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

从我们上面描述的 `onTurn` 功能开始构建。

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through info.log 
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    } else if(isMessage){
        // Store the users response
        info.log.push(`Message was: ${context.activity.text}`); 

        await context.onSendActivities(async (handlerContext, activities, handlerNext) => 
        {
            // Store the bot's reply
            info.log.push(`Reply was: ${activities[0].text}`);
            
            await handlerNext(); 
        });
        await next();
    }
}})

```

---

## <a name="sample-output"></a>示例输出

在完整示例中，机器人使用提示库来询问用户的姓名和最喜欢的数字。 可以在[提示用户](~/v4sdk/bot-builder-prompts.md)主题上找到有关提示库的详细信息。

以下是我们运行上述代码的示例机器人的示例会话。

![Cosmos 中间件示例会话](./media/cosmos-middleware-conversation.png)

## <a name="additional-resources"></a>其他资源

* [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/)
* [Cosmos DB 模拟器](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)

