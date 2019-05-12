---
title: 关于 Direct Line 通道
titleSuffix: Bot Service
description: Direct Line 通道功能
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.subservice: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: v-ivorb
ms.openlocfilehash: 0b108d90d18261cd22214db9a7926bdac1bfee40
ms.sourcegitcommit: a4181f35dbe6a8b107eea28122372f524e19880a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/02/2019
ms.locfileid: "65030190"
---
## <a name="about-direct-line"></a>关于 Direct Line

可以通过 Bot Framework Direct Line 通道轻松地将机器人集成到移动应用、网页或其他应用程序中。
Direct Line 以三种形式提供：
- Direct Line 服务 - 一种全局的功能强大的服务，适用于大多数开发人员
- Direct Line 应用服务扩展 - 专用的 Direct Line 功能，可以确保安全性和性能（个人预览版从 2019 年 5 月开始发布）
- Direct Line 语音 - 已针对高性能语音进行优化（个人预览版从 2019 年 5 月开始发布）

可以评估每个套餐提供的功能以及解决方案的需求，选择最适合自己的 Direct Line 套餐。 一段时间过后，这些套餐会被简化。

|                            | Direct Line | Direct Line 应用服务扩展 | Direct Line 语音 |
|----------------------------|-------------|-----------------------------------|--------------------|
| GA 可用性和 SLA    | 正式版 | 个人预览版，无 SLA  | 个人预览版，无 SLA |
| 语音识别和文本转语音性能 | 标准 | 标准 | 高性能 |
| 集成 OAuth           | 是 | 是 | 否 |
| 集成遥测       | 是 | 是 | 否 |
| 支持旧版 Web 浏览器 | 是 | 否 | 否 |
| Bot Framework SDK 支持 | 所有 v3、v4 | 需要 v4.5+ | 需要 v4.5+ |
| 客户端 SDK 支持    | JS、C# | JS、C# | C++、C#、Unity |
| 适用于 Web 聊天  | 是 | 是 | 否|
| 聊天历史记录 API | 是 | 是| 否|
| VNET | 否 | 预览版* | 否 |

_* Direct Line 应用服务扩展可以用在 VNET 中，但是仍然不允许限制出站调用。_

## <a name="addtional-resources"></a>其他资源
- [将机器人连接到 Direct Line](bot-service-channel-connect-directline.md)
- [将机器人连接到 Direct Line 语音](bot-service-channel-connect-directlinespeech.md)
