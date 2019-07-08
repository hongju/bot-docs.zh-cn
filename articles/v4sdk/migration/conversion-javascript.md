---
title: 将现有的 v3 JavaScript 机器人迁移到新的 v4 项目 | Microsoft Docs
description: 使用新项目将现有的 v3 JavaScript 机器人迁移到 v4 SDK。
keywords: JavaScript, 机器人迁移, 对话, v3 机器人
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 573dabba7a16f88db890f0d095a2d4a0f983660c
ms.sourcegitcommit: 41c8caf0e0c849beeeb50cdccf6dbc1ba7cce442
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/24/2019
ms.locfileid: "67344594"
---
# <a name="migrate-a-sdk-v3-javascript-bot-to-v4"></a>将 SDK v3 Javascript 机器人迁移到 v4

在本文中，我们会将 v3 SDK JavaScript [core-MultiDialogs-v3](https://aka.ms/v3-js-core-multidialog-migration-sample) 机器人移植到新的 v4 JavaScript 机器人。
这种转换分为以下阶段：

1. 创建新项目并添加依赖项。
1. 更新入口点并定义常量。
1. 创建对话，使用 SDK v4 重新实现它们。
1. 更新运行对话所需的机器人代码。
1. 移植 **store.js** 实用程序文件。

此过程结束时，我们会有一个可正常运行的 v4 机器人。 示例存储库 [core-MultiDialogs-v4](https://aka.ms/v4-js-core-multidialog-migration-sample) 中也存在已转换机器人的副本。

Bot Framework SDK v4 与 SDK v3 基于相同的基础 REST API。 但是，SDK v4 对以前的 SDK 版本作了重构，使开发人员对其机器人拥有更高的灵活性和控制度。 该 SDK 的主要更改包括：

- 通过状态管理对象和属性访问器管理状态。
- 我们处理轮次的方式已变化，换句话说，机器人接收和响应来自用户通道的传入活动的方式已变化。
- v4 不使用 `session` 对象，而是使用“轮次上下文”  对象，后者包含有关传入活动的信息，可以用来将响应活动发送回去。
- 采用新的对话库，该库与 v3 中的库有很大的不同。 需要使用组件和瀑布对话，将旧对话转换为新的对话系统。

<!-- TODO
For more information about specific changes, see [differences between the v3 and v4 JavaScript SDK](???.md).
-->

> [!NOTE]
> 在迁移过程中，我们还清理了一些代码，但我们只会突出显示在迁移过程中对 v3 逻辑所做的更改。

## <a name="prerequisites"></a>先决条件

- Node.js
- Visual Studio Code
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

## <a name="about-this-bot"></a>关于此机器人

我们要迁移的机器人演示了如何使用多个对话来管理聊天流。 此机器人可以查找航班或酒店信息。

- 主对话会询问用户他们在查找哪类信息。
- 酒店对话会提示用户输入搜索参数，然后执行模拟搜索。
- 航班对话生成一个错误，机器人会以适当方式捕获该错误并对其进行处理。

## <a name="create-and-open-a-new-v4-bot-project"></a>创建并打开新的 v4 机器人项目

1. 需要一个 v4 项目，以便将机器人代码移植到其中。 若要在本地创建项目，请参阅[通过 Bot Framework SDK for JavaScript 创建机器人](../../javascript/bot-builder-javascript-quickstart.md)。

    > [!TIP]
    > 也可在 Azure 上创建项目，请参阅[使用 Azure 机器人服务创建机器人](../../bot-service-quickstart.md)。
    > 但是，这两种方法会导致支持文件出现轻微差异。 本文的 v4 项目是作为本地项目创建的。

1. 然后，在 Visual Studio Code 中打开项目。

## <a name="update-the-packagejson-file"></a>更新 package.json 文件

1. 添加 **botbuilder-dialogs** 包的依赖项，方法是在 Visual Studio Code 的终端窗口中输入 `npm i botbuilder-dialogs`。

1. 编辑 **./package.json**，根据需要更新 `name`、`version`、`description` 等属性。

## <a name="update-the-v4-app-entry-point"></a>更新 v4 应用入口点

v4 模板为应用入口点创建 **index.js** 文件，为特定于机器人的逻辑创建 **bot.js** 文件。 在后续步骤中，我们会将 **bot.js** 文件重命名为 **bots/reservationBot.js**，并为每个对话添加一个类。

编辑 **./index.js**，此文件是机器人应用的入口点。 这样就会包含 v3 **app.js** 文件中用于设置 HTTP 服务器的部分。

1. 除了 `BotFrameworkAdapter`，还请从 **botbuilder** 包导入 `MemoryStorage` 和 `ConversationState`。 另请导入机器人和主对话模块。 （我们会很快创建这些模块，但需要在此处引用它们。）

    ```javascript
    // Import required bot services.
    // See https://aka.ms/bot-services to learn more about the different parts of a bot.
    const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');

    // This bot's main dialog.
    const { MainDialog } = require('./dialogs/main')
    const { ReservationBot } = require('./bots/reservationBot');
    ```

1. 为适配器定义一个 `onTurnError` 处理程序。

    ```javascript
    // Catch-all for errors.
    adapter.onTurnError = async (context, error) => {
        const errorMsg = error.message ? error.message : `Oops. Something went wrong!`;
        // This check writes out errors to console log .vs. app insights.
        console.error(`\n [onTurnError]: ${ error }`);
        // Clear out state
        await conversationState.delete(context);
        // Send a message to the user
        await context.sendActivity(errorMsg);
    };
    ```

    在 v4 中，我们使用机器人适配器  将传入活动路由到机器人。 可以在某个轮次完成之前，通过适配器捕获并响应错误。 在这里，我们会在发生应用程序错误的情况下清除聊天状态，以便重置所有对话，防止机器人处于损坏的聊天状态下。

1. 将用于创建机器人的模板代码替换为以下代码。

    ```javascript
    // Define state store for your bot.
    const memoryStorage = new MemoryStorage();

    // Create conversation state with in-memory storage provider.
    const conversationState = new ConversationState(memoryStorage);

    // Create the base dialog and bot
    const dialog = new MainDialog();
    const reservationBot = new ReservationBot(conversationState, dialog);
    ```

    内存中存储层现在由 `MemoryStorage` 类提供，我们需要显式创建一个聊天状态管理对象。

    对话定义代码已移到一个我们会很快定义的 `MainDialog` 类。 我们还会将机器人定义代码迁移到 `ReservationBot` 类中。

1. 最后，我们更新服务器的请求处理程序，以便使用适配器将活动路由到机器人。

    ```javascript
    // Listen for incoming requests.
    server.post('/api/messages', (req, res) => {
        adapter.processActivity(req, res, async (context) => {
            // Route incoming activities to the bot.
            await reservationBot.run(context);
        });
    });
    ```

    在 v4 中，我们的机器人派生自 `ActivityHandler`，后者定义的 `run` 方法用于接收某个轮次的活动。

## <a name="add-a-constants-file"></a>添加常量文件

创建一个 **./const.js** 文件，用于保存机器人的标识符。

```javascript
module.exports = {
    MAIN_DIALOG: 'mainDialog',
    INITIAL_PROMPT: 'initialPrompt',
    HOTELS_DIALOG: 'hotelsDialog',
    INITIAL_HOTEL_PROMPT: 'initialHotelPrompt',
    CHECKIN_DATETIME_PROMPT: 'checkinTimePrompt',
    HOW_MANY_NIGHTS_PROMPT: 'howManyNightsPrompt',
    FLIGHTS_DIALOG: 'flightsDialog',
};
```

在 v4 中，ID 分配给对话和提示对象，而对话和提示则通过 ID 调用。

## <a name="create-new-dialog-files"></a>创建新的对话文件

创建以下文件：

| 文件名 | 说明 |
|:---|:---|
| **./dialogs/flights.js** | 此文件将包含 `hotels` 对话的已迁移逻辑。 |
| **./dialogs/hotels.js** | 此文件将包含 `flights` 对话的已迁移逻辑。 |
| **./dialogs/main.js** | 此文件将包含机器人的已迁移逻辑，并将替代根  对话。 |

我们尚未迁移支持对话。 有关如何在 v4 中实现帮助对话的示例，请参阅[处理用户中断](../bot-builder-howto-handle-user-interrupt.md?tabs=javascript)。

### <a name="implement-the-main-dialog"></a>实现主对话

在 v3 中，所有机器人都是在对话系统的基础上构建的。 在 v4 中，机器人逻辑和对话逻辑现在是分开的。 我们利用了 v3 机器人中  根对话的内容，创建了一个 `MainDialog` 类来代替它。

编辑 **./dialogs/main.js**。

1. 导入需要用于对话的类和常量。

    ```javascript
    const { DialogSet, DialogTurnStatus, ComponentDialog, WaterfallDialog,
        ChoicePrompt } = require('botbuilder-dialogs');
    const { FlightDialog } = require('./flights');
    const { HotelsDialog } = require('./hotels');
    const { MAIN_DIALOG,
        INITIAL_PROMPT,
        HOTELS_DIALOG,
        FLIGHTS_DIALOG
    } = require('../const');
    ```

1. 定义并导出 `MainDialog` 类。

    ```javascript
    const initialId = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor() {
            super(MAIN_DIALOG);

            // Create a dialog set for the bot. It requires a DialogState accessor, with which
            // to retrieve the dialog state from the turn context.
            this.addDialog(new ChoicePrompt(INITIAL_PROMPT, this.validateNumberOfAttempts.bind(this)));
            this.addDialog(new FlightDialog(FLIGHTS_DIALOG));

            // Define the steps of the base waterfall dialog and add it to the set.
            this.addDialog(new WaterfallDialog(initialId, [
                this.promptForBaseChoice.bind(this),
                this.respondToBaseChoice.bind(this)
            ]));

            // Define the steps of the hotels waterfall dialog and add it to the set.
            this.addDialog(new HotelsDialog(HOTELS_DIALOG));

            this.initialDialogId = initialId;
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

    这样就声明了可供主对话直接引用的其他对话和提示。

    - 主瀑布对话，包含此对话的步骤。 组件对话在启动时会启动其初始对话。 
    - 选项提示，用于询问用户要执行哪项任务。 我们创建了带验证程序的选项提示。
    - 两个子对话：航班和酒店。

1. 将 `run` 帮助程序方法添加到类。

    ```javascript
    /**
     * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
     * If no dialog is active, it will start the default dialog.
     * @param {*} turnContext
     * @param {*} accessor
     */
    async run(turnContext, accessor) {
        const dialogSet = new DialogSet(accessor);
        dialogSet.add(this);

        const dialogContext = await dialogSet.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        if (results.status === DialogTurnStatus.empty) {
            await dialogContext.beginDialog(this.id);
        }
    }
    ```

    在 v4 中，机器人与对话系统交互的方式是先创建对话上下文，然后调用 `continueDialog`。 如果有活动对话，则会将控制传递给该对话，否则此调用会直接返回。 结果为 `empty` 表明没有活动的对话，因此我们在这里再次启动主对话。

    `accessor` 参数传入对话状态属性的访问器。 对话堆栈的状态  存储在此属性中。 若要详细了解如何在 v4 中使用状态和对话，请分别参阅[管理状态](../bot-builder-concept-state.md)和[对话库](../bot-builder-concept-dialog.md)。

1. 向类添加主对话的瀑布步骤以及选项提示的验证程序。

    ```javascript
    async promptForBaseChoice(stepContext) {
        return await stepContext.prompt(
            INITIAL_PROMPT, {
                prompt: 'Are you looking for a flight or a hotel?',
                choices: ['Hotel', 'Flight'],
                retryPrompt: 'Not a valid option'
            }
        );
    }

    async respondToBaseChoice(stepContext) {
        // Retrieve the user input.
        const answer = stepContext.result.value;
        if (!answer) {
            // exhausted attempts and no selection, start over
            await stepContext.context.sendActivity('Not a valid option. We\'ll restart the dialog ' +
                'so you can try again!');
            return await stepContext.endDialog();
        }
        if (answer === 'Hotel') {
            return await stepContext.beginDialog(HOTELS_DIALOG);
        }
        if (answer === 'Flight') {
            return await stepContext.beginDialog(FLIGHTS_DIALOG);
        }
        return await stepContext.endDialog();
    }

    async validateNumberOfAttempts(promptContext) {
        if (promptContext.attemptCount > 3) {
            // cancel everything
            await promptContext.context.sendActivity('Oops! Too many attempts :( But don\'t worry, I\'m ' +
                'handling that exception and you can try again!');
            return await promptContext.context.endDialog();
        }

        if (!promptContext.recognized.succeeded) {
            await promptContext.context.sendActivity(promptContext.options.retryPrompt);
            return false;
        }
        return true;
    }
    ```

    瀑布的第一步是通过启动选项提示（本身是一个对话）要求用户进行选择。 瀑布的第二步使用选项提示的结果。 它会启动子对话（如果用户进行了选择）或结束主对话（如果用户无法进行选择）。

    选项提示会返回用户的选择（如果用户进行了有效的选择），或者会重新提示用户再次进行选择。 验证程序会检查已向用户连续提示了多少次，并在 3 次尝试失败后取消提示，让控制返回到主瀑布对话。

### <a name="implement-the-flights-dialog"></a>实现航班对话

在 v3 机器人中，航班对话是一个存根，用于演示机器人如何处理聊天错误。 在这里，我们执行相同的操作。

编辑 **./dialogs/flights.js**。

```javascript
const { ComponentDialog, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'flightsWaterfallDialog';

class FlightDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async () => {
                throw new Error('Flights Dialog is not implemented and is instead ' +
                    'being used to show Bot error handling');
            }
        ]));
    }
}

exports.FlightDialog = FlightDialog;
```

### <a name="implement-the-hotels-dialog"></a>实现酒店对话

我们保留酒店对话的同一个总体流程：要求提供目的地、要求提供日期、要求提供要呆的夜晚数，然后为用户显示一系列与其搜索匹配的选项。

编辑 **./dialogs/hotels.js**。

1. 导入需要用于对话的类和常量。

    ```javascript
    const { ComponentDialog, WaterfallDialog, TextPrompt, DateTimePrompt } = require('botbuilder-dialogs');
    const { AttachmentLayoutTypes, CardFactory } = require('botbuilder');
    const store = require('../store');
    const {
        INITIAL_HOTEL_PROMPT,
        CHECKIN_DATETIME_PROMPT,
        HOW_MANY_NIGHTS_PROMPT
    } = require('../const');
    ```

1. 定义并导出 `HotelsDialog` 类。

    ```javascript
    const initialId = 'hotelsWaterfallDialog';

    class HotelsDialog extends ComponentDialog {
        constructor(id) {
            super(id);

            // ID of the child dialog that should be started anytime the component is started.
            this.initialDialogId = initialId;

            // Register dialogs
            this.addDialog(new TextPrompt(INITIAL_HOTEL_PROMPT));
            this.addDialog(new DateTimePrompt(CHECKIN_DATETIME_PROMPT));
            this.addDialog(new TextPrompt(HOW_MANY_NIGHTS_PROMPT));

            // Define the conversation flow using a waterfall model.
            this.addDialog(new WaterfallDialog(initialId, [
                this.destinationPromptStep.bind(this),
                this.destinationSearchStep.bind(this),
                this.checkinPromptStep.bind(this),
                this.checkinTimeSetStep.bind(this),
                this.stayDurationPromptStep.bind(this),
                this.stayDurationSetStep.bind(this),
                this.hotelSearchStep.bind(this)
            ]));
        }
    }

    exports.HotelsDialog = HotelsDialog;
    ```

1. 向类添加几个帮助程序函数，这些函数将用在对话步骤中。

    ```javascript
    addDays(startDate, days) {
        const date = new Date(startDate);
        date.setDate(date.getDate() + days);
        return date;
    };

    createHotelHeroCard(hotel) {
        return CardFactory.heroCard(
            hotel.name,
            `${hotel.rating} stars. ${hotel.numberOfReviews} reviews. From ${hotel.priceStarting} per night.`,
            CardFactory.images([hotel.image]),
            CardFactory.actions([
                {
                    type: 'openUrl',
                    title: 'More details',
                    value: `https://www.bing.com/search?q=hotels+in+${encodeURIComponent(hotel.location)}`
                }
            ])
        );
    }
    ```

    `createHotelHeroCard` 创建一张英雄卡，包含酒店的相关信息。

1. 向类添加在对话中使用的瀑布步骤。

    ```javascript
    async destinationPromptStep(stepContext) {
        await stepContext.context.sendActivity('Welcome to the Hotels finder!');
        return await stepContext.prompt(
            INITIAL_HOTEL_PROMPT, {
                prompt: 'Please enter your destination'
            }
        );
    }

    async destinationSearchStep(stepContext) {
        const destination = stepContext.result;
        stepContext.values.destination = destination;
        await stepContext.context.sendActivity(`Looking for hotels in ${destination}`);
        return stepContext.next();
    }

    async checkinPromptStep(stepContext) {
        return await stepContext.prompt(
            CHECKIN_DATETIME_PROMPT, {
                prompt: 'When do you want to check in?'
            }
        );
    }

    async checkinTimeSetStep(stepContext) {
        const checkinTime = stepContext.result[0].value;
        stepContext.values.checkinTime = checkinTime;
        return stepContext.next();
    }

    async stayDurationPromptStep(stepContext) {
        return await stepContext.prompt(
            HOW_MANY_NIGHTS_PROMPT, {
                prompt: 'How many nights do you want to stay?'
            }
        );
    }

    async stayDurationSetStep(stepContext) {
        const numberOfNights = stepContext.result;
        stepContext.values.numberOfNights = parseInt(numberOfNights);
        return stepContext.next();
    }

    async hotelSearchStep(stepContext) {
        const destination = stepContext.values.destination;
        const checkIn = new Date(stepContext.values.checkinTime);
        const checkOut = this.addDays(checkIn, stepContext.values.numberOfNights);

        await stepContext.context.sendActivity(`Ok. Searching for Hotels in ${destination} from 
            ${checkIn.toDateString()} to ${checkOut.toDateString()}...`);
        const hotels = await store.searchHotels(destination, checkIn, checkOut);
        await stepContext.context.sendActivity(`I found in total ${hotels.length} hotels for your dates:`);

        const hotelHeroCards = hotels.map(this.createHotelHeroCard);

        await stepContext.context.sendActivity({
            attachments: hotelHeroCards,
            attachmentLayout: AttachmentLayoutTypes.Carousel
        });

        return await stepContext.endDialog();
    }
    ```

    我们已将 v3 酒店对话的步骤迁移到 v4 酒店对话的瀑布步骤中。

## <a name="update-the-bot"></a>更新机器人

在 v4 中，机器人可以响应对话系统外部的活动。 `ActivityHandler` 类为常见活动类型定义处理程序，方便管理代码。

将 **./bot.js** 重命名为 **./bots/reservationBot.js** 并对其进行编辑。

1. 该文件已导入 `ActivityHandler`，后者提供机器人的基实现。

    ```javascript
    const { ActivityHandler } = require('botbuilder');
    ```

1. 将类重命名为 `ReservationBot`。

    ```javascript
    class ReservationBot extends ActivityHandler {
        // ...
    }

    module.exports.ReservationBot = ReservationBot;
    ```

1. 更新构造函数的签名，使之接受我们将要接收的对象。

    ```javascript
    /**
     *
     * @param {ConversationState} conversationState
     * @param {Dialog} dialog
     * @param {any} logger object for logging events, defaults to console if none is provided
    */
    constructor(conversationState, dialog, logger) {
        super();
        // ...
    }
    ```

1. 在构造函数中添加 null 参数检查并定义类构造函数属性。

    ```javascript
    if (!conversationState) throw new Error('[DialogBot]: Missing parameter. conversationState is required');
    if (!dialog) throw new Error('[DialogBot]: Missing parameter. dialog is required');
    if (!logger) {
        logger = console;
        logger.log('[DialogBot]: logger not passed in, defaulting to console');
    }

    this.conversationState = conversationState;
    this.dialog = dialog;
    this.logger = logger;
    this.dialogState = this.conversationState.createProperty('DialogState');
    ```

    我们在其中创建对话状态属性访问器，用于存储对话堆栈的状态。

1. 在构造函数中更新 `onMessage` 处理程序并添加 `onDialog` 处理程序。

    ```javascript
    this.onMessage(async (context, next) => {
        this.logger.log('Running dialog with Message Activity.');

        // Run the Dialog with the new message Activity.
        await this.dialog.run(context, this.dialogState);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });

    this.onDialog(async (context, next) => {
        // Save any state changes. The load happened during the execution of the Dialog.
        await this.conversationState.saveChanges(context, false);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` 将消息活动路由到 `onMessage`。 此机器人通过对话处理所有用户输入。

    在将控制返回到适配器之前，`ActivityHandler` 会在轮次结束时调用 `onDialog`。 我们需要在退出轮次之前显式保存状态。 否则，状态更改不会得到保存，对话不会正常运行。

1. 最后，更新构造函数中的 `onMembersAdded` 处理程序。

    ```javascript
    this.onMembersAdded(async (context, next) => {
        const membersAdded = context.activity.membersAdded;
        for (let cnt = 0; cnt < membersAdded.length; ++cnt) {
            if (membersAdded[cnt].id !== context.activity.recipient.id) {
                await context.sendActivity('Hello and welcome to Contoso help desk bot.');
            }
        }
        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` 调用 `onMembersAdded` 的前提是收到一项聊天更新活动，该活动指示机器人之外的参与者已添加到聊天中。 我们更新了此方法，一有用户加入聊天就会发送问候消息。

## <a name="create-the-store-file"></a>创建存储文件

创建供酒店对话使用的 **./store.js** 文件。 `searchHotels` 是一个模拟酒店搜索函数，与 v3 机器人中的相同。

```javascript
module.exports = {
    searchHotels: destination => {
        return new Promise(resolve => {

            // Filling the hotels results manually just for demo purposes
            const hotels = [];
            for (let i = 1; i <= 5; i++) {
                hotels.push({
                    name: `${destination} Hotel ${i}`,
                    location: destination,
                    rating: Math.ceil(Math.random() * 5),
                    numberOfReviews: Math.floor(Math.random() * 5000) + 1,
                    priceStarting: Math.floor(Math.random() * 450) + 80,
                    image: `https://placeholdit.imgix.net/~text?txtsize=35&txt=Hotel${i}&w=500&h=260`
                });
            }

            hotels.sort((a, b) => a.priceStarting - b.priceStarting);

            // complete promise with a timer to simulate async response
            setTimeout(() => { resolve(hotels); }, 1000);
        });
    }
};
```

## <a name="test-the-bot-in-the-emulator"></a>在模拟器中测试机器人

此时，我们应该能够在本地运行机器人，并使用模拟器连接到该机器人。

1. 在计算机本地运行示例。
    如果在 Visual Studio Code 中启动调试会话，则当你测试机器人时，系统会将日志记录信息发送到调试控制台。
1. 启动模拟器并连接到机器人。
1. 发送消息，对主对话、航班对话和酒店对话进行测试。

## <a name="additional-resources"></a>其他资源

v4 概念主题：

- [机器人工作原理](../bot-builder-basics.md)
- [管理状态](../bot-builder-concept-state.md)
- [对话框库](../bot-builder-concept-dialog.md)

v4 操作指南主题：

- [发送和接收文本消息](../bot-builder-howto-send-messages.md)
- [保存用户和聊天数据](../bot-builder-howto-v4-state.md)
- [实现顺序聊天流](../bot-builder-dialog-manage-conversation-flow.md)
- [使用模拟器进行调试](../../bot-service-debug-emulator.md)
- [将遥测功能添加到机器人](../bot-builder-telemetry.md)
