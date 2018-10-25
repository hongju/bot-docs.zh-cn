---
title: 设计机器人的第一个用户交互 | Microsoft Docs
description: 了解怎样才算的上是出色的第一次用户体验以及如何设计机器人以获得成功。
keywords: 第一印象, 开始, 语言和菜单
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f59a7acbdb7d580aebeef6ffe81d8b15505aed90
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999804"
---
# <a name="design-a-bots-first-user-interaction"></a>设计机器人的第一个用户交互

## <a name="first-impressions-matter"></a>第一印象非常重要

用户和机器人之间的第一次交互对用户体验至关重要。 在设计机器人时，请记住，第一条消息不仅仅是说“你好”。 生成应用时，设计第一个屏幕以提供重要的[导航](bot-service-design-navigation.md)提示。用户应该以直观方式了解以下内容：例如，菜单所在位置及其工作原理、可在何处寻求帮助、隐私策略的具体内容等等。 在设计机器人时，用户与机器人的第一次交互应该提供相同类型的信息。 

## <a name="language-versus-menus"></a>语言和菜单 

考虑以下两种设计：

### <a name="design-1"></a>设计 1

![机器人](~/media/bot-service-design-first-interaction/hello1.png)


### <a name="design-2"></a>设计 2

![机器人](~/media/bot-service-design-first-interaction/hello2.png)

一般不推荐使用开放式问题（例如，“有什么能为您效劳的吗？”）启动机器人 。 如果机器人可以执行一百种不同的任务，但用户很可能无法猜测出其中的大部分任务。 机器人没有告诉他们它能执行什么任务，那么，他们怎么可能知道呢？

菜单提供了针对该问题的简单解决方案。 首先，通过列出可用选项，机器人告诉用户其功能。 其次，菜单使用户不必进行过多输入，而只需单击即可。 最后，通过缩小机器人可以从用户接收的输入范围，菜单的使用可以显著简化自然语言模型。 

> [!TIP]
> 在设计机器人以获得出色的用户体验时，菜单是一个十分有用的工具；不要认为它们“不够聪明”而放弃使用菜单。 可以设计机器人使用菜单，同时仍支持自由格式输入。 如果用户通过键入而不是选择菜单选项来响应初始菜单，则机器人可能会尝试解析用户的文本输入。 

或者，如果机器人具有特定功能，那么可以提出更直接的问题来引导用户。 例如，如果机器人负责接受三明治订单，第一次互动可能是“您好！ 您可以点三明治了。 您想要什么样的面包？ 我们有白面包、全麦面包和黑麦面包。” 这样，用户就知道如何响应并通过会话给出导航提示。

## <a name="other-considerations"></a>其他注意事项

除了提供直观且易于导航的第一次交互之外，精心设计的机器人还可以让用户访问有关其隐私政策和使用条款的信息。 

> [!TIP]
> 如果机器人收集用户的个人数据，那么告知这一点并说明将如何处理这些数据非常重要。

## <a name="next-steps"></a>后续步骤

现在，你已经熟悉了设计用户和机器人之间第一次交互的一些基本原则，请了解有关[设计会话流](~/bot-service-design-conversation-flow.md)的详细信息。
