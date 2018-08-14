---
title: 使用对话容器创建模块化机器人逻辑 | Microsoft Docs
description: 了解如何使用对话容器在 Bot Builder SDK for Node.js 和 C# 中模块化机器人逻辑。
keywords: 复合控件, 模块化机器人逻辑
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2441a32167618ebb08e6a43d68d74076c3351d8f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298378"
---
# <a name="create-modular-bot-logic-with-a-dialog-container"></a>使用对话容器创建模块化机器人逻辑

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

想象一下，你正在创建一个可处理多项任务的酒店机器人，例如问候用户、预订餐位、点餐、设置闹钟、显示当前天气等等。 你可使用一个对话对象在机器人中处理其中每项任务，但这可能会让对话代码变得过长并导致机器人的主要文件出现混乱。

你可通过模块化解决此问题。 模块化可简化代码并让调试变得更容易。 此外，你可将代码拆分为多个部分，允许多个团队同时在机器人中工作。 我们可创建管理多个聊天流的机器人，方法是使用对话容器将它们拆分为多个组件。 我们将创建一些基本聊天流，并展示如何使用对话容器将它们组合在一起。

在此示例中，我们将创建结合入住登记、叫醒和餐位预订模块的酒店机器人。

## <a name="managing-state"></a>管理状态

可使用多种方法为使用复合对话的机器人设置状态管理。 在此机器人中，我们展示其中一种方法。

有关状态管理的详细信息，请参阅[使用聊天和用户属性保存状态](bot-builder-howto-v4-state.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

每个对话都会收集一些信息，我们会将这些信息保存到用户状态。 我们将为每个对话定义一个类，并且我们会将这些类用作用户状态类中的属性。

```csharp
/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}

/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}
```

在一个机器人回合中，对话集的 `CreateContext` 方法会建立对话状态。
该方法使用回合上下文和状态对象作为参数。

对于对话，此状态对象必须实现 `IDictionary<string, object>` 接口。 由于此机器人仅使用聊天状态来存放对话状态，我们的聊天状态类可以是简单的字典。

```csharp
using System.Collections.Generic;

/// <summary>
/// Conversation state information.
/// We are also using this directly for dialog state, which needs an <see cref="IDictionary{string, object}"/> object.
/// </summary>
public class ConversationInfo : Dictionary<string, object> { }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

为了跟踪用户的输入，我们将从父对话传入 `userState` 对象作为对话参数。 在每个对话类中，对话将内置于构造函数中，因此可将信息保存到 `userState`。 在整个对话中，我们可以在用户输入信息时，写入到定义为 `dc.activeDialog.state` 对象属性的本地状态对象。 对话完成后，本地状态对象会被释放。 因此，我们会将本地状态对象保存到父 `userState`，该对象会跨你与用户的所有聊天保存有关用户的信息。 

---

## <a name="define-a-modular-check-in-dialog"></a>定义模块化登记入住对话

首先，我们从简单的登记入住对话入手，该对话将询问用户的姓名及其想要入住的房间。 为了模块化此任务，我们创建了一个扩展 `DialogContainer` 的 `CheckIn` 类。 此类有一个定义根对话名称的构造函数，该对话定义为拥有三个步骤的*瀑布*。 对话对象的签名和构造与标准瀑布完全相同。

**登记入住对话步骤**

1. 询问客人的姓名。
1. 询问他们想要入住的房间。
1. 发送确认消息并完成对话。

有关对话和瀑布的详细信息，请参阅[使用对话管理聊天流](bot-builder-dialog-manage-conversation-flow.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`CheckIn` 类有一个私有构造函数，该构造函数定义登记入住对话的步骤、创建单一实例，并在静态 `Instance` 属性中暴露它。

在整个对话中，我们可写入到本地状态对象，该对象可通过对话上下文属性 `dc.ActiveDialog.State` 进行访问。 对话完成时，本地状态对象会被释放。 因此，我们会将本地状态对象保存到机器人的 `userState`，该对象会跨你与用户的所有聊天保存有关用户的信息。

有关状态管理的详细信息，请参阅[使用聊天和用户属性保存状态](bot-builder-howto-v4-state.md)。 

**CheckIn.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;

namespace HotelBot
{
    public class CheckIn : DialogContainer
    {
        public const string Id = "checkIn";

        private const string GuestKey = nameof(CheckIn);

        public static CheckIn Instance { get; } = new CheckIn();

        // You can start this from the parent using the dialog's ID.
        private CheckIn() : base(Id)
        {
            // Define the conversation flow using a waterfall model.
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the guest information and prompt for the guest's name.
                    dc.ActiveDialog.State[GuestKey] = new GuestInfo();
                    await dc.Prompt("textPrompt", "What is your name?");
                },
                async (dc, args, next) =>
                {
                    // Save the name and prompt for the room number.
                    var name = args["Value"] as string;

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Name = name;

                    await dc.Prompt("numberPrompt",
                        $"Hi {name}. What room will you be staying in?");
                },
                async (dc, args, next) =>
                {
                    // Save the room number and "sign off".
                    var room = (string)args["Text"];

                    var guestInfo = dc.ActiveDialog.State[GuestKey];
                    ((GuestInfo)guestInfo).Room = room;

                    await dc.Context.SendActivity("Great, enjoy your stay!");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Guest = (GuestInfo)guestInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("textPrompt", new TextPrompt());
            this.Dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**checkIn.js**
```JavaScript
const { DialogContainer, DialogSet, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

class CheckIn extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'checkIn' will start when class is called in the parent
        super('checkIn');

        // Defining the conversation flow using a waterfall model
        this.dialogs.add('checkIn', [
            async function (dc) {
                // Create a new local guestInfo state object
                dc.activeDialog.state.guestInfo = {};
                await dc.context.sendActivity("What is your name?");
            },
            async function (dc, name){
                // Save the name 
                dc.activeDialog.state.guestInfo.userName = name;
                await dc.prompt('numberPrompt', `Hi ${name}. What room will you be staying in?`);
            },
            async function (dc, room){
                // Save the room number
                dc.activeDialog.state.guestInfo.room = room
                await dc.context.sendActivity(`Great! Enjoy your stay!`);

                // Save dialog's state object to the parent's state object
                const user = userState.get(dc.context);
                user.guestInfo = dc.activeDialog.state.guestInfo;
                await dc.end();
            }
        ]);
        // Defining the prompt used in this conversation flow
        this.dialogs.add('textPrompt', new TextPrompt());
        this.dialogs.add('numberPrompt', new NumberPrompt());
    }
}
exports.CheckIn = CheckIn;
```

---

## <a name="define-modular-reserve-table-and-wake-up-dialogs"></a>定义模块化餐位预订和叫醒对话

使用对话容器的好处之一是能够将对话组合在一起。 每个 `DialogSet` 都保留了 `dialogs` 的互斥集，因此无法轻松共享或交叉引用 `dialogs`。 这便是对话容器能够发挥作用的时候。 你可以使用对话容器创建复合对话，以便更轻松地跨对话管理对话流。 为了说明这一点，让我们另外创建两个对话：一个用于询问用户他们想要预订的餐位，另一个用于创建叫醒电话。 同样，我们将为每个对话使用单独的类，且每个对话都将扩展 `DialogContainer`。

**餐位预订对话步骤**

1. 询问想要预订的餐位。
1. 发送确认消息并完成对话。

**叫醒对话步骤**

1. 询问他们想要设定的叫醒时间。
1. 发送确认消息并完成对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**ReserveTable.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Linq;
using Choice = Microsoft.Bot.Builder.Prompts.Choices.Choice;
using FoundChoice = Microsoft.Bot.Builder.Prompts.Choices.FoundChoice;

namespace HotelBot
{
    public class ReserveTable : DialogContainer
    {
        public const string Id = "reserveTable";

        private const string TableKey = "Table";

        public static ReserveTable Instance { get; } = new ReserveTable();

        private ReserveTable() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the table information and prompt for the table number.
                    dc.ActiveDialog.State[TableKey] = new TableInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    var prompt = $"Welcome {guestInfo.Name}, " +
                        $"which table would you like to reserve?";
                    var choices = new string[] { "1", "2", "3", "4", "5", "6" };
                    await dc.Prompt("choicePrompt", prompt,
                        new ChoicePromptOptions
                        {
                            Choices = choices.Select(s => new Choice { Value = s }).ToList()
                        });
                },
                async (dc, args, next) =>
                {
                    // Save the table number and "sign off".
                    var table = (args["Value"] as FoundChoice).Value;

                    var tableInfo = dc.ActiveDialog.State[TableKey];
                    ((TableInfo)tableInfo).Number = table;

                    await dc.Context.SendActivity(
                        $"Sounds great; we will reserve table number {table} for you.");

                    // Save dialog state to user state and end the dialog.
                    var userState = UserState<UserInfo>.Get(dc.Context);
                    userState.Table = (TableInfo)tableInfo;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("choicePrompt", new ChoicePrompt(Culture.English));
        }
    }
}
```

**WakeUp.cs**
```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
using System.Linq;
using DateTimeResolution = Microsoft.Bot.Builder.Prompts.DateTimeResult.DateTimeResolution;

namespace HotelBot
{
    public class WakeUp : DialogContainer
    {
        public const string Id = "wakeUp";

        private const string WakeUpKey = "WakeUp";

        public static WakeUp Instance { get; } = new WakeUp();

        private WakeUp() : base(Id)
        {
            this.Dialogs.Add(Id, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    // Clear the wake up call information and prompt for the alarm time.
                   dc.ActiveDialog.State[WakeUpKey] = new WakeUpInfo();

                    var guestInfo = UserState<UserInfo>.Get(dc.Context).Guest;

                    await dc.Prompt("datePrompt", $"Hi {guestInfo.Name}, " +
                        $"what time would you like your alarm set for?");
                },
                async (dc, args, next) =>
                {
                    // Save the alarm time and "sign off".
                    var resolution = (List<DateTimeResolution>)args["Resolution"];

                    var wakeUp = dc.ActiveDialog.State[WakeUpKey];
                    ((WakeUpInfo)wakeUp).Time = resolution?.FirstOrDefault()?.Value;

                    var userState = UserState<UserInfo>.Get(dc.Context);
                    await dc.Context.SendActivity(
                        $"Your alarm is set to {((WakeUpInfo)wakeUp).Time}" +
                        $" for room {userState.Guest.Room}.");

                    // Save dialog state to user state and end the dialog.
                    userState.WakeUp = (WakeUpInfo)wakeUp;

                    await dc.End();
                }
            });

            // Define the prompts used in this conversation flow.
            this.Dialogs.Add("datePrompt", new DateTimePrompt(Culture.English));
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**reserveTable.js**
```JavaScript
const { DialogContainer, DialogSet, ChoicePrompt } = require('botbuilder-dialogs');

class ReserveTable extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'reserve_table' will start when class is called in the parent
        super('reserve_table'); 

        // Defining the conversation flow using a waterfall model
        this.dialogs.add('reserve_table', [
            async function (dc, args) {
                // Get the user state from context
                const user = userState.get(dc.context);

                // Create a new local reserveTable state object
                dc.activeDialog.state.reserveTable = {};

                const prompt = `Welcome ${user.guestInfo.userName}, which table would you like to reserve?`;
                const choices = ['1', '2', '3', '4', '5', '6'];
                await dc.prompt('choicePrompt', prompt, choices);
            },
            async function(dc, choice){
                // Save the table number
                dc.activeDialog.state.reserveTable.tableNumber = choice.value;
                await dc.context.sendActivity(`Sounds great, we will reserve table number ${choice.value} for you.`);
                
                // Get the user state from context
                const user = userState.get(dc.context);
                // Persist dialog's state object to the parent's state object
                user.reserveTable = dc.activeDialog.state.reserveTable;

                // End the dialog
                await dc.end();
            }
        ]);

        // Defining the prompt used in this conversation flow
        this.dialogs.add('choicePrompt', new ChoicePrompt());
    }
}
exports.ReserveTable = ReserveTable;
```

**wakeUp.js**
```JavaScript
const { DialogContainer, DialogSet, DatetimePrompt } = require('botbuilder-dialogs');

class WakeUp extends DialogContainer {
    constructor(userState) {
        // Dialog ID of 'wakeup' will start when class is called in the parent
        super('wakeUp');

        this.dialogs.add('wakeUp', [
            async function (dc, args) {
                // Get the user state from context
                const user = userState.get(dc.context); 

                // Create a new local reserveTable state object
                dc.activeDialog.state.wakeUp = {};  
                             
                await dc.prompt('datePrompt', `Hello, ${user.guestInfo.userName}. What time would you like your alarm to be set?`);
            },
            async function (dc, time){
                // Get the user state from context
                const user = userState.get(dc.context);

                // Save the time
                dc.activeDialog.state.wakeUp.time = time[0].value

                await dc.context.sendActivity(`Your alarm is set to ${time[0].value} for room ${user.guestInfo.room}`);
                
                // Save dialog's state object to the parent's state object
                user.wakeUp = dc.activeDialog.state.wakeUp;

                // End the dialog
                await dc.end();
            }]);

        // Defining the prompt used in this conversation flow
        this.dialogs.add('datePrompt', new DatetimePrompt());
    }
}
exports.WakeUp = WakeUp;
```

---

## <a name="add-modular-dialogs-to-a-bot"></a>向机器人添加模块化对话

机器人主要文件将这三个模块化对话绑定到一个机器人中。

- 全部三个对话都会添加到机器人的主要对话集中。
- 餐位预订和叫醒对话内置于主要对话的聊天流中。
- 新的聊天开始时，我们没有活动的对话，机器人的回合逻辑会发挥作用。

**机器人的回合处理程序**

每当机器人回合位于活动对话范围之外时，机器人就会检查用户状态。
1. 如果已经拥有客人的信息，则会启动主要对话。
1. 否则，它会启动主要对话的登记入住子对话。

**主要对话步骤**

1. 询问客人他们想要做什么：预订餐位或设置叫醒电话。
1. 启动相应的子对话，或发送_不理解输入_消息并跳到下一步。
1. 跳回此对话的开头部分。


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

使用对话上下文的 `Continue` 方法更新对话流。 此方法在对话堆栈中运行瀑布的下一步。

**HotelBot.cs**
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

namespace HotelBot
{
    public class HotelBot : IBot
    {
        private const string MainMenuId = "mainMenu";

        private DialogSet _dialogs { get; } = ComposeMainDialog();

        /// <summary>
        /// Every Conversation turn for our bot calls this method. 
        /// </summary>
        /// <param name="context">The current turn context.</param>        
        public async Task OnTurn(ITurnContext context)
        {
            if (context.Activity.Type is ActivityTypes.Message)
            {
                // Get the user and conversation state from the turn context.
                var userInfo = UserState<UserInfo>.Get(context);
                var conversationInfo = ConversationState<ConversationInfo>.Get(context);

                // Establish dialog state from the conversation state.
                var dc = _dialogs.CreateContext(context, conversationInfo);

                // Continue any current dialog.
                await dc.Continue();

                // Every turn sends a response, so if no response was sent,
                // then there no dialog is currently active.
                if (!context.Responded)
                {
                    // If we don't yet have the guest's info, start the check-in dialog.
                    if (string.IsNullOrEmpty(userInfo?.Guest?.Name))
                    {
                        await dc.Begin(CheckIn.Id);
                    }
                    // Otherwise, start our bot's main dialog.
                    else
                    {
                        await dc.Begin(MainMenuId);
                    }
                }
            }
        }

        /// <summary>
        /// Composes a main dialog for our bot.
        /// </summary>
        /// <returns>A new main dialog.</returns>
        private static DialogSet ComposeMainDialog()
        {
            var dialogs = new DialogSet();

            dialogs.Add(MainMenuId, new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    var menu = new List<string> { "Reserve Table", "Wake Up" };
                    await dc.Context.SendActivity(MessageFactory.SuggestedActions(menu, "How can I help you?"));
                },
                async (dc, args, next) =>
                {
                    // Decide which dialog to start.
                    var result = (args["Activity"] as Activity)?.Text?.Trim().ToLowerInvariant();
                    switch (result)
                    {
                        case "reserve table":
                            await dc.Begin(ReserveTable.Id);
                            break;
                        case "wake up":
                            await dc.Begin(WakeUp.Id);
                            break;
                        default:
                            await dc.Context.SendActivity("Sorry, I don't understand that command. Please choose an option from the list below.");
                            await next();
                            break;
                    }
                },
                async (dc, args, next) =>
                {
                    // Show the main menu again.
                    await dc.Replace(MainMenuId);
                }
            });

            // Add our child dialogs.
            dialogs.Add(CheckIn.Id, CheckIn.Instance);
            dialogs.Add(ReserveTable.Id, ReserveTable.Instance);
            dialogs.Add(WakeUp.Id, WakeUp.Instance);

            return dialogs;
        }
    }
}
```

最后，我们需要更新 `StartUp` 类的 `ConfigureServices` 方法来连接机器人并添加状态中间件。

**Startup.cs**
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(HotelBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // Add state management middleware for conversation and user state.
        var path = Path.Combine(Path.GetTempPath(), nameof(HotelBot));
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
        IStorage storage = new FileStorage(path);

        options.Middleware.Add(new ConversationState<ConversationInfo>(storage));
        options.Middleware.Add(new UserState<UserInfo>(storage));
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

使用对话上下文的 `continue` 方法更新对话流。 此方法在对话堆栈中运行瀑布的下一步。

**app.js**
```JavaScript
const {BotFrameworkAdapter, FileStorage, ConversationState, UserState, BotStateSet, MessageFactory} = require("botbuilder");
const {DialogSet} = require("botbuilder-dialogs");
const restify = require("restify");
var azure = require('botbuilder-azure'); 

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

//Memory Storage
const storage = new FileStorage("c:/temp");
// ConversationState lasts for the entirety of a conversation then gets disposed of
const convoState = new ConversationState(storage);

// UserState persists information about the user across all of the conversations you have with that user
const userState  = new UserState(storage);

adapter.use(new BotStateSet(convoState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // State will store all of your information 
        const convo = convoState.get(context);
        const user = userState.get(context); // userState will not be used in this example

        const dc = dialogs.createContext(context, convo);
        // Continue the current dialog if one is currently active
        await dc.continue(); 

        // Default action
        if (!context.responded && isMessage) {

            // Getting the user info from the state
            const userinfo = userState.get(dc.context); 
            // If guest info is undefined prompt the user to check in
            if(!userinfo.guestInfo){
                await dc.begin('checkInPrompt');
            }else{
                await dc.begin('mainMenu'); 
            }           
        }
    });
});

const dialogs = new DialogSet();
dialogs.add('mainMenu', [
    async function (dc, args) {
        const menu = ["Reserve Table", "Wake Up"];
        await dc.context.sendActivity(MessageFactory.suggestedActions(menu));    
    },
    async function (dc, result){
        // Decide which module to start
        switch(result){
            case "Reserve Table":
                await dc.begin('reservePrompt');
                break;
            case "Wake Up":
                await dc.begin('wakeUpPrompt');
                break;
            default:
                await dc.context.sendActivity("Sorry, i don't understand that command. Please choose an option from the list below.");
                break;            
        }
    },
    async function (dc, result){
        await dc.replace('mainMenu'); // Show the menu again
    }

]);

// Importing the dialogs 
const checkIn = require("./checkIn");
dialogs.add('checkInPrompt', new checkIn.CheckIn(userState));

const reserve_table = require("./reserveTable");
dialogs.add('reservePrompt', new reserve_table.ReserveTable(userState));

const wake_up = require("./wake_up");
dialogs.add('wakeUpPrompt', new wake_up.WakeUp(userState));
```

---
<!-- TODO: These dialogs are not fully modularized, as there are cross dependencies:
    - Importantly, the dialogs need to know details about the bot's user state.
-->

如你所见，模块化对话以类似于向对话添加[提示](bot-builder-prompts.md)的方式添加到机器人的主要对话中。 你可以根据需要向主要对话添加任意数量的子对话。 每个模块都会添加机器人可向用户提供的其他功能和服务。

## <a name="next-steps"></a>后续步骤

你已了解如何使用对话模块化机器人逻辑，现在让我们看看如何使用语言理解 (LUIS) 来帮助机器人决定何时开始对话。

> [!div class="nextstepaction"]
> [使用 LUIS 进行语言理解](./bot-builder-howto-v4-luis.md)
