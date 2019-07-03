---
title: 向机器人添加自然语言理解 | Microsoft Docs
description: 了解如何借助 Bot Framework SDK 将 LUIS 用于自然语言理解。
keywords: 语言理解, LUIS, 意向, 识别器, 实体, 中间件
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e019d2d04d843cc0efd5a39135d65fe4cfc022f3
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404502"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>向机器人添加自然语言理解

[!INCLUDE[applies-to](../includes/applies-to.md)]

理解用户在会话和上下文中表达的含义是一项艰巨的任务，但可以让机器人更自然地进行聊天。 使用语言理解（称为 LUIS）能够实现此目标，使机器人能够识别用户消息的意向，接收用户更自然的语言，并更好地指导会话流程。 本主题详细介绍如何将 LUIS 添加到航班预订应用程序，以便识别用户输入中包含的不同意向和实体。 

## <a name="prerequisites"></a>先决条件
- [LUIS](https://www.luis.ai) 帐户
- 本文中的代码基于**核心机器人**示例。 需要 **[CSharp](https://aka.ms/cs-core-sample) 或 [JavaScript](https://aka.ms/js-core-sample)** 示例的副本。 
- 了解[机器人基础知识](bot-builder-basics.md)、[自然语言处理](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis)和[管理机器人资源](bot-file-basics.md)。

## <a name="about-this-sample"></a>关于此示例

此核心机器人编码示例演示了飞机航班预订应用程序的示例。 它使用 LUIS 服务来识别用户输入，并返回排名靠前的已识别 LUIS 意向。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
每次处理用户输入以后，`DialogBot` 都会保存 `UserState` 和 `ConversationState` 的当前状态。 收集所有必需的信息以后，编码示例会创建一个演示版的航班预订。 在本文中，我们将介绍此示例 LUIS 方面的内容。 但是，示例的常规流程如下所示：

- 连接新用户时，会调用 `OnMembersAddedAsync` 并显示欢迎卡片。 
- 每次收到用户输入都会调用 `OnMessageActivityAsync`。 

![LUIS 示例逻辑流](./media/how-to-luis/luis-logic-flow.png)

`OnMessageActivityAsync` 模块通过 `Run` 对话扩展方法运行相应的对话。 该主对话会调用 LUIS 帮助程序来查找排名靠前的评分用户意向。 如果用户输入的排名靠前的意向返回“Book_Flight”，则帮助程序会填充 LUIS 返回的用户的信息并启动 `BookingDialog`，后者会根据需要从用户获取其他信息，例如：

- `Origin` 始发城市。
- `TravelDate` 预订航班的日期。 
- `Destination` 目标城市。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
每次处理用户输入以后，`dialogBot` 都会保存 `userState` 和 `conversationState` 的当前状态。 收集所有必需的信息以后，编码示例会创建一个演示版的航班预订。 在本文中，我们将介绍此示例 LUIS 方面的内容。 但是，示例的常规流程如下所示：

- 连接新用户时，会调用 `onMembersAdded` 并显示欢迎卡片。 
- 每次收到用户输入都会调用 `OnMessage`。 

![LUIS 示例 JavaScript 逻辑流](./media/how-to-luis/luis-logic-flow-js.png)

`onMessage` 模块运行 `mainDialog`，后者将用户输入发送到 LUIS。 收到从 LUIS 返回的响应以后，`mainDialog` 会保存 LUIS 返回的用户信息并启动 `bookingDialog`。 `bookingDialog` 根据需要获取用户提供的其他信息，例如：

- `destination` 目标城市。
- `origin` 始发城市。
- `travelDate` 预订航班的日期。 

---

若要详细了解示例的其他方面，例如对话或状态，请参阅[使用对话提示收集用户输入](bot-builder-prompts.md)或[保存用户和聊天数据](bot-builder-howto-v4-state.md)。 

## <a name="create-a-luis-app-in-the-luis-portal"></a>在 LUIS 门户中创建 LUIS 应用
登录到 LUIS 门户，以创建自己的示例 LUIS 应用版本。 可在“我的应用”中创建和管理应用程序  。 

1. 选择“导入新应用”。  
1. 单击“选择应用文件(JSON 格式)...”  
1. 选择示例的 `CognitiveModels` 文件夹中的 `FlightBooking.json` 文件。 在“可选名称”中，输入 **FlightBooking**。  此文件包含三个意向：“预订航班”、“取消”和“None”。 当用户向机器人发送消息时，我们将使用这些意向来了解其意图。
1. [训练](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-train)应用。
1. 将应用[发布](https://docs.microsoft.com/azure/cognitive-services/LUIS/publishapp)到生产环境。 

### <a name="why-use-entities"></a>为何使用实体
LUIS 实体可让机器人智能理解不同于标准意向的某些事物或事件。 这样，你便可以从用户收集额外的信息，让机器人以更高的智能做出响应，或者在某些情况下跳过它向用户提出的有关该信息的问题。 除了“预订航班”、“取消”和“None”这三个 LUIS 意向的定义，FlightBooking.json 文件还包含一组实体，例如“From.Airport”和“To.Airport”。 LUIS 可以通过这些实体检测用户在发出新的旅行预订请求时其原始输入中包含的其他信息并将其返回。

若要了解实体信息如何在 LUIS 结果中显示，请参阅[从包含意图和实体的话语文本中提取数据](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-data-extraction)。

## <a name="obtain-values-to-connect-to-your-luis-app"></a>获取用于连接 LUIS 应用的值
发布 LUIS 应用后，可以在机器人中访问它。 需要记录多个值才能在机器人中访问该 LUIS 应用。 可以使用 LUIS 门户检索该信息。

### <a name="retrieve-application-information-from-the-luisai-portal"></a>从 LUIS.ai 门户检索应用程序信息
设置文件（`appsettings.json` 或 `.env`）将所有服务引用合并到一个位置。 检索的信息将添加到下一部分的该文件。 
1. 在 [luis.ai](https://www.luis.ai) 中选择已发布的 LUIS 应用。
1. 打开已发布的 LUIS 应用后，选择“管理”选项卡。  ![管理 LUIS 应用](./media/how-to-luis/manage-luis-app.png)
1. 在左侧选择“应用程序信息”选项卡，记录“应用程序 ID”(<YOUR_APP_ID>) 的显示值。  
1. 在左侧选择“密钥和终结点”选项卡，记录“创作密钥”(<YOUR_AUTHORING_KEY>) 的显示值。  
1. 向下滚动到页尾，记录“区域”(<YOUR_REGION>) 的显示值。 

### <a name="update-the-settings-file"></a>更新设置文件

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 `appsettings.json` 文件中添加访问 LUIS 应用所需的信息，包括应用程序 ID、创作密钥和区域。 前面已在发布的 LUIS 应用中保存了这些值。 请注意，API 主机名称应采用 `<your region>.api.cognitive.microsoft.com` 格式。

**appsetting.json**  
[!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/appsettings.json?range=1-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 `.env` 文件中添加访问 LUIS 应用所需的信息，包括应用程序 ID、创作密钥和区域。 前面已在发布的 LUIS 应用中保存了这些值。 请注意，API 主机名称应采用 `<your region>.api.cognitive.microsoft.com` 格式。

**.env**  
[!code[env](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/.env?range=1-5)]

---

## <a name="configure-your-bot-to-use-your-luis-app"></a>将机器人配置为使用你的 LUIS 应用

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

请确保为项目安装 **Microsoft.Bot.Builder.AI.Luis** NuGet 包。

为了连接到 LUIS 服务，机器人会从 appsetting.json 文件拉取你此前添加的信息。 `LuisHelper` 类包含的代码可以导入 appsetting.json 文件中的设置，并调用 `RecognizeAsync` 方法来查询 LUIS 服务。 如果返回的排名靠前的意向为“Book_Flight”，则会检查是否存在包含“目标城市”、“始发城市”和“TravelDate”预订信息的实体。

**LuisHelper.cs**  
[!code-csharp[luis helper](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/LuisHelper.cs?range=15-54)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

若要使用 LUIS，项目需安装 **botbuilder-ai** npm 包。

为了连接到 LUIS 服务，机器人会从 `.env` 文件拉取你此前添加的信息。 `LuisHelper` 类包含的代码可以导入 `.env` 文件中的设置，并调用 `recognize()` 方法来查询 LUIS 服务。 如果返回的排名靠前的意向为“Book_Flight”，则会检查是否存在包含“目标城市”、“始发城市”和“TravelDate”预订信息的实体。

[!code-javascript[luis helper](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/luisHelper.js?range=6-65)]

---

现在，你已为机器人配置并连接了 LUIS。 

## <a name="test-the-bot"></a>测试机器人

下载并安装最新的 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)

1. 在计算机本地运行示例。 如需说明，请参阅 [C# 示例](https://aka.ms/cs-core-sample)或 [JS 示例](https://aka.ms/js-core-sample)的自述文件。

1. 在模拟器中键入一条消息，例如“旅行到巴黎”或“从巴黎到柏林”。 使用在 FlightBooking.json 文件中发现的任何话语训练“预订航班”意向。

![LUIS 预订输入](./media/how-to-luis/luis-user-travel-input.png)

如果从 LUIS 返回的排名靠前的意向解析为“预订航班”，则机器人会询问其他问题，直至存储了进行旅行预订所需的足够信息。 此时机器人会将下述预订信息返回给用户。 

![LUIS 预订结果](./media/how-to-luis/luis-travel-result.png)

此时代码机器人逻辑会重置，你可以继续进行其他的预订。 

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [使用 QnA Maker 回答问题](./bot-builder-howto-qna.md)
