---
title: 本地化处理表单内容 | Microsoft Docs
description: 了解如何使用 FormFlow 和 Bot Builder SDK for .NET 实现表单内容的本地化。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: bb0ac4b8e3fa34ec8863bb323ae968db37972a6f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297867"
---
# <a name="localize-form-content"></a>本地化处理表单内容

表单的本地化语言由当前线程的 [CurrentUICulture](https://msdn.microsoft.com/en-us/library/system.threading.thread.currentuiculture(v=vs.110).aspx) 和 [CurrentCulture](https://msdn.microsoft.com/en-us/library/system.threading.thread.currentculture(v=vs.110).aspx) 决定。 默认情况下，区域性派生自当前消息的 Locale 字段，但用户可替代该默认行为。 根据机器人的构造方式，本地化信息可能来自三个不同的源：

- PromptDialog 和 FormFlow 的内置本地化
- 为表单中的静态字符串生成的资源文件
- 使用字符串为动态计算的字段、消息或确认创建的资源文件

## <a name="generate-a-resource-file-for-the-static-strings-in-your-form"></a>为表单中的静态字符串生成资源文件

表单中的静态字符串包括表单根据 C# 类中的信息生成的字符串，以及指定为提示、模板、消息或确认的字符串。 基于内置模板生成的字符串已本地化处理，因此不被视为静态字符串。 由于表单中的许多字符串会自动生成，因此不可直接使用常规的 C# 资源字符串。 相反，可通过调用 `IFormBuilder.SaveResources` 或使用 BotBuilder SDK for .NET 附带的 RView 工具为表单中的静态字符串生成资源文件。

### <a name="use-iformbuildersaveresources"></a>使用 IFormBuilder.SaveResources

可在表单上调用 [IFormBuilder.SaveResources][saveResources] 将字符串保存为 .resx 文件，从而生成资源文件。

### <a name="use-rview"></a>使用 RView

或者，可使用 BotBuilder SDK for .NET 附带的 <a href="https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Tools/RView" target="_blank">RView</a> 工具来生成基于 .dll 或 .exe 的资源文件。 要生成 .resx 文件，请执行 rview 并指定包含静态表单生成方法及其路径的程序集。 此片段演示如何使用 RView 生成 `Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.resx` 资源文件。 

```csharp
rview -g Microsoft.Bot.Sample.AnnotatedSandwichBot.dll Microsoft.Bot.Sample.AnnotatedSandwichBot.SandwichOrder.BuildForm
```

此节选内容展示了通过执行此 rview 命令生成的 .resx 文件的一部分。

```xml
<data name="Specials_description;VALUE" xml:space="preserve">
<value>Specials</value>
</data>
<data name="DeliveryAddress_description;VALUE" xml:space="preserve">
<value>Delivery Address</value>
</data>
<data name="DeliveryTime_description;VALUE" xml:space="preserve">
<value>Delivery Time</value>
</data>
<data name="PhoneNumber_description;VALUE" xml:space="preserve">
<value>Phone Number</value>
</data>
<data name="Rating_description;VALUE" xml:space="preserve">
<value>your experience today</value>
</data>
<data name="message0;LIST" xml:space="preserve">
<value>Welcome to the sandwich order bot!</value>
</data>
<data name="Sandwich_terms;LIST" xml:space="preserve">
<value>sandwichs?</value>
</data>
```

## <a name="configure-your-project"></a>配置项目

生成资源文件后，将其添加到项目中，然后通过完成以下步骤设置中性语言： 

1. 右键单击项目并选择“应用程序”。
2. 单击“程序集信息”。
3. 选择与开发机器人时所用语言对应的“中性语言”值。

创建表单时，[IFormBuilder.Build][build] 方法将自动查找包含你的表单类型名称的资源，并使用它们本地化处理表单中的静态字符串。 

> [!NOTE]
> 无法通过与静态字段相同的方式本地化处理使用 [Advanced.Field.SetDefine][setDefine]（如[使用动态字段](bot-builder-dotnet-formflow-formbuilder.md#dynamically-define-field-values-confirmations-and-messages)中所述）定义的动态计算的字段，这是因为动态计算字段的字符串是在填充表单时构造的。 但是，可使用常规 C# 本地化机制来本地化处理动态计算的字段。

### <a name="localize-resource-files"></a>本地化处理资源文件 

向项目添加资源文件后，可使用<a href="https://developer.microsoft.com/en-us/windows/develop/multilingual-app-toolkit" target="_blank">多语言应用工具包 (MAT)</a> 对其进行本地化处理。 安装 MAT，然后通过完成以下步骤为项目启用它：

1. 在 Visual Studio 解决方案资源管理器中选择你的项目。
2. 依次单击“工具”、“多语言应用工具包”和“启用”。
3. 右键单击项目，选择“多语言应用工具包”，再选择“添加翻译”以选择相关翻译。 这将创建能够自动或手动翻译的符合行业标准的 <a href="https://en.wikipedia.org/wiki/XLIFF" target="_blank">XLF</a> 文件。

> [!NOTE]
> 虽然本文介绍了如何使用多语言应用工具包本地化处理内容，但你也通过各种其他方式实现本地化。

## <a name="see-it-in-action"></a>实际操作

此代码示例基于[使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)中的代码，目的是实现上文所述的本地化。 在本例中，`DynamicSandwich` 类（此处未显示）包含动态计算字段、消息和确认的的本地化信息。

[!code-csharp[Build localized form](../includes/code/dotnet-formflow-localize.cs#buildLocalizedForm)]

此片段演示 `CurrentUICulture` 为法语时机器人与用户之间产生的交互。

```console
Bienvenue sur le bot d'ordre "sandwich" !
Quel genre de "sandwich" vous souhaitez sur votre "sandwich"?
 1. BLT
 2. Jambon Forêt Noire
 3. Poulet Buffalo
 4. Faire fondre le poulet et Bacon Ranch
 5. Combo de coupe à froid
 6. Boulette de viande Marinara
 7. Poulet rôti au four
 8. Rôti de boeuf
 9. Rotisserie poulet
 10. Italienne piquante
 11. Bifteck et fromage
 12. Oignon doux Teriyaki
 13. Thon
 14. Poitrine de dinde
 15. Veggie
> 2

Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> ?
* Vous renseignez le champ longueur.Réponses possibles:
* Vous pouvez saisir un numéro 1-2 ou des mots de la description. (Six pouces, ou Pied Long)
* Retourner à la question précédente.
* Assistance: Montrez les réponses possibles.
* Abandonner: Abandonner sans finir
* Recommencer remplir le formulaire. (Vos réponses précédentes sont enregistrées.)
* Statut: Montrer le progrès en remplissant le formulaire jusqu'à présent.
* Vous pouvez passer à un autre champ en entrant son nom. ("Sandwich", Longueur, Pain, Fromage, Nappages, Sauces, Adresse de remise, Délai de livraison, ou votre expérience aujourd'hui).
Quel genre de longueur vous souhaitez sur votre "sandwich"?
 1. Six pouces
 2. Pied Long
> 1

Quel genre de pain vous souhaitez sur votre "sandwich"?
 1. Neuf grains de blé
 2. Neuf grains miel avoine
 3. Italien
 4. Fromage et herbes italiennes
 5. Pain plat
> neuf
Par pain "neuf" vouliez-vous dire (1. Neuf grains miel avoine, ou 2. Neuf grains de blé)
```

## <a name="sample-code"></a>代码示例

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他资源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)
- [使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)
- [使用 JSON 架构定义表单](bot-builder-dotnet-formflow-json-schema.md)
- [使用模式语言自定义用户体验](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[build]: /dotnet/api/microsoft.bot.builder.formflow.formbuilder-1.build 

[setDefine]: /dotnet/api/microsoft.bot.builder.formflow.advanced.field-1.setdefine

[saveResources]: /dotnet/api/microsoft.bot.builder.formflow.iform-1.saveresources
