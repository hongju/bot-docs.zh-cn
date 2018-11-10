---
title: 企业机器人模板部署 | Microsoft Docs
description: 了解如何部署企业机器人的所有支持 Azure 资源。
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 06e91d4b7d320078e83c3523e1326b82ee3fe759
ms.sourcegitcommit: 49a76dd34d4c93c683cce6c2b8b156ce3f53280e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/26/2018
ms.locfileid: "50134697"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>企业机器人模板 - 部署机器人

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

## <a name="prerequisites"></a>先决条件

- 确保已将 [.NET Core](https://www.microsoft.com/net/download) 更新到最新版本。

- 确保安装了[节点包管理器](https://nodejs.org/en/)。

- 安装 Azure 机器人服务命令行 (CLI) 工具。 请务必执行此操作以确保你具有最新版本，即使以前已使用过这些工具。

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot luisgen chatdown
```

- 从[此处](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)安装 Azure 命令行工具 (CLI)。 如果已安装 Azure 机器人服务命令行 (CLI) 工具，请确保将其更新为最新版本，方法是：卸载当前版本，然后安装新版本。

- 安装适用于机器人服务的 AZ 扩展
```shell
az extension add -n botservice
```

## <a name="configuration"></a>配置

- 检索你的 LUIS 创作密钥
   - 查看[此](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions)文档页来了解适用于你打算部署到的区域的正确 LUIS 门户。 请注意，www.luis.ai 是指美国区域，从该门户检索的创作密钥不适用于欧洲部署。
   - 登录后，在右上角单击你的名字。
   - 选择”设置”并记下”创作密钥“以在下一步中使用。

## <a name="deployment"></a>部署

>如果你有多个 Azure 订阅并且希望确保部署选择正确的一个，请先运行以下命令，然后再继续操作。

 执行浏览器登录过程来登录到你的 Azure 帐户
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

企业模板机器人需要以下依赖项来执行端到端操作。
- Azure Web 应用
- Azure 存储帐户（脚本）
- Azure Application Insights（遥测）
- Azure CosmosDb（状态）
- Azure 认知服务 - 语言理解
- Azure 认知服务 - QnAMaker（包括 Azure 搜索、Azure Web 应用）
- Azure 认知服务 - 内容审查器（可选的手动步骤）

新的机器人项目有一个部署配方，它使得 `msbot clone services` 命令能够自动将上述所有服务部署到你的 Azure 订阅并确保更新项目中 .bot 文件中的所有服务（包括服务密钥），以实现机器人的平稳运行。

> 在部署后，复查已创建的服务的定价层，并进行调整以适应你的方案。

你创建的项目中的 README.md 包含一个示例 msbot 克隆服务命令行，它使用你创建的机器人名称进行了更新，下面显示了一个常规版本。 确保更新来自上一步骤的创作密钥并选择要使用的 Azure 数据中心位置（例如 westus 或 westeurope）。 确保在上一步检索的 LUIS 创作密钥适用于在下面指定的区域（例如，luis.ai 的 westus 或 eu.luis.ai 的 westeurope）。

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "YOUR_REGION"
```

> 某些用户会出现一个已知的问题，即在运行部署时可能会遇到以下错误`ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again`。 在这种情况下，请浏览到 https://apps.dev.microsoft.com 并手动创建一个检索 ApplicationID 和密码/机密的新应用程序。 请运行上面的 msbot clone services 命令，但需提供两个新的参数（`appId` 和 `appSecret`），用于传递刚检索的值。

msbot 工具将概述部署计划，包括位置和 SKU。 在继续操作之前，请确保进行复查。

![部署确认](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>在部署完成后，**必须**记下所提供的 .bot 文件机密，因为后续步骤将需要使用它。

- 使用新创建的 .bot 文件名和 .bot 文件机密更新 `appsettings.json` 文件。
- 运行以下命令并检索你的 Application Insights 实例的 InstrumentationKey，更新 `appsettings.json` 文件中的 InstrumentationKey。

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>测试

完成后，在开发环境中运行你的机器人项目，并打开 Bot Framework Emulator。 在 Emulator 中，从“文件”菜单中选择“打开机器人”并导航到目录中的 .bot 文件。

任何，键入 ```hi``` 来验证是否一切都正常工作。

如果 Bot Framework Emulator 出现问题，请先确保 Bot Framework Emulator 为最新版本。 如果旧版模拟器尚未正确更新，请将其卸载并重新安装。

## <a name="deploy-to-azure"></a>“部署到 Azure”

可以在本地执行端到端测试。 当准备好将机器人部署到 Azure 进行额外测试时，可以使用以下命令来发布源代码，任何时候希望推送源代码更新时，都可以运行此命令。

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>启用更多方案

你的机器人项目提供了可以通过以下步骤启用的其他功能。

### <a name="authentication"></a>身份验证

若要启用身份验证，请先在 Azure 门户中在你的机器人的设置中配置身份验证连接名称，然后执行以下步骤。 可以在此[文档](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0)中找到更多信息。

在 MainDialog 构造函数中注册 `SignInDialog`：
    
`AddDialog(new SignInDialog(_services.AuthConnectionName));`

在代码中在所需位置添加以下内容来测试简单的登录流：
    
`var signInResult = await dc.BeginDialogAsync(nameof(SignInDialog));`

### <a name="content-moderation"></a>内容审核

可以使用内容审核来识别发送给机器人的消息中的个人身份信息 (PII) 和成人内容。 若要启用此功能，请转到 Azure 门户并创建一个新的内容审查器服务。 收集你的订阅密钥和区域来配置 .bot 文件。 

> 将来，此步骤将自动执行。

在 startup 中，在 service.AddBot<>() 方法的底部添加以下代码，以在每次运行时启用内容审核。 可以通过机器人状态访问内容审核结果。 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
通过从对话堆栈内调用以下命令访问中间件结果
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>自定义机器人

验证已成功将机器人部署为现成可用后，可以根据你的方案和需求自定义机器人。 继续[自定义机器人](bot-builder-enterprise-template-customize.md)。
