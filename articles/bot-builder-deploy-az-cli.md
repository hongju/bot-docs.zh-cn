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
ms.date: 04/12/2019
ms.openlocfilehash: 4532fe55705524573de55017e633289255a20ab9
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/18/2019
ms.locfileid: "59508214"
---
# <a name="deploy-your-bot"></a>部署机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

创建机器人并在本地对其进行测试后，可将其部署到 Azure，以便可以从任何位置访问它。 将机器人部署到 Azure 需要支付服务使用费。 [计费和成本管理](https://docs.microsoft.com/en-us/azure/billing/)一文可帮助你了解 Azure 计费方式、如何监视使用量与费用，以及如何管理帐户和订阅。

本文介绍如何将 C# 和 JavaScript 机器人部署到 Azure。 在执行相关步骤之前最好是先阅读本文，以完全了解在部署机器人时所涉及到的工作。

## <a name="prerequisites"></a>先决条件
- 如果还没有 [Azure 订阅](http://portal.azure.com)，请在开始前创建一个免费帐户。
- 在本地计算机上开发的 [**CSharp**](./dotnet/bot-builder-dotnet-sdk-quickstart.md) 或 [**JavaScript**](./javascript/bot-builder-javascript-quickstart.md) 机器人。

## <a name="1-prepare-for-deployment"></a>1.准备部署
部署过程要求在 Azure 中有一个目标 Web 应用机器人，以便将本地机器人部署到其中。 Azure 中的目标 Web 应用机器人以及通过它预配的资源可供本地机器人用于部署。 这是必需的，因为本地机器人没有预配所有必需的 Azure 资源。 当你创建目标 Web 应用机器人时，系统会为你预配以下资源：
-   Web 应用机器人 - 使用此机器人是因为需要将本地机器人部署到其中。
-   应用服务计划 - 提供应用服务应用需要运行的资源。
-   应用服务 - 一项用于托管 Web 应用程序的服务
-   存储帐户 - 包含所有 Azure 存储数据对象：Blob、文件、队列、表和磁盘。

在创建目标 Web 应用机器人的过程中，也会为机器人生成应用 ID 和密码。 在 Azure 中，应用 ID 和密码支持[服务身份验证和授权](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)。 需检索部分此类信息，用在本地机器人代码中。 

> [!IMPORTANT]
> 在 Azure 门户中使用的机器人模板的编程语言必须与编写机器人时使用的编程语言匹配。

如果已在 Azure 中创建了想要使用的机器人，则可以选择性地阅读“创建新的 Web 应用机器人”。

1. 登录到 [Azure 门户](https://portal.azure.com)。
1. 在 Azure 门户左上角单击“创建新资源”链接，然后选择“AI + 计算机学习”>“Web 应用机器人”。
1. 此时会打开一个新的边栏选项卡，其中包含有关 Web 应用机器人的信息。 
1. 在“机器人服务”边栏选项卡中，提供所请求的有关机器人的信息。
1. 单击“创建”创建服务并将机器人部署到云。 此过程可能需要数分钟。

### <a name="download-the-source-code"></a>下载源代码
创建目标 Web 应用机器人以后，需将机器人代码从 Azure 门户下载到本地计算机。 下载代码是为了获取 appsettings.json 或 .env 文件中的服务引用（例如，MicrosoftAppID、MicrosoftAppPassword、LUIS 或 QnA）。 

1. 在“机器人管理”部分中，单击“生成”。
1. 单击右窗格中的“下载机器人源代码”链接。
1. 按照提示下载代码，然后解压缩该文件夹。
    1. [!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="update-your-local-appsettingsjson-or-env-file"></a>更新本地的 appsettings.json 或 .env 文件

打开下载的 appsettings.json 或 .env 文件。 复制其中列出的**所有**条目，将其添加到本地的 appsettings.json 或 .env 文件。 解析任何重复的服务条目或重复的服务 ID。 保留机器人依赖的其他服务引用。

保存文件。

### <a name="update-local-bot-code"></a>更新本地机器人代码
更新本地的 Startup.cs 或 index.js 文件，以便使用 appsettings.json 或 .env 文件而不是 .bot 文件。 .bot 文件已弃用，我们会更新 VSIX 模板、Yeoman 生成器、示例和剩余文档，使它们都使用 appsettings.json 或 .env 文件而不是 .bot 文件。 同时，你需要更改机器人代码。 

更新从 appsettings.json 或 .env 文件读取设置的代码。 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 `ConfigureServices` 方法中，使用 ASP.NET Core 提供的配置对象，例如： 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="jstabjs"></a>[JS](#tab/js)

在 JavaScript 中，引用 `process.env` 对象提供的 .env 变量，例如：
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

- 保存文件并测试机器人。

### <a name="setup-a-repository"></a>设置存储库

若要支持持续部署，请使用你偏好的 git 源代码管理提供程序创建 git 存储库。 将代码提交到该存储库。

确保存储库根目录具有正确的文件，详见[准备存储库](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository)。

### <a name="update-app-settings-in-azure"></a>在 Azure 中更新应用设置
本地机器人不使用加密的 .bot 文件，但 Azure 门户配置为使用加密的 .bot 文件。 可以通过删除 Azure 机器人设置中存储的 **botFileSecret** 来解决此问题。
1. 在 Azure 门户中，打开机器人的“Web 应用机器人”资源。
1. 打开机器人的“应用程序设置”。
1. 在“应用程序设置”窗口中，向下滚动到“应用程序设置”。
1. 查看机器人是否有 **botFileSecret** 和 **botFilePath** 条目。 如果有，请将其删除。
1. 保存更改。

## <a name="2-deploy-using-azure-deployment-center"></a>2.使用 Azure 部署中心进行部署

现在需将机器人代码上传到 Azure。 请遵照[持续部署到 Azure 应用服务](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment)主题中的说明操作。

请注意，我们建议使用 `App Service Kudu build server` 生成。

配置持续部署以后，会发布提交到存储库的更改。 但是，如果向机器人添加服务，则需将这些服务的条目添加到 .bot 文件。

## <a name="3-test-your-deployment"></a>3.测试部署

成功部署后，请等待几秒，然后视需要重启 Web 应用以清除所有缓存。 返回到“Web 应用机器人”边栏选项卡，并使用 Azure 门户中提供的“网络聊天”进行测试。

## <a name="additional-resources"></a>其他资源
- [How to investigate common issues with continuous deployment](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)（如何调查连续部署的常见问题）

