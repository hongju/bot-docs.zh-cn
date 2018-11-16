---
title: 企业机器人自定义 | Microsoft Docs
description: 了解如何自定义通过企业模板创建的机器人
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ea507bbdf916ff1955aea0db17b765791432f430
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645577"
---
# <a name="enterprise-bot-template---customize-your-bot"></a>企业机器人模板 - 自定义机器人

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

## <a name="net"></a>.NET
在按[此处](bot-builder-enterprise-template-deployment.md)的说明部署企业机器人模板并测试其是否自始至终正常工作后，可以根据你的方案和需求轻松自定义机器人。 模板的目标是提供一个坚实的基础，以便基于它构建聊天体验。

## <a name="project-structure"></a>项目结构

下面显示了你的机器人的文件夹结构，它表示我们建议的用于构建你的机器人项目和处理传入消息的最佳做法。

    | - YourBot.bot         // The .bot file containing all of your Bot configuration including dependencies
    | - README.md           // README file containing links to documentation
    | - Program.cs          // Default Program.cs file
    | - Startup.cs          // Core Bot Initialisation including Bot Configuration LUIS, Dispatcher, etc. 
    | - <BOTNAME>State.cs   // The Root State class for your Bot
    | - appsettings.json    // References above .bot file for Configuration information. App Insights key
    | - CognitiveModels     
        | - LUIS            // .LU file containing base conversational intents (Greeting, Help, Cancel)
        | - QnA             // .LU file containing example QnA items
    | - DeploymentScripts   // msbot clone recipe for deployment
    | - Dialogs             // All Bot dialogs sit under this folder
        | - Main            // Root Dialog for all messages
            | - MainDialog.cs       // Dialog Logic
            | - MainResponses.cs    // Dialog responses
            | - Resources           // Adaptive Card JSON, Resource File
        | - Onboarding
            | - OnboardingDialog.cs       // Onboarding dialog Logic
            | - OnboardingResponses.cs    // Onboarding dialog responses
            | - OnboardingState.cs        // Localised dialog state
            | - Resources                 Resource File
        | - Cancel
        | - Escalate
        | - Signin
    | - Middleware          // Telemetry, Content Moderator
    | - ServiceClients      // SDK libraries, example GraphClient provided for Auth example
   
## <a name="update-introduction-message"></a>更新介绍消息

介绍消息使用了[自适应卡片](https://www.adaptivecards.io)。 若要为机器人自定义此介绍，可以在 Dialogs/Main/Resources 文件夹中找到名为 ```Intro.json``` 的 JSON 文件。 使用[自适应卡片可视化工具](http://adaptivecards.io/visualizer)修改自适应卡片以满足你的机器人的要求。

## <a name="update-bot-responses"></a>更新机器人响应

项目中的每个对话都有一组响应，它们存储在支持资源 (.resx) 文件中。 可以在每个对话下的资源文件夹中找到它们。

可以如下所示在 Visual Studio 资源编辑器中更改响应来调整机器人的响应方式。

![自定义机器人响应](media/enterprise-template/EnterpriseBot-CustomisingResponses.png)

此方法使用标准资源文件本地化方法支持多语言响应。 可以在[此处](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.1)找到更多信息。

## <a name="updating-your-cognitive-models"></a>更新认知模型

默认情况下，企业模板包括了两个认知模型：一个示例 FAQ QnAMaker 知识库和一个用于一般意向（问候、帮助、取消，等等）的 LUIS 模型。 可以自定义这些模型来满足你的需求。 此外，还可以添加新的 LUIS 模型和 QnAMaker 知识库来扩展机器人的功能。

### <a name="updating-an-existing-luis-model"></a>更新现有 LUIS 模型
若要更新企业模板的现有 LUIS 模型，请执行以下步骤：
1. 在 [LUIS 门户](http://luis.ai)中或使用 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 和 [Luis](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS) CLI 工具对 LUIS 模型进行更改。 
2. 运行以下命令来更新调度模型以反映更改（确保正确的消息路由）：
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```
3. 从项目根目录为每个更新的模型运行以下命令来更新其关联的 LuisGen 类： 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen --cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qnamaker-knowledge-base"></a>更新现有 QnAMaker 知识库
若要更新现有 QnAMaker 知识库，请执行以下步骤：
1. 通过 [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) 和 [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) CLI 工具或 [QnAMaker 门户](https://qnamaker.ai)对 QnAMaker 知识库进行更改。
2. 运行以下命令来更新调度模型以反映更改（确保正确的消息路由）：
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```

### <a name="adding-a-new-luis-model"></a>添加新的 LUIS 模型

希望向项目中添加新的 LUIS 模型时，你需要更新机器人配置和调度程序来确保它知道附加的模型。 
1. 通过 LuDown/LUIS CLI 工具或 LUIS 门户创建 LUIS 模型
2. 运行以下命令来将新的 LUIS 应用连接到 .bot 文件：
```shell
    msbot connect luis --appId [LUIS_APP_ID] --authoringKey [LUIS_AUTHORING_KEY] --subscriptionKey [LUIS_SUBSCRIPTION_KEY] 
```
3. 通过以下命令将此新的 LUIS 模型添加到调度程序
```shell
    dispatch add -t luis -id YOUR_LUIS_APPID -bot "YOURBOT.bot" -secret YOURSECRET
```
4. 通过以下命令刷新调度模型以反映 LUIS 模型更改
```shell
    dispatch refresh -bot "YOURBOT.bot" -secret YOURSECRET
```

### <a name="adding-an-additional-qnamaker-knowledgebase"></a>添加额外的 QnAMaker 知识库

在某些情况下，你可能希望向机器人添加额外的 QnAMaker 知识库，可以通过以下步骤来执行此操作。

1. 使用在助手目录中执行的以下命令基于 JSON 文件创建一个新的 QnAMaker 知识库
```shell
qnamaker create kb --in <KB.json> --msbot | msbot connect qna --stdin --bot "YOURBOT.bot" --secret YOURSECRET
```
2. 运行以下命令来更新调度模型以反映更改
```shell
dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```
3. 更新强类型调度类来反映新的 QnA 源
```shell
msbot get dispatch --bot "YOURBOT.bot" | luis export version --stdin > dispatch.json
luisgen dispatch.json -cs Dispatch -o Dialogs\Shared
```
4.  根据提供的示例更新 `Dialogs\Main\MainDialog.cs` 文件以包括你的新 QnA 源的对应调用意向。

你现在应当能够将多个 QnA 源用作机器人的一部分。

## <a name="adding-a-new-dialog"></a>添加新对话

若要向机器人添加新对话，需要首先在对话下创建一个新文件夹，并确保此类派生自 `EnterpriseDialog`。 然后，你需要连接对话基础结构。 “加入”对话显示了一个你可以引用的简单示例，下面显示了一个摘录和步骤概述。

- 向构造函数添加瀑布对话
- 定义瀑布的步骤
- 创建瀑布步骤
- 调用 AddDialog 并传递你的瀑布
- 调用 AddDialog 并传递你在瀑布中使用的任何提示
- 将 InitialDialogId 设置为你希望组件运行的第一个对话

```
InitialDialogId = nameof(OnboardingDialog);

var onboarding = new WaterfallStep[]
{
    AskForName,
    AskForEmail,
    AskForLocation,
    FinishOnboardingDialog,
};

AddDialog(new WaterfallDialog(InitialDialogId, onboarding));
AddDialog(new TextPrompt(NamePrompt));
AddDialog(new TextPrompt(EmailPrompt));
AddDialog(new TextPrompt(LocationPrompt));
```

然后，你需要创建模板管理器来处理响应。 创建一个新类并从 TemplateManager 进行派生，OnboardingResponses.cs 文件中提供了一个示例，下面显示了摘录。

```
public const string _namePrompt = "namePrompt";
public const string _haveName = "haveName";
public const string _emailPrompt = "emailPrompt";
      
private static LanguageTemplateDictionary _responseTemplates = new LanguageTemplateDictionary
{
    ["default"] = new TemplateIdMap
    {
        {
            _namePrompt,
            (context, data) => OnboardingStrings.NAME_PROMPT
        },
        {
            _haveName,
            (context, data) => string.Format(OnboardingStrings.HAVE_NAME, data.name)
        },
        {
            _emailPrompt,
            (context, data) => OnboardingStrings.EMAIL_PROMPT
        },
```

要呈现响应，可以通过用于提示的 `ReplyWith` 或 `RenderTemplate` 使用模板管理器实例来访问这些响应。 下面显示了示例。

```
Prompt = await _responder.RenderTemplate(sc.Context, "en", OnboardingResponses._namePrompt),
await _responder.ReplyWith(sc.Context, OnboardingResponses._haveName, new { name });
```

对话基础结构的最后一部分用于创建作用域仅限于你的对话的一个状态类。 创建一个新类并确保它派生自 `DialogState`

在对话完成后，你需要使用 `AddDialog` 将该对话添加到 `MainDialog` 组件。 若要使用新对话，请从 `RouteAsync` 方法中调用 `dc.BeginDialogAsync()`，通过合适的 LUIS 意向进行触发（如果需要）。

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>使用 PowerBI 仪表板和 Application Insights 的聊天式见解
- 若要开始获取聊天式见解，请继续执行[使用 PowerBI 仪表板配置聊天式分析](bot-builder-enterprise-template-powerbi.md)。

