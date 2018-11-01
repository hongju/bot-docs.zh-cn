---
title: 使用 LUIS 识别意向和实体 | Microsoft Docs
description: 将机器人与 LUIS 集成可以检测用户的意图，并使用 Bot Builder SDK for Node.js 触发对话，以此做出适当的响应。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5df1352241485bf95a46fa981b9b16c3cb7e3925
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998694"
---
# <a name="recognize-intents-and-entities-with-luis"></a>使用 LUIS 识别意向和实体 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

本文以笔记记录机器人为例，演示语言理解 ([LUIS][LUIS]) 如何帮助机器人正确响应自然语言输入。 机器人通过识别用户意向来检测他们想要执行的操作。 此意向取决于语音或文本输入，或**话语**。 意向将话语映射到机器人执行的操作，例如调用对话。 机器人可能还需要提取**实体**，这些实体是话语中的重要单词。 有时需要使用实体来达成意向。 在笔记记录机器人的示例中，`Notes.Title` 实体标识每条笔记的标题。

## <a name="create-a-language-understanding-bot-with-bot-service"></a>使用机器人服务创建语言理解机器人

1. 在 [Azure 门户](https://portal.azure.com)上的菜单边栏选项卡中选择“创建新资源”，然后单击“全部查看”。

    ![新建资源](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. 在搜索框中，搜索“Web 应用机器人”。 

    ![新建资源](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. 在“机器人服务”边栏选项卡中提供所需的信息，然后单击“创建”。 此操作可创建机器人服务和 LUIS 应用并将其部署到 Azure。 
   * 将“应用名称”设置为机器人名称。 将机器人部署到云（例如，mynotesbot.azurewebsites.net）时，该名称用作子域。 此名称还用作与机器人关联的 LUIS 应用的名称。 复制该名称，稍后将用它查找与机器人关联的 LUIS 应用。
   * 选择“订阅”、“[资源组](/azure/azure-resource-manager/resource-group-overview)”、“应用服务计划”和“[位置](https://azure.microsoft.com/en-us/regions/)”。
   * 对于“机器人模板”字段，选择“语言理解 (Node.js)”模板。

     ![“机器人服务”边栏选项卡](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * 选中此框以确认服务条款。

4. 确认已部署机器人服务。
    * 单击“通知”（Azure 门户顶部边缘的钟形图标）。 通知将从“部署已开始”更改为“部署已成功”。
    * 通知更改为“部署已成功”后，在通知上单击“转到资源”。

## <a name="try-the-bot"></a>试用机器人

查看“通知”，确认已部署机器人。 通知将从“正在进行部署...”更改为“部署已成功”。 单击“转到资源”按钮，打开机器人的资源边栏选项卡。

注册机器人后，单击“通过网页聊天执行测试”，打开“网页聊天”窗格。 在网页聊天中键入“你好”。

  ![通过网上聊天测试机器人](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

机器人响应说：“你已经进行了问候。 你说了：你好。” 这可确认机器人已接收消息，并将其传递到其创建的默认 LUIS 应用。 此默认 LUIS 应用检测到了问候语意向。

## <a name="modify-the-luis-app"></a>修改 LUIS 应用

使用用于登录 Azure 的相同帐户登录 [https://www.luis.ai](https://www.luis.ai)。 单击“我的应用”。 在应用列表中找到所需应用，该应用的开头为创建机器人服务时在“机器人服务”边栏选项卡的“应用名称”中指定的名称。 

LUIS 应用开始时为 4 个意向：Cancel、Greeting、Help 和 None。 <!-- picture -->

以下步骤添加 Note.Create、Note.ReadAloud 和 Note.Delete 意向： 

1. 单击页面左下角的“预生成域”。 找到“Note”域，然后单击“添加域”。

2. 本教程不会使用“Note”预生成域中包含的所有意向。 在“意向”页中，单击以下每个意向名称，然后单击“删除意向”按钮。
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   只能在 LUIS 应用中保留以下意向： 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * 无
   * 帮助
   * Greeting
   * 取消

     ![LUIS 应用中显示的意向](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  单击右上角的“训练”按钮，训练应用。
4.  单击顶部导航栏中的“发布”，打开“发布”页。 单击“发布到生产槽”按钮。 发布成功后，会将一个 LUIS 应用部署到“发布应用”页的“终结点”列中显示的 URL（位于以资源名称 Starter_Key 开头的行中）。 该 URL 的格式类似于以下示例：`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=` 此 URL 中的应用 ID 和订阅密钥，与 ** “应用服务设置”>“应用程序设置”>“应用设置” ** 中的 LuisAppId 和 LuisAPIKey 相同。


## <a name="modify-the-bot-code"></a>修改机器人代码

单击“生成”，然后单击“打开联机代码编辑器”。

   ![打开联机代码编辑器](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

在代码编辑器中，打开 `app.js`。 它包含以下代码：

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> 在[笔记机器人示例][NotesSample]中也可以找到本文所述的示例代码。



## <a name="edit-the-default-message-handler"></a>编辑默认消息处理程序
机器人有一个默认的消息处理程序。 请按如下所示对其进行编辑： 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>处理 Note.Create 意向

复制以下代码并将其粘贴到 app.js 的末尾：

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

使用 `args` 参数将话语中的所有实体传递到对话。 [瀑布][waterfall]的第一个步骤调用 [EntityRecognizer.findEntity][EntityRecognizer_findEntity]，从 LUIS 响应中的所有 `Note.Title` 实体获取笔记的标题。 如果 LUIS 应用未检测到 `Note.Title` 实体，则机器人会提示用户输入笔记的名称。 瀑布的第二个步骤提示输入要包含在笔记中的文本。 机器人获取笔记的文本后，第三个步骤使用 [session.userData][session_userData] 在 `notes` 对象中保存笔记，并使用标题作为键。 有关 `session.UserData` 的详细信息，请参阅[管理状态数据](./bot-builder-nodejs-state.md)。 



## <a name="handle-the-notedelete-intent"></a>处理 Note.Delete 意向
与处理 `Note.Create` 意向时一样，机器人在 `args` 参数中检查标题。 如果未检测到标题，机器人会提示用户。 标题用于查找要从 `session.userData.notes` 中删除的笔记。 



复制以下代码并将其粘贴到 app.js 的末尾：
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

处理 `Note.Delete` 的代码使用 `noteCount` 函数来确定 `notes` 对象是否包含笔记。 

在 `app.js` 的末尾粘贴 `noteCount` 帮助器函数。

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>处理 Note.ReadAloud 意向

复制以下代码，并将其粘贴到 `Note.Delete` 处理程序后面的 `app.js` 中：

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

`session.userData.notes` 对象作为第三个参数传递到  `builder.Prompts.choice`，使提示内容向用户显示笔记列表。

添加新意向的处理程序后，`app.js` 的完整代码包含以下内容：

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>测试机器人

在 Azure 门户中，单击“通过网上聊天执行测试”以测试机器人。 尝试键入消息（例如“创建笔记”、“阅读我的笔记”和“删除笔记”），以调用添加到机器人的意向。
   ![通过网上聊天测试笔记机器人](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 如果发现机器人并未能始终识别正确意向或实体，请提供更多示例陈述对其进行训练，从而提高 LUIS 应用的性能。 无需对机器人代码进行任何修改即可重新训练 LUIS 应用。 请参阅[添加示例陈述](/azure/cognitive-services/LUIS/add-example-utterances)和[训练和测试 LUIS 应用](/azure/cognitive-services/LUIS/train-test)。


## <a name="next-steps"></a>后续步骤

在尝试操作机器人的过程中可以看到，识别器能够针对当前处于活动状态的对话触发中断。 允许和处理中断中断是一种灵活的设计，可以理解用户真正执行的操作。 详细了解可与识别的意向相关联的各种操作。

> [!div class="nextstepaction"]
> [处理用户操作](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
