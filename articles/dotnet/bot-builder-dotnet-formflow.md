---
title: FormFlow 的基本功能 | Microsoft Docs
description: 了解如何在用于 .NET 的 Bot Builder SDK 中使用 FormFlow 来引导聊天流。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cf3d69f7941d8c3177788bd00e4b58416a71cf5e
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904621"
---
# <a name="basic-features-of-formflow"></a>FormFlow 的基本功能

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[对话](bot-builder-dotnet-dialogs.md)是很强大且很灵活的功能，但处理引导式聊天（例如订购三明治）仍需很大周折。 在聊天中，每个时间点接下来要发生的事情都有许多可能性。 例如，可能需要澄清某个不清楚的地方、提供帮助、返回或显示进度。 在用于 .NET 的 Bot Builder SDK 中使用 **FormFlow**，可以大大简化管理此类引导式聊天的过程。 

FormFlow 可以根据指定的准则，自动生成管理引导式聊天所需的对话。 虽然与自行创建和管理对话相比，使用 FormFlow 会牺牲一些灵活性，但使用 FormFlow 设计引导式聊天可以大大缩短开发机器人所需的时间。 另外，在构建机器人时，可以组合使用 FormFlow 生成的对话和其他类型的对话。 例如，可以通过 FormFlow 对话引导用户执行完成窗体的过程，同时通过 [LuisDialog][LuisDialog] 评估用户输入，以便确定意向。

本文介绍如何创建一个机器人，以便使用 FormFlow 的基本功能从用户处收集信息。

## <a id="forms-and-fields"></a>窗体和字段

若要使用 FormFlow 创建机器人，必须指定机器人需要从用户处收集的信息。 例如，如果机器人的目的是获取用户的三明治订单，则必须定义一个窗体，其中包含的字段对应于机器人履行订单所需的数据。 可以通过创建 C# 类来定义窗体，该类包含一个或多个公共属性，表示机器人将要从用户处收集的数据。 每个属性必须是以下数据类型之一：

- 整形（sbyte、byte、short、ushort、int、uint、long、ulong）
- 浮点值（float、double）
- String
- DateTime
- 枚举
- 枚举列表

任何数据类型都可能为 null，这可以用来针对字段没有值的情况建模。 如果某个窗体字段所依据的枚举属性不可为 null，则枚举中的值 **0** 表示 **null**（表示字段没有值），枚举值应从 **1** 开始。 FormFlow 忽略所有其他的属性类型和方法。

对于复杂对象，必须为顶级 C# 类创建一个窗体，并为复杂对象创建另一个窗体。 可以使用典型的[对话](bot-builder-dotnet-dialogs.md)语义将窗体组合在一起。 也可直接定义一个窗体，方法是实现 [Advanced.IField][iField]，或者使用 [Advanced.Field][field] 并填充其中的字典。 

> [!NOTE]
> 可以使用 C# 类或 JSON 架构来定义窗体。 本文介绍如何使用 C# 类来定义窗体。 有关如何使用 JSON 架构的详细信息，请参阅[使用 JSON 架构定义窗体](bot-builder-dotnet-formflow-json-schema.md)。

## <a name="simple-sandwich-bot"></a>简单的三明治机器人

以简单的三明治机器人为例，该机器人旨在获取用户的三明治订单。 

### <a id="create-class"></a> 创建窗体

`SandwichOrder` 类定义窗体，枚举定义生成三明治所需的选项。 此类还包括静态的 `BuildForm` 方法，该方法使用 [FormBuilder][formBuilder] 创建窗体并定义简单的欢迎消息。 

若要使用 FormFlow，必须先导入 `Microsoft.Bot.Builder.FormFlow` 命名空间。

[!code-csharp[Define form](../includes/code/dotnet-formflow.cs#defineForm)]

### <a name="connect-the-form-to-the-framework"></a>将窗体连接到框架 

若要将窗体连接到框架，必须将其添加到控制器。 在此示例中，`Conversation.SendAsync` 方法调用静态的 `MakeRootDialog` 方法，后者反过来调用 `FormDialog.FromForm` 方法来创建 `SandwichOrder` 窗体。 

[!code-csharp[Connect form to framework](../includes/code/dotnet-formflow.cs#connectToFramework)]

### <a name="see-it-in-action"></a>在操作中查看

只需使用 C# 类定义窗体并将其连接到框架，即可通过 FormFlow 自动管理机器人和用户之间的聊天。 下面显示的示例交互演示了使用 FormFlow 的基本功能创建的机器人的功能。 在每个交互中，**>** 符号表示在该处用户输入了一个响应。 

#### <a name="display-the-first-prompt"></a>显示第一个提示 

此窗体填充 `SandwichOrder.Sandwich` 属性。 此窗体自动生成提示“请选择三明治”，提示中的“三明治”一词派生自属性名称 `Sandwich`。 `SandwichOptions` 枚举定义显示给用户的选项，每个枚举值都会在大小写变化和出现下划线时自动拆分成单词。

```console
Please select a sandwich
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

#### <a name="provide-guidance"></a>提供指导

在聊天过程中，用户可以随时输入“帮助”，获取有关如何填写窗体的指导。 例如，如果用户在出现三明治提示时输入“帮助”，则机器人会使用以下指导内容进行响应。 

```console
> help
* You are filling in the sandwich field. Possible responses:
* You can enter a number 1-15 or words from the descriptions. (BLT, Black Forest Ham, Buffalo Chicken, Chicken And Bacon Ranch Melt, Cold Cut Combo, Meatball Marinara, Oven Roasted Chicken, Roast Beef, Rotisserie Style Chicken, Spicy Italian, Steak And Cheese, Sweet Onion Teriyaki, Tuna, Turkey Breast, and Veggie)
* Back: Go back to the previous question.
* Help: Show the kinds of responses you can enter.
* Quit: Quit the form without completing it.
* Reset: Start over filling in the form. (With defaults from your previous entries.)
* Status: Show your progress in filling in the form so far.
* You can switch to another field by entering its name. (Sandwich, Length, Bread, Cheese, Toppings, and Sauce).
```

#### <a name="advance-to-the-next-prompt"></a>转到下一提示

如果用户输入“2”来响应初始的三明治提示，则机器人会显示一个提示，要求提供窗体定义的下一属性：`SandwichOrder.Length`。

```console
Please select a sandwich
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
> 2
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="return-to-the-previous-prompt"></a>返回到上一提示 

在聊天过程中，如果用户此时输入“返回”，机器人会返回到上一提示。 提示会显示用户当前的选择（“黑森林汉堡”）；用户可以输入另一数字来更改该选择，也可输入“c”确认该选择。

```console
> back
Please select a sandwich(current choice: Black Forest Ham)
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
> c
Please select a length (1. Six Inch, 2. Foot Long)
> 
```

#### <a name="clarify-user-input"></a>澄清用户输入

如果用户在进行选择时使用文本（而不是数字）进行响应，而该输入与多个选项匹配，机器人会自动要求用户进行澄清。 

```console
Please select a bread
 1. Nine Grain Wheat
 2. Nine Grain Honey Oat
 3. Italian
 4. Italian Herbs And Cheese
 5. Flatbread
> nine grain
By "nine grain" bread did you mean (1. Nine Grain Honey Oat, 2. Nine Grain Wheat)
> 1
```

如果用户输入没有直接匹配到任何有效选项，机器人会自动提示用户进行澄清。

```console
Please select a cheese (1. American, 2. Monterey Cheddar, 3. Pepperjack)
> amercan
"amercan" is not a cheese option.
> american smoked
For cheese I understood American. "smoked" is not an option.
```

如果用户输入为某个属性指定了多个选项，而机器人并不理解任何指定的选项，它会自动提示用户进行澄清。

```console
Please select one or more toppings
 1. Banana Peppers
 2. Cucumbers
 3. Green Bell Peppers
 4. Jalapenos
 5. Lettuce
 6. Olives
 7. Pickles
 8. Red Onion
 9. Spinach
 10. Tomatoes
> peppers, lettuce and tomato
By "peppers" toppings did you mean (1. Green Bell Peppers, 2. Banana Peppers)
> 1
```

#### <a name="show-current-status"></a>显示当前状态

如果用户在订购过程中的任何时间点输入“状态”，则机器人的响应会指示哪些值已指定，哪些值尚未指定。 

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> status
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Unspecified  
```

#### <a name="confirm-selections"></a>确认选择

当用户完成窗体时，机器人会要求用户确认所做的选择。

```console
Please select one or more sauce
 1. Honey Mustard
 2. Light Mayonnaise
 3. Regular Mayonnaise
 4. Mustard
 5. Oil
 6. Pepper
 7. Ranch
 8. Sweet Onion
 9. Vinegar
> 1
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
>
```

如果用户在响应时输入“否”，机器人会允许用户更新此前所做的任何选择。 如果用户在响应时输入“是”，则窗体完成，控制返回到调用对话。 

```console
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Six Inch
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> no
What do you want to change?
 1. Sandwich(Black Forest Ham)
 2. Length(Six Inch)
 3. Bread(Nine Grain Honey Oat)
 4. Cheese(American)
 5. Toppings(Lettuce, Tomatoes, and Green Bell Peppers)
 6. Sauce(Honey Mustard)
> 2
Please select a length (current choice: Six Inch) (1. Six Inch, 2. Foot Long)
> 2
Is this your selection?
* Sandwich: Black Forest Ham
* Length: Foot Long
* Bread: Nine Grain Honey Oat
* Cheese: American
* Toppings: Lettuce, Tomatoes, and Green Bell Peppers
* Sauce: Honey Mustard
> y
```

## <a name="handling-quit-and-exceptions"></a>处理退出和异常

如果用户在窗体中输入“退出”，或者在聊天的某个时间点发生异常，机器人需了解事件发生在哪一步、事件发生时窗体的状态，以及事件发生前窗体的哪些步骤已成功完成。 窗体通过 `FormCanceledException<T>` 类返回该信息。 

此代码示例演示如何捕获异常并根据已发生的事件显示一条消息。 

[!code-csharp[Handle exception or quit](../includes/code/dotnet-formflow.cs#handleExceptionOrQuit)]

## <a name="summary"></a>摘要

本文介绍如何使用 FormFlow 的基本功能创建一个具有下述功能的机器人：

- 自动生成和管理聊天
- 提供明确的指导和帮助
- 理解数字和文本条目
- 就理解和不理解的内容向用户提供反馈 
- 必要时要求用户澄清问题 
- 允许用户在步骤之间导航

虽然基本的 FormFlow 功能在某些情况下已足够，但也不妨考虑将 FormFlow 的部分更高级功能纳入机器人中，这样可能会为你带来很大优势。 有关详细信息，请参阅 [FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)和[使用 FormBuilder 自定义窗体](bot-builder-dotnet-formflow-formbuilder.md)。

## <a name="sample-code"></a>代码示例

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="next-steps"></a>后续步骤

FormFlow 可简化对话开发。 FormFlow 的高级功能可用于自定义 FormFlow 对象的行为。

> [!div class="nextstepaction"]
> [FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)

## <a name="additional-resources"></a>其他资源

- [使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)
- [本地化窗体内容](bot-builder-dotnet-formflow-localize.md)
- [使用 JSON 架构定义窗体](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式语言自定义用户体验](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[LuisDialog]: /dotnet/api/microsoft.bot.builder.dialogs.luisdialog-1

[iField]: /dotnet/api/microsoft.bot.builder.formflow.advanced.ifield-1

[field]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1

[formBuilder]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1
