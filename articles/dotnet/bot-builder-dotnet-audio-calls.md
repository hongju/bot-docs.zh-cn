---
title: 使用 Skype 进行音频通话 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 通过 Skype 进行音频通话。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 0d0489c23cd24a7323ba0160d5e8e5e914be3011
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998844"
---
# <a name="conduct-audio-calls-with-skype"></a>使用 Skype 进行音频通话

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

支持音频通话的机器人体系结构与典型的机器人非常类似。 下面的代码示例展示了如何启用对使用 Bot Builder SDK for .NET 通过 Skype 进行音频通话的支持。 

## <a name="enable-support-for-audio-calls"></a>启用对音频通话的支持

若要启用机器人对音频通话的支持，请定义 `CallingController`。

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> 除了支持音频通话的 `CallingController`，机器人可能还包含 `MessagesController` 来支持消息。 提供这两个选项使用户能够以他们喜欢的方式与机器人进行交互。 <!-- docs on MessagesController are where? -->

##  <a name="answer-the-call"></a>接听电话

每当用户从 Skype 启动与此机器人的通话，就会执行任务 `ProcessIncomingCallAsync`。
构造函数注册 `IVRBot` 类，该类包含一个预定义的 `incomingCallEvent` 处理程序。

工作流中的第一个操作应确定机器人是响应还是拒绝传入呼叫。 此工作流指示机器人响应传入呼叫，然后播放一条欢迎消息。 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>机器人答复后

如果机器人响应呼叫，工作流中指定的后续操作将指示 Skype 呼叫机器人平台播放提示、录制音频、识别语音，或从拨号盘收集数字。 工作流的最后一个操作可能是结束呼叫。 

此代码示例定义处理程序，该程序将在欢迎消息完成后设置一个菜单。

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

`CreateIvrOptions` 方法定义将向用户显示的菜单。

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

`RecognitionOption` 类定义语音应答以及相应的双音多频 (DTMF) 变体。 DTMF 让用户能够通过在键盘上键入相应的数字（而不是通过语言）来应答。

`OnRecognizeCompleted` 方法处理用户选择，并输入包含用户选择值的参数 `recognizeOutcomeEvent`。

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>支持自然语言
机器人还可以设计为支持自然语言响应。 必应语音 API 使机器人能够识别用户语音回复中的单词。

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>代码示例

有关演示如何使用 Bot Builder SDK for .NET 通过 Skype 支持音频通话的完整示例，请参阅 GitHub 中的 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype 呼叫机器人示例</a>。

## <a name="additional-resources"></a>其他资源

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Skype 呼叫机器人示例 (GitHub)</a>
