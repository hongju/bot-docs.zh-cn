---
title: 部署机器人 | Microsoft Docs
description: 将机器人部署到 Azure 云。
keywords: 部署机器人, azure 部署机器人, 发布机器人
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 02/13/2019
ms.openlocfilehash: 8db2f0629b0d95dda0cb5d10dea5c9225e5d8d83
ms.sourcegitcommit: 4139ef7ebd8bb0648b8af2406f348b147817d4c7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/19/2019
ms.locfileid: "58073783"
---
# <a name="deploy-your-bot"></a>部署机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

创建机器人并在本地对其进行测试后，可将其部署到 Azure，以便可以从任何位置访问它。 将机器人部署到 Azure 需要支付服务使用费。 [计费和成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文可帮助你了解 Azure 计费方式、如何监视使用量与费用，以及如何管理帐户和订阅。

本文介绍如何将 C# 和 JavaScript 机器人部署到 Azure。 在执行相关步骤之前最好是先阅读本文，以完全了解在部署机器人时所涉及到的工作。

## <a name="prerequisites"></a>先决条件

- 安装最新版本的 [msbot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot) 工具。
- 在本地计算机上开发的 [CSharp](./dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [JavaScript](./javascript/bot-builder-javascript-quickstart.md) 机器人。

## <a name="1-prepare-for-deployment"></a>1.准备部署
部署过程要求在 Azure 中有一个目标 Web 应用机器人，以便将本地机器人部署到其中。 Azure 中的目标 Web 应用机器人以及通过它预配的资源可供本地机器人用于部署。 这是必需的，因为本地机器人没有预配所有必需的 Azure 资源。 当你创建目标 Web 应用机器人时，系统会为你预配以下资源：
-   Web 应用机器人 - 使用此机器人是因为需要将本地机器人部署到其中。
-   应用服务计划 - 提供应用服务应用需要运行的资源。
-   应用服务 - 一项用于托管 Web 应用程序的服务
-   存储帐户 - 包含所有 Azure 存储数据对象：Blob、文件、队列、表和磁盘。

在创建目标 Web 应用机器人的过程中，也会为机器人生成应用 ID 和密码。 在 Azure 中，应用 ID 和密码支持[服务身份验证和授权](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)。 需检索部分此类信息，用在本地机器人代码中。 

> [!IMPORTANT]
> 此服务的机器人模板的语言必须与编写机器人时使用的语言匹配。

如果已在 Azure 中创建了想要使用的机器人，则可以选择性地阅读“创建新的 Web 应用机器人”。

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 在 Azure 门户左上角单击“创建新资源”链接，然后选择“AI + 计算机学习”>“Web 应用机器人”。
1. 此时会打开一个新的边栏选项卡，其中包含有关 Web 应用机器人的信息。 
1. 在“机器人服务”边栏选项卡中，提供所请求的有关机器人的信息。
1. 单击“创建”创建服务并将机器人部署到云。 此过程可能需要数分钟。

### <a name="download-the-source-code"></a>下载源代码
创建目标 Web 应用机器人以后，需将机器人代码从 Azure 门户下载到本地计算机。 下载代码是为了获取 [.bot 文件](./v4sdk/bot-file-basics.md)中的服务引用。 这些服务引用适用于 Web 应用机器人、应用服务计划、应用服务和存储帐户。 

1. 在“机器人管理”部分中，单击“生成”。
1. 单击右窗格中的“下载机器人源代码”链接。
1. 按照提示下载代码，然后解压缩该文件夹。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-bot-file"></a>解密 .bot 文件

从 Azure 门户下载的源代码包含加密的 .bot 文件。 需要解密该文件才能将值复制到本地 .bot 文件中。 若要复制实际的服务引用而不是加密的服务引用，必须执行此步骤。  

1. 在 Azure 门户中打开机器人的 Web 应用机器人资源。
1. 打开机器人的“应用程序设置”。
1. 在“应用程序设置”窗口中，向下滚动到“应用程序设置”。
1. 找到 **botFileSecret** 并复制其值。
1. 使用 `msbot cli` 解密该文件。

    ```cmd
    msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
    ```

### <a name="update-your-local-bot-file"></a>更新本地 .bot 文件

打开已解密的 .bot 文件。 复制 `services` 节下列出的**所有**条目，将其添加到本地 .bot 文件。 解析任何重复的服务条目或重复的服务 ID。 保留机器人依赖的其他服务引用。 例如：

```json
"services": [
    {
        "type": "abs",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<bot-service-name>",
        "name": "<friendly-service-name>",
        "id": "1",
        "appId": "<app-id>"
    },
    {
        "type": "blob",
        "connectionString": "<connection-string>",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<blob-service-name>",
        "id": "2"
    },
    {
        "type": "endpoint",
        "appId": "",
        "appPassword": "",
        "endpoint": "<local-endpoint-url>",
        "name": "development",
        "id": "3"
    },
    {
        "type": "endpoint",
        "appId": "<app-id>",
        "appPassword": "<app-password>",
        "endpoint": "<hosted-endpoint-url>",
        "name": "production",
        "id": "4"
    },
    {
        "type": "appInsights",
        "instrumentationKey": "<instrumentation-key>",
        "applicationId": "<appinsights-app-id>",
        "apiKeys": {},
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group>",
        "serviceName": "<appinsights-service-name>",
        "id": "5"
    }
],
```

保存文件。

在发布之前，可以使用 msbot 工具生成新机密并加密 .bot 文件。 如果重新加密了 .bot 文件，请在 Azure 门户中更新机器人的 **botFileSecret** 来包含新机密。

```cmd
msbot secret --bot <name-of-bot-file> --new
```

> [!TIP]
> 对于 .bot 文件的文件属性，请在 Visual Studio 中确保“复制到输出目录”设置为“始终复制”。

### <a name="setup-a-repository"></a>设置存储库

若要支持持续部署，请使用你偏好的 git 源代码管理提供程序创建 git 存储库。 将代码提交到该存储库。

确保存储库根目录具有正确的文件，详见[准备存储库](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)。

### <a name="update-app-settings-in-azure"></a>在 Azure 中更新应用设置
本地机器人不使用加密的 .bot 文件，但 Azure 门户配置为使用加密的 .bot 文件。 可以通过删除 Azure 机器人设置中存储的 **botFileSecret** 来解决此问题。
1. 在 Azure 门户中，打开机器人的“Web 应用机器人”资源。
1. 打开机器人的“应用程序设置”。
1. 在“应用程序设置”窗口中，向下滚动到“应用程序设置”。
1. 找到 **botFileSecret** 并将其删除。 （如果你重新加密了 .bot 文件，请确保 **botFileSecret** 包含新机密且不要删除此设置。）
1. 更新机器人文件的名称，以便与签入到存储库中的文件相匹配。
1. 保存更改。

## <a name="2-deploy-using-azure-deployment-center"></a>2.使用 Azure 部署中心进行部署

现在需将机器人代码上传到 Azure。 请遵照[持续部署到 Azure 应用服务](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)主题中的说明操作。

请注意，我们建议使用 `App Service Kudu build server` 生成。

配置持续部署以后，会发布提交到存储库的更改。 但是，如果向机器人添加服务，则需将这些服务的条目添加到 .bot 文件。

## <a name="3-test-your-deployment"></a>3.测试部署

成功部署后，请等待几秒，然后视需要重启 Web 应用以清除所有缓存。 返回到“Web 应用机器人”边栏选项卡，并使用 Azure 门户中提供的“网络聊天”进行测试。

## <a name="additional-resources"></a>其他资源

- [How to investigate common issues with continuous deployment](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)（如何调查连续部署的常见问题）

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->
