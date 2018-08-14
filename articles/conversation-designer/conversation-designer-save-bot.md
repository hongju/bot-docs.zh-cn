---
title: 保存聊天设计器机器人 | Microsoft Docs
description: 了解如何在聊天设计器机器人中保存和训练语言理解模型并启动语音识别。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 3a911158c379f879c0be604fb5e8ba30ab22ee44
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298428"
---
# <a name="saving-your-conversation-designer-bot"></a>保存聊天设计器机器人
> [!IMPORTANT]
> 聊天设计器尚无法供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

在聊天设计器中工作时，所有工作都缓存在浏览器内存中。 要提交所做的更改，请单击左侧导航菜单左上角的“保存”按钮。 要避免工作丢失，建议经常保存工作。 除了单击“保存”按钮，还可使用 Ctrl+S 键盘快捷方式保存工作。

## <a name="trains-luis-and-primes-speech-recognition"></a>训练 LUIS 并启动语音识别

单击“保存”按钮时，所做更改将保存到机器人中并执行一些其他任务。 与键盘快捷方式不同，单击“保存”按钮还指示聊天设计器执行以下任务：

- 训练机器人了解所有新的 LUIS 意向和实体，并在机器人服务中本地发布 LUIS 模型（如果需要）。 这些意向可以添加到聊天设计器或机器人对应的 [LUIS](https://luis.ai) 应用中。
- 更新聊天运行时以使用新的 LUIS 模型。
- 通过准备并将示例话语发送到 Microsoft 认知服务来启动语音识别，这极大地提高了此机器人语音识别的准确性。

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [测试机器人](conversation-designer-debug-bot.md)
