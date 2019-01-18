---
title: 创建自己的提示来收集用户输入 | Microsoft Docs
description: 了解如何在 Bot Framework SDK 中使用原始提示来管理聊天流。
keywords: 聊天流, 提示, 聊天状态, 用户状态, 自定义提示
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2e591f19f7df8fa6281573c0ac7f1330d95f4c53
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225432"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>创建自己的提示来收集用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人与用户之间的聊天通常涉及到请求（提示）用户输入信息、分析用户的响应，然后对该信息采取措施。

机器人应该跟踪聊天上下文，以便可以管理聊天行为并记住先前问题的回答。 机器人的状态是它为了正确响应传入消息而跟踪的信息。

## <a name="prerequisites"></a>先决条件

- 本文中的代码基于“提示用户输入”示例。 需要获取 [C# ](https://aka.ms/cs-primitive-prompt-sample) 或 [JS](https://aka.ms/js-primitive-prompt-sample) 示例的副本。
- 了解[管理状态](bot-builder-concept-state.md)以及如何[保存用户和聊天数据](bot-builder-howto-v4-state.md)。
- 用于在本地测试机器人的 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。

## <a name="about-the-sample-code"></a>关于示例代码

在本文中，我们将向用户提出一系列问题，验证他们的某些回答，然后保存其输入。
我们将使用机器人的轮次处理程序以及用户和聊天状态属性来管理聊天流与输入的收集。

1. 定义并配置状态
1. 使用状态属性来引导聊天
   1. 更新机器人的轮次处理程序。
   1. 实现一个帮助器方法以管理用户数据的收集。
   1. 针对用户输入实现验证方法。

## <a name="define-and-configure-state"></a>定义并配置状态

需将机器人配置为跟踪以下信息：

- 用户的姓名、年龄和所选日期：将在用户状态中定义。
- 刚刚向用户提了哪些问题：将在聊天状态中定义。

由于我们不打算部署此机器人，因此会将用户和聊天状态配置为使用内存存储。 下面介绍配置代码的一些重要方面。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

定义以下类型。

- 一个 `UserProfile` 类，用于跟踪机器人要收集的用户信息。
- 一个 `ConversationFlow` 类，用于跟踪有关我们在聊天中所处位置的信息。
- 一个内部 `ConversationFlow.Question` 枚举，用于跟踪我们在聊天中所处的位置。
- 一个 `CustomPromptBotAccessors` 类，将在其中捆绑状态管理信息。

机器人访问器类包含状态管理和状态属性访问器对象，它将通过 ASP.NET Core 中的依赖项注入传递给机器人。 在机器人中，我们将记录当机器人创建每个轮次时你会收到的状态属性访问器信息。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

创建状态管理对象，并在创建机器人时传入这些对象。
在机器人中，我们将定义状态属性的标识符用于跟踪我们在聊天中所处的位置，然后记录状态管理对象，并在机器人的构造函数中创建状态属性访问器。

---

## <a name="use-state-properties-to-direct-the-conversation"></a>使用状态属性来引导聊天

配置状态属性后，可以在机器人中使用它们。

- 定义[轮次处理程序](#the-bots-turn-handler)以访问状态并调用帮助器方法。
- 实现一个[帮助器方法](#filling-out-the-user-profile)以管理用户个人资料的收集。
- 实现[验证方法](#parse-and-validate-input)以分析和验证用户输入。

### <a name="the-bots-turn-handler"></a>机器人的轮次处理程序

使用状态属性访问器从轮次上下文获取状态属性。
如果需要填写用户个人资料，则调用帮助器方法，然后保存所有状态更改。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>填写用户个人资料

首先收集信息。 每个应用程序提供类似的界面。

- 返回值将指示输入是否为此问题的有效回答。
- 如果通过了验证，则生成可保存的已分析规范化值。
- 如果未通过验证，则生成一条消息，机器人可凭此再次请求提供信息。

 在[下一部分](#parse-and-validate-input)，我们将定义用于分析和验证用户输入的帮助器方法。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>分析和验证输入

我们将使用以下条件来验证输入。

- **姓名**必须为非空字符串。 我们将通过修剪空白字符来规范化此值。
- **年龄**必须介于 18 与 120 之间。 我们将通过返回整数来规范化此值。
- **日期**必须是将来至少一小时后的任何日期或时间。
  我们将通过只返回已分析输入的日期部分来规范化此值。

> [!NOTE]
> 对于年龄和日期输入，我们将使用 [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) 库来执行初始分析。
> 尽管我们提供了示例代码，但不会解释文本识别器库的工作原理，它只是分析输入的一种方式。
> 有关这些库的详细信息，请参阅存储库的**自述文件**。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

将以下验证方法添加到机器人。

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

将以下验证方法添加到机器人。

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>在本地测试机器人
1. 在计算机本地运行示例。 如需说明，请参阅 [C#](https://aka.ms/cs-primitive-prompt-sample) 或 [JS](https://aka.ms/js-primitive-prompt-sample) 示例的自述文件。
1. 按如下所示使用仿真器测试机器人。

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>其他资源

[对话库](bot-builder-concept-dialog.md)提供的某些类可以从多个方面将聊天管理自动化。 

## <a name="next-step"></a>后续步骤

> [!div class="nextstepaction"]
> [实现顺序聊天流](bot-builder-dialog-manage-conversation-flow.md)
