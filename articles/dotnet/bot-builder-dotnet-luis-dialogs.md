---
title: 使用 LUIS 识别意向和实体 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for .NET 中使用 LUIS 对话，使机器人理解自然语言。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f95335149fa2c896d905834832089ffbfa960bf2
ms.sourcegitcommit: d4afc924b0e1907c4d6f7a6fc5ac1fe521aeef7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447383"
---
# <a name="recognize-intents-and-entities-with-luis"></a>使用 LUIS 识别意向和实体 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

本文以笔记记录机器人为例，演示语言理解 ([LUIS][LUIS]) 如何帮助机器人正确响应自然语言输入。 机器人通过识别用户意向来检测他们想要执行的操作。 根据口头或文本输入，或者话语确定此意向。 意向将话语映射到机器人采取的操作。 例如，笔记记录机器人识别到 `Notes.Create` 意向，便调用创建笔记功能。 机器人可能还需要提取实体，这些实体是话语中的重要字词。 在笔记记录机器人的示例中，`Notes.Title` 实体标识每条笔记的标题。

## <a name="create-a-language-understanding-bot-with-bot-service"></a>使用机器人服务创建语言理解机器人

1. 在 [Azure 门户](https://portal.azure.com)上的菜单边栏选项卡中选择“创建新资源”，然后单击“全部查看”。

    ![新建资源](../media/bot-builder-dotnet-use-luis/bot-service-creation.png)

2. 在搜索框中，搜索“Web 应用机器人”。 

    ![新建资源](../media/bot-builder-dotnet-use-luis/bot-service-selection.png)

3. 在“机器人服务”边栏选项卡中提供所需的信息，然后单击“创建”。 此操作可创建机器人服务和 LUIS 应用并将其部署到 Azure。 
   * 将“应用名称”设置为机器人名称。 将机器人部署到云（例如，mynotesbot.azurewebsites.net）时，该名称用作子域。 此名称还用作与机器人关联的 LUIS 应用的名称。 复制该名称，稍后将用它查找与机器人关联的 LUIS 应用。
   * 选择“订阅”、“[资源组](/azure/azure-resource-manager/resource-group-overview)”、“应用服务计划”和“[位置](https://azure.microsoft.com/en-us/regions/)”。
   * 对于“机器人模板”字段，选择“语言理解(C#)”模板。

     ![“机器人服务”边栏选项卡](../media/bot-builder-dotnet-use-luis/bot-service-setting-callout-template.png)

   * 选中此框以确认服务条款。

4. 确认已部署机器人服务。
    * 单击“通知”（Azure 门户顶部边缘的钟形图标）。 通知将从“部署已开始”更改为“部署已成功”。
    * 通知更改为“部署已成功”后，在通知上单击“转到资源”。

## <a name="try-the-bot"></a>试用机器人

查看“通知”，确认已部署机器人。 通知将从“正在进行部署...”更改为“部署已成功”。 单击“转到资源”按钮，打开机器人的资源边栏选项卡。

注册机器人后，单击“通过网页聊天执行测试”，打开“网页聊天”窗格。 在网页聊天中键入“你好”。

  ![通过网上聊天测试机器人](../media/bot-builder-dotnet-use-luis/bot-service-web-chat.png)

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

     ![LUIS 应用中显示的意向](../media/bot-builder-dotnet-use-luis/luis-intent-list.png)

3. 单击右上角的“训练”按钮，训练应用。
4. 单击顶部导航栏中的“发布”，打开“发布”页。 单击“发布到生产槽”按钮。 发布成功后，复制“发布应用”页的“终结点”列中显示的 URL（位于以资源名称 Starter_Key 开头的行中）。 保存此 URL 以便稍后在机器人代码中使用。 该 URL 的格式类似于以下示例：`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`

## <a name="modify-the-bot-code"></a>修改机器人代码

单击“生成”，然后单击“打开联机代码编辑器”。
    ![打开联机代码编辑器](../media/bot-builder-dotnet-use-luis/bot-service-build.png)

在代码编辑器中，打开 `BasicLuisDialog.cs`。 它包含以下用于处理 LUIS 应用中的意向的代码。
```cs
using System;
using System.Configuration;
using System.Threading.Tasks;

using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Luis;
using Microsoft.Bot.Builder.Luis.Models;

namespace Microsoft.Bot.Sample.LuisBot
{
    // For more information about this template visit http://aka.ms/azurebots-csharp-luis
    [Serializable]
    public class BasicLuisDialog : LuisDialog<object>
    {
        public BasicLuisDialog() : base(new LuisService(new LuisModelAttribute(
            ConfigurationManager.AppSettings["LuisAppId"], 
            ConfigurationManager.AppSettings["LuisAPIKey"], 
            domain: ConfigurationManager.AppSettings["LuisAPIHostName"])))
        {
        }

        [LuisIntent("None")]
        public async Task NoneIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        // Go to https://luis.ai and create a new intent, then train/publish your luis app.
        // Finally replace "Greeting" with the name of your newly created intent in the following handler
        [LuisIntent("Greeting")]
        public async Task GreetingIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Cancel")]
        public async Task CancelIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        [LuisIntent("Help")]
        public async Task HelpIntent(IDialogContext context, LuisResult result)
        {
            await this.ShowLuisResult(context, result);
        }

        private async Task ShowLuisResult(IDialogContext context, LuisResult result) 
        {
            await context.PostAsync($"You have reached {result.Intents[0].Intent}. You said: {result.Query}");
            context.Wait(MessageReceived);
        }
    }
}
```
### <a name="create-a-class-for-storing-notes"></a>创建用于存储笔记的类

在 BasicLuisDialog.cs 中添加以下 `using` 语句。

```cs
using System.Collections.Generic;
```

在构造函数定义后，将以下代码添加到 `BasicLuisDialog` 类中。

```cs
        // Store notes in a dictionary that uses the title as a key
        private readonly Dictionary<string, Note> noteByTitle = new Dictionary<string, Note>();
        
        [Serializable]
        public sealed class Note : IEquatable<Note>
        {

            public string Title { get; set; }
            public string Text { get; set; }

            public override string ToString()
            {
                return $"[{this.Title} : {this.Text}]";
            }

            public bool Equals(Note other)
            {
                return other != null
                    && this.Text == other.Text
                    && this.Title == other.Title;
            }

            public override bool Equals(object other)
            {
                return Equals(other as Note);
            }

            public override int GetHashCode()
            {
                return this.Title.GetHashCode();
            }
        }

        // CONSTANTS        
        // Name of note title entity
        public const string Entity_Note_Title = "Note.Title";
        // Default note title
        public const string DefaultNoteTitle = "default";
```

### <a name="handle-the-notecreate-intent"></a>处理 Note.Create 意向
若要处理 Note.Create 意向，请将以下代码添加到 `BasicLuisDialog` 类。

```cs
        private Note noteToCreate;
        private string currentTitle;
        [LuisIntent("Note.Create")]
        public Task NoteCreateIntent(IDialogContext context, LuisResult result)
        {
            EntityRecommendation title;
            if (!result.TryFindEntity(Entity_Note_Title, out title))
            {
                // Prompt the user for a note title
                PromptDialog.Text(context, After_TitlePrompt, "What is the title of the note you want to create?");
            }
            else
            {
                var note = new Note() { Title = title.Entity };
                noteToCreate = this.noteByTitle[note.Title] = note;

                // Prompt the user for what they want to say in the note           
                PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");
            }
            
            return Task.CompletedTask;
        }
        
        
        private async Task After_TitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            EntityRecommendation title;
            // Set the title (used for creation, deletion, and reading)
            currentTitle = await result;
            if (currentTitle != null)
            {
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = currentTitle };
            }
            else
            {
                // Use the default note title
                title = new EntityRecommendation(type: Entity_Note_Title) { Entity = DefaultNoteTitle };
            }

            // Create a new note object 
            var note = new Note() { Title = title.Entity };
            // Add the new note to the list of notes and also save it in order to add text to it later
            noteToCreate = this.noteByTitle[note.Title] = note;

            // Prompt the user for what they want to say in the note           
            PromptDialog.Text(context, After_TextPrompt, "What do you want to say in your note?");

        }

        private async Task After_TextPrompt(IDialogContext context, IAwaitable<string> result)
        {
            // Set the text of the note
            noteToCreate.Text = await result;
            
            await context.PostAsync($"Created note **{this.noteToCreate.Title}** that says \"{this.noteToCreate.Text}\".");
            
            context.Wait(MessageReceived);
        }
```

### <a name="handle-the-notereadaloud-intent"></a>处理 Note.ReadAloud 意向
如果未检测到笔记标题，机器人可使用 `Note.ReadAloud` 意向来显示某一笔记或所有笔记的内容。

将以下代码粘贴到 `BasicLuisDialog` 类中。
```cs
        [LuisIntent("Note.ReadAloud")]
        public async Task NoteReadAloudIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                await context.PostAsync($"**{note.Title}**: {note.Text}.");
            }
            else
            {
                // Print out all the notes if no specific note name was detected
                string NoteList = "Here's the list of all notes: \n\n";
                foreach (KeyValuePair<string, Note> entry in noteByTitle)
                {
                    Note noteInList = entry.Value;
                    NoteList += $"**{noteInList.Title}**: {noteInList.Text}.\n\n";
                }
                await context.PostAsync(NoteList);
            }

            context.Wait(MessageReceived);
        }
        
        public bool TryFindNote(string noteTitle, out Note note)
        {
            bool foundNote = this.noteByTitle.TryGetValue(noteTitle, out note); // TryGetValue returns false if no match is found.
            return foundNote;
        }
        
        public bool TryFindNote(LuisResult result, out Note note)
        {
            note = null;

            string titleToFind;

            EntityRecommendation title;
            if (result.TryFindEntity(Entity_Note_Title, out title))
            {
                titleToFind = title.Entity;
            }
            else
            {
                titleToFind = DefaultNoteTitle;
            }

            return this.noteByTitle.TryGetValue(titleToFind, out note); // TryGetValue returns false if no match is found.
        }
```

### <a name="handle-the-notedelete-intent"></a>处理 Note.Delete 意向
将以下代码粘贴到 `BasicLuisDialog` 类中。

```cs
        [LuisIntent("Note.Delete")]
        public async Task NoteDeleteIntent(IDialogContext context, LuisResult result)
        {
            Note note;
            if (TryFindNote(result, out note))
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {                             
                // Prompt the user for a note title
                PromptDialog.Text(context, After_DeleteTitlePrompt, "What is the title of the note you want to delete?");                         
            }           
        }

        private async Task After_DeleteTitlePrompt(IDialogContext context, IAwaitable<string> result)
        {
            Note note;
            string titleToDelete = await result;
            bool foundNote = this.noteByTitle.TryGetValue(titleToDelete, out note);

            if (foundNote)
            {
                this.noteByTitle.Remove(note.Title);
                await context.PostAsync($"Note {note.Title} deleted");
            }
            else
            {
                await context.PostAsync($"Did not find note named {titleToDelete}.");
            }

            context.Wait(MessageReceived);
        }
```

## <a name="build-the-bot"></a>生成机器人
在代码编辑器中右键单击“build.cmd”，然后选择“从控制台运行”。

   ![运行 build.cmd](../media/bot-builder-dotnet-use-luis/bot-service-run-console.png)

## <a name="test-the-bot"></a>测试机器人

在 Azure 门户中，单击“通过网上聊天执行测试”以测试机器人。 尝试键入“创建笔记”、“阅读笔记”和“删除笔记”等消息。
   ![通过网上聊天测试笔记机器人](../media/bot-builder-dotnet-use-luis/bot-service-test-notebot.png)

> [!TIP]
> 如果发现机器人并未能始终识别正确意向或实体，请提供更多示例陈述对其进行训练，从而提高 LUIS 应用的性能。 无需对机器人代码进行任何修改即可重新训练 LUIS 应用。 请参阅[添加示例陈述](/azure/cognitive-services/LUIS/add-example-utterances)和[训练和测试 LUIS 应用](/azure/cognitive-services/LUIS/train-test)。

> [!TIP]
> 如果机器人代码遇到问题，请检查以下内容：
> * 是否已[生成机器人](./bot-builder-dotnet-luis-dialogs.md#build-the-bot)。
> * 机器人代码是否为 LUIS 应用中的每个意向定义了一个处理程序。

## <a name="next-steps"></a>后续步骤

通过试用机器人，可了解 LUIS 意向如何调用任务。 但是，该简单示例不允许中断当前活动的对话。 允许出现“帮助”或“取消”等中断并能有效处理这种情况是一种灵活的设计，可解释用户真正执行的操作。 了解有关使用可评分项对话的详细信息，以便对话可处理中断。

> [!div class="nextstepaction"]
> [使用可评分项的全局消息处理程序](bot-builder-dotnet-scorable-dialogs.md)

## <a name="additional-resources"></a>其他资源

- [对话框](bot-builder-dotnet-dialogs.md)
- [使用对话框管理会话流](bot-builder-dotnet-manage-conversation-flow.md)
- <a href="https://www.luis.ai" target="_blank">LUIS</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[LUIS]: https://www.luis.ai/
[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample
[NotesSampleJSON]: https://github.com/Microsoft/BotFramework-Samples/blob/master/docs-samples/CSharp/Simple-LUIS-Notes-Sample/Notes.json
