---
title: 为 Skype 构建实时媒体机器人 | Microsoft Docs
description: 了解如何构建一个机器人，以使用 Bot Builder SDK for .NET 和 Bot Builder-RealTimeMediaCalling SDK for .NET 通过 Skype 进行实时音频/视频通话。
author: MalarGit
ms.author: malarch
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 35aca6f5f50602d0a90c41997eff2e8b1d2cdb4e
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905609"
---
# <a name="build-a-real-time-media-bot-for-skype"></a>为 Skype 构建实时媒体机器人

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

机器人实时媒体平台是一项高级功能，允许机器人逐帧发送和接收语音和视频内容。 机器人具有对语音、视频和屏幕共享实时形式的“原始”访问权限。 本文提供了有关构建音频/视频通话机器人和访问实时形式的概述。

本文中，机器人在 Azure 云服务中作为自托管 ASP.NET Web API 框架的 Web 角色或辅助角色运行。

> [!IMPORTANT]
> 本文只进行了初步介绍；若要了解有关示例实时媒体机器人的完整代码，请参阅 <a href="https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling">BotBuilder RealTimeMediaCalling</a> GitHub 存储库中的“示例”文件夹。

## <a name="configure-the-service-hosting-the-real-time-media-bot"></a>配置托管实时媒体机器人的服务

若要使用实时媒体平台，以下服务配置是必需的。

* 机器人必须知道其服务的完全限定域名 (FQDN)。 Azure RoleEnvironment API 不提供此信息；而是 FQDN 必须存储在机器人的云服务配置中，并在服务启动期间读取。

* 机器人服务必须具有由公认的证书颁发机构颁发的证书。 证书指纹必须存储在机器人的云服务配置中，并在服务启动期间读取。

* 必须预配公共<a href="/azure/cloud-services/cloud-services-enable-communication-role-instances#instance-input-endpoint">实例输入终结点</a>。 这将为机器人服务中的每个虚拟机 (VM) 实例分配一个唯一的公共端口。 此端口被实时媒体平台用来与 Skype 呼叫云进行通信。
  ```xml
  <InstanceInputEndpoint name="InstanceMediaControlEndpoint" protocol="tcp" localPort="20100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="20200" min="20101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

  为呼叫相关的回调和通知创建另一个实例输入终结点也很有用。 使用实例输入终结点可以确保回调和通知传递到服务部署中相同的 VM 实例，该实例承载呼叫的实时媒体会话。
  ```xml
  <InstanceInputEndpoint name="InstanceCallControlEndpoint" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
    <FixedPortRange max="10200" min="10101" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
  ```

* 每个 VM 实例都必须具有实例层级公共 IP 地址 (ILPIP)。 在启动过程中，机器人必须发现分配给每个服务实例的 ILPIP 地址。 有关获取和配置 ILPIP 的详细信息，请参阅 <a href="/azure/virtual-network/virtual-networks-instance-level-public-ip">ILPIP</a>。
  ```xml
  <NetworkConfiguration>
  <AddressAssignments>
    <InstanceAddress roleName="WorkerRole">
    <PublicIPs>
        <PublicIP name="InstancePublicIP" domainNameLabel="InstancePublicIP" />
    </PublicIPs>
    </InstanceAddress>
  </AddressAssignments>
  </NetworkConfiguration>
  ```

* 在服务实例启动期间，脚本 `MediaPlatformStartupScript.bat`（作为 Nuget 包的一部分）需要作为启动任务在提升的权限下运行。 在调用平台的初始化方法之前，必须完成脚本执行。 

```xml
<Startup>
<Task commandLine="MediaPlatformStartupScript.bat" executionContext="elevated" taskType="simple" />      
</Startup> 
```

## <a name="initialize-the-media-platform-on-service-startup"></a>在服务启动时初始化媒体平台

在服务实例启动期间，必须初始化实时媒体平台。 在此实例中，在机器人能够接受任何音频/视频 Skype 通话之前，此操作必须只进行一次。 媒体平台的初始化需要提供各种服务配置设置，包括服务的 FQDN、公共 ILPIP 地址、实例输入终结点端口值和机器人的 Microsoft 应用 ID。

> [!NOTE]
> 若要查找机器人的“AppID”和“AppPassword”，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

```cs
var mediaPlatformSettings = new MediaPlatformSettings()
{
    MediaPlatformInstanceSettings = new MediaPlatformInstanceSettings()
    {
        CertificateThumbprint = certificateThumbprint,
        InstanceInternalPort = instanceMediaControlEndpointInternalPort,
        InstancePublicIPAddress = instancePublicIPAddress,
        InstancePublicPort = instanceMediaControlEndpointPublicPort,
        ServiceFqdn = serviceFqdn
    },

    ApplicationId = MicrosoftAppId
};

MediaPlatform.Initialize(mediaPlatformSettings);            
```

## <a name="register-to-receive-incoming-call-requests"></a>注册以接收传入呼叫请求

定义 `CallController` 类。 这使得机器人服务可以为传入的 Skype 呼叫进行注册，并确保回调和通知请求转发到相应的 `RealTimeMediaCall` 对象。

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallController : ApiController
{
    static CallController()
    {
        RealTimeMediaCalling.RegisterRealTimeMediaCallingBot(
            c => { return new RealTimeMediaCall(c); },
            new RealTimeMediaCallingBotServiceSettings()
        );
    }

    [Route("call")]
    public async Task<HttpResponseMessage> OnIncomingCallAsync()
    {
        // forwards the incoming call to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.IncomingCall);
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> OnCallbackAsync()
    {
        // forwards the incoming callback to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.CallingEvent);
    }

    [Route("notification")]
    public async Task<HttpResponseMessage> OnNotificationAsync()
    {
        // forwards the incoming notification to the associated RealTimeMediaCall object
        return await RealTimeMediaCalling.SendAsync(this.Request, RealTimeMediaCallRequestType.NotificationEvent);
    }
}
```

`RealTimeMediaCallingBotServiceSettings` 实现 `IRealTimeMediaCallServiceSettings` 并为回调和通知事件提供 webhook 链接。

## <a name="register-for-incoming-events-for-the-call"></a>为呼叫注册传入事件

定义用于实现 `IRealTimeMediaCall` 的 `RealTimeMediaCall` 类。 对于由机器人接收的每个呼叫，Bot Framework 创建了一个 `RealTimeMediaCall` 实例。 传递给构造函数的 `IRealTimeMediaCallService` 对象允许机器人注册事件来处理与实时媒体呼叫相关的事件。

```cs
internal class RealTimeMediaCall : IRealTimeMediaCall
{
     public RealTimeMediaCall(IRealTimeMediaCallService callService)
     {
         if (callService == null)
             throw new ArgumentNullException(nameof(callService));

         CallService = callService;
         CorrelationId = callService.CorrelationId;
         CallId = CorrelationId + ":" + Guid.NewGuid().ToString();

         // Register for the call events
         CallService.OnIncomingCallReceived += OnIncomingCallReceived;
         CallService.OnAnswerAppHostedMediaCompleted += OnAnswerAppHostedMediaCompleted;
         CallService.OnCallStateChangeNotification += OnCallStateChangeNotification;
         CallService.OnRosterUpdateNotification += OnRosterUpdateNotification;
         CallService.OnCallCleanup += OnCallCleanup;
     }
}
```

## <a name="create-audio-and-video-sockets"></a>创建音频和视频套接字
在机器人接受传入音频/视频 Skype 通话之前，它必须创建 `AudioSocket` 和 `VideoSocket` 对象，以支持发送和接收实时媒体。 （如果机器人不希望支持视频，则应只创建一个 AudioSocket。）

机器人必须预先决定它想要支持的形式，并创建适当的 AudioSocket 和 VideoSocket 对象。 在接受传入呼叫后，机器人不能更改它支持呼叫的形式。

对于每个 AudioSocket 和 VideoSocket，机器人指定媒体套接字是支持发送和接收媒体，还是只发送或只接收。 例如，机器人可能希望发送和接收音频（“Sendrecv”），但只发送视频（“Sendonly”）。 机器人还必须指定每个媒体套接字支持的媒体格式。 对于 AudioSocket，当前支持的格式是“Pcm16K”：（已签名）16 位 PCM 编码，16 KHz 采样率。 对于 VideoSocket，发送和接收的媒体格式是单独指定的。 接收视频仅支持“NV12”格式，而发送支持多种不同格式。

```cs
_audioSocket = new AudioSocket(new AudioSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    SupportedAudioFormat = AudioFormat.Pcm16K,
    CallId = correlationId
});

_videoSocket = new VideoSocket(new VideoSocketSettings
{
    StreamDirections = StreamDirection.Sendrecv,
    ReceiveColorFormat = VideoColorFormat.NV12,
    SupportedSendVideoFormats = new VideoFormat[]
    {
        VideoFormat.Yuy2_1280x720_30Fps,
        VideoFormat.Yuy2_720x1280_30Fps,
    },
    CallId = correlationId
});

_audioSocket.AudioMediaReceived += OnAudioMediaReceived;
_audioSocket.AudioSendStatusChanged += OnAudioSendStatusChanged;
_audioSocket.DominantSpeakerChanged += OnDominantSpeakerChanged;
_videoSocket.VideoMediaReceived += OnVideoMediaReceived;
_videoSocket.VideoSendStatusChanged += OnVideoSendStatusChanged;
```                             

## <a name="create-a-mediaconfiguration"></a>创建 MediaConfiguration
创建媒体套接字后，接下来机器人必须创建 `MediaConfiguration` 对象，需要使用该对象将媒体套接字与传入音频/视频 Skype 通话联系起来。

```cs
var mediaConfiguration = MediaPlatform.CreateMediaConfiguration(_audioSocket, _videoSocket);
```

##  <a name="answer-an-incoming-audiovideo-call"></a>传入音频/视频通话应答
调用 `OnIncomingCallReceived` 事件可以让机器人接受传入的 Skype 音频/视频通话。 若要执行此操作，机器人需创建带 `MediaConfiguration` 对象的 `AnswerAppHostedMedia` 对象。 机器人注册与此 Skype 通话相关的通知。

```cs
private Task OnIncomingCallReceived(RealTimeMediaIncomingCallEvent incomingCallEvent)
{
    // ... create Audio/VideoSocket objects and MediaConfiguration ...

    incomingCallEvent.RealTimeMediaWorkflow.Actions = new ActionBase[]
    {
        new AnswerAppHostedMedia
        {
            MediaConfiguration = mediaConfiguration,
            OperationId = Guid.NewGuid().ToString()
        }
    };

    // subscribe for roster and call state changes
    incomingCallEvent.RealTimeMediaWorkflow.NotificationSubscriptions = new NotificationType[]
    {
        NotificationType.CallStateChange,
        NotificationType.RosterUpdate
    };
}
```

## <a name="outcome-of-the-call"></a>呼叫结果
当完成 `AnswerAppHostedMedia` 操作，将引发 `OnAnswerAppHostedMediaCompleted`。 `AnswerAppHostedMediaOutcomeEvent` 中的 `Outcome` 属性指示成功或失败。 如果无法建立呼叫，机器人应处理它为呼叫创建的 AudioSocket 和 VideoSocket 对象。

## <a name="receive-audio-media"></a>接收音频媒体
如果创建的 `AudioSocket` 能够接收音频，则将在每次接收音频帧时调用 `AudioMediaReceived` 事件。 机器人应该能够以每秒 50 次的速度处理该事件，而不考虑任何可以提供音频内容的对等点（因为如果没有从对等点接收到音频，会在本地生成舒适噪音缓冲区）。 每个包的音频内容都在 `AudioMediaBuffer` 对象中提供。 此对象包含一个指向本机堆分配的内存缓冲区的指针，其中包含解码音频内容。 

```cs
void OnAudioMediaReceived(
            object sender,
            AudioMediaReceivedEventArgs args)
{
   var buffer = args.Buffer;

   // native heap-allocated memory containing decoded content
   IntPtr rawData = buffer.Data;            
}
```

事件处理程序必须快速返回。 建议应用程序对 `AudioMediaBuffer` 进行排列以便以异步方式进行处理。 `OnAudioMediaReceived` 事件将由实时媒体平台序列化（也就是说，在当前事件返回之前不会引发下一个事件）。 一旦使用了 `AudioMediaBuffer`，应用程序就必须调用缓冲区的 Dispose 方法，以便基础非托管内存可以被媒体平台回收。 

```cs
   // release/dispose buffer when done 
   buffer.Dispose();
```

> [!IMPORTANT]
> 在完成缓冲区访问之前，机器人不得调用缓冲区的 Dispose 方法。

## <a name="receive-video-media"></a>接收视频媒体
接收视频类似于接收音频，只不过每秒的缓冲区数将取决于帧速率的值。 `VideoMediaBuffer` 具有 `VideoFormat` 和 `OriginalVideoFormat` 属性。 当提供 `OriginalVideoFormat` 时，它表示缓冲区的原始格式。 只有通过 `IVideoSocket.VideoMediaReceived` 事件处理程序接收视频缓冲区时才可用。 如果缓冲区在传输之前已调整大小，`OriginalVideoFormat` 属性将具有原始的 Width 和 Height，而 `VideoFormat` 将具有重设大小后的当前 Width 和 Height。 如果 `OriginalVideoFormat` 的 Width 和 Height 属性与 `VideoFormat` 属性不同，`VideoMediaReceived` 事件中引发的 `VideoMediaBuffer` 的使用者应调整缓冲区大小以适应 `OriginalVideoFormat` 大小。 对于接收，目前仅支持 NV12 格式。

## <a name="send-audio-media"></a>发送音频媒体
如果 `AudioSocket` 配置为发送媒体，机器人应注册 `AudioSocket` 上的 `AudioSendStatusChanged` 事件处理程序，以获取有关发送状态更改的通知。 机器人应仅在发送状态更改为“活动”之后开始发送音频。

```cs
void AudioSocket_OnSendStatusChanged(
             object sender,
             AudioSendStatusChangedEventArgs args)
{
    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

若要发送音频媒体，我们假定 PCM 音频内容包含在一个本机堆分配缓冲区中。 机器人必须派生自 `AudioMediaBuffer` 抽象类。 媒体通过 AudioSocket 的 `Send` 方法以异步方式发送，发送完成时，平台将调用 `AudioMediaBuffer` 上的 `Dispose`。 返回 `Send` 时，机器人不应发布（或返回到池分配器）非托管资源。 它必须等待调用 `Dispose`。

## <a name="send-video-media"></a>发送视频媒体
发送视频媒体类似于音频媒体。 机器人应注册 `VideoSendStatusChanged` 事件并等待 `MediaSendStatus` 变为 `Active`。 在调用 `Dispose` 方法之前，机器人不得发布或回收缓冲区的非托管资源。 支持 RGB24、NV12 和 Yuy2 颜色格式。

虽然有多个支持发送视频的 `VideoFormat`，但目前首选的 `VideoFormat` 是通过 `VideoSendStatusChanged` 事件与机器人通信。 发送视频的首选 `VideoFormat` 可能会由于网络带宽条件或对等客户端调整其视频窗口大小而改变。

```cs
void VideoSocket_OnSendStatusChanged(
            object sender,
            VideoSendStatusChangedEventArgs args)
{
    VideoFormat preferredVideoFormat;

    switch (args.MediaSendStatus)
    {
    case MediaSendStatus.Active:
        // notify bot to begin sending audio 
        // bot is recommended to use this format for sourcing video content.
        preferredVideoFormat = args.PreferredVideoSourceFormat;
        break;
     
    case MediaSendStatus.Inactive:
        // notify bot to stop sending audio
        break;
    }
}
```

每个 `VideoMediaBuffer` 都具有 VideoFormat 属性，指示该特定缓冲区的视频内容的格式。 虽然 `VideoFormat` 属性不需要匹配 `VideoSendStatusChanged` 事件中指示的 `PreferredVideoSourceFormat` 属性，但强烈建议使用所指示的 `PreferredVideoSourceFormat` 以避免在视频帧重设大小上浪费 CPU 周期。

## <a name="call-notifications"></a>来电通知
### <a name="call-state-changes"></a>通话状态变化
机器人可以通过订阅 `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow` 的 `NotificationSubscriptions` 中的 `NotificationType.CallStateChange` 来获取通话状态更改通知。

```cs
private Task OnCallStateChangeNotification(CallStateChangeNotification callStateChangeNotification)
{
    if (callStateChangeNotification.CurrentState == CallState.Terminated)
    {   
        // stop sending media and dispose the media sockets
        _audioSocket.Dispose();
        _videoSocket.Dispose();
    }

    return Task.CompletedTask;
}
 ```

### <a name="roster-update"></a>名册更新
如果将机器人添加到会议，它可以通过订阅 `RealTimeMediaIncomingCallEvent.RealTimeMediaWorkflow` 的 `NotificationSubscriptions` 中的 `NotificationType.RosterUpdate` 来侦听会议名册。 `RosterUpdateNotification` 拥有会议中所有参与者的详细信息。 机器人可以选择等待参与者发送视频通知，然后在它的一个 `VideoSocket` 对象上 `Subscribe` 参与者视频。

```cs
private async Task OnRosterUpdateNotification(RosterUpdateNotification rosterUpdateNotification)
{
    // Find a video source in the conference to subscribe
    foreach (RosterParticipant participant in rosterUpdateNotification.Participants)
    {
        if (participant.MediaType == ModalityType.Video
            && (participant.MediaStreamDirection == "sendonly" || participant.MediaStreamDirection == "sendrecv")
            )
        {
            var videoSubscription = new VideoSubscription
            {
                ParticipantIdentity = participant.Identity,
                OperationId = Guid.NewGuid().ToString(),
                SocketId = _videoSocket.SocketId,
                VideoModality = ModalityType.Video,
                VideoResolution = ResolutionFormat.Hd720p
            };

            await CallService.Subscribe(videoSubscription).ConfigureAwait(false);
            break;
        }
    }  
}
```

## <a name="end-the-call"></a>结束通话

### <a name="handle-call-termination-from-the-caller"></a>处理来自呼叫方的呼叫终止
当用户在对等会话中断开与机器人的通话，或者当机器人从会议中移除时，机器人会收到呼叫状态变更通知，指示 CallState 为终止状态。 机器人应处置它所创建的任何媒体套接字。

### <a name="terminate-the-call-from-the-bot"></a>终止来自机器人的呼叫
机器人可以通过调用 `IRealTimeMediaCallService` 上的 `EndCall` 选择结束通话。

### <a name="handle-call-clean-up-by-the-bot-framework"></a>由 Bot Framework 处理通话清理
在错误情况下（例如，如果未在合理时间内收到 `AnswerAppHostedMediaOutcomeEvent`），Bot Framework 可能会终止通话。 机器人应注册 `OnCallCleanup` 事件并处置媒体套接字。

