---
title: 使用 Cortana 技能构建支持语音的机器人 | Microsoft Docs
description: 了解如何借助 Cortana 技能和 Bot Builder SDK for Node.js 构建支持语音的机器人。
author: DeniseMak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c213c1155b1eef5f5c776ba42a221d95b74f99a5
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906072"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>使用 Cortana 技能构建支持语音的机器人

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)

Bot Builder SDK for Node.js 通过将支持语音的机器人作为 Cortana 技能连接到 Cortana 通道，使你能够构建支持语音的机器人。 Cortana 技能让你能够通过 Cortana 提供功能，响应用户的语音输入。

> [!TIP]
> 有关技能的概念及其作用的详细信息，请参阅 [Cortana 技能套件][CortanaGetStarted]。

使用 Bot Framework 创建 Cortana 技能只需少量的 Cortana 特定知识，主要包括构建机器人。 与已创建的其他机器人的一个主要区别是 Cortana 具有视觉和音频组件。 对于视觉组件，Cortana 提供画布区域用于呈现卡片等内容。 对于音频组件，可在机器人的消息中提供文本或 SSML，Cortana 会向用户读出这些内容，使机器人拥有语音功能。 

> [!NOTE]
> Cortana 可用于多种不同的设备。 有些设备带有屏幕，而有些设备可能没有屏幕，例如独立的扬声器。 应确保机器人能够处理这两种情况。 请参阅 [Cortana 特定的实体][CortanaSpecificEntities]，了解如何检查设备信息。

## <a name="adding-speech-to-your-bot"></a>将语音添加到机器人

来自机器人的语音消息以语音合成标记语言 (SSML) 的形式呈现。 使用 Bot Builder SDK 可在机器人的响应中包含 SSML，控制机器人讲述的内容以及显示的内容。

### <a name="sessionsay"></a>session.say

机器人使用 session.say 方法向用户讲话，用于代替 session.send。 它包含用于发送 SSML 输出的可选参数，以及卡片等附件。 

该方法具有以下格式：

```session.say(displayText: string, speechText: string, options?: object)```

| 参数 | Description |
|------|------|
| **displayText** | 在 Cortana 的 UI 中显示的文本消息。|
| **speechText** | Cortana 读取给用户的文本或 SSML。 |
| **options** | [IMessage][IMessage] 对象，可以包含附件或输入提示。 输入提示指示机器人接受、期待或忽略输入。 卡片附件显示在 displayText 信息下方 Cortana 的画布中。   |

inputHint 属性有助于向 Cortana 指示机器人是否期待输入。 如果使用内置提示，此值会自动设置为 expectingInput 的默认值。


| 值 | Description |
|------|------|
| **acceptingInput** | 机器人被动地准备好接收输入，但并不等待响应。 如果用户按住麦克风按钮，Cortana 接受来自用户的输入。|
| **expectingInput** | 指示机器人主动期待来自用户的响应。 Cortana 会收听用户对着麦克风的讲话。  |
| **ignoringInput** | Cortana 将忽略输入。 如果机器人正在主动处理请求，并在请求完成之前忽略用户的输入，则机器人可能发送此提示。  |


下面的示例显示 Cortana 如何读取纯文本或 SSML：

```javascript
// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');
```

此示例演示如何让 Cortana 知道机器人期待用户输入。 麦克风将保持打开状态。
```javascript
// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});
```
<!-- TODO: tip about time limit and batching -->


### <a name="prompts"></a>提示

除使用 session.say() 方法外，还可以使用 speak 和 retrySpeak 选项向内置提示传递文本或 SSML。  

```javascript
builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});
```



<!-- TODO: Link to SSML library -->

若要为用户提供选项列表，请使用 Prompts.choice。 synonyms 选项允许更灵活地识别用户话语。 value 选项在 results.response.entity 中返回。 action 选项指定机器人为该选择显示的标签。

Prompts.choice 支持序号选择。 这意味着用户可以说“第一”、“第二”或“第三”来选择列表中的项目。 例如，给定以下提示，如果用户向 Cortana 提出“第二个选项”，则提示符将返回 8 值。

```javascript
        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });
```

在前面的示例中，通过使用存储在本地化提示文件中具有以下格式的字符串，对提示的 speak 属性的 SSML 进行格式设置。 

```json
{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}
```


然后，帮助程序函数构建语音合成标记语言 (SSML) 文档的必需根元素。 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> 可以在 [Roller 示例技巧](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill)中找到一个小型实用程序模块 (ssml.js)，用于构建机器人基于 SSML 的响应。
> 通过 [npm](https://www.npmjs.com/search?q=ssml) 还提供几个有用的 SSML 库，这样可以轻松创建格式规范的 SSML。

## <a name="display-cards-in-cortana"></a>在 Cortana 中显示卡片

除语音响应外，Cortana 还可显示卡片附件。 Cortana 支持以下资讯卡：
* [HeroCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html)
* [ReceiptCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html)
* [ThumbnailCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html)

请参阅[卡片设计最佳做法][CardDesign]，查看这些卡片在 Cortana 中的外观。 有关如何将资讯卡添加到机器人的示例，请参阅[发送资讯卡](bot-builder-nodejs-send-rich-cards.md)。 

下面的代码演示如何将 speak 和 inputHint 属性添加到包含 Hero 卡片的消息中。

```javascript 

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });


/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>示例：RollerSkill
以下部分中的代码摘自掷骰子的示例 Cortana 技能。 从 [BotBuilder-Samples 存储库](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill)下载机器人的完整代码。

可通过向 Cortana 说出技能的[调用名称][InvocationNameGuidelines]来调用该技能。 对于掷骰子技能，[将机器人添加到 Cortana 通道][CortanaChannel]并将其注册为 Cortana 技能后，可以通过向 Cortana 说出“掷骰子”或“掷骰子”来调用该技能。

### <a name="explore-the-code"></a>浏览代码

RollerSkill 示例首先打开带有一些按钮的卡片，告知用户可以使用的选项。

```javascript
/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>提示用户输入

下面的对话设置自定义游戏供机器人玩耍。  机器人询问用户他们希望骰子具有的面数，以及应该滚动的面数。 机器人构建游戏结构后，会将其传递给单独的“PlayGameDialog”。

要启动对话，此对话上的 triggerAction() 处理程序允许用户说出“我想玩玩投骰子”之类的内容。 它使用正则表达式来匹配用户的输入，但你可以使用 [LUIS 意向](./bot-builder-nodejs-recognize-intent-luis.md)，用起来同样简单。 


```javascript


bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});
```

### <a name="render-results"></a>呈现结果

 此对话是我们主要的游戏循环。 机器人在 session.conversationData 中存储游戏结构，因此如果用户说"再投一次"，我们可以再次重新抛掷同一组骰子。

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });
```

## <a name="next-steps"></a>后续步骤
如果正在本地运行机器人或已在云端部署机器人，则可以从 Cortana 调用该机器人。 请参阅[测试 Cortana 技能](../bot-service-debug-cortana-skill.md)，了解试用 Cortana 技能所需的步骤。


## <a name="additional-resources"></a>其他资源
* [Cortana 技能套件][CortanaGetStarted]
* [向消息添加语音](bot-builder-nodejs-text-to-speech.md)
* [SSML 参考][SSMLRef]
* [Cortana 的语音设计最佳做法][VoiceDesign]
* [Cortana 的卡片设计最佳做法][CardDesign]
* [Cortana 开发人员中心][CortanaDevCenter]
* [Cortana 的测试和调试最佳做法][Cortana-TestBestPractice]


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
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
