---
title: 实时媒体机器人的要求和注意事项 | Microsoft Docs
description: 了解使用 Bot Builder SDK for .NET SDK 创建用于 Skype 的实时媒体机器人的相关重要要求和注意事项。
author: ssulzer
ms.author: ssulzer
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 091c90da9b14c0abe70d08f45f528a3cce818cef
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904031"
---
# <a name="requirements-and-considerations-for-real-time-media-bots"></a>实时媒体机器人的要求和注意事项

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

并非所有适用于开发消息传递和 IVR 呼叫机器人的指南同样适用于构建实时媒体机器人。 本文介绍了开发实时媒体机器人的一些重要要求和注意事项。 

> [!NOTE]
> 由于机器人实时媒体平台是一项预览版技术，因此本文中的指南可能会发生更改。

## <a name="real-time-media-bot-development-requires-cnet-and-windows-server"></a>开发实时媒体机器人要求使用 C# /.NET 和 Windows Server

- 实时媒体机器人需要 `Microsoft.Skype.Bots.Media` .NET 库（可通过 <a href="https://www.nuget.org/" target="_blank">NuGet 获得</a>）来访问音频和视频媒体，机器人必须在 Windows Server 计算机（或在 Azure 中的 Windows Server 来宾 OS）上部署。 因此，应使用 C# 和标准 .NET Framework 来开发机器人并将在 Azure 中部署。 不能使用 C++ 和 Node.js API 来访问实时媒体。

- 实时媒体机器人必须在最新版本的 `Microsoft.Skype.Bots.Media` .NET 库上运行。 机器人应使用 <a href="https://www.nuget.org/" target="_blank">NuGet</a> 包的最新可用版本，或者使用距最新版本不超过三个月的版本。 旧版本的库将被弃用，几个月后将无法使用。

## <a name="real-time-media-calls-stay-on-the-machine-where-they-were-created"></a>实时媒体调用保留在创建它们的计算机上

- 实时媒体机器人是非常有状态的。 实时媒体调用被固定在接受了传入调用的虚拟机 (VM) 实例上：来自 Skype 调用方的语音和视频媒体将流向该 VM 实例，并且机器人发送回调用方的媒体也必须来自该 VM。

- 如果 VM 停止时有任何实时媒体调用正在进行，则这些调用将突然终止。 如果机器人能够知道即将发生的 VM 关闭，它可以尝试“优雅地”结束调用。

## <a name="real-time-media-bots-must-be-directly-accessible-on-the-internet"></a>必须可以在 Internet 上直接访问实时媒体机器人

- 托管的实时媒体机器人的每个 VM 实例必须可以从 Internet 直接访问。 对于托管在 Azure 中的实时媒体机器人，每个 VM 实例必须具有实例级公共 IP 地址 (ILPIP)。 有关获取和配置 ILPIP 的信息，请参阅<a href="/azure/virtual-network/virtual-networks-instance-level-public-ip" target="_blank">实例级公共 IP（经典）概述</a>。 默认情况下，Azure 订阅可以获得 5 个 ILPIP 地址，请联系 Azure 支持来增加订阅配额。

- 由于公共 IP 地址要求，必须在“IaaS”Azure 虚拟机或“经典”Azure 云服务中托管实时媒体机器人。 其他运行时环境（如机器人服务或具有 VM 规模集的 Service Fabric）不受支持，因为这些环境不支持 ILPIP。

- 实时媒体智能机器人也不受 [Bot Framework Emulator](../bot-service-debug-emulator.md) 支持。

## <a name="scalability-and-performance-considerations"></a>可扩展性和性能考虑

- 实时媒体机器人比消息传递机器人需要更多计算和网络（带宽）容量，并且可能会明显产生更高的运营成本。 实时媒体机器人开发人员必须仔细测量机器人的可扩展性，并确保机器人接受的同时调用不超过它所能管理的。 对于每个 CPU 核心，支持视频的机器人可能只能维持一个或两个并发实时媒体调用。

- 机器人实时媒体平台的当前预览版具有一定的可扩展性限制，机器人开发人员必须注意（这些限制可能在以后的版本中得到改进）： 
  1. 单个 VM 实例在任何单个时刻都不能创建超过 10 个的视频套接字。
  2. 实时媒体平台目前没有利用 VM 上可用的任何图形处理单元 (GPU) 来减免 H.264 视频编码/解码工作。 相反，进行视频编码和解码在软件中 CPU 上完成。 如果 GPU 可用，机器人可以利用它进行自己的图形绘制（例如，如果机器人使用 3D 图形引擎）。

- 托管实时媒体机器人的 VM 实例必须至少具有 2 个 CPU 内核。 对于 Azure，建议使用 Dv2 系列虚拟机。 有关 Azure VM 类型的详细的信息，请参阅 <a href="/azure/virtual-machines/windows/sizes-general" target="_blank">Azure 文档</a>。 
