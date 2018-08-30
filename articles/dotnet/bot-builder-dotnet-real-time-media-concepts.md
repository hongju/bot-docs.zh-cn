---
title: 使用 Skype 进行实时媒体通话 | Microsoft Docs
description: 理解通过 Bot Builder SDK for .NET 构建可以使用 Skype 进行实时音频和视频通话的机器人的关键概念。
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: eed4ac20664d15fbc40969551a1f23fc91b4f120
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906221"
---
# <a name="real-time-media-calling-with-skype"></a>使用 Skype 进行实时媒体通话

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

适用于机器人的实时媒体平台通过实现实时语音、视频和屏幕共享形式为机器人与用户互动的方式增加了一个新的维度。 它提供构建引人注目以及交互式娱乐、教育和辅助机器人的能力。 用户使用 Skype 与实时媒体机器人进行通信。

这是一个高级功能，允许机器人逐帧发送和接收语音和视频内容。 机器人具有对语音、视频和屏幕共享实时形式的“原始”访问权限。 例如，当用户说话时，机器人将每秒接收 50 个音频帧，每帧包含 20 毫秒 (ms) 的音频。 机器人可以在收到音频帧时执行实时语音识别，而不必等用户停止说话之后再开始录制。 机器人还可以以每秒 30 帧的速度向用户发送高清分辨率视频以及音频。

适用于机器人的实时媒体平台与 Skype 呼叫云协同工作，负责通话建立和媒体会话建立，使机器人能够与 Skype 呼叫方进行语音和（可选）视频会话。 该平台为机器人提供了一个简单的像“套接字”这样的 API 以发送和接收媒体，并使用用于音频的 SILK 和用于视频的 H.264 等编解码器来处理媒体的实时编码和解码。 实时媒体机器人还可以参与与多个参与者一起进行的 Skype 群组视频通话。

本文介绍了与构建可以使用 Skype 进行实时音频和视频通话的机器人相关的关键概念，并提供了指向相关开发者资源的链接。

## <a name="media-session"></a>媒体会话
当实时媒体机器人应答传入的 Skype 呼叫时，它决定是支持音频和视频形式，还是仅支持音频。 对于支持的每种形式，机器人可以同时发送和接收媒体，也可以仅接收或仅发送。 例如，机器人可能希望同时发送和接收音频，但只发送视频（因为它不想接收 Skype 呼叫方的视频）。 在 Skype 呼叫方和机器人之间建立的这组音频和视频形式称为媒体会话。

支持两种视频形式，分别是“主要视频”和“屏幕共享”。 主要视频将机器人生成或播放的视频传输给呼叫方，并将呼叫方的视频（通常来自用户的网络摄像头）传输给机器人。 屏幕共享形式允许呼叫方与机器人共享他或她的屏幕（作为视频）。 机器人无法向用户发送屏幕共享视频。

加入多方 Skype 群组视频通话时，实时媒体机器人可以支持同时接收多个主要视频流。 这让机器人能够“看到”群组视频通话中的多个参与者。

## <a name="frames-and-frame-rate"></a>帧和帧速率
实时媒体机器人直接与 Skype 通话的音频和视频形式进行交互。 这意味着机器人正在作为帧序列发送和/或接收媒体，其中每个帧代表一个内容单元。 一秒音频可以作为 50 帧的序列传输，每帧包含 20 毫秒 (ms)（1/50 秒）的内容。 一秒钟的视频可被切片成 30 个静止图像的序列，在显示下一个视频帧之前，每个静止图像仅供观看 33 ms（1/30 秒）。 每秒传输或渲染的帧数称为帧速率。 “30fps”表示每秒 30 帧。

## <a name="audio-format"></a>音频格式
每秒音频表示为 16,000 个样本，每个样本存储 16 位数据。 20 ms 音频帧包含 320 个样本（640 字节数据）。

## <a name="video-format"></a>视频格式
支持多种视频格式。 视频格式的两个关键属性是其“帧大小”和“颜色格式”。 支持的帧大小包括 640x360（“360p”）、1280x720（“720p”）和 1920x1080（“1080p”）。 支持的颜色格式包括 NV12（每像素 12 位）和 RGB24（每像素 24 位）。

“720p”视频帧包含 921,600 像素 (1280 × 720)。 在 RGB24 颜色格式中，每个像素表示为 3 个字节（24 位），每个字节由红色、绿色和蓝色分量组成。 因此，单个 720p RGB24 视频帧需要 2,764,800 字节的数据（921,600 像素 × 3 字节/像素）。 在帧速率为 30fps 时，发送 720p RGB24 视频帧意味着处理大约 80 MB/s 的内容（在网络传输之前实质上是由 H.264 视频编解码器压缩）。

## <a name="active-and-dominant-speakers"></a>活跃和主导的说话人
当加入由多个参与者组成的 Skype 群组视频通话时，机器人可以识别当前正在说话的参与者。 活跃说话人用于标识在每个收到的音频帧中正在被倾听的参与者。 主导说话人用于标识群组会话中当前最活跃（或“主导”）的参与者，即使在每个音频帧中他们的声音可能不被听到。 由于不同的参与者轮流说话，这组主导说话人可能会改变。

## <a name="video-subscription"></a>视频订阅
在与单个 Skype 呼叫方的通话中，机器人将自动接收呼叫方的视频（如果机器人已被启用接收主要视频）。 在多方 Skype 群组视频通话中，机器人必须向实时媒体平台指出它想要看到哪些参与者。 视频订阅是机器人发出的用于接收参与者的视频的请求。 当群组视频通话中的参与者进行会话时，机器人可以基于主导说话人组的更新来修改其所需的视频订阅。

## <a name="developer-resources"></a>开发人员资源 

### <a name="sdks"></a>SDK

若要开发实时媒体机器人，必须在 Visual Studio 项目中安装这些 NuGet 包：

- [Bot Builder SDK for .NET](bot-builder-dotnet-overview.md)
- [适用于 .NET 的 Bot Builder 实时媒体通话](https://www.nuget.org/packages?q=Bot.Builder.RealTimeMediaCalling)
- [Microsoft.Skype.Bots.Media .NET 库](https://www.nuget.org/packages?q=Microsoft.Skype.Bots.Media)

### <a name="code-samples"></a>代码示例

[BotBuilder-RealTimeMediaCalling](https://github.com/Microsoft/BotBuilder-RealTimeMediaCalling) GitHub 存储库包含说明如何构建适用于 Skype 的实时媒体机器人的示例。
