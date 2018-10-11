---
title: 发送主动消息 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 发送主动消息。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c00913c4e6fa7133dad734e69167d2723257f75a
ms.sourcegitcommit: 6c2426c43cd2212bdea1ecbbf8ed245145b3c30d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/08/2018
ms.locfileid: "48852302"
---
# <a name="send-proactive-messages"></a>发送主动消息

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>主动消息的类型 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>发送临时主动消息

下面的代码示例演示如何使用 Bot Builder SDK for .NET 发送临时主动消息。

为了能够向用户发送临时消息，机器人必须先从当前聊天中收集和存储一些有关用户的信息。 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> 为简单起见，此示例未指定如何存储用户数据。 只要机器人稍后能够检索数据，数据的存储方式无关紧要。

现在数据已经存储，机器人便可检索数据，构造临时主动消息并发送消息。 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> 如果机器人指定以前存储的聊天 ID，则消息可能会在客户端的现有聊天窗口中传递给用户。 如果机器人生成新的聊天 ID，只要客户端支持多个聊天窗口，就会在客户端的新聊天窗口中将消息传递给用户。 

## <a name="send-a-dialog-based-proactive-message"></a>发送基于对话的主动消息

下面的代码示例演示如何使用 Bot Builder SDK for .NET 发送基于对话的主动消息。

为了能够向用户发送基于对话的主动消息，机器人必须先从当前聊天中收集和保存信息。 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

需要发送消息时，机器人将创建一个新的对话，并将其添加到对话堆栈的顶部。 新的对话将成为主角，传递主动消息，然后关闭，再将控制权返回给堆栈中的上一对话。 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
`SurveyDialog` 控制聊天，直至结束。 其任务完成后，它将调用 `context.Done`然后关闭，将控制返回给上一对话。 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>代码示例

有关演示如何使用 Bot Builder SDK for .NET 发送主动消息的完整示例，请参阅 GitHub 中的<a href="https://aka.ms/proactive-messaging-cs-v3 " target="_blank">主动消息示例</a>。 在主动消息示例中，<a href="https://aka.ms/proactive-sendmessage-cs-v3 " target="_blank">simpleSendMessage</a> 演示了如何发送临时主动消息，而 <a href="https://aka.ms/proactive-newdialog-cs-v3 " target="_blank">startNewDialog</a> 演示了如何发送基于对话的主动消息。 

## <a name="additional-resources"></a>其他资源

- [设计和控制聊天流](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">主动消息示例 (GitHub)</a>

