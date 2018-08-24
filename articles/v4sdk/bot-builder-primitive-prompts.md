---
title: 使用原始提示管理聊天流 | Microsoft Docs
description: 了解如何在 Bot Builder SDK 中使用原始提示来管理聊天流。
keywords: 聊天流, 提示
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1514e1bcafc87be9e8449382bca7aa156e512ed9
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/20/2018
ms.locfileid: "40228340"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>使用自己的提示来提示用户输入
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人与用户之间的聊天通常涉及到请求（提示）用户输入信息、分析用户的响应，然后对该信息采取措施。 有关[使用“对话”库提示用户](bot-builder-prompts.md)的主题详细介绍了如何使用“对话”库来提示用户输入。 除此之外，“对话”库还会负责跟踪当前聊天，以及当前提出的问题。 它还提供方法用于请求不同类型的信息，例如文本、数字、日期和时间，等等。 

在某些情况下，内置的“对话”可能不是适合机器人的解决方案；“对话”可能会给简单的机器人带来过多的开销、过于严格，或者无法让机器人达到你自己的目的。 在这种情况下，可以跳过该库，并实现自己的提示逻辑。 本主题介绍如何设置基本的聊天机器人，以便可以使用自己的提示来管理聊天。

## <a name="track-prompt-states"></a>跟踪提示状态

在提示驱动的聊天中，需要跟踪你目前处于聊天的哪个位置，以及当前提出的问题。 在代码中，这相当于管理多个标志。 

例如，可以创建多个想要跟踪的属性。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

此处的提示信息中嵌入了用户个人资料类，因此可将所有这些属性统一存储在机器人[状态](bot-builder-howto-v4-state.md)中。

```csharp
// Where user information will be stored
public class UserProfile
{
    public string userName = null;
    public string workPlace = null;
    public int age = 0;
}

// Flags to keep track of our prompts, and the 
// user profile object for this conversation
public class PromptInformation
{
    public string topicTitle = null;
    public string prompt = null;
    public UserProfile userProfile = null;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

这些状态只会跟踪我们当前所处的主题和提示。 为了确保在部署到云后这些标志可按预期方式运行，我们将其存储在[聊天状态](bot-builder-howto-v4-state.md)中。 请将此代码放在主机器人逻辑中。

**app.js**
```javascript
// Define a topicStates object if it doesn't exist in the convoState.
if(!convo.topicStates){
    convo.topicStates = {
        "topicTitle": undefined, // Current conversation topic in progress
        "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
    };
}
```

---

## <a name="manage-a-topic-of-conversation"></a>管理聊天主题

在有序聊天（例如，从用户收集信息的聊天）中，需要知道用户何时进入了聊天，以及用户在聊天中所处的位置。 可以通过在主机器人轮次处理程序中设置并检查前面定义的提示标志，然后采取相应的措施，来跟踪此信息。 以下示例演示如何使用这些标志通过聊天来收集用户的个人资料信息。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Get our current state, as defined above
var convoState = context.GetConversationState<PromptInformation>();

if (convoState.userProfile == null)
{
    await context.SendActivity("Welcome new user, please fill out your profile information.");
    convoState.topicTitle = "profileTopic"; // Start the userProfile topic
    convoState.userProfile = new UserProfile();
}

// Start or continue a conversation if we are in one
if (convoState.topicTitle == "profileTopic")
{
    if (convoState.userProfile.userName == null && convoState.prompt == null)
    {
        convoState.prompt = "askName";
        await context.SendActivity("What is your name?");
    }
    else if (convoState.prompt == "askName")
    {
        // Save the user's response
        convoState.userProfile.userName = context.Activity.Text;

        // Ask next question
        convoState.prompt = "askAge";
        await context.SendActivity("How old are you?");
    }
    else if (convoState.prompt == "askAge")
    {
        // Save user's response
        if (!Int32.TryParse(context.Activity.Text, out convoState.userProfile.age))
        {
            // Failed to convert to int
            await context.SendActivity("Failed to convert string to int");
        }
        else
        {
            // Ask next question
            convoState.prompt = "workPlace";
            await context.SendActivity("Where do you work?");
        }

    }
    else if (convoState.prompt == "workPlace")
    {
        // Save user's response
        convoState.userProfile.workPlace = context.Activity.Text;

        // Done
        convoState.topicTitle = null; // Reset conversation flag
        convoState.prompt = null;     // Reset the prompt flag
        
        await context.SendActivity("Thank you. Your profile is complete.");
    }
    else // Somehow our flags got inappropriately set
    {
        await context.SendActivity("Looks like something went wrong, let's start over");
        convoState.userProfile = null;
        convoState.prompt = null;
        convoState.topicTitle = null;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);

        // Defined flags to manage the conversation flow and prompt states
        // convo.topicStates.topicTitle - Current conversation topic in progress
        // convo.topicStates.prompt - Current prompt state in progress - indicate what question is being asked.
        
        if (isMessage) {
            // Define a topicStates object if it doesn't exist in the convoState.
            if(!convo.topicStates){
                convo.topicStates = {
                    "topicTitle": undefined, // Current conversation topic in progress
                    "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
                };
            }
            
            // If user profile is not defined then define it.
            if(!convo.userProfile){
                
                await context.sendActivity(`Welcome new user, please fill out your profile information.`);
                convo.topicStates.topicTitle = "profileTopic"; // Start the userProfile topic
                convo.userProfile = { // Define the user's profile object
                    "userName": undefined,
                    "age": undefined,
                    "workPlace": undefined
                }; 
            }

            // Start or continue a conversation if we are in one
            if(convo.topicStates.topicTitle == "profileTopic"){
                if(!convo.userProfile.userName && !convo.topicStates.prompt){
                    convo.topicStates.prompt = "askName";
                    await context.sendActivity("What is your name?");
                }
                else if(convo.topicStates.prompt == "askName"){
                    // Save the user's response
                    convo.userProfile.userName = context.activity.text; 

                    // Ask next question
                    convo.topicStates.prompt = "askAge";
                    await context.sendActivity("How old are you?");
                }
                else if(convo.topicStates.prompt == "askAge"){
                    // Save user's response
                    convo.userProfile.age = context.activity.text;

                    // Ask next question
                    convo.topicStates.prompt = "workPlace";
                    await context.sendActivity("Where do you work?");
                }
                else if(convo.topicStates.prompt == "workPlace"){
                    // Save user's response
                    convo.userProfile.workPlace = context.activity.text;

                    // Done
                    convo.topicStates.topicTitle = undefined; // Reset conversation flag
                    convo.topicStates.prompt = undefined;     // Reset the prompt flag
                    await context.sendActivity("Thank you. Your profile is complete.");
                }
            }

            // Check for valid intents
            else if(context.activity.text && context.activity.text.match(/hi/ig)){
                await context.sendActivity(`Hi ${convo.userProfile.userName}.`);
            }
            else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

    });
});

```

---

以上示例代码检查内存中是否定义了用户的个人资料。 如果未定义并且这是一个新用户，则将标志自动设置为启动该主题。 然后，该示例演示如何通过将 `prompt` 标志设置为当前所提问题的值，来继续进行聊天。 如果此标志设置为正确的值，则条件子句将捕获用户在每个轮次的响应，并做相应的处理。 

最后完成信息请求后，必须重置这些标志，使机器人不会陷在循环中。 可以扩展此基本设置来管理机器人所需的更复杂聊天流：只需定义其他标志，或者根据用户输入将聊天分支即可。

## <a name="manage-multiple-topics-of-conversations"></a>管理多个聊天主题

能够处理一个聊天主题之后，可以扩展机器人逻辑来处理多个聊天主题。 如同单个聊天主题一样，只需检查触发特定主题的条件，然后采用相应的路径，即可处理多个主题。

在上述示例中，可将代码重构为允许其他函数和主题（例如预订餐桌或订餐）。 可以根据需要，将其他信息添加到用户个人资料或主题状态标志，以跟踪聊天。

帮助更好地管理多个聊天主题的方法之一是提供主菜单。 使用[建议的操作](bot-builder-howto-add-suggested-actions.md)，可让用户选择他们想要参与的主题，并深入到该主题。 另一种可能有帮助的做法是将代码拆分为独立的函数，以便更轻松地跟踪聊天流。

## <a name="validate-user-input"></a>验证用户输入

“对话”库提供内置的方法用于验证用户的输入，但我们也可以使用自己的提示来实现此目的。 例如，如果我们请求用户告知其年龄，则应确保得到一个数字，而不是类似于“Bob”这样的答复。

分析数字或日期和时间是一项复杂的任务，不在本主题的讨论范畴之内。 幸运的是，我们可以利用库。 若要正确分析此信息，可以使用 [Microsoft 文本识别器](https://github.com/Microsoft/Recognizers-Text)库。 此包通过 NuGet 提供，也可以从存储库中下载。 此外，它包含在“对话”库中。我们建议关注此库，但本文未使用它。

分析通用语言，或者有关日期、时间或电话号码等内容的复杂响应时，此库特别有用。 此示例只验证聚会订餐人数，但可以延用相同的思路来执行更深度的验证。 如果你不熟悉此库，请查看该站点上的内容，以了解其功能。

此示例只演示 `RecognizeNumber` 的用法。 有关如何使用该库中其他识别器方法的详细信息，请参阅该[存储库的文档](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要使用识别器库，请将其添加到 using 语句。

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq; // Used to get the first result from the recognizer
```

然后，定义实际执行验证的函数。

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(input, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        Int32.TryParse(result.First().Text, out int partySize);
        
        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivity("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用识别器库，需将其添加到 **app.js**：

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

然后，定义实际执行验证的函数。

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if(value < 6) {
            throw new Error(`Party size too small.`);
        }
        else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    }
    catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

在要验证的提示步骤中调用验证函数，然后转到下一提示。 如果验证失败，则再次提问。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (convoState.prompt == "partySize")
{
    if (await ValidatePartySize(context, context.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which 
        // is a new class we've added to our state
        convoState.ReservationInfo.partySize = context.Activity.Text;

        // Ask next question
        convoState.prompt = "reserveName";
        await context.SendActivity("Who's name will this be under?");
    }
    else
    {
        // Ask again
        await context.SendActivity("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if(convo.topicStates.prompt == "partySize"){
    if(await validatePartySize(context, context.activity.text)){
        // Save user's response
        convo.reservationInfo.partySize = context.activity.text;
        
        // Ask next question
        topicStates.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    }
    else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>后续步骤

掌握如何自行管理提示状态后，接下来让我们了解如何利用“对话”库来提示用户输入。

> [!div class="nextstepaction"]
> [使用“对话”提示用户输入](bot-builder-prompts.md)
