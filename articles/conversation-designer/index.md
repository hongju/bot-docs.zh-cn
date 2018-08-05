---
title: 对话设计器 | Microsoft Docs
description: 了解对话设计器。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 42e9e8137619fff2ca86f82fea1fe81aa348449b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39111551"
---
# <a name="conversation-designer"></a>对话设计器
> [!IMPORTANT]
> 对话设计器尚无法供所有客户使用。 关于对话设计器可用性的更多详细信息将在今年晚些时候发布。

对话设计器是一种强大的框架，可以提供视觉机器人生成器、用于定义机器人的声明性模型，以及简化机器人创建过程的运行时。

若要生成有用的对话机器人，必须使用统一的方法集成对话控件、语言理解、对话和语言生成功能，并始终关注用户体验以及与现有业务逻辑的集成。 使用对话设计器可以通过集成路径来生成机器人，该路径涵盖创作、调试、LUIS、分析、语音识别功能并兼容现有的 Bot Framework SDK 机器人。

使用对话设计器生成的机器人可以轻松适应可用的输入/输出形式，并充分利用以下功能： 

- 强大的运行时，可以尽量减少状态管理开销
- 内置的对话状态，可以轻松地为对话进行视觉建模
- 将代码与机器人 UI 分开
- 对 <a href="https://luis.ai" target="_blank">LUIS</a> 和<a href="https://www.microsoft.com/cognitive-services/en-us/speech-api" target="_blank">语音识别</a>等 AI 框架的内置支持
- 对使用简单的条件响应模板来生成语言的内置支持
- 对[自适应卡](conversation-designer-adaptive-cards.md)的支持

## <a name="prerequisites"></a>先决条件

- 对话设计器需要 Azure 订阅。 可以从<a href="https://azure.microsoft.com/en-us/" target="_blank">这里</a>开始
- 使用它们创建帐户之后，请确保至少登录 [LUIS 门户](https://luis.ai)一次（如果尚未这样做）。
- 熟悉 JavaScript 编程。 自定义脚本函数以 JavaScript 编写。
- Microsoft Edge 或 Google Chrome

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [创建机器人](conversation-designer-create-bot.md)

## <a name="additional-resources"></a>其他资源
详细了解如何使用对话设计器生成机器人：
- [语言理解](conversation-designer-luis.md)：为机器人设置语言理解
- [对话](conversation-designer-dialogues.md)：为机器人设置对话
- [响应模板](conversation-designer-response-templates.md)：关于如何为机器人设置响应模板的快速提示
- [自适应卡](conversation-designer-adaptive-cards.md)：使用自适应卡为机器人设置丰富的视觉交互
