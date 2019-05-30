---
title: 使用多个 LUIS 和 QnA 模型 | Microsoft Docs
description: 了解如何在机器人中使用 LUIS 和 QnA Maker。
keywords: Luis, QnA, Dispatch 工具, 多个服务, 路由意向
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 942ab2d5b3a43ca071c877b5cc18e8141838d604
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/24/2019
ms.locfileid: "66214254"
---
# <a name="use-multiple-luis-and-qna-models"></a>使用多个 LUIS 和 QnA 模型

[!INCLUDE[applies-to](../includes/applies-to.md)]

如果机器人使用多个 LUIS 模型和 QnA Maker 知识库 (KB)，则你可以使用 Disptach 工具来确定哪些 LUIS 模型或 QnA Maker KB 最符合用户输入。 为了实现此目的，Disptach 工具会创建单个 LUIS 应用将用户输入路由到正确的模型。 有关 Disptach 的详细信息（包括 CLI 命令），请参阅[自述文件][dispatch-readme]。

## <a name="prerequisites"></a>先决条件
- 了解[机器人基础知识](bot-builder-basics.md)、[LUIS][howto-luis] 和 [QnA Maker][howto-qna]。 
- [Dispatch 工具](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- [C# 示例][cs-sample]或 [JS 示例][js-sample]代码存储库中 **NLP with Dispatch** 的副本。
- 一个用于发布 LUIS 应用的 [luis.ai](https://www.luis.ai/) 帐户。
- 一个用于发布 QnA 知识库的 [QnA Maker](https://www.qnamaker.ai/) 帐户。

## <a name="about-this-sample"></a>关于此示例

此示例基于一组预定义的 LUIS 和 QnA Maker 应用。

## <a name="ctabcs"></a>[C#](#tab/cs)

![代码示例逻辑流](./media/tutorial-dispatch/dispatch-logic-flow.png)

每次收到用户输入都会调用 `OnMessageActivityAsync`。 此模块会查找最相关的评分用户意向，并将该结果传递到 `DispatchToTopIntentAsync`。 而 DispatchToTopIntentAsync 又会调用相应的应用处理程序

- `ProcessSampleQnAAsync` - 用于提出机器人 FAQ 问题。
- `ProcessWeatherAsync` - 用于发出天气查询。
- `ProcessHomeAutomationAsync` - 用于执行家庭照明命令。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![代码示例逻辑流](./media/tutorial-dispatch/dispatch-logic-flow-js.png)

每次收到用户输入都会调用 `onMessage`。 此模块会查找最相关的评分用户意向，并将该结果传递到 `dispatchToTopIntentAsync`。 而 dispatchToTopIntentAsync 又会调用相应的应用处理程序

- `processSampleQnA` - 用于提出机器人 FAQ 问题。
- `processWeather` - 用于发出天气查询。
- `processHomeAutomation` - 用于执行家庭照明命令。

---

该处理程序调用 LUIS 或 QnA Maker 服务，并将生成的结果返回给用户。

## <a name="create-luis-apps-and-qna-kb"></a>创建 LUIS 应用和 QnA KB
在创建调度模型之前，需要先创建并发布 LUIS 应用和 QnA KB。 在本文中，我们将发布 `\CognitiveModels` 文件夹中 _NLP With Dispatch_ 示例随附的以下模型： 

| 名称 | 说明 |
|------|------|
| 家庭自动化 | 一个可以识别包含关联实体数据的家庭自动化意向的 LUIS 应用。|
| 天气 | 一个可以识别包含位置数据的天气相关意向的 LUIS 应用。|
| QnAMaker  | 一个可以提供有关机器人的一些简单问题的答案的 QnA Maker 知识库。 |

### <a name="create-luis-apps"></a>创建 LUIS 应用
1. 登录到 [LUIS Web 门户](https://www.luis.ai/)。 在“我的应用”  部分，选择“导入新应用”选项卡。  此时会出现以下对话框：

![导入 LUIS json 文件](./media/tutorial-dispatch/import-new-luis-app.png)

2. 选择“选择应用文件”按钮，导航到示例代码的 CognitiveModel 文件夹，然后选择“HomeAutomation.json”文件。  将可选名称字段留空。 

3. 选择“完成”  。

4. 当 LUIS 打开“家庭自动化”应用以后，请选择“训练”按钮。  这样就会使用刚刚通过“home-automation.json”文件导入的话语集来训练应用。

5. 训练完成后，请选择“发布”按钮。  此时会出现以下对话框：

![发布 LUIS 应用](./media/tutorial-dispatch/publish-luis-app.png)

6. 选择“生产”环境，然后选择“发布”按钮  。

7. 发布新的 LUIS 应用以后，请选择“管理”选项卡。  在“应用程序信息”页中，记下“_app-id-for-app_”的 `Application ID` 值，以及“_name-of-app_”的 `Display name` 值。 在“密钥和终结点”页中，记下“_your-luis-authoring-key_”的 `Authoring Key` 值，以及“_your-region_”的 `Region` 值。 稍后要在“appsetting.json”文件中使用这些值。

8. 完成后，针对“Weather.json”文件重复上述步骤，以训练并发布 LUIS 天气应用和 LUIS 调度应用。  

### <a name="create-qna-maker-kb"></a>创建 QnA Maker KB

若要设置 QnA Maker KB，首先请在 Azure 中设置一个 QnA Maker 服务。 为此，请按[此处](https://aka.ms/create-qna-maker)提供的分步说明操作。

在 Azure 中创建 QnA Maker 服务以后，需记录为 QnA Maker 服务提供的认知服务 _Key 1_。 将 QnA 添加到 Dispatch 应用程序时，会将此项用作 \<azure-qna-service-key1>。 以下步骤为你提供此密钥：
    
![选择认知服务](./media/tutorial-dispatch/select-qna-cognitive-service.png)

1. 在 Azure 门户中选择 QnA Maker 认知服务。

![选择认知服务密钥](./media/tutorial-dispatch/select-cognitive-service-keys.png)

2. 在左侧菜单的“资源管理”  部分下选择“密钥”图标。

![选择认知服务 Key1](./media/tutorial-dispatch/select-cognitive-service-key1.png)

3. 将 _Key 1_ 的值复制到剪贴板并将其保存到本地。 随后，将 QnA 添加到 Dispatch 应用程序时，会将此项用于 (-k) 密钥值 \<azure-qna-service-key1>。

现在登录到 [QnAMaker Web 门户](https://qnamaker.ai)。 向下移动到步骤 2

![创建 QnA 的步骤 2](./media/tutorial-dispatch/create-qna-step-2.png) 

并选择
1. Azure AD 帐户。
1. Azure 订阅名称。
1. 为 QnA Maker 服务创建的名称。 （如果你的 Azure QnA 服务一开始没有显示在此下拉列表中，请尝试刷新页面。） 

移动到步骤 3

![创建 QnA 的步骤 3](./media/tutorial-dispatch/create-qna-step-3.png)

为 QnA Maker 知识库提供一个名称。 对于本示例，我们将使用名称“sample-qna”。

移动到步骤 4

![创建 QnA 的步骤 4](./media/tutorial-dispatch/create-qna-step-4.png)

选择“+ 添加文件”选项，导航到示例代码的 CognitiveModel 文件夹，然后选择“QnAMaker.tsv”文件 

可以通过其他选项向知识库添加聊天个性化内容，但我们的示例不包括该选项。 

转到步骤 5

选择“创建知识库”  。

基于上传的文件创建知识库后，选择“保存并训练”；完成后，选择“发布”选项卡并发布该应用。  

发布 QnA Maker 应用以后，请选择“设置”选项卡，然后向下滚动到“部署详细信息”。  记录 _Postman_ 示例 HTTP 请求中的以下值。

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL.
Authorization: EndpointKey <qna-maker-resource-key>
```

主机名的完整 URL 字符串类似于“https://< >.azure.net/qnamaker”。

稍后要在 `appsettings.json` 或 `.env` 文件中使用这些值。

请记下 LUIS 应用和 QnA Maker 知识库名称与 ID。 另请记下 LUIS 创作密钥和认知服务订阅密钥。 需要所有这些信息才能完成此过程。

## <a name="create-the-dispatch-model"></a>创建调度模型

Dispatch 工具的 CLI 接口可以创建用于调度到正确服务的模型。

1. 打开命令提示符或终端窗口，将目录切换到 **CognitiveModels** 目录
1. 确保已安装最新版本的 npm 和 Dispatch 工具。

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. 使用 `dispatch init` 初始化并创建调度模型的 .dispatch 文件。 请使用可方便识别的文件名创建此文件。

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. 使用 `dispatch add` 将 LUIS 应用和 QnA Maker 知识库添加到该 .dispatch 文件。

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<azure-qna-service-key1>" --intentName q_sample-qna
    ```

1. 使用 `dispatch create` 基于该 .dispatch 文件生成调度模型。

    ```cmd
    dispatch create
    ```

1. 使用生成的调度模型 JSON 文件发布调度 LUIS 应用。

## <a name="use-the-dispatch-model"></a>使用调度模型

生成的模型定义了每个应用和知识库的意向；如果言语中没有合适的意向，则会定义 _none_ 意向。

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

请注意，要使机器人正常运行，需要以适当的名称发布这些服务。

机器人需要有关已发布服务的信息，这样它才能访问这些服务。

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>安装包

首次运行此应用之前，请确保已安装多个 Nuget 包：

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>手动更新 appsettings.json 文件

创建所有服务应用以后，需将每个应用的信息添加到“appsettings.json”文件中。 初始的 [C# 示例][cs-sample]代码包含一个空的 appsettings.json 文件：

**appsettings.json** [!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

对于下面所示的每个实体，请在这些指令中添加前面记下的值：

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAAuthKey": "<qna-maker-resource-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-authoring-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```
当所有更改都已完成后，保存此文件。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="installing-packages"></a>安装包

首次运行此应用之前，需要安装多个 npm 包。

```powershell
npm install --save botbuilder
npm install --save botbuilder-ai
```
若要使用 .env 配置文件，需在机器人中包含一个附加的包：

```powershell
npm install --save dotenv
```

### <a name="manually-update-your-env-file"></a>手动更新 .env 文件

创建所有服务应用以后，需将每个应用的信息添加到“.env”文件中。 初始的 [JavaScript 示例][js-sample]代码包含一个空的 .env 文件。 

**.env** [!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

如下所示添加服务连接值：

**.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-authoring-key>
LuisAPIHostName=<your-dispatch-app-region>

```
当所有更改都已完成后，保存此文件。

---

### <a name="connect-to-the-services-from-your-bot"></a>从机器人连接到服务

若要连接到 Dispatch、LUIS 和 QnA Maker 服务，机器人需要从前面提供的设置中提取信息。

## <a name="ctabcs"></a>[C#](#tab/cs)

在 **BotServices.cs** 中，包含在配置文件 _appsettings.json_ 中的信息用于将调度机器人连接到 `Dispatch` 和 `SampleQnA` 服务。 构造函数使用提供的值连接到这些服务。

**BotServices.cs** [!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **dispatchBot.js** 中，包含在配置文件 _.env_ 中的信息用于将调度机器人连接到 _LuisRecognizer(dispatch)_ 和 _QnAMaker_ 服务。 构造函数使用提供的值连接到这些服务。

**dispatchBot.js** [!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>从机器人调用服务

对于用户提供的每项输入，机器人逻辑会根据组合的调度模型检查用户输入，查找最相关的返回意向，并使用该信息来针对该输入调用相应的服务。

## <a name="ctabcs"></a>[C#](#tab/cs)

在 **DispatchBot.cs** 文件中，每当调用 `OnMessageActivityAsync` 方法时，我们都会根据调度模型检查传入的用户消息。 然后，我们将调度模型的 `topIntent` 和 `recognizerResult` 传递到正确的方法，以调用服务并返回结果。

**DispatchBot.cs** [!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 **dispatchBot.js** `onMessage` 方法中，我们将根据调度模型检查用户输入消息，查找 _topIntent_，然后通过调用 _dispatchToTopIntentAsync_ 传递此信息。

**dispatchBot.js**

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>处理识别结果

## <a name="ctabcs"></a>[C#](#tab/cs)

当模型生成结果时，它会指示哪个服务最适合用于处理话语。 此机器人中的代码将请求路由到相应的服务，然后汇总被调用服务返回的响应。 根据 Dispatch 返回的意向，此代码使用返回的意向路由到正确的 LUIS 模型或 QnA 服务。 

**DispatchBot.cs** [!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

如果调用了 `ProcessHomeAutomationAsync` 或 `ProcessWeatherAsync` 方法，将在这些方法的 _luisResult.ConnectedServiceResult_ 中传递调度模型返回的结果。 然后，指定的方法将提供用户反馈，其中显示了调度模型的最相关意向，加上检测到的所有意向和实体的排名列表。

如果调用了 `q_sample-qna` 方法，此方法将使用 turnContext 中包含的用户输入来基于知识库生成回答，并向用户显示该结果。

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

当模型生成结果时，它会指示哪个服务最适合用于处理话语。 此示例中的代码使用识别到的 _topIntent_ 来演示如何将请求路由到相应的服务。

**DispatchBot.cs** [!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

如果调用了 `processHomeAutomation` 或 `processWeather` 方法，将在这些方法的 _recognizerResult.luisResult_ 中传递调度模型返回的结果。 然后，指定的方法将提供用户反馈，其中显示了调度模型的最相关意向，加上检测到的所有意向和实体的排名列表。

如果调用了 `q_sample-qna` 方法，此方法将使用 turnContext 中包含的用户输入来基于知识库生成回答，并向用户显示该结果。

---

> [!NOTE]
> 如果这是一个生产应用程序，则选定的 LUIS 方法将在此上下文中连接到其指定服务、传入用户输入，并处理返回的 LUIS 意向和实体数据。

## <a name="test-your-bot"></a>测试机器人

使用开发环境启动示例代码。 记下应用打开的浏览器窗口地址栏中显示的 localhost 地址：“https://localhost:<Port_Number>”。打开 Bot Framework Emulator 后，选择 `create new bot configuration` 下面所述的蓝色测试。

![创建新配置](./media/tutorial-dispatch/emulator-create-new-configuration.png)

输入记下的 localhost 地址，并在末尾添加“/api/messages”：“https://localhost:<Port_Number>/api/messages”

![连接仿真器](./media/tutorial-dispatch/emulator-create-and-connect.png)

现在请单击 `Save and connect` 按钮访问正在运行的机器人。 为便于参考，下面提供了为机器人生成的服务所用的某些问题和命令：

- QnA Maker
  - `hi`、`good morning`
  - `what are you`、`what do you do`
- LUIS（家庭自动化）
  - `turn on bedroom light`
  - `turn off bedroom light`
  - `make some coffee`
- LUIS（天气）
  - `whats the weather in redmond washington`
  - `what's the forecast for london`
  - `show me the forecast for nebraska`

## <a name="additional-information"></a>其他信息

待机器人运行以后，即可删除类似的或重叠的话语，改进机器人的性能。 例如，假设在 `Home Automation` LUIS 应用中，“开灯”请求将映射到“TurnOnLights”意向，而“为什么灯未打开？”请求 将映射到“None”意向，以便可将其传递给 QnA Maker。 使用 Dispatch 合并 LUIS 应用和 QnA Maker 服务时，需执行以下某个操作：

- 从原始 `Home Automation` LUIS 应用中删除“None”意向，改将该意向中的话语添加到调度程序应用中的“None”意向。
- 如果不从原始 LUIS 应用中删除“None”意向，则需在机器人中添加逻辑，将那些与“None”意向匹配的消息传递给 QnA Maker 服务。

上述两项操作中的任一操作都会减少机器人使用消息“找不到答案”来回应用户的次数。

### <a name="to-update-or-create-a-new-luis-model"></a>更新或创建新的 LUIS 模型

此示例基于预先配置的 LUIS 模型。 有关如何更新此模型或创建新的 LUIS 模型的其他信息，可参阅[此文](https://aka.ms/create-luis-model#updating-your-cognitive-models)。

### <a name="to-delete-resources"></a>删除资源

此示例将创建许多应用程序和资源，可以使用下面列出的步骤将其删除，但不应删除其他任何应用或服务依赖的资源。 

若要删除 LUIS 资源：

1. 登录到 [luis.ai](https://www.luis.ai) 门户。
1. 转到“我的应用”页。 
1. 选择本示例创建的应用。
   - `Home Automation`
   - `Weather`
   - `NLP-With-Dispatch-BotDispatch`
1. 单击“删除”，然后单击“确定”以确认。  

若要删除 QnA Maker 资源：

1. 登录到 [qnamaker.ai](https://www.qnamaker.ai/) 门户。
1. 转到“我的知识库”页。 
1. 单击 `Sample QnA` 知识库对应的删除按钮，然后单击“删除”以确认。 

### <a name="best-practice"></a>最佳做法

若要改进本示例中使用的服务，请参阅 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) 和 [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices) 的最佳做法。


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch
