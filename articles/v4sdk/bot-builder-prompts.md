---
title: 使用对话框库收集用户输入 | Microsoft Docs
description: 了解如何在 Bot Builder SDK 中使用对话框库提示用户输入。
keywords: 提示, 对话框, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, 重新提示, 验证
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 150d5f0a68d897ac278026a7cf36609aca05bb80
ms.sourcegitcommit: 984705927561cc8d6a84f811ff24c8c71b71c76b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/02/2018
ms.locfileid: "50965715"
---
# <a name="use-dialog-library-to-gather-user-input"></a>使用对话框库收集用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

通过发布问题来收集信息是机器人与用户交互的主要方式之一。 可以使用 [turn context](~/v4sdk/bot-builder-basics.md#defining-a-turn) 对象的 _send activity_ 方法直接这样操作，然后将下一个传入消息作为响应处理。 不过，Bot Builder SDK 提供一个[对话库](bot-builder-concept-dialog.md)，该库提供的方法旨在方便提问，并确保响应符合特定的数据类型或满足自定义验证规则。 本主题详细介绍如何使用提示对象来请求用户输入，以便达到此目的。

本文介绍如何创建提示并从对话内部调用它们。
有关如何不使用对话提示输入，请参阅[使用自己的提示来提示用户输入](bot-builder-primitive-prompts.md)。
有关如何使用对话的一般信息，请参阅[使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)。

## <a name="prompt-types"></a>提示类型

在幕后，提示是由两个步骤组成的对话。 首先，提示会请求输入；其次，它会返回有效值，或者使用重新提示从头开始。

对话库提供多种基本提示，每个提示用于收集不同类型的响应。

| Prompt | Description | 返回值 |
|:----|:----|:----|
| 附件提示 | 请求提供一个或多个附件，例如文档或图像。 | 附件对象的集合。 |
| 选项提示 | 请求从一组选项中选择一个选项。 | 找到的选项对象。 |
| 确认提示 | 请求确认。 | 布尔值。 |
| 日期时间提示 | 请求提供日期时间。 | 日期时间解析对象的集合。 |
| 数字提示 | 要求提供数字。 | 数字值。 |
| 文本提示 | 请求提供常规文本输入。 | 一个字符串。 |

该库还包含用于获取 OAuth 令牌的 OAuth 提示，代表用户访问另一应用程序时需要该令牌。 有关身份验证的详细信息，请参阅如何[将身份验证添加到机器人](bot-builder-authentication.md)。

基本提示可以解释解释自然语言输入，例如，“ten”或“a dozen”表示数字，“tomorrow”或“Friday at 10am”表示日期时间。

## <a name="using-prompts"></a>使用提示

仅当对话和提示位于同一对话集时，对话才能使用提示。

1. 为对话状态定义状态属性访问器。
1. 创建对话集
1. 创建提示并将其添加到对话集。
1. 创建使用提示的对话，并将其添加到对话集。
1. 在该对话中，添加对提示的调用并检索提示结果。

本文将介绍如何创建提示，以及如何从瀑布对话中调用它们。
有关对话的一般详细信息，请参阅[对话库](bot-builder-concept-dialog.md)一文。
有关使用对话和提示的完整机器人的介绍，请参阅如何[使用对话管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)。

可以在某个对话的多个步骤中，以及在同一个对话集的多个对话中，使用同一个提示。
但是，在初始化时，需将自定义验证关联到提示。
因此，如果需要对相同类型的提示使用不同的验证，则需要创建提示类型的多个实例，每个实例具有自身的验证代码。

### <a name="create-a-prompt"></a>创建提示

若要提示用户输入，请使用某个内置类（例如“文本提示”）定义一个提示，并将其添加到对话集。

* 提示具有固定的 ID。 （标识符必须在对话集中唯一。）
* 提示可以包含自定义验证程序。 （请参阅[自定义验证](#custom-validation)。）
* 对于某些提示，可以指定默认区域设置。

一般情况下，应在初始化机器人时创建提示和对话并将其添加到对话集。 然后，对话集可以在机器人从用户接收输入时解析提示的 ID。

例如，以下代码将创建两个文本提示，并将其添加到现有的对话集。 第二个文本提示引用了此处并未显示的验证方法。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处，`_dialogs` 包含现有对话集，`NameValidator` 是验证方法。

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处，`this.dialogs` 包含现有对话集，`NameValidator` 是验证函数。

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

使用区域设置来确定**选项**、**确认**、**日期时间**和**数字**提示的特定于语言的行为。 对于来自用户的任意给定输入：

* 如果通道在用户的消息中提供了 _locale_ 属性，则使用该属性。
* 否则，如果设置了提示的默认区域设置（通过在调用提示的构造函数时提供，或者在以后进行设置），则会使用该区域设置。
* 否则，将使用“英语”("en-us") 作为区域设置。

> [!NOTE]
> 区域设置是由 2、3 或 4 个字符组成的 ISO 639 代码，代表某种语言或语言系列。

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>从瀑布对话调用提示

添加提示后，可在瀑布对话的某个步骤中调用它，并在下一个对话步骤中获取提示结果。
若要从瀑布步骤内部调用提示，请调用瀑布步骤上下文对象的 _prompt_ 方法。 第一个参数是要使用的提示的 ID，第二个参数包含提示的选项，例如，用于请求用户输入的文本。

假设用户正在与机器人交互，该机器人具有活动的瀑布对话，并且该对话中的下一个步骤使用某个提示。

1. 当用户向机器人发送消息时，会执行以下操作：
   1. 机器人的轮次处理程序创建一个对话上下文，并调用其 _continue_ 方法。
   1. 控制权将传递给活动对话（在本例中为瀑布对话）中的下一个步骤。
   1. 该步骤调用该对话的瀑布步骤上下文的 _prompt_ 方法，以请求用户输入。
   1. 瀑布步骤上下文将提示推送到堆栈并将其启动。
   1. 提示向用户发送活动，以请求提供输入。
1. 当用户向机器人发送文本消息时，会执行以下操作：
   1. 机器人的轮次处理程序创建一个对话上下文，并调用其 _continue_ 方法。
   1. 控制权将传递给活动对话中的下一个步骤，这是提示的第二个轮次。
   1. 提示验证用户的输入。
      * 如果其输入无效，则重启提示，以便重新提示输入，在下一轮次会重复这一组步骤。
      * 否则，提示将会结束，并向父对话返回对话轮次结果对象。 控制权将传递给瀑布对话的下一个步骤，瀑布步骤上下文的 _result_ 属性中会提供提示结果。

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

当提示返回时，瀑布步骤上下文的 _result_ 属性将设置为提示的返回值。

此示例演示了两个连续瀑布步骤的组成部分。 第一个步骤使用提示来请求用户输入其姓名。 第二个步骤获取提示的返回值。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处，`name` 是文本提示的 ID，`NameStepAsync` 和 `GreetingStepAsync` 是瀑布对话的两个连续步骤委托。

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

此处，`name` 是文本提示的 ID，`nameStep` 和 `greetingStep` 是瀑布对话的两个连续步骤函数。

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>从机器人的轮次处理程序调用提示

可以使用对话上下文的 _prompt_ 方法，从轮次处理程序直接调用提示。
需要在下一个轮次调用对话上下文的 _continue dialog_ 方法并查看其返回值（对话轮次结果对象）。 有关如何执行此操作的示例，请参阅提示验证示例 ([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample))，或参阅如何[使用自己的提示来提示用户输入](bot-builder-primitive-prompts.md)（替代方法）。

## <a name="prompt-options"></a>提示选项

_prompt_ 方法的第二个参数采用提示选项对象，该对象包含以下属性。

| 属性 | Description |
| :--- | :--- |
| _prompt_ | 发送给用户的、以请求输入的初始活动。 |
| _retry prompt_ | 未验证用户的第一次输入时发送给用户的活动。 |
| _choices_ | 供用户选择的选项列表，与选项提示配合使用。 |

一般而言，prompt 和 retry prompt 属性属于活动，不过，在不同的编程语言中，这些属性的处理方式有一定的差异。

始终应该指定要向用户发送的初始提示活动。

当用户输入采用的是提示无法分析的格式（例如，用于数字提示的“tomorrow”），或者不符合验证条件，因而无法验证时，指定 retry prompt 很有用。 在这种情况下，如果未提供 retry prompt，则提示将使用初始提示活动来重新提示用户输入。

对于选项提示，应始终提供可用选项的列表。

此示例演示如何使用提供了所有三个属性的选项提示。 _favorite color_ 方法用作瀑布对话中的步骤，对话集包含瀑布对话以及 ID 为 `colorChoice` 的选项提示。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 JavaScript SDK 中，可为 `prompt` 和 `retryPrompt` 属性提供一个字符串。 提示会自动将这些信息转换为消息活动。

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

## <a name="custom-validation"></a>自定义验证

可以在将值返回到**瀑布**对话框的下一个步骤之前验证提示响应。 验证程序函数包含提示验证程序上下文参数，并返回一个布尔值用于指示输入是否通过了验证。

提示验证程序上下文包含以下属性：

| 属性 | Description |
| :--- | :--- |
| _上下文_ | 机器人的当前轮次上下文。 |
| _Recognized_ | 提示识别器结果，包含识别器处理的有关用户输入的信息。 |

提示识别器结果包含以下属性：

| 属性 | Description |
| :--- | :--- |
| 成功 | 指示识别器是否能够分析输入。 |
| _值_ | 识别器的返回值。 如果必要，验证代码可以修改此值。 |

### <a name="setup"></a>设置

添加验证代码之前需要完成少量的设置。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在机器人的 **.cs** 文件中，为预订信息定义一个内部类。

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

在 **BotAccessors.cs** 中，为预订数据添加一个状态属性访问器。

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

在 **Startup.cs** 中，更新 `ConfigureServices` 以设置访问器。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

无需对 JavaScript 的 HTTP 服务代码进行更改，可将 **index.js** 文件保留原样。

在 **bot.js** 中更新 require 语句，并添加状态属性访问器的标识符。

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

在机器人文件中，添加对话和提示的标识符。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>定义提示和对话

在机器人的构造函数代码中，创建对话集、添加提示，然后添加预订对话。
在创建提示时包含自定义验证。 接下来，我们将实现验证函数。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>实现验证代码

实现群体大小验证程序。 我们将预订限制为 6 到 20 个人的群体。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the party size is appropriate to make a reservation.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations can be made for groups of 6 to 20 people.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
private async Task<bool> PartySizeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

日期时间提示返回与用户输入匹配的可能日期时间解析列表或数组。 例如，9:00 可能表示 9 AM 或 9 PM，而 Sunday 也可能有歧义。 此外，日期时间解析可以表示日期、时间、日期时间或范围。 日期时间提示使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 分析用户输入。

实现预订日期验证程序。 我们将预订限制为当前时间后的一小时或以上。 我们将保留与条件匹配的第一个解析，并清除剩余的解析。

此验证代码并不详尽。 它最适合用于分析成日期和时间的输入。 此代码演示了一些用于验证日期时间提示的选项，你的实现将取决于尝试从用户收集的信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the reservation date is appropriate.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations must be made at least an hour in advance.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);
    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
// Check whether the input could be recognized as an integer.
if (!promptContext.recognized.succeeded) {
    await promptContext.context.sendActivity(
        "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
    return false;
}

// Check whether any of the recognized date-times are appropriate,
// and if so, return the first appropriate date-time.
const earliest = Date.now() + (60 * 60 * 1000);
let value = null;
promptContext.recognized.value.forEach(candidate => {
    // TODO: update validation to account for time vs date vs date-time vs range.
    const time = new Date(candidate.value || candidate.start);
    if (earliest < time.getTime()) {
        value = candidate;
    }
});
if (value) {
    promptContext.recognized.value = [value];
    return true;
}

await promptContext.context.sendActivity(
    "I'm sorry, we can't take reservations earlier than an hour from now.");
return false;
}
```

---

### <a name="implement-the-dialog-steps"></a>实现对话步骤

使用已添加到对话集的提示。 在机器人的构造函数中创建提示时，我们已在这些提示中添加了验证。 当提示首次请求用户输入时，它会从提供的选项中发送 _prompt_ 活动。 如果验证失败，则发送 _retry prompt_ 活动，以请求用户提供不同的输入。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>First step of the main dialog: prompt for party size.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

/// <summary>Second step of the main dialog: record the party size and prompt for the
/// reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

/// <summary>Third step of the main dialog: return the collected party size and reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>更新轮次处理程序

更新机器人的轮次处理程序，以启动对话，并在对话完成时接受对话的返回值。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        default:
            break;
    }
}
```

---

有关更多示例，请参阅[示例存储库](https://aka.ms/bot-samples-readme)。

可以使用类似技术来验证任何提示类型的提示响应。

## <a name="handling-prompt-results"></a>处理提示结果

如何处理提示结果取决于从用户请求信息的原因。 选项包括：

* 使用信息来控制对话流（例如，在用户对确认或选项提示做出响应时）。
* 在对话的状态中缓存信息（例如，在瀑布步骤上下文的 _values_ 属性中设置一个值），然后在对话结束时返回收集的信息。
* 将信息保存到机器人状态。 这需要将对话设计为有权访问机器人的状态属性访问器。

请参阅其他资源中涵盖了这些方案的主题和示例。

## <a name="additional-resources"></a>其他资源

* [管理简单的聊天流](bot-builder-dialog-manage-conversation-flow.md)
* [管理复杂的聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)
* [创建一组集成的对话](bot-builder-compositcontrol.md)
* [持久保存对话框中的数据](bot-builder-tutorial-persist-user-inputs.md)
* **多轮次提示**示例 ([C#](https://aka.ms/cs-multi-prompts-sample) | [JS](https://aka.ms/js-multi-prompts-sample))

## <a name="next-steps"></a>后续步骤

现已了解如何提示用户进行输入，我们可以通过对话框管理各种会话流来增强机器人代码和用户体验。

> [!div class="nextstepaction"]
> [管理复杂的聊天流](bot-builder-dialog-manage-complex-conversation-flow.md)
