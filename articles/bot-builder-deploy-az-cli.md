---
title: 使用 Azure CLI 部署机器人 | Microsoft Docs
description: 将机器人部署到 Azure 云。
keywords: 部署机器人, azure 部署, 发布机器人, az 部署机器人, visual studio 部署机器人, msbot 发布, msbot 克隆
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654332"
---
# <a name="deploy-your-bot-using-azure-cli"></a>使用 Azure CLI 部署机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

创建机器人并在本地对其进行测试后，可将其部署到 Azure，以便可以从任何位置访问它。 将机器人部署到 Azure 需要支付服务使用费。 [计费和成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文可帮助你了解 Azure 计费方式、如何监视使用量与费用，以及如何管理帐户和订阅。

本文介绍如何使用 `az` 和 `msbot` cli 将 C# 和 JavaScript 机器人部署到 Azure。 在执行相关步骤之前最好是先阅读本文，以完全了解在部署机器人时所涉及到的工作。


## <a name="prerequisites"></a>先决条件
- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。
- 安装最新版本的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。
- 安装 `az` 工具的最新 `botservice` 扩展。
  - 首先，使用 `az extension remove -n botservice` 命令删除旧版本。 接下来，使用 `az extension add -n botservice` 命令安装最新版本。
- 安装最新版本的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
- 安装最新发布的 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) 版本。
- 安装并配置 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- 了解 [.bot](v4sdk/bot-file-basics.md) 文件。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# 机器人

你已在本地创建并测试一个机器人，现在想要将它部署到 Azure。 这些步骤假定你已创建所需的 Azure 资源。

打开命令提示符以登录到 Azure 门户。

```cmd
az login
```

此时会打开一个用于登录的浏览器窗口。

### <a name="set-the-subscription"></a>设置订阅

设置要使用的默认订阅。

```cmd
az account set --subscription "<azure-subscription>"
```

如果你不确定要使用哪个订阅来部署机器人，可以使用 `az account list` 命令查看帐户的 `subscriptions` 列表。

导航到 bot 文件夹。

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>创建 Web 应用机器人

创建机器人资源，以便将机器人发布到其中。

在继续之前，请先根据用于登录到 Azure 的电子邮件帐户类型阅读适用的说明。

#### <a name="msa-email-account"></a>MSA 电子邮件帐户

如果使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 电子邮件帐户，则需在应用程序注册门户中创建可以与 `az bot create` 命令配合使用的应用 ID 和应用密码。

1. 转到[**应用程序注册门户**](https://apps.dev.microsoft.com/)。
1. 单击“添加应用”以注册应用程序，创建**应用程序 ID**，然后单击“生成新密码”。 如果已经有应用程序和密码，但却忘记了该密码，则需在“应用程序机密”部分生成新密码。
1. 保存刚刚生成的应用程序 ID 和新密码，以便可以在 `az bot create` 命令中使用这些信息。  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 选项 | Description |
|:---|:---|
| --name | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。 |
| --location | 用于创建机器人服务资源的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 |
| --lang | 用来创建机器人的语言：`Csharp` 或 `Node`；默认为 `Csharp`。 |
| --resource-group | 要在其中创建机器人的资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。 |
| --appid | 可以与机器人配合使用的 Microsoft 帐户 ID (MSA ID)。 |
| --password | 机器人的 Microsoft 帐户 (MSA) 密码。 |

#### <a name="business-or-school-account"></a>企业或学校帐户

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| 选项 | Description |
|:---|:---|
| --name | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。 |
| --location | 用于创建机器人服务资源的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 |
| --lang | 用来创建机器人的语言：`Csharp` 或 `Node`；默认为 `Csharp`。 |
| --resource-group | 要在其中创建机器人的资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。 |

### <a name="download-the-bot-from-azure"></a>从 Azure 下载此机器人

接下来，下载刚创建的机器人。 以下命令将在 save-path 下创建一个子目录，但指定的路径必须已存在。

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| 选项 | Description |
|:---|:---|
| --name | Azure 中机器人的名称。 |
| --resource-group | 机器人所在的资源组的名称。 |
| --save-path | 一个现有目录，可将机器人代码下载到其中。 |

### <a name="decrypt-the-downloaded-bot-file"></a>解密已下载的 .bot 文件

.bot 文件中的敏感信息已加密。

获取加密密钥。

1. 登录到 [Azure 门户](http://portal.azure.com/)。
1. 打开机器人的 Web 应用机器人资源。
1. 打开机器人的“应用程序设置”。
1. 在“应用程序设置”窗口中，向下滚动到“应用程序设置”。
1. 找到 **botFileSecret** 并复制其值。

解密 .bot 文件。

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| 选项 | Description |
|:---|:---|
| --bot | 已下载的 .bot 文件的相对路径。 |
| --secret | 加密密钥。 |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>使用项目中的已下载 .bot 文件

将解密的 .bot 文件复制到本地机器人项目所在的目录。

更新机器人，以便使用此新的 .bot 文件。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 **appsettings.json** 中更新 **botFilePath** 属性，使之指向新的 .bot 文件。

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

在 **.env** 中更新 **botFilePath** 属性，使之指向新的 .bot 文件。

---

### <a name="update-the-bot-file"></a>更新 .bot 文件

如果机器人使用 LUIS、QnA Maker 或 Dispatch 服务，则需向 .bot 文件添加对这些服务的引用， 否则可以跳过此步骤。

1. 使用新的 .bot 文件在 BotFramework Emulator 中打开机器人。 此机器人不需在本地运行。
1. 在“机器人资源管理器”面板中展开“服务”部分。
1. 若要添加对 LUIS 应用的引用，请单击“服务”右侧的加号 (+)。
   1. 选择“添加语言理解(LUIS)”。
   1. 如果提示你登录到 Azure 帐户，请照做。
   1. 此时会显示一个列表，其中包含你有权访问的 LUIS 应用程序。 请选择适用于机器人的应用程序。
1. 若要添加对 QnA Maker 知识库的引用，请单击“服务”右侧的加号 (+)。
   1. 选择“添加 QnA Maker”。
   1. 如果提示你登录到 Azure 帐户，请照做。
   1. 此时会显示一个列表，其中包含你有权访问的知识库。 请选择适用于机器人的应用程序。
1. 若要添加对 Dispatch 模型的引用，请单击“服务”右侧的加号 (+)。
   1. 选择“添加 Dispatch”。
   1. 如果提示你登录到 Azure 帐户，请照做。
   1. 此时会显示一个列表，其中包含你有权访问的 Dispatch 模型。 请选择适用于机器人的应用程序。

### <a name="test-your-bot-locally"></a>在本地测试机器人

目前，机器人的工作方式应该与使用旧 .bot 文件时的工作方式相同。 请确保它在使用新的 .bot 文件时的工作方式符合预期。

### <a name="publish-your-bot-to-azure"></a>将机器人发布到 Azure

<!-- TODO: re-encrypt your .bot file? -->

将本地机器人发布到 Azure。 此步骤可能需要一定的时间。

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| 选项 | Description |
|:---|:---|
| --name | Azure 中机器人的资源名称。 |
| --proj-file | 需发布的启动项目文件名称（没有 .csproj）。 例如：EnterpriseBot。 |
| --resource-group | 资源组的名称。 |
| --code-dir | 一个目录，可以从其上传机器人代码。 |

此步骤完成后如果出现“部署成功!” 消息，则表明机器人已部署在 Azure 中。

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

清除加密密钥设置。

1. 登录到 [Azure 门户](http://portal.azure.com/)。
1. 打开机器人的 Web 应用机器人资源。
1. 打开机器人的“应用程序设置”。
1. 在“应用程序设置”窗口中，向下滚动到“应用程序设置”。
1. 找到 **botFileSecret** 并将其删除。

## <a name="additional-resources"></a>其他资源

部署机器人时，通常会在 Azure 门户中创建以下资源：

| 资源      | Description |
|----------------|-------------|
| Web 应用机器人 | 部署到 Azure 应用服务的 Azure 机器人服务机器人。|
| [应用服务](https://docs.microsoft.com/en-us/azure/app-service/)| 用于生成和托管 Web 应用程序。|
| [应用服务计划](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 为要运行的 Web 应用定义一组计算资源。|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| 提供用于收集和分析遥测数据的工具。|
| [存储帐户](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| 提供高度可用、安全、持久、可缩放且冗余的云存储。|

若要查看有关 `az bot` 命令的文档，请参阅[参考](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest)主题。

如果你不熟悉 Azure 资源组，请参阅此[术语](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology)主题。

## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [设置持续部署](bot-service-build-continuous-deployment.md)
