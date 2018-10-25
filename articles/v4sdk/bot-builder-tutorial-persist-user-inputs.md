---
title: 保存用户数据 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK 将用户状态数据保存到存储区。
keywords: 保存用户数据, 存储, 聊天数据
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/19/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 36e8efefb276e5b9fb45ba6243b1b472476d5046
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997940"
---
# <a name="persist-user-data"></a>保存用户数据

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

当机器人要求用户进行输入时，你可能希望将某些信息保存到某种形式的存储中。 Bot Builder SDK 允许使用内存中存储或数据库存储（如 CosmosDB）来存储用户输入。 本地存储类型主要用于机器人的测试或原型制作过程。 但是，持久性存储类型（例如数据库存储）则最适合生产性机器人。

本主题介绍如何定义存储对象，以及如何将用户的输入内容保存到存储对象中，使其可持久保留。 我们将使用对话框向用户询问其名称（如果还没有该名称）。 无论你选择使用哪种存储类型，连接和保存数据的过程都是相同的。 本主题中的代码使用 `CosmosDB` 作为存储来持久保存数据。

## <a name="prerequisites"></a>先决条件

某些资源是必需的，具体取决于要使用的开发环境。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [安装 Visual Studio 2015 或更高版本](https://www.visualstudio.com/downloads/)。
* [安装 BotBuilder V4 模板](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [安装 Visual Studio Code](https://www.visualstudio.com/downloads/)。
* [安装 Node.js v8.5 或更高版本](https://nodejs.org/en/)。
* [安装 Yeoman](http://yeoman.io/)。
* 安装 Node.js v4 Botbuilder 模板生成器。

    ```shell
    npm install generator-botbuilder
    ```

---

本教程使用以下包。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

我们从一个基本的 EchoBot 模板着手。 有关说明，请参阅[适用于 .NET 的快速入门](~/dotnet/bot-builder-dotnet-quickstart.md)。

从 NuGet 包管理器安装这些额外的包。

* **Microsoft.Bot.Builder.Azure**
* **Microsoft.Bot.Builder.Dialogs**

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

我们从一个基本的 EchoBot 模板着手。 有关说明，请参阅[适用于 JavaScript 的快速入门](~/javascript/bot-builder-javascript-quickstart.md)。

安装这些额外的 npm 包。

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

若要测试在本教程中创建的机器人，需安装 [BotFramework Emulator](https://github.com/Microsoft/BotFramework-Emulator)。

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>创建 CosmosDB 服务并更新应用程序设置

若要设置 CosmosDB 服务和数据库，请按[使用 CosmosDB](bot-builder-howto-v4-storage.md#using-cosmos-db) 中的说明操作。 步骤归纳如下：

1. 在新浏览器窗口中，登录到 <a href="http://portal.azure.com/" target="_blank">Azure 门户</a>。
1. 单击“创建资源”>“数据库”>“Azure Cosmos DB”。
1. 在“新建帐户”页的“ID”字段中提供唯一的名称。 至于 **API**，请选择“SQL”，然后提供“订阅”、“位置”、“资源组”信息。
1. 单击“创建”。

然后，将集合添加到该服务，以便与此机器人配合使用。

记录用于添加集合的数据库 ID 和集合 ID，以及集合的密钥设置中的 URI 和主密钥，因为我们需要有这些才能将机器人连接到服务。

### <a name="update-your-application-settings"></a>更新应用程序设置

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

更新 **appsettings.json** 文件，使之包含 CosmosDB 的连接信息。

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在项目文件夹中找到 **.env** 文件，向其添加这些项以及特定于 Cosmos 的数据。

**.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

然后在机器人的主 **index.js** 文件中替换 `storage`，以便使用 `CosmosDbStorage` 而不是 `MemoryStorage`。 在运行时会拉入环境变量并用其填充这些字段。

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>创建存储、状态管理器和状态属性访问器对象

机器人使用状态管理和存储对象来管理并持久保存状态。 管理器提供一个抽象层，让你可以使用状态属性访问器来访问状态属性（不考虑基础存储的类型）。 使用状态管理器将数据写入存储中。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>为用户数据定义一个类

将文件 **CounterState.cs** 重命名为 **UserData.cs**，将 **CounterState** 类重命名为 **UserData**。

更新此类，以便用其保存要收集的数据。

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>为状态和状态属性访问器对象定义一个类

将文件 **EchoBotAccessors.cs** 重命名为 **BotAccessors.cs**，将 **EchoBotAccessors** 类重命名为 **BotAccessors**。

更新此类，以便存储机器人需要的状态对象和状态属性访问器。

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>更新机器人的启动代码

在 **Startup.cs** 文件中更新 using 语句。

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Integration;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Configuration;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
```

在 `ConfigureServices` 方法中，从创建后备存储对象的位置开始更新 AddBot 调用，然后注册机器人访问器对象。

我们需要 `DialogState` 对象的聊天状态才能跟踪对话框状态。 我们将为机器人要使用的对话框状态属性访问器和对话框集注册单一实例。 机器人将针对用户状态创建自己的状态属性访问器。

可以通过 `BotAccessors` 访问器有效地管理机器人的多个状态对象的存储。

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>更新服务器代码

在项目的 **index.js** 文件中，更新以下 require 语句。

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

将使用 `UserState` 来存储本教程的数据。 需创建新的 `userState` 对象并更新此行代码，以便将另一个参数传递到 `MainDialog` 类。

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

如果遇到常规错误，请清除聊天和用户状态。

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

另请更新 HTTP 服务器循环，以便调用机器人对象。

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>更新机器人逻辑

在 `MyBot` 类中，需要机器人运行所需的库。 在本教程中，我们将使用**对话框**库。

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

更新 `MyBot` 类的构造函数，接受另一个参数：`userState`。 另请更新用于定义本教程所需状态、对话框和提示的构造函数。 在此示例中，我们将定义一个双步瀑布框。其中，步骤 1 要求用户提供其名称，步骤 2 返回用户的响应。 将由机器人的主逻辑来持久保存该信息。

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

需要保存用户数据时，可以有一些选择。 此 SDK 提供一些具有不同范围的状态对象供你选择。 在这里，我们将使用聊天状态来管理对话框状态对象，使用用户状态来管理用户数据。

有关常规情况下存储和状态的详细信息，请参阅[存储](bot-builder-howto-v4-storage.md)和[如何管理聊天和用户状态](bot-builder-howto-v4-state.md)。

## <a name="create-a-greeting-dialog"></a>创建问候对话框

将使用对话框来收集用户的名称。 为了简化此方案，可以让对话框返回用户的名称，让机器人管理用户数据对象和状态。

创建一个 **GreetingsDialog** 类，使之包含以下代码。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

请查看上面的部分，可以看到对话框在 `MainDialog` 的构造函数中创建。

---

有关对话框的详细信息，请参阅[如何提示用户输入](bot-builder-prompts.md)和[如何管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>更新机器人，让其使用对话框和用户状态

我们将分别讨论机器人构造和用户输入管理。

### <a name="add-the-dialog-and-a-user-state-accessor"></a>添加对话框和用户状态访问器

需跟踪用户数据的对话框实例和状态属性访问器。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

添加初始化机器人所需的代码。

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

请查看上面的部分，可以看到这些状态访问器在 `MainDialog` 的构造函数中定义。

---

### <a name="update-the-turn-handler"></a>更新轮次处理程序

轮次处理程序会在用户首次加入聊天时问候用户，并在用户向机器人发送消息时进行响应。 不管什么时候，如果机器人还不知道用户的名称，它会启动问候对话框，要求用户提供其名称。 当对话框完成操作以后，我们会使用一个由状态属性访问器生成的状态对象将用户名称保存到用户状态。 该轮次结束时，访问器和状态管理器会将对对象所做的更改写入存储。

我们还会增加对“删除用户数据”活动的支持。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

更新机器人的 `OnTurnAsync` 方法。

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

更新机器人的 `onTurn` 处理程序。

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>在 Visual Studio 中启动机器人

生成并运行应用程序。

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。
2. 选择创建 Visual Studio 解决方案时所在目录中的 .bot 文件。

## <a name="interact-with-your-bot"></a>与机器人交互

向机器人发送消息，机器人将回复消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [管理对话和用户状态](bot-builder-howto-v4-state.md)
