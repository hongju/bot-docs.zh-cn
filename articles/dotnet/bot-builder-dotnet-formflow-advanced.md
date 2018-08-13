---
title: FormFlow 的高级功能 | Microsoft Docs
description: 了解如何使用 FormFlow 和用于 .NET 的 Bot Builder SDK 自定义用户体验。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8860c6b3fa0eaaf9f9bfc92984a501c066f2fe5c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297062"
---
# <a name="advanced-features-of-formflow"></a>FormFlow 的高级功能

[FormFlow 的基本功能](bot-builder-dotnet-formflow.md)介绍了基本的 FormFlow 实现，提供了相当通用的用户体验。 若要使用 FormFlow 提供自定义程度更高的用户体验，可以指定初始窗体状态、添加业务逻辑来管理字段之间的相互依赖关系并处理用户输入，以及使用属性来自定义提示、重写模板、指定可选字段、匹配用户输入和验证用户输入。 

## <a name="specify-initial-form-state-and-entities"></a>指定初始窗体状态和实体

启动 [FormDialog][formDialog] 时，可以选择传入一个状态实例。 如果确实传入了一个状态实例，则默认情况下，FormFlow 会跳过已包含值的字段的步骤；系统不会提示用户输入这些字段的值。 若要强制窗体提示用户输入所有字段（包括那些在初始状态中已包含值的字段）的值，请在启动 `FormDialog` 时传入 [FormOptions.PromptFieldsWithValues][promptFieldsWithValues]。 如果某个字段包含初始值，则提示会使用该值作为默认值。

也可传入要绑定到该状态的 [LUIS](https://luis.ai/) 实体。 如果 `EntityRecommendation.Type` 是 C# 类中某个字段的路径，则会通过要绑定到字段的识别器传入 `EntityRecommendation.Entity`。 FormFlow 会跳过已绑定到某个实体的字段的步骤；系统不会提示用户输入这些字段的值。 

## <a name="add-business-logic"></a>添加业务逻辑 

在获取或设置字段值的过程中，若要处理窗体字段之间的相互依赖关系或应用特定的逻辑，可以在验证函数中指定业务逻辑。 验证函数用于操作状态并返回一个 [ValidateResult][validateResult] 对象，该对象可能包含： 

- 一个反馈字符串，用于描述值无效的原因
- 一个转换后的值
- 一组用于澄清某个值的选择

此代码示例演示 `Toppings` 字段的验证函数。 如果字段的输入包含 `ToppingOptions.Everything` 枚举值，则此函数会确保 `Toppings` 字段值包含浇汁的完整列表。

[!code-csharp[Validation function](../includes/code/dotnet-formflow-advanced.cs#validationFunction)]

除了验证函数，还可以添加 [Term](#match-user-input-using-the-terms-attribute) 属性来匹配用户表述（例如“一切”或“不”）。

[!code-csharp[Terms for Toppings](../includes/code/dotnet-formflow-advanced.cs#toppingsTerms)]

此代码片段使用上述验证函数演示了当用户请求“除墨西哥辣椒之外的一切”时机器人和用户之间的交互。 

```console
Please select one or more toppings (current choice: No Preference)
 1. Everything
 2. Avocado
 3. Banana Peppers
 4. Cucumbers
 5. Green Bell Peppers
 6. Jalapenos
 7. Lettuce
 8. Olives
 9. Pickles
 10. Red Onion
 11. Spinach
 12. Tomatoes
> everything but jalapenos
For sandwich toppings you have selected Avocado, Banana Peppers, Cucumbers, Green Bell Peppers, Lettuce, Olives, Pickles, Red Onion, Spinach, and Tomatoes.
```

## <a name="formflow-attributes"></a>FormFlow 属性

可以将以下 C# 属性添加到类，以便自定义 FormFlow 对话的行为。

| 属性 | 目的 |
|----|----| 
| [Describe][describeAttribute] | 更改某个字段或值在模板或卡中的显示方式 |
| [Numeric][numericAttribute] | 限制数字字段接受的值 |
| [Optional][optionalAttribute] | 将字段标记为可选 |
| [Pattern][patternAttribute] | 定义一个用于验证字符串字段的正则表达式 |
| [Prompt][promptAttribute] | 定义用于某个字段的提示 |
| [Template][templateAttribute] | 定义一个模板，用于生成提示或提示中的值 |
| [Terms][termsAttribute] | 定义与字段或值匹配的输入术语 |

## <a name="customize-prompts-using-the-prompt-attribute"></a>使用 Prompt 属性自定义提示

窗体中每个字段的默认提示是自动生成的，但可使用 `Prompt` 属性为任何字段指定自定义提示。 例如，如果 `SandwichOrder.Sandwich` 字段的默认提示为“请选择三明治”，则可添加 `Prompt` 属性，以便为该字段指定自定义提示。

[!code-csharp[Prompt attribute](../includes/code/dotnet-formflow-advanced.cs#promptAttribute)]

此示例使用[模式语言](bot-builder-dotnet-formflow-pattern-language.md)在运行时使用窗体数据对提示进行动态填充：`{&}` 替换为字段的说明，`{||}` 替换为枚举中选项的列表。 

> [!NOTE]
> 默认情况下，字段的说明是根据字段的名称生成的。 若要指定某个字段的自定义说明，请添加 `Describe` 属性。

以下代码片段演示通过上述示例指定的自定义提示。

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

`Prompt` 属性也可指定那些影响窗体显示提示的参数。 例如，`ChoiceFormat` 参数决定了窗体如何呈现选项列表。

[!code-csharp[Prompt attribute ChoiceFormat parameter](../includes/code/dotnet-formflow-advanced.cs#promptChoice)]

在此示例中，`ChoiceFormat` 参数的值指示选项应以项目符号列表（而不是编号列表）的形式显示。

```console
What kind of sandwich would you like?
- BLT
- Black Forest Ham
- Buffalo Chicken
- Chicken And Bacon Ranch Melt
- Cold Cut Combo
- Meatball Marinara
- Oven Roasted Chicken
- Roast Beef
- Rotisserie Style Chicken
- Spicy Italian
- Steak And Cheese
- Sweet Onion Teriyaki
- Tuna
- Turkey Breast
- Veggie
>
```

## <a name="customize-prompts-using-the-template-attribute"></a>使用 Template 属性自定义提示

`Prompt` 属性用于自定义单个字段的提示，而 `Template` 属性则用于替换 FormFlow 用来自动生成提示的默认模板。 以下代码示例使用 `Template` 属性来重新定义窗体处理所有枚举字段的方式。 此属性指示用户只能选择一个项、通过[模式语言](bot-builder-dotnet-formflow-pattern-language.md)设置提示文本，并指定窗体一行只能显示一个项。 

[!code-csharp[Template attribute](../includes/code/dotnet-formflow-advanced.cs#templateAttribute)]

此代码片段演示针对 `Bread` 字段和 `Cheese` 字段生成的提示。

```console
What kind of bread would you like on your sandwich?
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> 

What kind of cheese would you like on your sandwich? 
 1. American
 2. Monterey Cheddar
 3. Pepperjack
> 
```

如果使用 `Template` 属性来替换 FormFlow 用来生成提示的默认模板，则可能需要将某些变体插入到窗体生成的提示和消息中。 为此，可以使用[模式语言](bot-builder-dotnet-formflow-pattern-language.md)定义多个文本字符串，窗体就会在每次需要显示提示或消息时随机地从可用选项中进行选择。

以下代码示例重新定义了 [TemplateUsage.NotUnderstood][notUnderstood] 模板，为消息指定了两个不同的变体。 当机器人需要表示它不理解用户的输入时，它会从这两个文本字符串中随机地选择一个来确定消息内容。 

[!code-csharp[Template variations of message](../includes/code/dotnet-formflow-advanced.cs#templateMessages)]

此代码片段以示例方式演示了在机器人和用户之间进行的交互。 

```console
What size of sandwich do you want? (1. Six Inch, 2. Foot Long)
> two feet
I do not understand "two feet".
> two feet
Try again, I don't get "two feet"
> 
```

## <a name="designate-a-field-as-optional-using-the-optional-attribute"></a>使用 Optional 属性将字段指定为可选

若要将字段指定为可选，请使用 `Optional` 属性。 此代码示例指定 `Cheese` 字段为可选。

[!code-csharp[Optional attribute](../includes/code/dotnet-formflow-advanced.cs#optionalAttribute)]

如果某个字段为可选但尚未指定值，则当前选项会显示为“无首选项”。

```console
What kind of cheese would you like on your sandwich? (current choice: No Preference)
  1. American
  2. Monterey Cheddar
  3. Pepperjack
 >
```

如果某个字段为可选且用户已指定值，则会在列表中将“无首选项”显示为最后一个选项。

```console
What kind of cheese would you like on your sandwich? (current choice: American)
 1. American
 2. Monterey Cheddar
 3. Pepperjack
 4. No Preference
>
```

## <a name="match-user-input-using-the-terms-attribute"></a>使用 Terms 属性来匹配用户输入

当用户向使用 FormFlow 生成的机器人发送消息时，该机器人会尝试将输入与术语列表进行匹配，以便确定用户输入的含义。 默认情况下，生成术语列表时会对字段或值应用以下步骤： 

1. 在大小写变化和出现下划线 (_) 时进行拆分。
2. 生成每个 <a href="https://en.wikipedia.org/wiki/N-gram" target="_blank">n-gram</a> 时，有一个最大长度。
3. 添加“s?” 到每个单词的末尾（目的是支持复数形式）。 

例如，值“AngusBeefAndGarlicPizza”会生成以下术语： 

- 'angus?'
- 'beefs?'
- 'garlics?'
- 'pizzas?'
- 'angus? beefs?'
- 'garlics? pizzas?'
- 'angus beef and garlic pizza'

若要重写此默认行为并定义术语列表，以便使用它将用户输入与字段或字段中的值进行匹配，请使用 `Terms` 属性。 例如，考虑到用户可能会拼错“rotisserie”一词，可以使用 `Terms` 属性（和一个正则表达式）。 

[!code-csharp[Terms attribute](../includes/code/dotnet-formflow-advanced.cs#termsAttribute)]

使用 `Terms` 属性可以提高系统将用户输入匹配到某个有效选项的可能性。 此示例中的 `Terms.MaxPhrase` 参数导致 `Language.GenerateTerms` 生成术语的其他变体。 

此代码片段演示当用户拼错“Rotisserie”时机器人与用户之间产生的交互。

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
> rotissary checkin
For sandwich I understood Rotisserie Style Chicken. "checkin" is not an option.
```

## <a name="validate-user-input-using-the-numeric-attribute-or-pattern-attribute"></a>使用 Numeric 或 Pattern 属性验证用户输入

若要限制某个数字字段的允许值的范围，请使用 `Numeric` 属性。 此代码示例使用 `Numeric` 属性，目的是指定 `Rating` 字段的输入必须是 1 到 5 之间的数字。 

[!code-csharp[Numeric attribute](../includes/code/dotnet-formflow-advanced.cs#numericAttribute)]

若要为特定字段的值指定必需格式，请使用 `Pattern` 属性。 此代码示例使用 `Pattern` 属性，目的是指定 `PhoneNumber` 字段的值的必需格式。

[!code-csharp[Pattern attribute](../includes/code/dotnet-formflow-advanced.cs#patternAttribute)]

## <a name="summary"></a>摘要

本文介绍如何使用 FormFlow 提供自定义的用户体验，方法是指定初始窗体状态、添加业务逻辑来管理字段之间的相互依赖关系并处理用户输入，以及使用属性来自定义提示、重写模板、指定可选字段、匹配用户输入和验证用户输入。 若要了解如何使用 FormFlow 通过其他方式来自定义用户体验，请参阅[使用 FormBuilder 自定义窗体](bot-builder-dotnet-formflow-formbuilder.md)。

## <a name="sample-code"></a>代码示例

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他资源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [使用 FormBuilder 自定义窗体](bot-builder-dotnet-formflow-formbuilder.md)
- [本地化窗体内容](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 架构定义窗体](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式语言自定义用户体验](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">用于 .NET 的 Bot Builder SDK 参考</a>

[formDialog]: /dotnet/api/microsoft.bot.builder.formflow.formdialog

[promptFieldsWithValues]: /dotnet/api/microsoft.bot.builder.formflow.formoptions.promptfieldswithvalues

[validateResult]: /dotnet/api/microsoft.bot.builder.formflow.validateresult

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[optionalAttribute]: /dotnet/api/microsoft.bot.builder.formflow.optionalattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[termsAttribute]: /dotnet/api/microsoft.bot.builder.formflow.termsattribute

[notUnderstood]: /dotnet/api/microsoft.bot.builder.formflow.templateusage.notunderstood
