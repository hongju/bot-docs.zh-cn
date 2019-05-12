---
ms.openlocfilehash: 4b5181babf728861107a0c7bc28f844491761a7a
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033881"
---
在开始部署之前，请确保你有最新版本的 [Azure cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) 和 [dotnet cli](https://dotnet.microsoft.com/download)。 如果没有 dotnet cli，请使用上面提供的链接中的“.Net Core 运行时”选项来安装它。 

### <a name="login-to-azure-cli-and-set-your-subscription"></a>登录到 Azure CLI 并设置订阅
你已在本地创建并测试一个机器人，现在想要将它部署到 Azure。 打开命令提示符以登录到 Azure 门户。

```cmd
az login
```
### <a name="set-the-subscription"></a>设置订阅

设置要使用的默认订阅。

```cmd
az account set --subscription "<azure-subscription>"
```

如果你不确定要使用哪个订阅来部署机器人，可以使用 `az account list` 命令查看帐户的订阅列表。

导航到 bot 文件夹。
`cd <local-bot-folder>`

### <a name="create-a-web-app-bot-in-azure"></a>在 Azure 中创建 Web 应用机器人 

如果还没有可以向其发布本地机器人的资源组，请创建一个：

```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| 选项     | 说明 |
|:-----------|:---|
| 名称     | 资源组的唯一名称。 切勿在此名称中包含空格或下划线。 |
| location | 用于创建资源组的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 使用 `az account list-locations` 来列出位置。 |

然后创建机器人资源，以便将机器人发布到其中。 这样就会在 Azure 中预配必需的资源并创建一个机器人 Web 应用，后者可以被本地机器人覆盖。 

在继续之前，请先根据用于登录到 Azure 的电子邮件帐户类型阅读适用的说明。

#### <a name="msa-email-account"></a>MSA 电子邮件帐户
如果使用 MSA 电子邮件帐户，则需在应用程序注册门户中创建可以与 `az bot create` 命令配合使用的应用 ID 和应用密码。
1. 转到[**应用程序注册门户**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)。
1. 单击“添加应用”以注册应用程序，创建**应用程序 ID**，然后单击“生成新密码”。 如果已经有应用程序和密码，但却忘记了该密码，则需在“应用程序机密”部分生成新密码。
1. 保存刚刚生成的应用程序 ID 和新密码，以便可以在 `az bot create` 命令中使用这些信息。  

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| 选项 | 说明 |
|:---|:---|
| 名称 | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。 |
| location | 用于创建机器人服务资源的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 |
| resource-group | 要在其中创建机器人的资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。 |
| appid | 可以与机器人配合使用的 Microsoft 帐户 ID (MSA ID)。 |
| password | 机器人的 Microsoft 帐户 (MSA) 密码。 |

#### <a name="business-or-school-account"></a>企业或学校帐户

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```
| 选项 | 说明 |
|:---|:---|
| 名称 | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。 |
| location | 用于创建机器人服务资源的地理位置。 例如 `eastus`、`westus`、`westus2` 等。 |
| lang | 用来创建机器人的语言：`Csharp` 或 `Node`；默认为 `Csharp`。 |
| resource-group | 要在其中创建机器人的资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。 |

#### <a name="update-appsettingsjson-or-env-file"></a>更新 appsettings.json 或 .env 文件
创建机器人以后，应该会在控制台窗口中看到显示的以下信息： 

```JSON
{
  "appId": "as234-345b-4def-9047-a8a44b4s",
  "appPassword": "34$#w%^$%23@334343",
  "endpoint": "https://mybot.azurewebsites.net/api/messages",
  "id": "mybot",
  "name": "mybot",
  "resourceGroup": "botresourcegroup",
  "serviceName": "mybot",
  "subscriptionId": "234532-8720-5632-a3e2-a1qw234",
  "tenantId": "32f955bf-33f1-43af-3ab-23d009defs47",
  "type": "abs"
}
```

需复制 `appId` 和 `appPassword` 值并将其粘贴到 appsettings.json 或 .env 文件中。 例如：

```JSON
{
  MicrosoftAppId: "as234-345b-4def-9047-a8a44b4s",
  MicrosoftAppPassword: "34$#w%^$%23@334343"
}
```
请注意，如果 appsettings.json 或 .env 文件有其他的项，而这些项是用于你为机器人预配的其他服务的，请勿删除这些项。

保存文件。

接下来，按照你在创建机器人时使用的编程语言（**C#** 或 **JS**），执行相应的步骤。

**C# 机器人：** 

打开命令提示窗口，然后导航到项目文件夹。 从命令行运行以下命令。

| 任务 | 命令 |
|:-----|:--------|
| 1.还原项目依赖项 | `dotnet restore`|
| 2.生成项目     | `dotnet build` |
| 3.压缩项目文件 | 使用任何实用程序压缩项目文件。 转到 .csproj 文件所在的文件夹，选择该层级的所有文件和文件夹，以便创建压缩的文件夹。 |
| 4.设置内部版本部署设置 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
| 5.设置脚本生成器参数 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_SCRIPT_GENERATOR_ARGS="--aspNetCore mybot.csproj"`|

**JS 机器人：**
1. 从[此处](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps)下载 web.config 并将其保存到项目文件夹中。 
1. 编辑文件，将所有“server.js”替换为“index.js”。 
1. 保存文件。

打开命令提示窗口，然后导航到项目文件夹。 从命令行运行以下命令。

| 任务 | 命令 |
|:-----|:--------|
| 1.安装节点模块 | `npm install` |
| 2.压缩项目文件 | 使用任何实用程序压缩项目文件。 转到 .csproj 文件所在的文件夹，选择该层级的所有文件和文件夹，以便创建压缩的文件夹。 |
| 3.设置内部版本部署设置 | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
