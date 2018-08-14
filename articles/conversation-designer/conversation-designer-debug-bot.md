---
title: 测试机器人 | Microsoft Docs
description: 测试和调试聊天设计器机器人。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: b08bd96bc7c413ff7cb6db4899c8c0d3ad613ab8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298452"
---
# <a name="test-your-conversation-designer-bot"></a>测试聊天设计器机器人
> [!IMPORTANT]
> 聊天设计器尚无法供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

聊天设计器通过几个不同的输出窗口提供了有用的调试工具（位于屏幕底部）。 可单击窗口右上角“测试”按钮开始测试和调试。 

## <a name="validation-errors"></a>验证错误
下面几种情况下会发生验证错误。 例如： 
- 缺少对触发的响应 
- 部分完成聊天流上的信息
- 缺少带条件的应答模板、决定和进程状态的触发器和代码后置
- 模板引用中检测到递归或无限循环 
- 对话具有与聊天流无关的孤立状态。
- 已添加任务，但缺少触发或操作 


## <a name="javascript-output"></a>JavaScript 输出
可将脚本附加到答复的执行以提供其他功能。 如果脚本中存在错误，将在此处显示。 可在机器人的业务逻辑中添加 `console.log()``error.log()` 或 `debug.log()`，这将在输出窗口中列出消息。 例如：

``` javascript
module.exports.Respond_beforeResponse = function(context) {
    console.log(JSON.stringify(context));
}
```

## <a name="runtime-output"></a>运行时输出
此处会显示运行时生成的错误或异常。 例如，如果机器人未能答复，请查看输出，调查导致异常或错误的原因。 如果输出窗口中消息过多，可单击“全部清除”，然后通过向机器人发送消息再次测试机器人，以查看错误详细信息。 

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [导入和导出机器人](conversation-designer-export-import-bot.md)
