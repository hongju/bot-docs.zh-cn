---
title: 配置自适应卡 | Microsoft Docs
description: 了解如何配置自适应卡。
author: vkannan
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: fee70da7288b3214ff7f384998a69b40f91b3226
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464520"
---
# <a name="configure-adaptive-cards"></a>配置自适应卡
> [!IMPORTANT]
> 聊天设计器尚未可供所有客户使用。 关于聊天设计器可用性的更多详细信息将在今年晚些时候发布。

<a href="http://adaptivecards.io" target="_blank">自适应卡</a>是一种新架构，用于定义丰富的 UI 卡，以便在包括 Microsoft Bot Framework 通道在内的多个不同终结点中使用。 

聊天设计器提供了一个深度集成的创作环境，用于在机器人中创作、预览和使用自适应卡。 

可以在几个不同的关键位置定义自适应卡。

- 对任务操作的简单响应。
- 在对话的反馈状态中。
- 在对话的提示状态中。 请注意，提示可以使用单独的卡：一个用于响应，另一个用于重新提示。

若要定义自适应卡，请导航到相关的编辑器。 浏览并选择现有的自适应卡模板之一，或在 JSON 代码编辑器中构建自己的模板。 

构建卡时，会在创作门户中呈现卡的丰富预览。

> [!NOTE]
> 自适应卡的功能仍在不断开发中。 目前，所有通道都并非支持所有自适应卡功能。 若要查看每个通道支持的功能，请参阅“通道状态”部分。

## <a name="input-form"></a>输入窗体

自适应卡可以包含输入窗体。 在“聊天设计器”中，这些窗体与任务实体集成在一起。 例如，如果某个字段的 `id` 为 **myName** 并且执行了窗体 `Submit` 操作，则会创建名为 **myName** 的 `taskEntity`，其中包含该字段的值。 

下面的代码片段显示了如何在代码中定义 **myName** 实体：

```javascript
{
   "type": "Input.Text",
   "id": "myName",
   "placeholder": "Last, First"
}
```

此外，如果字段的 id 为 `@task`，则该字段的值将用作任务名称。 触发此字段时（例如：单击按钮），将执行指定的任务。 

以此代码片段为例：

```javascript
{
  'type': 'Action.Submit',
  'title': 'Search',
  'speak': '<s>Search</s>',
  'data': {
    '@task': 'Hotel Search'
  }
}
```

单击此按钮时，将触发提交操作，`context.sticky` 将设置为 `Hotel Search`。 这将导致执行“酒店搜索”  任务。 若要使用此功能，请确保 `@task` 与在聊天设计器中定义的任务名称相匹配。

## <a name="use-entities-and-language-generation-templates"></a>使用实体和语言生成模板
自适应卡支持全语言生成解决方案。

* `entityName` 使用卡内的实体。
* `responseTemplateName` 使用卡内的简单或条件响应模板。

可在此处了解有关自适应卡片的详细信息  TODO：在自适应卡片架构文档中插入链接 -->

## <a name="sample-adaptive-card-payload"></a>示例自适应卡有效负载

以下 JSON 显示了自适应卡的有效负载。

```json
{
    "$schema": "https://microsoft.github.io/AdaptiveCards/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.0",
    "body": [
        {
            "speak": "<s>Serious Pie is a Pizza restaurant which is rated 9.3 by customers.</s>",
            "type": "ColumnSet",
            "columns": [
                {
                    "type": "Column",
                    "size": "2",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "[Greeting], [TimeOfDayTemplate], You can eat in {location}",
                            "weight": "bolder",
                            "size": "extraLarge"
                        },
                        {
                            "type": "TextBlock",
                            "text": "9.3 · $$ · Pizza",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "text": "[builtin.feedback.display]",
                            "wrap": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "size": "1",
                    "items": [
                        {
                            "type": "Image",
                            "url": "http://res.cloudinary.com/sagacity/image/upload/c_crop,h_670,w_635,x_0,y_0/c_scale,w_640/v1397425743/Untitled-4_lviznp.jpg",
                            "size":"auto"
                        }
                    ]
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "More Info",
            "url": "http://foo.com"
        },
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "View on Foursquare",
            "url": "http://foo.com"
        }
    ]
}
```

