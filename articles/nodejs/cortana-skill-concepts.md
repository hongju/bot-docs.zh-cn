---
title: 使用 Node.js 构建 Cortana 技能 | Microsoft Docs
description: 了解在 Bot Framework SDK for Node.js 中构建 Cortana 技能的核心概念。
keywords: Bot Framework, Cortana 技能, 语音, Node.js, Bot Builder, SDK, 关键概念, 核心概念
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5de773f6f8f4d46c0c1fe880588f2530c3c68f56
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/18/2019
ms.locfileid: "59746061"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>使用 Node.js 为 Cortana 技能构建机器人的关键概念
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> 本文属于初步内容，将进行更新。

本文介绍在 Bot Framework SDK for Node.js 中构建 Cortana 技能的关键概念。 

## <a name="what-is-a-cortana-skill"></a>什么是 Cortana 技能？
Cortana 技能是一种可以通过使用 Cortana 客户端来调用的机器人，就像 Windows 10 中内置的那样。 用户通过说出与机器人相关联的一些关键字或短语来启动机器人。 使用 Bot Framework 门户来配置启动机器人的关键字。 

可将 Cortana 视为一种支持语音的通道，可以发送和接收语音消息及文本聊天。 作为 Cortana 技能发布的机器人应该设计用于语音和文本。 Bot Framework 提供了指定语音合成标记语言 (SSML) 的方法，以定义机器人发送的语音消息。

## <a name="acknowledge-user-utterances"></a>确认用户话语 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


创建支持语音的机器人时，应尝试在聊天中建立共识和相互理解。 机器人应通过提示用户被听到和被理解来“落实”用户的话语。

如果系统未能落实他们的话语，用户会感到困惑。 例如，当机器人询问“接下来干什么？”时，下面的聊天可能有点令人困惑：

> **Cortana**：你想要查看更多个人资料内容吗？  
> **用户**：不是。  
> **Cortana**：后续步骤

如果机器人添加“好的”作为确认，则对用户来说更为友好：

> **Cortana**：你想要查看更多个人资料内容吗？  
> **用户**：不是。  
> **Cortana**：**好的**，接下来你想要做什么？

落实程度，从最弱到最强：

1. 持续关注
2. 下一相关表述
3. 确认：最短的回应或提示下文话语："是的"、"嗯"、"好的"、"太好了"
4. 示范：通过重新阐述、完成表述来表示理解。
5. 显示：重复全部或部分内容。

### <a name="acknowledgement-and-next-relevant-contribution"></a>确认和下一相关表述

> **用户**：我需要在五月旅行。  
> **Cortana**：**好的**。 你想在五月的哪一天旅行？  
> **用户**：嗯，我需要 12 号至 15 号到那儿。  
> **Cortana**：**好的**。 你要飞到哪个城市？  

### <a name="grounding-by-demonstration"></a>通过示范进行落实

> **用户**：我需要在五月旅行。  
> **Cortana**：那么，你想在五月**哪一天**旅行？  
> **用户**：嗯，我需要 12 号至 15 号到那儿。  
> **Cortana**：**那么**，你要飞到哪个城市？  
    
### <a name="closure"></a>结尾

执行操作的机器人应提供成功执行的证据。 表明失败或理解也很重要。 

* 非语音结尾：如果按下电梯按钮，其指示灯将亮起。  
这是一个两步过程：
    * 表示（按按钮时）
    * 接受（按钮亮起）

## <a name="differences-in-content-presentation"></a>内容呈现的差异
请记住，许多设备支持 Cortana，但只有一部分有屏幕。 在设计支持语音的机器人时需考虑的一个事项是，语音对话通常与机器人显示的文本消息不同。
<!-- If there are differences in what the bot will say, in the text vs the speak fields of a prompt or in a waterfall, for example, discuss them here.

## Speech

You bot uses the **session.say** method to speak to the user. The speak method has three overloads:
* If you pass only one parameter to **session.say**, it can be a text parameter.
* If you pass two parameters to **session.say**, it can take text and SSML.
* If you pass three parameters, the third parameter takes an options structure that specifies all the options you can pass to build an **IMessage** object.

```javascript
var bot = new builder.UniversalBot(connector, function (session) {
    session.say("Hello... I'm a decision making bot.'.", 
        ssml.speak("Hello. I can help you answer all of life's tough questions."));
    session.replaceDialog('rootMenu');
});

```
## Speech in messages

The **IMessage** object provides a **speak** property for SSML. It can be used to play a .wav file.

The **inputHint** property helps indicate to Cortana whether your bot is expecting input. If you're using a built-in prompt, this value is automatically set to the default of **expectingInput**.

The **inputHint** property can take the following values: 
* **expectingInput**: Indicates that the bot is actively expecting a response from the user. Cortana listens for the user to speak into the microphone.
* **acceptingInput**: Indicates that the bot is passively ready for input but is not waiting on a response. Cortana accepts input from the user if the user holds down the microphone button.
* **ignoringInput**: Cortana is ignoring input. Your bot may send this hint if it is actively processing a request and will ignore input from users until the request is complete.

Prompts must use the `speak:` option.

```javascript
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

Prompts.number has *ordinal support*, meaning that you can say "the last", "the first", "the next-to-last" to choose an item in a list.

## Using synonyms

<!-- Axl Rose example -->
```javascript   
         var choices = [
            { 
                value: 'flipCoinDialog',
                action: { title: "Flip A Coin" },
                synonyms: 'toss coin|flip quarter|toss quarter'
            },
            {
                value: 'rollDiceDialog',
                action: { title: "Roll Dice" },
                synonyms: 'roll die|shoot dice|shoot die'
            },
            {
                value: 'magicBallDialog',
                action: { title: "Magic 8-Ball" },
                synonyms: 'shake ball'
            },
            {
                value: 'quit',
                action: { title: "Quit" },
                synonyms: 'exit|stop|end'
            }
        ];
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

## <a name="configuring-your-bot"></a>配置机器人

## <a name="prompts"></a>提示

## <a name="additional-resources"></a>其他资源

Cortana 文档：[Cortana 技能文档](/cortana/skills/)

Cortana SSML 参考：[语音合成标记语言 (SSML) 参考](/cortana/skills/speech-synthesis-markup-language)
