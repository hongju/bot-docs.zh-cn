---
title: 使用 .NET 构建 Cortana 技能 | Microsoft Docs
description: 了解在 Bot Framework SDK for .NET 中构建 Cortana 技能的核心概念。
keywords: Bot Framework, Cortana 技能, 语音, .NET, SDK, 关键概念, 核心概念
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 98fc10a806a4c8d1a4d6563934d92b0e0cdbb771
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224772"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>使用 Cortana 技能构建支持语音的机器人

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)


使用 Bot Framework SDK for .NET 可以通过将支持语音的机器人作为 Cortana 技能连接到 Cortana 通道，来构建该机器人。 


> [!TIP]
> 有关技能的概念及其作用的详细信息，请参阅 [Cortana 技能套件][CortanaGetStarted]。

使用 Bot Framework 创建 Cortana 技能只需少量的 Cortana 特定知识，主要包括构建机器人。 与以前创建的其他机器人相比，Cortana 的一个主要差别可能在于，它具有视觉和音频组件。 对于视觉组件，Cortana 提供画布区域用于呈现卡片等内容。 对于音频组件，可在机器人的消息中提供文本或 SSML，Cortana 会向用户读出这些内容，使机器人拥有语音功能。 

> [!NOTE]
> Cortana 可用于多种不同的设备。 有些设备带有屏幕，而有些设备可能没有屏幕，例如独立的扬声器。 应确保机器人能够处理这两种情况。 请参阅 [Cortana 特定的实体][CortanaSpecificEntities]，了解如何检查设备信息。

## <a name="adding-speech-to-your-bot"></a>将语音添加到机器人

来自机器人的语音消息以语音合成标记语言 (SSML) 的形式呈现。 使用 Bot Framework SDK 可在机器人的响应中包含 SSML，控制机器人讲述的内容以及显示的内容。  此外，可以通过指定机器人是接受、需要还是忽略用户输入，来控制 Cortana 的麦克风状态。

设置 `IMessageActivity` 对象的 `Speak` 属性可以指定 Cortana 要讲述的消息。 如果指定纯文本，Cortana 会确定单词的发音。 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

若要更好地控制音节、音调和强调语气，请将 `Speak` 属性设置为[语音合成标记语言 (SSML)](http://www.w3.org/TR/speech-synthesis/) 格式。  

以下代码示例指定应该使用中等的强调语气读出单词“text”：
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


**InputHint** 属性帮助向 Cortana 指示机器人是否需要输入。 对于提示，该属性的默认值为 **ExpectingInput**；对于其他类型的响应，默认值为 **AcceptingInput**。


| 值 | Description |
|------|------|
| **AcceptingInput** | 机器人已被动准备好接受输入，但不是在等待响应。 如果用户按住麦克风按钮，Cortana 会接受来自用户的输入。|
| **ExpectingInput** | 指示机器人主动要求用户的响应。 Cortana 会收听用户对着麦克风的讲话。  |
| **IgnoringInput** | Cortana 将忽略输入。 如果机器人正在主动处理请求，并在请求完成之前忽略用户的输入，则机器人可能发送此提示。  |

<!-- TODO: tip about time limit and batching -->

此示例演示如何让 Cortana 知道机器人需要用户输入。 麦克风将保持打开状态。
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>在 Cortana 中显示卡片

除语音响应外，Cortana 还可显示卡片附件。 Cortana 支持以下丰富卡片：

| 卡类型 | Description |
|----|----|
| [HeroCard][heroCard] | 一种通常包含单个大图像、一个或多个按钮和文本的卡。 |
| [ThumbnailCard][thumbnailCard] | 一种通常包含单个缩略图图像、一个或多个按钮和文本的卡。 |
| [ReceiptCard][receiptCard] | 一种可让机器人向用户提供收据的卡。 它通常包含要包括在收据中的项目列表、税款和总计信息以及其他文本。 |
| [SignInCard][signinCard] | 一种可让机器人请求用户登录的卡。 它通常包含文本和一个或多个按钮，用户可以单击这些按钮来启动登录进程。 |


请参阅[卡片设计最佳做法][CardDesign]，查看这些卡片在 Cortana 中的外观。 有关如何在机器人中使用丰富卡片的示例，请参阅[将丰富卡片附件添加到消息](bot-builder-dotnet-add-rich-card-attachments.md)。 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>示例：RollerSkill
以下部分中的代码摘自掷骰子的示例 Cortana 技能。 从 [BotBuilder-Samples 存储库](https://github.com/Microsoft/BotBuilder-Samples/)下载机器人的完整代码。

可通过向 Cortana 说出技能的[调用名称][InvocationNameGuidelines]来调用该技能。 对于掷骰子技能，[将机器人添加到 Cortana 通道][CortanaChannel]并将其注册为 Cortana 技能后，可以通过向 Cortana 说出“掷骰子”或“掷骰子”来调用该技能。

### <a name="explore-the-code"></a>浏览代码



若要调用相应的对话，`RootDispatchDialog.cs` 中定义的活动处理程序将使用正则表达式来匹配用户的输入。 例如，如果用户说出类似于“I'd like to roll some dice”的句子，将触发以下示例中的处理程序。 正则表达式中包含同义词，使类似的话语可以触发对话。
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

`CreateGameDialog` 对话设置机器人玩的自定义游戏。 它使用 `PromptDialog` 来询问用户希望骰子具有的面数，以及应该滚动的面数。 请注意，用于初始化提示的 `PromptOptions` 对象包含提示语音版本的 `speak` 属性。

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

为呈现结果，`PlayGameDialog` 会在 `HeroCard` 中显示结果，并使用 `Speak` 方法来生成要讲述的语音消息。

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>后续步骤

如果机器人在本地运行并已部署到云中，则你可以从 Cortana 调用它。 请参阅[测试 Cortana 技能](../bot-service-debug-cortana-skill.md)，了解试用 Cortana 技能所需的步骤。


## <a name="additional-resources"></a>其他资源
* [Cortana 技能套件][CortanaGetStarted]
* [向消息添加语音](bot-builder-dotnet-text-to-speech.md)
* [SSML 参考][SSMLRef]
* [Cortana 的语音设计最佳做法][VoiceDesign]
* [Cortana 的卡片设计最佳做法][CardDesign]
* [Cortana 开发人员中心][CortanaDevCenter]
* [Cortana 的测试和调试最佳做法][Cortana-TestBestPractice]
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Framework SDK for .NET 参考</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines  
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 


