---
title: 使用模式语言自定义用户体验 | Microsoft Docs
description: 了解如何在 Bot Builder SDK for .NET 中使用模式语言来自定义 FormFlow 提示和重写 FormFlow 模板。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8c1fe15436a00487483755af06897c4af1c70be2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326494"
---
# <a name="customize-user-experience-with-pattern-language"></a>使用模式语言自定义用户体验

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

自定义提示或重写默认模板时，可以使用模式语言指定提示的内容和/或格式。 

## <a name="prompts-and-templates"></a>提示和模板

[提示][promptAttribute]定义发送给用户以请求信息片段或请求确认的消息。 可以使用 [Prompt 属性](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-prompt-attribute)自定义提示，或者通过 [IFormBuilder<T>.Field][field] 隐式自定义提示。 

窗体使用模板来自动构建提示和其他内容，例如帮助。 可以使用 [Template 属性](bot-builder-dotnet-formflow-advanced.md#customize-prompts-using-the-template-attribute)重写某个类或字段的默认模板。 

> [!TIP]
> [FormConfiguration.Templates][formConfiguration] 类定义一组内置模板，这些模板提供有关如何使用模式语言的极佳示例。

## <a name="elements-of-pattern-language"></a>模式语言的元素

模式语言使用大括号 (`{}`) 来标识要在运行时替换为实际值的元素。 下表列出了模式语言的元素。

| 元素 | Description |
|----|----|
| `{<format>}` | 显示当前字段（该属性应用到的字段）的值。 |
| `{&}` | 显示当前字段的说明（除非另有指定，否则是该字段的名称）。 |
| `{<field><format>}` | 显示命名字段的值。 | 
| `{&<field>}` | 显示命名字段的说明。 |
| <code>{&#124;&#124;}</code> | 显示当前选项，可以是字段的当前值、“无首选项”或枚举的值。 |
| `{[{<field><format>} ...]}` | 显示命名字段中的值列表，该列表中的各个值以 [Separator][separator] 和 [LastSeparator][lastSeparator] 分隔。 |
| `{*}` | 为每个活动字段显示一行；每行包含字段说明和当前值。 | 
| `{*filled}` | 为包含实际值的每个活动字段显示一行；每行包含字段说明和当前值。 |
| `{<nth><format>}` | 应用到模板第 n 个参数的常规 C# 格式说明符。 有关可用参数的列表，请参阅 [TemplateUsage][templateUsage]。 |
| `{?<textOrPatternElement>...}` | 条件替代。 如果引用模式元素的所有对象包含值，则会替代值，并使用整个表达式。 |

对于上面列出的元素：

- `<field>` 占位符是 form 类中的字段名称。 例如，如果类包含名称为 `Size` 的字段，则你可以指定 `{Size}` 作为模式元素。 

- 模式元素中的省略号 (`"..."`) 指示该元素可能包含多个值。

- `<format>` 占位符是 C# 格式说明符。 例如，如果类包含名称为 `Rating`、类型为 `double` 的字段，则你可以指定 `{Rating:F2}` 作为模式元素来以两位数的精度显示数据。

- `<nth>` 占位符引用模板的第 n 个参数。

### <a name="pattern-language-within-a-prompt-attribute"></a>Prompt 属性中的模式语言

此示例使用 `{&}` 元素显示 `Sandwich` 字段的说明，并使用`{||}` 元素显示该字段的选项列表。

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns1)]

生成的提示如下：

```console
What kind of sandwich would you like?
1. BLT
2. Black Forest Ham
3. Buffalo Chicken
4. Chicken And Bacon Ranch Melt
5. Cold Cut Combo
6. Meatball Marinara
7. Oven Roasted Chicken
8. Roast Beef
9. Rotisserie Style Chicken
10. Spicy Italian
11. Steak And Cheese
12. Sweet Onion Teriyaki
13. Tuna
14. Turkey Breast
15. Veggie
>
```

## <a name="formatting-parameters"></a>设置参数格式

提示和模板支持以下格式参数。

| 使用情况 | Description |
|----|----|
| `AllowDefault` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定窗体是否应显示字段的当前值作为可能的选项。 如果为 `true`，则显示当前值作为可能值。 默认为 `true`。 |
| `ChoiceCase` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定是否规范化每个选项的文本（例如，是否将每个单词的第一个字母大写）。 默认为 `CaseNormalization.None`。 有关可能值，请参阅 [CaseNormalization][caseNormalization]。 |
| `ChoiceFormat` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定是要以编号列表还是项目符号列表的形式显示选项列表。 若要使用编号列表，请将 `ChoiceFormat` 设置为 `{0}`（默认值）。 若要使用项目编号列表，请将 `ChoiceFormat` 设置为 `{1}`。 |
| `ChoiceLastSeparator` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定内联选项列表是否在最后一个选项的前面包含分隔符。 |
| `ChoiceParens` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定是否在括号中显示内联选项列表。 如果为 `true`，则在括号中显示选项列表。 默认为 `true`。 |
| `ChoiceSeparator` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定内联选项列表是否在除最后一个选项以外的每个选项的前面包含分隔符。 | 
| `ChoiceStyle` | 适用于 <code>{&#124;&#124;}</code> 模式元素。 确定是要内联还是逐行的形式显示选项列表。 默认值为 `ChoiceStyleOptions.Auto`，确定在运行时是要以内联还是列表形式显示选项。 有关可能值，请参阅 [ChoiceStyleOptions][choiceStyleOptions]。 |
| `Feedback` | 仅适用于提示。 确定窗体是否回显用户的选项，以指示窗体能够识别所做的选择。 默认值为 `FeedbackOptions.Auto`，即，仅当有一部分用户输入内容不可识别时，才回显该内容。 有关可能值，请参阅 [FeedbackOptions][feedbackOptions]。 |
| `FieldCase` | 确定是否规范化字段说明的文本（例如，是否将每个单词的第一个字母大写）。 默认值为 `CaseNormalization.Lower`，即，将说明转换为小写。 有关可能值，请参阅 [CaseNormalization][caseNormalization]。 |
| `LastSeparator` | 适用于 `{[]}` 模式元素。 确定项数组是否在最后一个项的前面包含分隔符。 |
| `Separator` | 适用于 `{[]}` 模式元素。 确定项数组是否在数组中除最后一项以外的每个项的前面包含分隔符。 |
| `ValueCase` | 确定是否规范化字段值的文本（例如，是否将每个单词的第一个字母大写）。 默认值为 `CaseNormalization.InitialUpper`，即，将每个单词的第一个字母转换为大写。 有关可能值，请参阅 [CaseNormalization][caseNormalization]。 |

### <a name="prompt-attribute-with-formatting-parameter"></a>包含格式参数的 Prompt 属性 

此示例使用 `ChoiceFormat` 参数来指定应以项目符号列表的形式显示选项列表。

[!code-csharp[Patterns example](../includes/code/dotnet-formflow-pattern-language.cs#patterns2)]

生成的提示如下：

```console
What kind of sandwich would you like?
* BLT
* Black Forest Ham
* Buffalo Chicken
* Chicken And Bacon Ranch Melt
* Cold Cut Combo
* Meatball Marinara
* Oven Roasted Chicken
* Roast Beef
* Rotisserie Style Chicken
* Spicy Italian
* Steak And Cheese
* Sweet Onion Teriyaki
* Tuna
* Turkey Breast
* Veggie
>
```

## <a name="sample-code"></a>代码示例

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他资源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)
- [使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)
- [本地化窗体内容](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 架构定义窗体](bot-builder-dotnet-formflow-json-schema.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[field]: /dotnet/api/microsoft.bot.builder.formflow.iformbuilder-1.field

[formConfiguration]: /dotnet/api/microsoft.bot.builder.formflow.formconfiguration

[separator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.separator

[lastSeparator]: /dotnet/api/microsoft.bot.builder.formflow.advanced.templatebaseattribute.lastseparator

[templateUsage]: /dotnet/api/microsoft.bot.builder.formflow.templateusage

[caseNormalization]: /dotnet/api/microsoft.bot.builder.formflow.casenormalization

[choiceStyleOptions]: /dotnet/api/microsoft.bot.builder.formflow.choicestyleoptions

[feedbackOptions]: /dotnet/api/microsoft.bot.builder.formflow.feedbackoptions
