---
title: 使用“对话”库提示用户输入 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用“对话”库提示用户输入。
keywords: 提示, 对话框, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 验证
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b238ed510fd1d6fda82734af373f344b0dc28e3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905361"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>使用“对话”库提示用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

通常机器人通过向用户提出问题收集其信息。 只需使用[轮次上下文](bot-builder-concept-activity-processing.md#turn-context)对象的 _send activity_ 方法来请求字符串输入，即可向用户发送标准消息；但是，Bot Builder SDK 提供了一个可用于请求不同类型的信息的“对话”库。 本主题详细介绍如何使用提示来请求用户输入。

本文介绍如何使用对话框中的提示。 有关使用对话的一般信息，请参阅[使用对话管理聊天流](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="prompt-types"></a>提示类型

对话框库提供多种不同类型的提示，每个提示请求不同类型的响应。

| Prompt | Description |
|:----|:----|
| **AttachmentPrompt** | 提示用户提供附件，例如文档或图像。 |
| **ChoicePrompt** | 提示用户从一组选项中进行选择。 |
| **ConfirmPrompt** | 提示用户确认其操作。 |
| **DatetimePrompt** | 提示用户输入日期时间。 用户可以使用自然语言响应，如“明天晚上 8 点”或“星期五上午 10 点”。 Bot Framework SDK 使用 LUIS `builtin.datetimeV2` 预生成实体。 有关详细信息，请参阅 [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2)。 |
| **NumberPrompt** | 提示用户输入数字。 用户可以使用“10”或“十”进行响应。 如果响应为“10”，例如，提示会将响应转换为一个数字并返回 `10` 作为结果。 |
| **TextPrompt** | 提示用户输入文本字符串。 |

## <a name="add-references-to-prompt-library"></a>添加对提示库的引用

可以通过向机器人添加对话框包获取对话框库。 [使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)中介绍了对话，本文将在提示中使用对话。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

从 NuGet 安装 **Microsoft.Bot.Builder.Dialogs** 包。

然后，在机器人代码中包含引用库。

```cs
using Microsoft.Bot.Builder.Dialogs;
```

可以在机器人代码文件中将对话框定义为类或将内联定义为属性。

本文中的代码是针对定义为类的对话框编写的。
下面的示例假定你将代码添加到对话框的构造函数。

对话框中的主要流是其步骤集合，需要提供一个 ID。 机器人可以使用此 ID 来检索对话框，因此它是将此公开为常量的好办法。

```cs
public class MyDialog : DialogSet
{
    public const string Name = "mainDialog";

    public MyDialog()
    {
        // Define your dialog's prompts and steps here.
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

通过 NPM 安装对话框包：

```cmd
npm install --save botbuilder-dialogs@preview
```

若要在机器人中使用对话框，请将其包含在机器人代码中。

在 app.js 文件中，添加以下内容。

```javascript
const {DialogSet} = require("botbuilder-dialogs");
const dialogs = new DialogSet();
```

---

## <a name="prompt-the-user"></a>提示用户

若要提示用户进行输入，可以向对话框添加提示。 例如，可以定义 TextPrompt 类型的提示，并为其提供 textPrompt 的对话框 ID：

添加提示对话框后，可以在简单的两步骤瀑布式对话框中使用它，或在多个步骤瀑布式对话框中同时使用多个提示。 瀑布式对话框只是定义一系列步骤的方法。 有关详细信息，请参阅[使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)中的[使用对话](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps)部分。

在第一轮中，对话框提示用户输入其名称，在第二轮中，对话框将用户输入作为提示答案处理。

例如，以下对话框提示用户输入其名称，然后按名称问候他们：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此外，还为对话框中使用的每个提示指定了一个名称，由对话框或机器人用于访问提示。 在所有这些示例中，我们将提示 ID 公开为常量。

对对话框上下文的 Prompt 或 End 方法的调用表示对话框步骤结束。 没有这些语句的情况下，该对话框将无法正常运行。

```csharp
/// <summary>Defines a simple greeting dialog that asks for the user's name.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            // Each step takes in a dialog context, arguments, and the next delegate.
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];
                await dc.Context.SendActivity($"Hi {user}!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {TextPrompt} = require("botbuilder-dialogs");
```

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('textPrompt', new TextPrompt());
dialogs.add('greetings', [
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.end();
    }
]);
```

---

> [!NOTE]
> 若要启动对话框，获取一个对话框的上下文，并使用其 begin 方法。 有关详细信息，请参阅[使用对话管理简单的聊天流](./bot-builder-dialog-manage-conversation-flow.md)。

## <a name="reusable-prompts"></a>可重用的提示

可以重复使用提示，以使用相同类型的提示要求不同信息。 例如，上面的示例代码定义文本提示，并使用它向用户询问其名称。 例如，如果需要，还可以使用相同的提示请求用户提供另一个文本字符串；例如，“您在哪里工作？”。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>Defines a simple greeting dialog that asks for the user's name and place of work.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];

                // Ask them where they work.
                await dc.Prompt(Inputs.Text, $"Hi {user}! Where do you work?");
            },
            async(dc, args, next) =>
            {
                var workplace = (string)args["Text"];

                await dc.Context.SendActivity($"{workplace} is a cool place!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);

        // Ask them where they work.
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, workPlace){
        await dc.context.sendActivity(`${workPlace} is a cool place!`);

        await dc.end();
    }
]);
```

---

但是，如果希望将提示与提示所请求的期望值配对，可以为每个提示提供一个唯一的 dialogId。 向对话框添加一个唯一 ID。 使用不同的 ID，还可以创建多个相同类型的提示对话框。 例如，可以为上面的示例创建两个 TextPrompt 对话框：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>The ID of the main dialog in the set.</summary>
public const string Name = "mainDialog";

/// <summary>Defines the IDs of the prompts in the set.</summary>
public struct Inputs
{
    /// <summary>The ID of the name prompt.</summary>
    public const string Name = "namePrompt";

    /// <summary>The ID of the work prompt.</summary>
    public const string Work = "workPrompt";
}

/// <summary>Defines the prompts and steps of the dialog.</summary>
public MyDialog()
{
    Add(Inputs.Name, new TextPrompt());
    Add(Inputs.Work, new TextPrompt());
    Add(Name, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the user's name.
            await dc.Prompt(Inputs.Name, "What is your name?");
        },
        async(dc, args, next) =>
        {
            var user = (string)args["Text"];

            // Ask them where they work.
            await dc.Prompt(Inputs.Work, $"Hi {user}! Where do you work?");
        },
        async(dc, args, next) =>
        {
            var workplace = (string)args["Text"];

            await dc.Context.SendActivity($"{workplace} is a cool place!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add('namePrompt', new TextPrompt());
dialogs.add('workPlacePrompt', new TextPrompt());
```

---

为了代码可重用性，定义单个 `textPrompt` 将适用于所有这些提示，因为它们要求提供一个文本字符串作为响应。 但是，在需要验证提示输入时，命名对话框才会变得简单易行。 在这种情况下，提示可能会使用 TextPrompt，但每个提示都在寻找一组不同的值。 让我们看看如何使用 `NumberPrompt` 来验证提示响应。

## <a name="specify-prompt-options"></a>指定提示选项

在对话框步骤内使用提示时，还可以提供提示选项，例如重新提示字符串。

当用户输入无法满足提示时，指定重新提示字符串很有用，因为它采用的是提示无法解析的格式（例如，用于数字提示的“明天”），或者是因为输入使验证条件失效。

> [!TIP]
> 创建数字提示时，需要指定它将使用的输入区域性。 数字提示可以解释各种输入，例如“十二”或“一个季度”以及“12”和“0.25”。 输入区域性可帮助提示更正确地解释用户输入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

输入区域性在其他库中定义。

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
```

下面的代码将向现有对话框集（即对话框）添加数字提示。

```csharp
dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
```

在对话框步骤中，下面的代码会提示用户输入，并在他们的输入不能解释为数字的情况下提供要使用的重新提示字符串。

```csharp
await dc.Prompt("numberPrompt", "How many people are in your party?", new PromptOptions()
{
    RetryPromptString = "Sorry, please specify the number of people in your party."
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {NumberPrompt} = require("botbuilder-dialogs");
```

```javascript
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

```javascript
dialogs.add('numberPrompt', new NumberPrompt());
```

---

具体而言，选择提示需要一些其他信息，即向用户提供的选择列表。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此示例使用以下命名空间中的类型。

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
```


当我们使用 ChoicePrompt 要求用户在一组选项之间进行选择，必须提供包含此组选项的提示，在 ChoicePromptOptions 对象中提供。 在这里，我们使用 ChoiceFactory 将选项列表转换为相应的格式。

我们还使用 SuggestedActions 活动作为重新提示，作为重新为用户提供输入选项的一种方法。


```csharp
/// <summary>Defines a dialog that asks for a choice of color.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the color prompt.</summary>
        public const string Color = "colorPrompt";
    }

    /// <summary>The available colors to choose from.</summary>
    public List<string> Colors = new List<string> { "Green", "Blue" };

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Color, new ChoicePrompt(Culture.English));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for a color. A choice prompt requires that you specify choice options.
                await dc.Prompt(Inputs.Color, "Please make a choice.", new ChoicePromptOptions()
                {
                    Choices = ChoiceFactory.ToChoices(Colors),
                    RetryPromptActivity =
                        MessageFactory.SuggestedActions(Colors, "Please choose a color.") as Activity
                });
            },
            async(dc, args, next) =>
            {
                var color = (FoundChoice)args["Value"];

                await dc.Context.SendActivity($"You chose {color.Value}.");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ChoicePrompt} = require("botbuilder-dialogs");
```

```javascript
dialogs.add('choicePrompt', new ChoicePrompt());
```

```javascript
// A choice prompt requires that you specify choice options.
const list = ['green', 'blue'];
await dc.prompt('choicePrompt', 'Please make a choice', list, {retryPrompt: 'Please choose a color.'});
```

---

## <a name="validate-a-prompt-response"></a>验证提示响应

在将有效值返回到瀑布式对话框的下一个步骤之前，可以验证提示响应。 例如，若要验证 NumberPrompt 是否在 6 和 20 之间的数字范围内，可以使用与以下类似的验证逻辑：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Defines a dialog that asks for the number of people in a party.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the party size prompt.</summary>
        public const string Size = "parytySize";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        // Include a validation function for the party size prompt.
        Add(Inputs.Size, new NumberPrompt<int>(Culture.English, async (context, result) =>
        {
            if (result.Value < 6 || result.Value > 20)
            {
                result.Status = PromptStatus.OutOfRange;
            }
        }));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the party size.
                await dc.Prompt(Inputs.Size, "How many people are in your party?", new PromptOptions()
                {
                    RetryPromptString = "Please specify party size between 6 and 20."
                });
            },
            async(dc, args, next) =>
            {
                var size = (int)args["Value"];

                await dc.Context.SendActivity($"Okay, {size} people!");
                await dc.End();
            }
        });
    }
}
```

验证还可以封装在其自己的私有方法中，并以此方式添加。

```cs
/// <summary>Validates input for the partySize prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task PartySizeValidator(ITurnContext context, Int32Result result)
{
    if (result.Value < 6 || result.Value > 20)
    {
        result.Status = PromptStatus.OutOfRange;
    }
}
```

在对话框中，指定要用于验证输入的方法。

```cs
// Include a validation function for the party size prompt.
Add(Inputs.Size, new NumberPrompt<int>(Culture.English, PartySizeValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt( async (context, value) => {
    try {
        if(value < 6 ){
            throw new Error('Party size too small.');
        }
        else if(value > 20){
            throw new Error('Party size too big.')
        }
        else {
            return value; // Return the valid value
        }
    } catch (err) {
        await context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
        return undefined;
    }
}));
```

---

同样，如果想要验证未来某个日期和时间的 DatetimePrompt 响应，可以使用与以下类似的验证逻辑：

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using DateTimeResult = Microsoft.Bot.Builder.Prompts.DateTimeResult;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Validates input for the reservationTime prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task TimeValidator(ITurnContext context, DateTimeResult result)
{
    if (result.Resolution.Count == 0)
    {
        await context.SendActivity("Sorry, I did not recognize the time that you entered.");
        result.Status = PromptStatus.NotRecognized;
    }

    // Find any recognized time that is not in the past.
    var now = DateTime.Now;
    DateTime time = default(DateTime);
    var resolution = result.Resolution.FirstOrDefault(
        res => DateTime.TryParse(res.Value, out time) && time > now);

    if (resolution != null)
    {
        // If found, keep only that result.
        result.Resolution.Clear();
        result.Resolution.Add(resolution);
    }
    else
    {
        // Otherwise, flag the input as out of range.
        await context.SendActivity("Please enter a time in the future, such as \"tomorrow at 9am\"");
        result.Status = PromptStatus.OutOfRange;
    }
}
```

```csharp
Add(Inputs.Time, new DateTimePrompt(Culture.English, TimeValidator));
```

有关更多示例，请参阅[示例存储库](https://github.com/Microsoft/botbuilder-dotnet)。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt( async (context, values) => {
    try {
        if (values.length < 0) { throw new Error('missing time') }
        if (values[0].type !== 'datetime') { throw new Error('unsupported type') }
        const value = new Date(values[0].value);
        if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }
        return value;
    } catch (err) {
        await context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
        return undefined;
    }
}));
```

有关更多示例，请参阅[示例存储库](https://github.com/Microsoft/botbuilder-js)。

---

> [!TIP]
> 如果用户提供的答案不明确，日期时间提示可以解析为几个不同的日期。 根据使用意图，你可能想要检查提示结果提供的所有解析，而不仅仅是第一个。

可以使用类似技术来验证任何提示类型的提示响应。

## <a name="save-user-data"></a>保存用户数据

提示用户输入时，可以通过几个选项来处理此输入。 例如，可以使用和放弃输入、可以将其保存到全局变量、可以将其保存到一个临时的或内存中存储容器、可以将其保存到文件中，或者可以将其保存到外部数据库。 有关如何保存用户数据的详细信息，请参阅[管理用户数据](bot-builder-howto-v4-state.md)。

## <a name="next-steps"></a>后续步骤

现已了解如何提示用户进行输入，我们可以通过对话框管理各种会话流来增强机器人代码和用户体验。


