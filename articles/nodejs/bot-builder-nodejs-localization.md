---
title: 支持本地化 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for Node.js 确定用户所在位置并启用本地化功能。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f9af8cbbc5f457d5e8684ad57bb147f9d42a62c7
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905141"
---
# <a name="support-localization"></a>支持本地化

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Bot Builder 包含一个丰富的本地化系统，用于构建可以用多种语言与用户交流的机器人。 可以使用存储在机器人目录结构中的 JSON 文件将机器人的所有提示本地化。 如果使用 LUIS 等系统执行自然语言处理，则可以为机器人支持的每种语言的 [LuisRecognizer][LUISRecognizer] 配置单独的模型，SDK 会自动选择与用户的首选区域设置匹配的模型。

## <a name="determine-the-locale-by-prompting-the-user"></a>通过提示用户确定区域设置
本地化机器人的第一步是添加识别用户首选语言的功能。 SDK 提供了 [session.preferredLocale()][preferredLocal] 方法，以便基于每个用户保存和检索此首选项。 以下示例是一个对话，用于提示用户输入其首选语言，然后保存其选择。

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>通过分析确定区域设置
确定用户区域设置的另一种方法是使用[文本分析 API](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) 之类的服务根据用户发送的消息文本自动检测用户的语言。

下面的代码片段演示了如何将该服务合并到自己的机器人中。
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

将上述代码片段添加到机器人后，调用 [session.preferredLocale()][preferredLocal] 将自动返回检测到的语言。 `preferredLocale()` 的搜索顺序如下所示：
1. 通过调用 `session.preferredLocale()` 保存的区域设置。 此值存储在 `session.userData['BotBuilder.Data.PreferredLocale']` 中。
2. 检测到分配给 `session.message.textLocale` 的区域设置。
3. 机器人配置的默认区域设置（例如：英语 (‘en’)）。

可以使用机器人的构造函数配置其默认区域设置：

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>本地化提示
Bot Builder SDK 的默认本地化系统是基于文件的，并允许机器人使用存储在磁盘上的 JSON 文件支持多种语言。 默认情况下，本地化系统将在 **./locale/<IETF TAG>/index.json** 文件中搜索机器人的提示，其中 <IETF TAG> 是有效的 [IETF 语言标记][IEFT]，表示要查找其提示的首选区域设置。 

以下屏幕截图显示了支持三种语言的机器人的目录结构：英语、意大利语和西班牙语。

![三个区域设置的目录结构](../media/locale-dir.png)

该文件的结构是消息 ID 到本地化文本字符串的简单 JSON 映射。 如果值是数组而不是字符串，则使用 [session.localizer.gettext()][GetText] 检索该值时，会随机选择数组中的一个提示。 

如果在调用 [session.send()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send) 时传递消息 ID 而不是特定于语言的文本，则机器人会自动检索消息的本地化版本：

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

在内部，SDK 会调用 [`session.preferredLocale()`][preferredLocale] 以获取用户的首选区域设置，然后在调用 [`session.localizer.gettext()`][GetText] 时使用该区域设置将消息 ID 映射到其本地化文本字符串。  有时你可能需要手动调用本地化程序。 例如，传递给 [`Prompts.choice()`][promptsChoice] 的枚举值永远不会自动本地化，因此你可能需要在调用提示之前手动检索本地化列表：

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

默认本地化程序在多个文件中搜索消息 ID，如果找不到 ID（或者没有提供本地化文件），它将只返回 ID 文本，使本地化文件的使用变得透明和可选。  按以下顺序搜索文件：

1. 搜索 [`session.preferredLocale()`][preferredLocale] 返回的区域设置下的 **index.json** 文件。
2. 如果区域设置包含可选的子标记（例如 **en-US**），则会搜索 **en** 的根标记。
3. 搜索机器人的已配置默认区域设置。

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>使用命名空间自定义和本地化提示
默认本地化程序支持为提示建立命名空间，以避免消息 ID 之间发生冲突。  机器人可以重写带命名空间的提示，以便自定义或改写来自另一个命名空间的提示。  可以利用此功能自定义 SDK 的内置消息，从而可以添加对其他语言的支持，或者只是改写 SDK 的当前消息。  例如，可以通过简单地将名为 **BotBuilder.json** 的文件添加到机器人的区域设置目录，然后为 `default_error` 消息 ID 添加条目来更改 SDK 的默认错误消息：

![用于针对区域建立命名空间的 BotBuilder.json](../media/locale-namespacing.png)


## <a name="additional-resources"></a>其他资源

若要了解如何本地化识别器，请参阅[识别意向](bot-builder-nodejs-recognize-intent-messages.md)。


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://github.com/Microsoft/BotBuilder/blob/master/Node/examples/basics-naturalLanguage/app.js
[DisambiguationSample]: https://github.com/Microsoft/BotBuilder/tree/master/Node/examples/feature-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag

