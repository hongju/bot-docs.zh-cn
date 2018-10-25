---
title: 使用 FormBuilder 自定义表单 | Microsoft Docs
description: 了解如何使用适用于 Bot Builder SDK for .NET 的 FormBuilder 动态更改和自定义聊天流和内容。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1c4e60f76ecebfa01664500b8343d60ccff0064c
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997694"
---
# <a name="customize-a-form-using-formbuilder"></a>使用 FormBuilder 自定义表单

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[FormFlow 的基本功能](bot-builder-dotnet-formflow.md)介绍了能提供比较通用的用户体验的基本 FormFlow 实现，[FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)则介绍了如何使用业务逻辑和属性自定义用户体验。 本文介绍如何通过指定表单执行步骤的序列并动态定义字段值、确认和消息，使用 [FormBuilder][formBuilder] 更进一步地自定义用户体验。 

## <a name="dynamically-define-field-values-confirmations-and-messages"></a>动态定义字段值、确认和消息

你可使用 FormBuilder 动态定义字段值、确认和消息。

### <a name="dynamically-define-field-values"></a>动态定义字段值 

三明治机器人旨在向要求一英寸长三明治的所有订单添加一份免费的饮品或饼干，它使用 `Sandwich.Specials` 字段来存储免费物品的相关数据。 这样的话，必须根据订单中是否要求一英尺长的三明治，为每个订单动态设置 `Sandwich.Specials` 字段的值。 

`Specials` 字段已指定为可选，“None”则指定为表示无偏好的选项的文本。

[!code-csharp[Field definition](../includes/code/dotnet-formflow-formbuilder.cs#fieldDefinition)]

以下代码示例展示了如何动态设置 `Specials` 字段的值。 

[!code-csharp[Define value](../includes/code/dotnet-formflow-formbuilder.cs#defineValue)]

在本例中，[Advanced.Field.SetType][setType] 方法指定字段类型（`null` 表示枚举字段）。 [Advanced.Field.SetActive][setActive] 方法指定仅在三明治的长度为 `Length.FootLong` 时才启用该字段。 最后，[Advanced.Field.SetDefine][setDefine] 方法指定一个用于定义字段的异步委托。 会向委托传递当前状态对象以及要动态定义的 [Advanced.Field][field]。 委托使用字段的 fluent 方法来动态定义值。 在此示例中，值是字符串，`AddDescription` 和 `AddTerms` 方法指定每个值的说明和对应物品。

> [!NOTE]
> 要动态定义字段值，可自行实现 [Advanced.IField][iField]，也可使用 [Advanced.FieldReflector][FieldReflector] 类简化此过程，如上例所示。 

### <a name="dynamically-define-messages-and-confirmations"></a>动态定义消息和确认

你还可使用 FormBuilder 动态定义消息和确认。 只有在表单中之前的步骤处于非活动状态或已完成时，每个消息和确认才会运行。 

以下代码示例展示了一个自动生成的计算三明治成本的确认消息。 

[!code-csharp[Define confirmation](../includes/code/dotnet-formflow-formbuilder.cs#defineConfirmation)]

## <a name="customize-a-form-using-formbuilder"></a>使用 FormBuilder 自定义表单

以下代码示例使用 FormBuilder 定义表单的步骤、[验证所选内容](bot-builder-dotnet-formflow-advanced.md#add-business-logic)以及[动态定义字段值和确认](#dynamically-define-field-values-confirmations-and-messages)。 默认情况下，表单中的步骤将按列出的序列执行。 但是，对于已包含值的字段或在已指定显式导航的情况下，可能会跳过步骤。 

[!code-csharp[FormBuilder form](../includes/code/dotnet-formflow-formbuilder.cs#formBuilderForm)]

在本例中，表单执行以下步骤：

- 显示一条欢迎消息。 
- 填写 `SandwichOrder.Sandwich`。 
- 填写 `SandwichOrder.Length`。 
- 填写 `SandwichOrder.Bread`。 
- 填写 `SandwichOrder.Cheese`。 
- 填写 `SandwichOrder.Toppings` 并添加缺失值（如果用户已选择 `ToppingOptions.Everything`）。 -. 显示确认所选浇汁的消息。 
- 填写 `SandwichOrder.Sauces`。 
- [动态定义](#dynamically-define-field-values) `SandwichOrder.Specials` 的字段值。 
- [动态定义](#dynamically-define-messages-and-confirmations)三明治成本的确认。 
- 填写 `SandwichOrder.DeliveryAddress` 并[验证](bot-builder-dotnet-formflow-advanced.md#add-business-logic)所生成的字符串。 如果该地址不以数字开头，则表单返回一条消息。 
- 使用自定义提示符填写 `SandwichOrder.DeliveryTime`。 
- 确认订单。 
- 添加已在类中定义但 `Field` 未显式引用的任何剩余字段。 （如果示例未调用 `AddRemainingFields` 方法，则表单不会包括未显式引用的任何字段。） 
- 显示感谢消息。 
- 定义 `OnCompletionAsync` 处理程序以处理订单。 

## <a name="sample-code"></a>代码示例

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他资源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)
- [本地化处理表单内容](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 架构定义窗体](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式语言自定义用户体验](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1

[setType]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.settype

[setActive]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setactive

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[FieldReflector]: /dotnet/api/microsoft.bot.builder.formflow.advanced.fieldreflector-1
