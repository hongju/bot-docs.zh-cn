---
title: 提取类型化 LUIS 结果 | Microsoft Docs
description: 了解如何使用 LUIS 通过 Bot Builder SDK 提取实体。
keywords: 意向, 实体, LUISGen, 提取
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e1817179d8459ace444f669791d8a302fb6fb5c9
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999774"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>使用 LUISGen 提取意向和实体

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

除了识别意向外，LUIS 应用还可以提取实体，这些实体是实现用户请求的重要词汇。 例如，在餐馆预订的示例中，LUIS 应用可能能够从用户的消息中提取聚会大小、预订日期或餐馆位置。 


可以使用 [LUISGen 工具](https://aka.ms/botbuilder-tools-luisgen)生成类，以便更轻松地从机器人代码的 LUIS 中提取实体。

在 Node.js 命令行中，将 `luisgen` 安装到全局路径。
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="generate-a-luis-results-class"></a>生成 LUIS 结果类

下载 [CafeBot LUIS 示例](https://aka.ms/contosocafebot-luis)，并在其根文件夹中运行 LUISGen：

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>检查生成的代码
这将生成 **cafeLUISModel.cs**，可以将其添加到项目。 它提供了 `cafeLuisModel` 类，用于从 LUIS 获取强类型结果。

此类包含用于获取 LUIS 应用中定义的意向的枚举。
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
它还有 `Entities` 属性。 由于用户消息中可能多次出现某个实体，因此 `_Entities` 类为每种类型的实体定义一个数组。 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.Luis.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.Luis.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> 所有实体类型都是数组，因为 LUIS 可能在用户的话语中检测到多个指定类型的实体。 例如，如果用户说“明天下午 5 点和下周六晚上 9 点预订”，则 `datetime` 结果中将返回“明天下午 5 点”和“下周六晚上 9 点”。
>

|实体 | 类型 | 示例 | 说明 |
|-------|-----|------|---|
|partySize| string[]| `four` 人的聚会| 简单实体会识别字符串。 在此示例中，Entities.partySize[0] 是 `"four"`。
|datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| 在 `9pm tomorrow` 预订| 每个 **DateTimeSpec** 对象都有一个 timex 字段，其中包含以 **timex** 格式指定的可能时间值。 有关 timex 的更多信息，请访问： http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3；有关进行识别的库的更多信息，请访问： https://github.com/Microsoft/Recognizers-Text
|数字| double[]| `four` 人的聚会，其中包括 `2` 个孩子 | `number` 会识别所有数字，而不仅仅是聚会的大小。 <br/> 在“4 人的聚会，其中包括 2 个孩子”这句话中，`Entities.number[0]` 是 4，`Entities.number[1]` 是2。
|cafelocation| string[][] | 在 `Seattle` 位置预订。| cafeLocation 是一个列表实体，这意味着它包含已识别的成员列表。 如果已识别的实体是多个列表的成员，则它是数组的数组。 例如，“在华盛顿预订”可以对应于华盛顿州和华盛顿特区的列表

`_Instance` 属性提供 [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha)，以便用户获取有关每个已识别实体的更多详细信息。

## <a name="check-intents-in-your-bot"></a>在机器人中检查意向
在 **CafeBot.cs** 中，查看 `OnTurn` 内的代码。 你可以看到机器人调用 LUIS 的位置，并检查意向以决定开始哪个对话。 调用 [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) 的 LUIS 结果将作为参数传递给 `BookTable` 对话。



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>在对话中提取实体

现在来看看 `Dialogs/BookTable.cs`。 `BookTable` 对话包含一系列瀑布式步骤，每个步骤都检查传递给 `args` 的 LUIS 结果中的实体。 如果找不到实体，对话将通过调用 `next()` 跳过提示。 如果找到实体，则对话会提示它，并在下一个瀑布式步骤中收到用户对提示的回答。

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>运行示例

在 Visual Studio 2017 中打开 `ContosoCafeBot.sln`，并运行机器人。 使用 [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) 连接到示例机器人。

在模拟器中，说“`reserve a table`”以启动预订对话。

![运行机器人](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

下载 [CafeBot_LUIS 示例](https://aka.ms/contosocafebot-typescript-luis-dialogs)，并在其根文件夹中运行 LUISGen：

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

这将生成 **CafeLUISModel.ts**，可以将其添加到项目。 可以使用生成的文件中的类型从 LUIS 识别器获取类型化结果。


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>将类型化结果传递给对话

检查 **luisbot.ts** 中的代码。 在 `processActivity` 处理程序中，机器人将类型化结果传递给对话。

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>检查对话中的现有实体

在 **luisbot.ts** 中，`reserveTable` 对话调用 `SaveEntities` 帮助程序函数来检查 LUIS 应用检测到的实体。 如果找到实体，它们将保存到对话状态。 对话中的每个瀑布式步骤都会检查实体是否已保存到对话状态，如果没有，则会提示它。

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

`SaveEntities` 帮助程序函数会检查是否存在 `datetime` 和 `partysize` 实体。 `datetime` 实体是[预生成实体](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)。

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>运行示例

1. 如果未安装 TypeScript 编译器，请使用以下命令安装它：

```
npm install --global typescript
```

2. 在运行机器人之前，通过在示例的根目录中运行 `npm install` 安装依赖项：

```
npm install
```

3. 从根目录中使用 `tsc` 生成示例。 这将生成 `luisbot.js`。

4. 在 `lib` 目录中运行 `luisbot.js`。

5. 使用 [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator) 运行示例。

6. 在模拟器中，说“`reserve a table`”以启动预订对话。

![运行机器人](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>其他资源

有关 LUIS 的更多背景信息，请参阅[语言理解](./bot-builder-concept-luis.md)。


## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 Dispatch 工具组合 LUIS 和 QnA](./bot-builder-tutorial-dispatch.md)


