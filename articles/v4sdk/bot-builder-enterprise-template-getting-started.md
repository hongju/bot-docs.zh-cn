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
ms.openlocfilehash: e888b2473269cf576fd9edda0d99a30aa6212f7b
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/07/2019
ms.locfileid: "57571840"
---
# <a name="enterprise-bot-template---getting-started"></a>Enterprise Bot Template - 入门

> [!NOTE]
> 本主题适用于 SDK 的 v4 版本。 

## <a name="prerequisites"></a>先决条件

安装以下项目：
- [Enterprise Bot Template VSIX](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4enterprise)
- [.NET Core SDK](https://www.microsoft.com/net/download)（最新版本）
- [Node 包管理器](https://nodejs.org/en/)
- [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0)（最新版本）
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure 机器人服务 CLI 工具](https://github.com/Microsoft/botbuilder-tools)（最新版本）
    ```shell
    npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
    ```
- [LuisGen](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/LUISGen/src/npm/readme.md)
    ```shell
    dotnet tool install -g luisgen
    ```

## <a name="create-your-bot-project"></a>创建机器人项目
1. 在 Visual Studio 中，单击“文件”>“新建项目”。
1. 在“机器人”下，选择“Enterprise Bot Template”。

![文件新建项目模板](media/enterprise-template/new_project.jpg)

1. 为项目命名并单击“创建”。
1. 右键单击项目，然后单击“生成”以还原 NuGet 包。

## <a name="deploy-your-bot"></a>部署机器人

Enterprise Template Bots 需要以下 Azure 服务来执行端到端操作：
- Azure Web 应用
- Azure 存储帐户（脚本）
- Azure Application Insights（遥测）
- Azure CosmosDb（聊天状态存储）
- Azure 认知服务 - 语言理解
- Azure 认知服务 - QnA Maker（包括 Azure 搜索、Azure Web 应用）

以下步骤演示如何使用提供的部署脚本来部署这些服务：

1. 检索 LUIS 创作密钥。
   - 查看[此](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions)文档页，了解适用于部署区域的 LUIS 门户。 请注意， www.luis.ai 是指美国区域，从该门户检索的创作密钥不适用于欧洲部署。
   - 登录后，在右上角单击你的名字。
   - 选择“设置”，记下可供以后使用的“创作密钥”。
    
    ![LUIS 创作密钥屏幕截图](./media/enterprise-template/luis_authoring_key.jpg)

1. 打开命令提示符。
1. 使用 Azure CLI 登录到 Azure 帐户。 可以在 Azure 门户的[订阅](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)页中找到有权访问的订阅列表。
    ```shell
    az login
    az account set --subscription "YOUR_SUBSCRIPTION_NAME"
    ```

1. 运行 msbot clone services 命令，以便部署服务并在项目中配置 .bot 文件。 **注意：部署完成后，必须记下在命令提示符窗口中显示的供以后使用的机器人文件机密。**

    ```shell
    msbot clone services --name "YOUR-BOT-NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
    ```

    > **注释**：
    >- **YOUR-BOT-NAME** 参数必须**全局独一无二**，只能包含小写字母、数字和短划线（“-”）。
    >- 确保在这一步提供的部署区域与 LUIS 创作密钥门户的区域匹配（例如，westus 对应于 luis.ai，westeurope 对应于 eu.luis.ai）。
    >- 某些用户在预配 MSA AppId 和密码时可能会遇到一个已知的问题。 如果收到“`ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again`”错误，请访问 [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com) 并手动创建一个新应用程序，记下 AppId 和密码/机密。 请再次运行上面的 msbot clone services 命令，提供两个新的参数（`--appId` 和 `--appSecret`），其值是刚检索的值。 可能需要将密码中可能被 shell 解释为命令的任何特殊字符进行转义：
    >   - 对于 *Windows 命令提示符*，请将 appSecret 括在双引号内。 例如，`msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret "YOUR_APP_SECRET"`
    >   - 对于 *Windows PowerShell，请尝试在 --% 参数后传入 appSecret。 例如，`msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --% --appSecret "YOUR_APP_SECRET"`
    >   - 对于 *MacOS 或 Linux*，请将 appSecret 括在单引号内。 例如，`msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret 'YOUR_APP_SECRET'`

1. 部署完成后，使用机器人文件机密更新 **appsettings.json**。 
    
    ```
    "botFilePath": "./YOUR_BOT_FILE.bot",
    "botFileSecret": "YOUR_BOT_SECRET",
    ```
1. 运行以下命令并检索 Application Insights 实例的 InstrumentationKey。
    ```
    msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"
    ```

1. 使用检测密钥更新 **appsettings.json**：

    ```
    "ApplicationInsights": {
        "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
    }
    ```

## <a name="test-your-bot"></a>测试机器人

完成后，在开发环境中运行机器人项目，并打开 **Bot Framework Emulator**。 在 Emulator 中单击“文件”>“打开机器人”，导航到目录中的 .bot 文件。

![Emulator 开发终结点屏幕截图](./media/enterprise-template/development_endpoint.jpg)

应该会在聊天开始时收到简介消息。

键入 ```hi```，验证是否一切正常。

## <a name="publish-your-bot"></a>发布机器人

可以在本地执行端到端测试。 当准备好将机器人部署到 Azure 进行额外测试时，可以使用以下命令来发布源代码：

```shell
az bot publish -g YOUR-BOT-NAME -n YOUR-BOT-NAME --proj-name YOUR-BOT-NAME.csproj --version v4
```

## <a name="view-your-bot-analytics"></a>查看机器人分析
Enterprise Bot Template 带有一个预配置的 Power BI 仪表板，该仪表板在连接到 Application Insights 服务以后即可提供聊天分析。 在本地测试机器人以后，即可打开此仪表板来查看数据。 

1. 在[此处](https://github.com/Microsoft/AI/blob/master/solutions/analytics/ConversationalAnalyticsSample_02132019.pbit)下载 Power BI 仪表板（.pbit 文件）。
1. 在 [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) 中打开仪表板。
1. 输入 Application Insights 应用程序 ID（可以在 .bot 文件中找到）。

    ![在机器人文件的何处查找 AppInsights appId](./media/enterprise-template/appInsights_appId.jpg)

1. 在[此处](https://github.com/Microsoft/AI/tree/master/solutions/analytics)详细了解 Power BI 仪表板功能。

# <a name="next-steps"></a>后续步骤

成功部署现成的机器人以后，即可根据方案和需求自定义该机器人。 继续[自定义机器人](bot-builder-enterprise-template-customize.md)。
