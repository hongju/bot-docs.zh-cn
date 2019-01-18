---
redirect_url: /bot-framework/bot-builder-prompts
ms.openlocfilehash: d45ec888a0082ee17718c93fc34a3df99431a254
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225412"
---
<a name="--"></a><!--
---
title:向用户提问 | Microsoft Docs description:了解如何使用瀑布模型要求用户在 Bot Framework SDK 中进行多次输入。
keywords: 瀑布, 对话框, 提问, 提示 author: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date:5/10/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="ask-the-user-questions"></a>向用户提问

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

其核心是围绕与用户的聊天构建机器人。 聊天可采用[多种形式](bot-builder-conversations.md)：可以是简短聊天或更复杂的聊天，可以是提问，也可以是解答。 聊天的形式取决于多个因素，但它们都与聊天有关。

本教程将指导你建立聊天，涵盖从提出简单问题到多回合的机器人等等内容。 我们的示例将主要讲解如何预订餐位，但你可设想一个机器人，它通过多轮聊天执行下单、回答常见问题和预约等各项任务。

交互式聊天机器人可答复用户的输入或向用户请求特定输入。 本教程将演示如何使用 `Prompts` 库（`Dialogs` 的一部分）向用户提问。 可将[对话](../bot-service-design-conversation-flow.md)视为定义聊天结构的容器，而要更深入了解对话中的提示，请参阅对话自带的[操作指南文章](bot-builder-prompts.md)。

## <a name="prerequisite"></a>先决条件

本教程中的代码将基于通过[入门](~/bot-service-quickstart.md)体验所创建的基本机器人。

## <a name="get-the-package"></a>获取包

# <a name="ctabcstab"></a>[C#](#tab/cstab)

从 Nuget 数据包管理器安装 Microsoft.Bot.Builder.Dialogs 包。

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
导航到机器人的项目文件夹并从 NPM 安装 `botbuilder-dialogs` 包：

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>将包导入机器人

# <a name="ctabcstab"></a>[C#](#tab/cstab)

在机器人代码中添加对话和提示的引用。

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

打开 app.js 文件并将 `botbuilder-dialogs` 库包含在机器人代码中。

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

这将让你有权访问要用于向用户提问的 `DialogSet` 和 `Prompts` 库。 `DialogSet` 只是对话的集合，其以瀑布模式进行构建。 这只表示一个对话紧接着一个对话。

## <a name="instantiate-a-dialogs-object"></a>实例化对话对象

实例化 `dialogs` 对象。 将使用此对话对象管理问答进程。

# <a name="ctabcstab"></a>[C#](#tab/cstab)
在 bot 类中声明一个成员变量，并在机器人的构造函数中将其初始化。 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>定义一个瀑布式对话

若要提问，将需要一个至少包含两个步骤的瀑布式对话。 在本例中，你将构造一个包含两个步骤的瀑布式对话，其中步骤一是询问用户的姓名，步骤二是用姓名问候用户。 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

修改机器人的构造函数，添加对话：
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

使用 `Prompts` 库附带的 `textPrompt` 方法提问。 `Prompts` 库提供了一组提示，它们让你能够向用户询问各种类型的信息。 有关其他提示类型的详细信息，请参阅[提示用户输入](~/v4sdk/bot-builder-prompts.md)。

要使提示有效，将需要使用 dialogId `textPrompt` 将提示添加到 `dialogs`，并使用 `TextPrompt()` 构造函数创建它。

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
用户回答问题后，可在步骤 2 的 `args` 参数中找到答复。

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
用户回答问题后，可在步骤 2 的 `results` 参数中找到答复。 在此情况下，`results` 分配到本地变量 `userName`。 在后面的教程中，我们将展示如何将用户输入的内容保存到所选的存储中。

---


你已定义 `dialogs` 进行提问，接下来需要调用对话来启动提示过程。

## <a name="start-the-dialog"></a>启动对话

# <a name="ctabcstab"></a>[C#](#tab/cstab)

修改机器人逻辑，使其如下所示：

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

机器人逻辑采用 `OnTurn()` 方法。 用户说“你好”之后，机器人将启动 `greetings` 对话。 `greetings` 对话的第一步是提示用户输入姓名。 用户将以消息形式回复他们的姓名，结果通过 `dc.Continue()` 方法发送到瀑布式对话的步骤 2。 根据你的定义，瀑布式对话的第二步将用姓名问候用户并结束对话。 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

修改基本机器人的 `processActivity()` 方法，使其如下所示：

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

机器人逻辑采用 `processActivity()` 方法。 用户说“你好”之后，机器人将启动 `greetings` 对话。 `greetings` 对话的第一步是提示用户输入姓名。 用户将以活动的 `text` 消息形式回复他们的姓名。 由于消息不符合任何预期目的，并且机器人尚未向用户发送任何答复，因此结果通过 `dc.continue()` 方法发送到瀑布式对话的步骤 2。 根据你的定义，瀑布式对话的第二步将用姓名问候用户并结束对话。 例如，如果步骤 2 未向用户发送问候消息，则 `processActivity` 方法将以发送给用户的默认消息结束。

---



## <a name="define-a-more-complex-waterfall-dialog"></a>定义一个更复杂的瀑布式对话

我们已经介绍了瀑布式对话的工作方式和构建方式，接下来让我们尝试构建一个以餐位预订为目的的更复杂的对话。

要管理餐位预订请求，你将需要定义具有 4 个步骤的瀑布式对话。 在此聊天中，你还将在 `TextPrompt` 之外使用 `DatetimePrompt` 和 `NumberPrompt`。



# <a name="ctabcstab"></a>[C#](#tab/cstab)

首先来使用回显机器人模板，并将机器人重命名为 CafeBot。 添加 `DialogSet` 和一些静态成员变量。

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

然后定义 `reserveTable` 对话。 可以在 bot 类构造函数中添加对话。
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

`reserveTable` 对话如下所示：

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

在 `reserveTable` 对话的聊天流，将通过瀑布式聊天的前 3 个步骤向用户提出 3 个问题。 第 4 步是处理最后一个问题的答案，并向用户发送预订确认。



# <a name="ctabcstab"></a>[C#](#tab/cstab)
`reserveTable` 对话的每个瀑布式步骤都使用提示来向用户询问信息。 以下代码用于将提示添加到对话集。

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

要使瀑布式对话有效，还需要将这些提示添加到 `dialogs` 对象：

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

现在，你已准备好将此消息连接到机器人逻辑中。

## <a name="start-the-dialog"></a>启动对话

# <a name="ctabcstab"></a>[C#](#tab/cstab)
修改机器人的 `OnTurn`，使其包含以下代码：
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


在 Startup.cs 中，更改 ConversationState 中间件的初始化，以使用派生自 `Dictionary<string,object>`（而非派生自 `EchoState`）的类。

例如，在 `Configure()` 中：
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

要捕获此请求的用户意向，请向 `processActivity()` 方法添加几行代码。 修改机器人的 `processActivity()` 方法，使其如下所示：

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

在执行时，每当用户发送包含字符串 `reserve table` 的消息，机器人都将启动 `reserveTable` 聊天。

---



## <a name="next-steps"></a>后续步骤

在本教程中，机器人将用户输入的内容保存到机器人中的变量。 如果要存储或保留此信息，需要将状态添加到中间件层。 下一个教程中将更深入地讲解如何保存用户状态数据。 

> [!div class="nextstepaction"]
> [保存用户数据](bot-builder-tutorial-persist-user-inputs.md)

-->
