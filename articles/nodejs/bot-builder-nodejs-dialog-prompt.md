---
title: 提示用户输入 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for Node.js 中使用提示来收集用户输入。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: aa20dc396b68ede3271d12a8deab2e673a79d1d1
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904479"
---
# <a name="prompt-for-user-input"></a>提示用户输入

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Builder SDK for Node.js 提供一组内置的提示来简化从用户收集输入的过程。 

每当机器人需要用户的输入时，都会使用提示。 可以通过在瀑布中链接提示，使用提示来请求用户提供一系列输入。 可将提示与[瀑布](bot-builder-nodejs-dialog-waterfall.md)相结合，以帮助在机器人中[管理聊天流](bot-builder-nodejs-manage-conversation-flow.md)。 

本文将帮助你了解提示的工作原理，以及如何使用提示从用户收集信息。

## <a name="prompts-and-responses"></a>提示和响应

每当需要用户的输入时，可以发送一条提示，等待用户使用输入做出响应，然后处理输入并向用户发送响应。

以下代码示例提示用户输入其姓名，并使用一条问候消息做出响应。

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

使用此基本构造，可以通过添加机器人所需的任意数目的提示和响应，来为聊天流建模。

## <a name="prompt-results"></a>提示结果 

内置提示实现为在 `results.response` 字段中返回用户响应的[对话](bot-builder-nodejs-dialog-overview.md)。 对于 JSON 对象，响应在 `results.response.entity` 字段中返回。 任何类型的[ 对话处理程序](bot-builder-nodejs-dialog-overview.md#dialog-handlers)都可以接收提示结果。 机器人收到响应后，可以通过调用 [`session.endDialogWithResult`][EndDialogWithResult] 方法来使用该响应，或将其传递回到调用方对话。

以下代码示例演示如何使用 `session.endDialogWithResult` 方法，将提示结果返回给调用方对话。 在此示例中，`greetings` 对话使用 `askName` 对话返回的提示结果，来问候用户并显示其姓名。

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="prompt-types"></a>提示类型
Bot Builder SDK for Node.js 包含多种不同类型的内置提示。 

|**提示类型**     | **说明** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | 请求用户输入文本字符串。 |     
|[Prompts.confirm](#promptsconfirm) | 请求用户确认操作。| 
|[Prompts.number](#promptsnumber) | 请求用户输入数字。     |
|[Prompts.time](#promptstime) | 请求用户输入时间或日期/时间。      |
|[Prompts.choice](#promptschoice) | 请求用户从选项列表中进行选择。    |
|[Prompts.attachment](#promptsattachment) | 请求用户上传图片或视频。|       

以下部分提供了有关每种类型的提示的更多详细信息。

### <a name="promptstext"></a>Prompts.text

使用 [Prompts.text()][PromptsText] 方法可以请求用户输入**文本字符串**。 该提示将返回 [IPromptTextResult][IPromptTextResult] 形式的用户响应。

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

使用 [Prompts.confirm()][PromptsConfirm] 方法可以请求用户使用 **yes/no** 响应确认某个操作。 该提示将返回 [IPromptConfirmResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html) 形式的用户响应。

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

使用 [Prompts.number()][PromptsNumber] 方法可以请求用户输入**数字**。 该提示将返回 [IPromptNumberResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html) 形式的用户响应。

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

使用 [Prompts.time()][PromptsTime] 方法可以请求用户输入**时间**或**日期/时间**。 该提示将返回 [IPromptTimeResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html) 形式的用户响应。 框架使用 [Chrono](https://github.com/wanasit/chrono) 库来分析用户的响应，并支持相对响应（例如，“在 5 分钟内”）和非相对响应（例如，“6 月 6 日下午 2 点”）。

表示用户响应的 [Results.response](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response) 字段包含一个指定日期和时间的 [entity](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html) 对象。 若要将日期和时间解析成 JavaScript `Date` 对象，请使用 [EntityRecognizer.resolveTime()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime) 方法。

> [!TIP] 
> 根据托管机器人的服务器的时区，将用户输入的时间转换为 UTC 时间。 服务器与用户所在的时区可能不同，因此请务必考虑到时区。 若要将日期和时间转换为用户的本地时间，请考虑请求用户指定他们所在的时区。

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

使用 [Prompts.choice()][PromptsChoice] 方法可以请求用户**从选项列表中进行选择**。 用户可以通过以下方式指明所做的选择：输入与所选选项关联的数字，或者输入所选选项的名称。 支持选项名称的完全匹配和部分匹配。 该提示将返回 [IPromptChoiceResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html) 形式的用户响应。 

若要指定提供给用户的列表的样式，请设置 [IPromptOptions.listStyle](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle) 属性。 下表显示此属性的 `ListStyle` 枚举值。


`ListStyle` 枚举值如下所示：

| 索引 | 名称 | Description |
| ---- | ---- | ---- |
| 0 | 无 | 不呈现任何列表。 将列表包含为提示的一部分时使用此值。 |
| 1 | inline | 以窗体“1. red, 2. green, or 3. blue”的内联列表形式呈现选项。 |
| 2 | list | 以带编号列表的形式呈现选项。 |
| 3 | button | 以支持按钮的通道的按钮形式呈现选项。 对于其他通道，将以文本形式呈现选项。 |
| 4 | auto | 根据通道和选项数目自动选择样式。 | 

可以从 `builder` 对象访问此枚举，或者，可以提供一个索引来选择 `ListStyle`。 例如，以下代码片段中的两条语句可以实现相同的目的。

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

若要指定选项列表，可以使用竖线 (`|`) 分隔的字符串、字符串数组或对象映射。

竖线分隔的字符串： 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

字符串数组：

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

对象映射： 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

使用 [Prompts.attachment()][PromptsAttachment] 方法可以请求用户上传文件，例如图像或视频。 该提示将返回 [IPromptAttachmentResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html) 形式的用户响应。

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>后续步骤

了解如何引导用户完成瀑布流程中的每个步骤并提示他们输入信息后，接下来请了解如何更好地管理聊天流。

> [!div class="nextstepaction"]
> [管理聊天流](bot-builder-nodejs-dialog-manage-conversation-flow.md)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#reponse

[ResumeReason]: https://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html
