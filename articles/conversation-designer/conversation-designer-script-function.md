---
title: 将脚本函数定义为“执行”操作 | Microsoft Docs
description: 了解如何将脚本操作函数设置为“执行”操作。
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 674781c2cb5a8700941feb59b2ce7ab902979d4f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298420"
---
# <a name="define-a-script-function-as-a-do-action"></a>将脚本函数定义为“执行”操作

[脚本操作函数](conversation-designer-context-object.md#script-callback-functions)会执行你定义的自定义脚本定义来帮助完成任务。 例如，如果用户说“将恒温器设置为 74 度”，则会调用服务将恒温器实际设置为 74 度。 

要将自定义脚本用作操作，请选择“脚本操作”作为任务编辑器中的“执行”操作类型。 然后输入实现该操作的函数名称。 单击“编辑”以调出脚本编辑器，然后编写此函数的实现。 

## <a name="script-action-function-parameter"></a>脚本操作函数参数

始终将使用 [`context`](conversation-designer-context-object.md) 对象调用指定的操作回调函数。

`context` 对象包括 `taskEntities` 和 `contextEntities`。 任务实体是为此任务定义或生成的实体，上下文实体是可在与用户的聊天中保留实体的属性包。

## <a name="return-value"></a>返回值
脚本操作函数应返回一个布尔值。

## <a name="sample-script-action-function"></a>脚本操作函数示例
在返回匹配的触发前，以下示例函数会进行 HTTP 调用以获得响应。

```javascript
module.exports.NewTask_do_onRun = function(context) {
    var options =  {
        host: 'HOST',
        path: 'PATH',
        method: 'post',
        headers: {
            "HEADER1" : "VALUE"
        }, 
        body: {
            "BODY": VALUE
        }
    };

    return http.request(options).then(function(response) {
      // parse response payload
      // Send a message to user
      context.responses.push({text: "Done", type: "message"});
    });
} 
```

## <a name="next-step"></a>后续步骤
> [!div class="nextstepaction"]
> [对话操作](conversation-designer-dialogues.md)
