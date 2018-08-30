---
title: 翻译用户输入的内容，让机器人会讲多种语言 | Microsoft Docs
description: 如何将用户输入的内容自动翻译成机器人的本机语言，以及如何再翻译回到用户的语言。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e316ff90b68f860274579f06e7196deec364e082
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905630"
---
# <a name="translate-from-the-users-language-to-make-your-bot-multilingual"></a>翻译用户的语言，让机器人会讲多种语言

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人可使用 [Microsoft Translator](https://www.microsoft.com/en-us/translator/) 将消息自动翻译成它理解的语言，并选择性地将其回复翻译回用户的语言。
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

## <a name="get-a-text-services-key"></a>获取文本服务密钥

首先，需要密钥才能使用 Microsoft Translator 服务。 可以在 Azure 门户中获取[免费试用版密钥](https://www.microsoft.com/en-us/translator/trial.aspx#get-started)。

## <a name="installing-packages"></a>安装包

请确保已安装将翻译功能添加到机器人时所需的包。

# <a name="ctabcsrefs"></a>[C#](#tab/csrefs)

添加对以下 NuGet 包的预发布版的[引用](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui)：

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai`（翻译时需要）

要将结合使用翻译功能和语言理解 (LUIS)，还要添加对以下包的引用：

* `Microsoft.Bot.Builder.Luis`（LUIS 需要）

# <a name="javascripttabjsrefs"></a>[JavaScript](#tab/jsrefs)

可使用 botbuilder-ai 包向机器人添加上述任一服务。 可通过 npm 向项目添加以下包：
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>配置翻译

只需将翻译工具添加到机器人的中间件堆栈，即可将机器人配置为调用该翻译工具来翻译从用户收到的每条消息。 该中间件通过上下文对象使用翻译结果来修改用户的消息。


# <a name="ctabcssetuptranslate"></a>[C#](#tab/cssetuptranslate)

先来看看 SDK 中的 EchoBot 示例，同时更新 `Startup.cs` 文件中的 `ConfigureServices` 方法，向机器人添加 `TranslationMiddleware`。 这会将机器人配置为翻译从用户收到的每条消息。 <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->

**Startup.cs**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<TranslatorBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

    });
}


```

> [!TIP] 
> BotBuilder SDK 根据用户刚提交的消息自动检测用户的语言。 要重写此功能，可提供其他回调参数，以添加自己的逻辑来检测和更改用户的语言。  



请查看 `EchoBot.cs` 中的代码，其中发送了“You sent”短语，后接用户所述的内容：

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

添加翻译中间件时，可选参数可用于指定是否要将回复内容翻译回用户的语言。 `Startup.cs` 中指定了 `false`，表示只将用户消息翻译成机器人的语言。

```cs
        // The first parameter is a list of languages the bot recognizes
        // The second parameter is your API key
        // The third parameter specifies whether to translate the bot's replies back to the user's language
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));
```

# <a name="javascripttabjssetuptranslate"></a>[JavaScript](#tab/jssetuptranslate)

要使用回显机器人设置翻译中间件，请将以下内容粘贴到 app.js 中。

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['fr', 'de'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>运行机器人并查看翻译后的输入内容

运行机器人并以其他语言键入一些消息。 你将会看到机器人已翻译用户消息并在答复中指出了所翻译的内容。

![机器人检测语言并翻译输入](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>调用采用机器人本机语言的逻辑

现在，添加用于检查英语单词的逻辑。 如果用户以另一种语言说出“help”或“cancel”一词，机器人会将它翻译成英语，同时调用逻辑来检查英语单词“help”或“cancel”。

# <a name="ctabcshelp"></a>[C#](#tab/cshelp)
```cs
        case ActivityTypes.Message:
            // check the message text before calling context.SendActivity
            var messagetext = context.Activity.Text.Trim().ToLower();
            if (string.Equals("help", messagetext))
            {
                await context.SendActivity("You asked for help.");
            }
```

# <a name="javascripttabjshelp"></a>[JavaScript](#tab/jshelp)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![机器人检测 help 的法语翻译](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>将回复翻译回用户的语言

此外，可通过将最后一个构造函数参数设置为 `true`，将回复翻译回用户的语言。

# <a name="ctabcstranslateback"></a>[C#](#tab/cstranslateback)
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjstranslateback"></a>[JavaScript](#tab/jstranslateback)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>运行机器人以查看按用户语言给出的回复

运行机器人并以其他语言键入一些消息。 你将看到机器人检测到了用户语言，并翻译了答复。

![机器人检测语言并翻译答复](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>添加用于检测或更改用户语言的逻辑

如果不让 Botbuilder SDK 自动检测用户的语言，可提供回调来添加自己的逻辑，用于确定用户的语言或确定用户的语言何时发生了更改。

> [!TIP] 
> 要获取实现回调来检测和更改用户语言的示例机器人，请查看 SDK 中的翻译示例。 

在下例中，`CheckUserChangedLanguage` 回调检查特定的用户消息以更改语言。 

# <a name="ctabcschangelanguage"></a>[C#](#tab/cschangelanguage)
```cs
// In Startup.cs
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", null, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

// Add a file TranslatorLocalHelper.cs with the following:

    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) => context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) => _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjschangelanguage"></a>[JavaScript](#tab/jschangelanguage)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>将翻译功能与 LUIS/QnA 结合使用

要将翻译功能与机器人中的其他服务（例如 LUIS 或 QnA 生成器）结合使用，请先添加翻译中间件，以便在将消息传递到预期使用机器人本机语言的其他中间件之前对消息进行翻译。

# <a name="ctabcslanguageluis"></a>[C#](#tab/cslanguageluis)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

# <a name="javascripttabjslanguageluis"></a>[JavaScript](#tab/jslanguageluis)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});

```

---

在机器人代码中，LUIS 结果基于已翻译成机器人本机语言的输入。 尝试修改机器人代码，以检查 LUIS 应用的结果：

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

## <a name="bypass-translation-for-specified-patterns"></a>绕过指定模式的翻译
你可能不希望机器人翻译某些单词，例如专有名词。 可提供正则表达式，指出不该翻译的模式。 例如，如果用户以机器人的非本机语言说出“My name is ...”，而你想要避免翻译该用户的姓名，可使用模式来指定此要求。

# <a name="ctabcsbypass"></a>[C#](#tab/csbypass)
```cs
    // Startup.cs

    // Pattern representing input to not translate
    Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
    // single pattern for fr language, to avoid translating what follows "my name is"
    patterns.Add("fr", new List<string> { "mon nom est (.+)" });
    
    middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR API KEY>", patterns, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjsbypass"></a>[JavaScript](#tab/jsbypass)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![机器人绕过模式翻译](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>将日期本地化

如果需要将日期本地化，可以添加 `LocaleConverterMiddleware`。 例如，如果知道机器人需要 `MM/DD/YYYY` 格式的日期，而采用其他区域设置的用户可能以 `DD/MM/YYYY` 格式输入日期，则区域设置转换器中间件可将日期自动转换为机器人所需的格式。

> [!NOTE]
> 区域设置转换器中间件只能用于转换日期。 它并不知道翻译中间件的结果。 如果使用翻译中间件，请谨慎地将它与区域设置转换器相结合。 翻译中间件会连同其他文本输入一起翻译某些采用文本格式的日期，但不会翻译日期

例如，下图显示的机器人在将英语翻译成法语后回复用户输入。 它使用 `TranslationMiddleware` 而不是使用 `LocaleConverterMiddleware`。

![翻译日期而不转换日期的机器人](./media/how-to-bot-translate/locale-date-before.png)

下面显示了在添加 `LocaleConverterMiddleware` 后，同一机器人的聊天结果。

![翻译日期而不转换日期的机器人](./media/how-to-bot-translate/locale-date-after.png)

区域设置转换器支持英语、法语、德语和中文区域设置。 <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcstranslation"></a>[C#](#tab/cstranslation)
```cs
// Startup.cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(TranslatorLocaleHelper.GetActiveLocale, TranslatorLocaleHelper.CheckUserChangedLocale, "en-us", LocaleConverter.Converter));

// TranslatorLocaleHelper.cs
        public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
        {
            bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
            //use a specific message from user to change language
            if (context.Activity.Type == ActivityTypes.Message)
            {
                var messageActivity = context.Activity.AsMessageActivity();
                if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
                {
                    changeLocale = true;
                }
                if (changeLocale)
                {
                    var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
                    if (!string.IsNullOrWhiteSpace(newLocale)
                            && IsSupportedLanguage(newLocale))
                    {
                        SetLocale(context, newLocale);
                        await context.SendActivity($@"Changing your language to {newLocale}");
                    }
                    else
                    {
                        await context.SendActivity($@"{newLocale} is not a supported locale.");
                    }
                    //intercepts message
                    return true;
                }
            }

            return false;
        }
        public static string GetActiveLocale(ITurnContext context)
        {
            if (currentLocale != null)
            {
                //the user has specified a different locale so update the bot state
                if (context.GetConversationState<CurrentUserState>() != null
                    && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
                {
                    SetLocale(context, currentLocale);
                }
            }
            if (context.Activity.Type == ActivityTypes.Message
                && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
            {
                return context.GetConversationState<CurrentUserState>().Locale;
            }

            return "en-us";
        }
```

# <a name="javascripttabjstranslation"></a>[JavaScript](#tab/jstranslation)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---
