---
title: 有关如何创建和部署基本机器人的教程 | Microsoft Docs
description: 了解如何创建基本机器人并将其部署到 Azure。
keywords: echo 机器人, 部署, azure, 教程
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fd6b1b7ed38b57e8245098679d7753bd5cbb9387
ms.sourcegitcommit: dbbfcf45a8d0ba66bd4fb5620d093abfa3b2f725
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2019
ms.locfileid: "67464685"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>教程：创建和部署基本机器人

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

本教程介绍如何使用 Bot Framework SDK 来创建基本机器人并将其部署到 Azure。 如果已创建基本机器人并在本地运行它，请跳转到[部署机器人](#deploy-your-bot)部分。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建基本的 Echo 机器人
> * 在本地运行该机器人并与之交互
> * 发布机器人

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>部署机器人

### <a name="prerequisites"></a>先决条件
- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。
- 在本地计算机上运行的具有上述功能的机器人。
- 最新版本的 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest)。

### <a name="1-prepare-for-deployment"></a>1.准备部署
使用 Visual Studio 或 Yeoman 模板创建机器人时，生成的源代码将包含 `deploymentTemplates` 文件夹和 ARM 模板。 本文所述的部署过程使用 ARM 模板通过 Azure CLI 在 Azure 中预配机器人所需的资源。 

#### <a name="login-to-azure"></a>登录到 Azure

你已在本地创建并测试一个机器人，现在想要将它部署到 Azure。 打开命令提示符以登录到 Azure 门户。

```cmd
az login
```
此时会打开一个用于登录的浏览器窗口。

#### <a name="set-the-subscription"></a>设置订阅
设置要使用的默认订阅。

```cmd
az account set --subscription "azure-subscription"
```

如果你不确定要使用哪个订阅来部署机器人，可以使用 `az account list` 命令查看帐户的订阅列表。 导航到 bot 文件夹。

#### <a name="create-an-app-registration"></a>创建应用注册
注册应用程序意味着可以使用 Azure AD 对用户进行身份验证并请求访问用户资源。 机器人需要在 Azure 中注册一个应用，该应用可让机器人访问 Bot Framework 服务，以发送和接收经过身份验证的消息。 若要通过 Azure CLI 创建并注册应用，请执行以下命令：

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| 选项   | 说明 |
|:---------|:------------|
| display-name | 应用程序的显示名称。 |
| password | 应用密码，也称为“客户端机密”。 该密码的长度必须至少为 16 个字符，至少包含 1 个大写或小写字母字符，并至少包含 1 个特殊字符。 |
| available-to-other-tenants| 可以从任何 Azure AD 租户使用该应用程序。  允许的值：false、true。 默认值为 True。 使机器人能够使用 Azure 机器人服务通道|

上述命令将输出包含密钥 `appId` 的 JSON，请保存 ARM 部署的此密钥值，稍后要将此值用于 `appId` 参数。 提供的密码将用于 `appSecret` 参数。

可将机器人部署到新资源组或现有资源组中。 请选择最合适的选项。

# <a name="deploy-via-arm-template-with-new-resource-grouptabnewrg"></a>[通过 ARM 模板部署（使用**新**资源组）](#tab/newrg)

#### <a name="create-azure-resources"></a>创建 Azure 资源

将在 Azure 中创建新资源组，然后使用 ARM 模板来创建其中指定的资源。 在本例中，我们将提供应用服务计划、Web 应用和机器人通道注册。

```cmd
az deployment create --name "name-of-deployment" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="msa-app-guid" appSecret="msa-app-password" botId="id-or-name-of-bot" botSku=F0 newAppServicePlanName="name-of-app-service-plan" newWebAppName="name-of-web-app" groupName="new-group-name" groupLocation="location" newAppServicePlanLocation="location"
```

| 选项   | 说明 |
|:---------|:------------|
| 名称 | 部署的易记名称。 |
| template-file | ARM 模板的路径。 可以使用项目的 `deploymentTemplates` 文件夹中提供的 `template-with-new-rg.json` 文件。 |
| location |位置。 `az account list-locations` 中的值。 可以使用 `az configure --defaults location=<location>` 配置默认位置。 |
| parameters | 提供部署参数值。 运行 `az ad app create` 命令后获取的 `appId` 值。 `appSecret` 是在上一步骤中提供的密码。 `botId` 参数应全局唯一，用作不可变的机器人 ID。 此参数还用于配置机器人的可变显示名称。 `botSku` 是定价层，可以是 F0（免费）或 S1（标准）。 `newAppServicePlanName` 是应用服务计划的名称。 `newWebAppName` 要创建的 Web 应用的名称。 `groupName` 要创建的 Azure 资源组的名称。 `groupLocation` 是 Azure 资源组的位置。 `newAppServicePlanLocation` 是应用服务计划的位置。 |

# <a name="deploy-via-arm-template-with-existing--resource-grouptaberg"></a>[通过 ARM 模板部署（使用**现有**资源组）](#tab/erg)

#### <a name="create-azure-resources"></a>创建 Azure 资源

使用现有的资源组时，可以使用现有的应用服务计划或新建一个计划。 下面列出了这两种选项的步骤。 

**选项 1：现有的应用服务计划** 

在本例中，我们将使用现有的应用服务计划，但会创建新的 Web 应用和机器人通道注册。 

_注意：botId 参数应全局唯一，用作不可变的机器人 ID。此参数还用于配置机器人的可变显示名称。_

```cmd
az group deployment create --name "name-of-deployment" --resource-group "name-of-resource-group" --template-file "template-with-preexisting-rg.json" --parameters appId="msa-app-guid" appSecret="msa-app-password" botId="id-or-name-of-bot" newWebAppName="name-of-web-app" existingAppServicePlan="name-of-app-service-plan" appServicePlanLocation="location"
```

**选项 2：新的应用服务计划** 

在本例中，我们将创建应用服务计划、Web 应用和机器人通道注册。 

```cmd
az group deployment create --name "name-of-deployment" --resource-group "name-of-resource-group" --template-file "template-with-preexisting-rg.json" --parameters appId="msa-app-guid" appSecret="msa-app-password" botId="id-or-name-of-bot" newWebAppName="name-of-web-app" newAppServicePlanName="name-of-app-service-plan" appServicePlanLocation="location"
```

| 选项   | 说明 |
|:---------|:------------|
| 名称 | 部署的易记名称。 |
| resource-group | Azure 资源组的名称 |
| template-file | ARM 模板的路径。 可以使用项目的 `deploymentTemplates` 文件夹中提供的 `template-with-preexisting-rg.json` 文件。 |
| location |位置。 `az account list-locations` 中的值。 可以使用 `az configure --defaults location=<location>` 配置默认位置。 |
| parameters | 提供部署参数值。 运行 `az ad app create` 命令后获取的 `appId` 值。 `appSecret` 是在上一步骤中提供的密码。 `botId` 参数应全局唯一，用作不可变的机器人 ID。 此参数还用于配置机器人的可变显示名称。 `newWebAppName` 要创建的 Web 应用的名称。 `newAppServicePlanName` 是应用服务计划的名称。 `newAppServicePlanLocation` 是应用服务计划的位置。 |

---

#### <a name="retrieve-or-create-necessary-iiskudu-files"></a>检索或创建所需的 IIS/Kudu 文件

**对于 C# 机器人**

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

必须提供 .csproj 文件的相对于 --code-dir 的路径 可以通过 --proj-file-path 参数执行此操作。 该命令会将 --code-dir 和 --proj-file-path 解析为“./MyBot.csproj”

**对于 JavaScript 机器人**

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

此命令将提取 Node.js 应用在 Azure 应用服务中使用 IIS 所需的 web.config 文件。 请务必将 web.config 保存到机器人的根目录。

#### <a name="zip-up-the-code-directory-manually"></a>手动压缩代码目录

使用未配置的 [zip deploy API](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url) 部署机器人的代码时，Web 应用/Kudu 的行为如下所述：

默认情况下，Kudu 假设 zip 文件中的部署已准备好运行，并且在部署期间不需要额外的生成步骤，例如，不需要执行 npm 安装或 dotnet 还原/dotnet 发布。 

因此，必须将生成代码以及所有必要的依赖项包含在要部署到 Web 应用的 zip 文件中，否则机器人不会按预期方式工作。

> [!IMPORTANT]
> 在压缩项目文件之前，请确保在正确的文件夹中进行压缩。  
> - 对于 C# 机器人，正确的文件夹是包含 .csproj 文件的文件夹。 
> - 对于 JS 机器人，正确的文件夹是包含 app.js 或 index.js 文件的文件夹。 
>
> **在该文件夹中**选择所有文件并将其压缩，然后运行此命令，此时仍在该文件夹中。
>
> 如果根文件夹位置不正确，**机器人将无法在 Azure 门户中运行**。

### <a name="2-deploy-code-to-azure"></a>2.将代码部署到 Azure
现在，我们已准备好将代码部署到 Azure Web 应用。 在命令行中运行以下命令，以使用 Web 应用的 Kudu zip 推送部署来执行部署。

```cmd
az webapp deployment source config-zip --resource-group "new-group-name" --name "name-of-web-app" --src "code.zip" 
```

| 选项   | 说明 |
|:---------|:------------|
| resource-group | 先前在 Azure 中创建的资源组名称。 |
| 名称 | 先前使用的 Web 应用的名称。 |
| src  | 创建的压缩文件的路径。 |

### <a name="3-test-in-web-chat"></a>3.通过网页聊天执行测试
- 在 Azure 门户中，转到 Web 应用机器人边栏选项卡。
- 在“机器人管理”部分中，单击“通过网上聊天执行测试”   。 Azure 机器人服务将加载网上聊天控件，并连接到机器人。
- 成功部署后，请等待几秒，然后视需要重启 Web 应用以清除所有缓存。 返回到“Web 应用机器人”边栏选项卡，并使用 Azure 门户中提供的“网络聊天”进行测试。

## <a name="additional-resources"></a>其他资源

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [在机器人中使用 QnA Maker 来回答问题](bot-builder-tutorial-add-qna.md)
