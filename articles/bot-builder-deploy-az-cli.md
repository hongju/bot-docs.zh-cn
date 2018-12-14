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
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286613"
---
# <a name="deploy-your-bot-using-azure-cli"></a>使用 Azure CLI 部署机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

创建机器人并在本地对其进行测试后，可将其部署到 Azure，以便可以从任何位置访问它。 将机器人部署到 Azure 需要支付服务使用费。 [计费和成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文可帮助你了解 Azure 计费方式、如何监视使用量与费用，以及如何管理帐户和订阅。

本文介绍如何使用 `msbot` 工具将 C# 和 JavaScript 机器人部署到 Azure。 在执行相关步骤之前最好是先阅读本文，以完全了解在部署机器人时所涉及到的工作。


## <a name="prerequisites"></a>先决条件
- 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。
- 安装 [.NET Core SDK](https://dotnet.microsoft.com/download) v2.2 或更高版本。 
- 安装最新版本的 [Azure CLI 工具](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)。
- 安装 `az` 工具的最新 `botservice` 扩展。 
  - 首先，使用 `az extension remove -n botservice` 命令删除旧版本。 接下来，使用 `az extension add -n botservice` 命令安装最新版本。
- 安装最新版本的 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
  - 如果克隆操作包括 LUIS 或 Dispatch 资源，则需要 [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation)。
  - 如果克隆操作包括 QnA Maker 资源，则需要 [QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli)。
- 安装 [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started)。
- 安装并配置 [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29)。
- 了解 [.bot](v4sdk/bot-file-basics.md) 文件。

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>使用 az cli 部署 JavaScript 和 C# 机器人
你已创建一个机器人，现在想要将它部署到 Azure。 这些步骤假设你已创建所需的 Azure 资源，并已使用 `msbot connect` 命令或 Bot Framework Emulator 更新了 .bot 文件中的服务引用。 如果 .bot 文件不是最新的，则部署过程也许可以完成且不出现错误或警告，但部署的机器人不会正常工作。

打开命令提示符以登录到 Azure 门户。

```cmd
az login
```
此时会打开一个用于登录的浏览器窗口。 

### <a name="set-the-subscription"></a>设置订阅 
使用以下命令设置订阅：

```cmd
az account set --subscription "<azure-subscription>"
``` 

如果你不确定要使用哪个订阅来部署机器人，可以使用 `az account list` 命令查看帐户的 `subscriptions` 列表。

导航到 bot 文件夹。 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Azure 订阅帐户
在继续之前，请先根据用于登录到 Azure 的电子邮件帐户类型阅读适用的说明。

**MSA 电子邮件帐户**

如果使用 [MSA](https://en.wikipedia.org/wiki/Microsoft_account) 电子邮件帐户，则需要创建要在 `msbot clone services` 命令中使用的 appId 和 appSecret。 

- 转到[应用程序注册门户](https://apps.dev.microsoft.com/)。 单击“添加应用”以注册应用程序，创建**应用程序 ID**，然后单击“生成新密码”。 
- 保存刚刚生成的应用程序 ID 和新密码，以便可以在 `msbot clone services` 命令中使用这些信息。 
- 若要部署机器人，请使用机器人适用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**企业或学校帐户**

如果使用企业或学校提供的电子邮件帐户登录到 Azure，则无需创建应用程序 ID 和密码。 若要部署机器人，请使用机器人适用的命令。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>保存用于解密 .bot 文件的机密
必须注意，部署过程会创建新的 .bot 文件并使用机密将其加密。 在部署机器人期间，命令行中会显示以下消息，要求保存 .bot 文件机密。 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
请保存 .bot 文件机密供稍后使用。 在 Azure 门户中，我们将使用通过 botFileSecret 加密的新 .bot 文件。 如果以后需要更改机器人文件名或机密，请转到门户中的“应用服务设置”->“应用程序设置”部分。 请注意，在 appsettings.json 或 .env 文件中，将使用最新创建的机器人文件更新机器人文件名。  

### <a name="test-your-bot"></a>测试机器人
在仿真器中，使用生产终结点来测试应用。 若要在本地进行测试，请确保机器人在本地计算机上运行。 

### <a name="to-update-your-bot-code-in-azure"></a>更新 Azure 中的机器人代码
切勿使用 `msbot clone services` 命令更新 Azure 中的机器人代码。 必须按如下所示使用 `az bot publish` 命令：

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| 参数        | Description |
|----------------  |-------------|
| `name`      | 首次将机器人部署到 Azure 时使用的名称。|
| `proj-file` | 对于 C# 机器人，它是 .csproj 文件。 对于 JS/TS 机器人，它是本地机器人的启动项目文件名（例如 index.js 或 index.ts）。|
| `resource-group` | `msbot clone services` 命令使用的 Azure 资源组。|
| `code-dir`  | 指向本地 bot 文件夹。|



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
