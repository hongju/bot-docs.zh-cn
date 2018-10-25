---
title: 使用 JSON 架构和 FormFlow 定义表单 | Microsoft Docs
description: 了解如何通过 Bot Builder SDK for .NET 使用 JSON 架构和 FormFlow 来定义表单。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8bcc957dbe2d69790cdfa7c2d7c377ed28b5fa12
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000324"
---
# <a name="define-a-form-using-json-schema"></a>使用 JSON 架构定义表单

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

如果使用 FormFlow 创建机器人时使用 [C# 类](bot-builder-dotnet-formflow.md#create-class)来定义表单，则表单派生自 C# 中类型的静态定义。 或者，可以改而使用 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 架构</a>来定义表单。 使用 JSON 架构定义的表单是纯数据驱动的，只需通过更新架构就可以更改表单（因此，也可以更改机器人的行为）。 

JSON 架构描述了 <a href="http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Linq_JObject.htm" target="_blank">JObject</a> 中的字段，并包含控制提示、模板和术语的批注。 要使用 JSON 架构与 FormFlow，必须将 `Microsoft.Bot.Builder.FormFlow.Json` NuGet 包添加到项目中，并导入 `Microsoft.Bot.Builder.FormFlow.Json` 命名空间。

## <a name="standard-keywords"></a>标准关键字 

FormFlow 支持这些标准 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 架构</a>关键字：

| 关键字 | Description | 
|----|----|
| type | 定义字段包含的数据类型。 |
| 枚举 | 定义字段的有效值。 |
| minimum | 定义字段允许的最小数值（如 [NumericAttribute][numericAttribute] 中所述）。 |
| maximum | 定义字段允许的最大数值（如 [NumericAttribute][numericAttribute] 中所述。 |
| 必填 | 定义哪些字段是必填。 |
| pattern | 验证字符串值（如 [PatternAttribute][patternAttribute] 中所述）。 |

## <a name="extensions-to-json-schema"></a>JSON 架构扩展

FormFlow 扩展了标准 <a href="http://json-schema.org/documentation.html" target="_blank">JSON 架构</a>以支持其他数个属性。

### <a name="additional-properties-at-the-root-of-the-schema"></a>架构根处的其他属性

| 属性 | 值 |
|----|----|
| OnCompletion | 带参数 `(IDialogContext context, JObject state)` 的 C# 脚本，用于完成表单。 |
| 参考 | 要包含在脚本中的引用。 例如，`[assemblyReference, ...]`。 路径应为当前目录的绝对或相对路径。 默认情况下，脚本包括 `Microsoft.Bot.Builder.dll`。 |
| 导入 | 要包含在脚本中的导入。 例如，`[import, ...]`。 默认情况下，脚本包括 `Microsoft.Bot.Builder`、`Microsoft.Bot.Builder.Dialogs`、`Microsoft.Bot.Builder.FormFlow`、`Microsoft.Bot.Builder.FormFlow.Advanced`、`System.Collections.Generic` 和 `System.Linq` 命名空间。 |

### <a name="additional-properties-at-the-root-of-the-schema-or-as-peers-of-the-type-property"></a>架构根处的其他属性或作为类型属性对等项的其他属性

| 属性 | 值 |
|----|----|
| 模板 | `{ TemplateUsage: { Patterns: [string, ...], <args> }, ...}` |
| Prompt | `{ Patterns:[string, ...] <args>}` |

要指定 JSON 架构中的模板和提示，请使用 [TemplateAttribute][templateAttribute] 和 [PromptAttribute][promptAttribute] 定义的相同词汇。 架构中的属性名称和值应匹配基础 C# 枚举中的属性名称和值。 例如，此架构片段定义了一个替代 `TemplateUsage.NotUnderstood` 的模板，并指定了 `TemplateBaseAttribute.ChoiceStyle`： 

```json
"Templates":{ "NotUnderstood": { "Patterns": ["I don't get it"], "ChoiceStyle":"Auto"}}
```

### <a name="additional-properties-as-peers-of-the-type-property"></a>作为类型属性对等项的其他属性

|   属性   |          内容           |                                                   Description                                                    |
|--------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
|   DateTime   |            bool             |                                  表示字段是否为 `DateTime` 字段。                                  |
|   Describe   |      字符串或对象       |                  如 [DescribeAttribute][describeAttribute] 中所述的字段描述。                  |
|    术语     |       `[string,...]`        |                  如 TermsAttribute 中所述的匹配字段值的正则表达式。                  |
|  MaxPhrase   |             int             |                  通过 `Language.GenerateTerms(string, int)` 运行你的术语来将其展开。                   |
|    值    | `{ string: {Describe:string |                                  object, Terms:[string, ...], MaxPhrase}, ...}`                                  |
|    活动    |           脚本            | 带参数 `(JObject state)->bool` 的 C# 脚本，用来测试字段、消息或确认是否可用。  |
|   验证   |           脚本            |      带参数 `(JObject state, object value)->ValidateResult` 的 C# 脚本，用来验证字段值。      |
|    Define    |           脚本            |        带参数 `(JObject state, Field<JObject> field)` 的 C# 脚本，用来动态定义字段。        |
|     下一步     |           脚本            | 带参数 `(object value, JObject state)` 的 C# 脚本，用来在填写字段后确定下一步。 |
|    之前    |          `[confirm          |                                                  message, ...]`                                                  |
|    之后     |          `[confirm          |                                                  message, ...]`                                                  |
| 依赖项 |        [string, ...]        |                           此字段、消息或确认所依赖的字段。                           |

在 Before 属性或 After 属性的值内使用 `{Confirm:script|[string, ...], ...templateArgs}` 来定义确认，方法是使用带参数 `(JObject state)` 的 C# 脚本，或使用随可选模板参数一起随机选择的一组模式。

在 Before 属性或 After 属性的值内使用 `{Message:script|[string, ...] ...templateArgs}` 来定义消息，方法是使用带参数 `(JObject state)` 的 C# 脚本，或使用随可选模板参数一起随机选择的一组模式。

## <a name="scripts"></a>脚本

上述几个属性包含脚本作为属性值。 脚本可以是通常会在方法正文中找到的任何 C# 代码片段。 可以通过使用 References 属性和/或 Imports 属性来添加引用。 特殊的全局变量包括：

| 变量 | Description |
|----|----|
| choice | 用于要执行的脚本的内部调度。 |
| state | 所有脚本的绑定的 `JObject` 表单状态。 |
| ifield | `IField<JObject>` 可允许对所有脚本（消息/确认提示生成器除外）的当前字段进行推理。 |
| 值 | 要为 Validate 进行验证的对象值。 |
| 字段 | `Field<JObject>` 可允许动态更新 Define 中的字段。 |
| 上下文 | `IDialogContext` 上下文可允许在 OnCompletion 中发布结果。 |

通过 JSON 架构定义的字段具有与任何其他字段相同的以编程方式扩展或替代定义的功能。 它们也可以用相同的方式进行本地化。

## <a name="json-schema-example"></a>JSON 架构示例

定义表单最简单的方法是直接在 JSON 架构中定义所有内容，包括任何 C# 代码。 此示例显示了[使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)中描述的带注释的三明治机器人的 JSON 架构。

```json
{
  "References": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.dll" ],
  "Imports": [ "Microsoft.Bot.Sample.AnnotatedSandwichBot.Resource" ],
  "type": "object",
  "required": [
    "Sandwich",
    "Length",
    "Ingredients",
    "DeliveryAddress"
  ],
  "Templates": {
    "NotUnderstood": {
      "Patterns": [ "I do not understand \"{0}\".", "Try again, I don't get \"{0}\"." ]
    },
    "EnumSelectOne": {
      "Patterns": [ "What kind of {&} would you like on your sandwich? {||}" ],
      "ChoiceStyle": "Auto"
    }
  },
  "properties": {
    "Sandwich": {
      "Prompt": { "Patterns": [ "What kind of {&} would you like? {||}" ] },
      "Before": [ { "Message": [ "Welcome to the sandwich order bot!" ] } ],
      "Describe": { "Image": "https://placeholdit.imgix.net/~text?txtsize=16&txt=Sandwich&w=125&h=40&txttrack=0&txtclr=000&txtfont=bold" },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "BLT",
        "BlackForestHam",
        "BuffaloChicken",
        "ChickenAndBaconRanchMelt",
        "ColdCutCombo",
        "MeatballMarinara",
        "OvenRoastedChicken",
        "RoastBeef",
        "RotisserieStyleChicken",
        "SpicyItalian",
        "SteakAndCheese",
        "SweetOnionTeriyaki",
        "Tuna",
        "TurkeyBreast",
        "Veggie"
      ],
      "Values": {
        "RotisserieStyleChicken": {
          "Terms": [ "rotis\\w* style chicken" ],
          "MaxPhrase": 3
        }
      }
    },
    "Length": {
      "Prompt": {
        "Patterns": [ "What size of sandwich do you want? {||}" ]
      },
      "type": [
        "string",
        "null"
      ],
      "enum": [
        "SixInch",
        "FootLong"
      ]
    },
    "Ingredients": {
      "type": "object",
      "required": [ "Bread" ],
      "properties": {
        "Bread": {
          "type": [
            "string",
            "null"
          ],
          "Describe": {
            "Title": "Sandwich Bot",
            "SubTitle": "Bread Picker"
          },
          "enum": [
            "NineGrainWheat",
            "NineGrainHoneyOat",
            "Italian",
            "ItalianHerbsAndCheese",
            "Flatbread"
          ]
        },
        "Cheese": {
          "type": [
            "string",
            "null"
          ],
          "enum": [
            "American",
            "MontereyCheddar",
            "Pepperjack"
          ]
        },
        "Toppings": {
          "type": "array",
          "items": {
            "type": "integer",
            "enum": [
              "Everything",
              "Avocado",
              "BananaPeppers",
              "Cucumbers",
              "GreenBellPeppers",
              "Jalapenos",
              "Lettuce",
              "Olives",
              "Pickles",
              "RedOnion",
              "Spinach",
              "Tomatoes"
            ],
            "Values": {
              "Everything": { "Terms": [ "except", "but", "not", "no", "all", "everything" ] }
            }
          },
          "Validate": "var values = ((List<object>) value).OfType<string>(); var result = new ValidateResult {IsValid = true, Value = values} ; if (values != null && values.Contains(\"Everything\")) { result.Value = (from topping in new string[] {  \"Avocado\", \"BananaPeppers\", \"Cucumbers\", \"GreenBellPeppers\", \"Jalapenos\", \"Lettuce\", \"Olives\", \"Pickles\", \"RedOnion\", \"Spinach\", \"Tomatoes\"} where !values.Contains(topping) select topping).ToList();} return result;",
          "After": [ { "Message": [ "For sandwich toppings you have selected {Ingredients.Toppings}." ] } ]
        },
        "Sauces": {
          "type": [
            "array",
            "null"
          ],
          "items": {
            "type": "string",
            "enum": [
              "ChipotleSouthwest",
              "HoneyMustard",
              "LightMayonnaise",
              "RegularMayonnaise",
              "Mustard",
              "Oil",
              "Pepper",
              "Ranch",
              "SweetOnion",
              "Vinegar"
            ]
          }
        }
      }
    },
    "Specials": {
      "Templates": {
        "NoPreference": { "Patterns": [ "None" ] }
      },
      "type": [
        "string",
        "null"
      ],
      "Active": "return (string) state[\"Length\"] == \"FootLong\";",
      "Define": "field.SetType(null).AddDescription(\"cookie\", DynamicSandwich.FreeCookie).AddTerms(\"cookie\", Language.GenerateTerms(DynamicSandwich.FreeCookie, 2)).AddDescription(\"drink\", DynamicSandwich.FreeDrink).AddTerms(\"drink\", Language.GenerateTerms(DynamicSandwich.FreeDrink, 2)); return true;",
      "After": [ { "Confirm": "var cost = 0.0; switch ((string) state[\"Length\"]) { case \"SixInch\": cost = 5.0; break; case \"FootLong\": cost=6.50; break;} return new PromptAttribute($\"Total for your sandwich is {cost:C2} is that ok?\");" } ]
    },
    "DeliveryAddress": {
      "type": [
        "string",
        "null"
      ],
      "Validate": "var result = new ValidateResult{ IsValid = true, Value = value}; var address = (value as string).Trim(); if (address.Length > 0 && (address[0] < '0' || address[0] > '9')) {result.Feedback = DynamicSandwich.BadAddress; result.IsValid = false; } return result;"
    },
    "PhoneNumber": {
      "type": [ "string", "null" ],
      "pattern": "(\\(\\d{3}\\))?\\s*\\d{3}(-|\\s*)\\d{4}"
    },
    "DeliveryTime": {
      "Templates": {
        "StatusFormat": {
          "Patterns": [ "{&}: {:t}" ],
          "FieldCase": "None"
        }
      },
      "DateTime": true,
      "type": [
        "string",
        "null"
      ],
      "After": [ { "Confirm": [ "Do you want to order your {Length} {Sandwich} on {Ingredients.Bread} {&Ingredients.Bread} with {[{Ingredients.Cheese} {Ingredients.Toppings} {Ingredients.Sauces} to be sent to {DeliveryAddress} {?at {DeliveryTime}}?" ] } ]
    },
    "Rating": {
      "Describe": "your experience today",
      "type": [
        "number",
        "null"
      ],
      "minimum": 1,
      "maximum": 5,
      "After": [ { "Message": [ "Thanks for ordering your sandwich!" ] } ]
    }
  },
  "OnCompletion": "await context.PostAsync(\"We are currently processing your sandwich. We will message you the status.\");"
}
```

## <a name="implement-formflow-with-json-schema"></a>使用 JSON 架构实现 FormFlow

要使用 JSON 架构实现 FormFlow，请使用 `FormBuilderJson`，它支持与 `FormBuilder` 相同的流畅界面。 此代码示例演示如何实现[使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)中所述的带批注的三明治机器人的 JSON 架构。

[!code-csharp[Use JSON schema](../includes/code/dotnet-formflow-json-schema.cs#useSchema)]

## <a name="sample-code"></a>代码示例

[!INCLUDE [Sample code](../includes/snippet-dotnet-formflow-samples.md)]

## <a name="additional-resources"></a>其他资源

- [FormFlow 的基本功能](bot-builder-dotnet-formflow.md)
- [FormFlow 的高级功能](bot-builder-dotnet-formflow-advanced.md)
- [使用 FormBuilder 自定义表单](bot-builder-dotnet-formflow-formbuilder.md)
- [本地化处理表单内容](bot-builder-dotnet-formflow-localize.md)
- [使用模式语言自定义用户体验](bot-builder-dotnet-formflow-pattern-language.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>

[numericAttribute]: /dotnet/api/microsoft.bot.builder.formflow.numericattribute

[patternAttribute]: /dotnet/api/microsoft.bot.builder.formflow.patternattribute

[templateAttribute]: /dotnet/api/microsoft.bot.builder.formflow.templateattribute

[promptAttribute]: /dotnet/api/microsoft.bot.builder.formflow.promptattribute

[describeAttribute]: /dotnet/api/microsoft.bot.builder.formflow.describeattribute
