---
title: 使用 .bot 文件管理资源 | Microsoft Docs
description: 介绍机器人文件的用途和用法。
keywords: 机器人文件, .bot, .bot 文件, 机器人资源, 管理机器人资源
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/30/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 14552c55da4b1f9b581b81917496de179e92762b
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/18/2019
ms.locfileid: "58811499"
---
# <a name="manage-bot-resources"></a>管理机器人资源

机器人往往使用不同的服务，例如 [LUIS.ai](https://luis.ai) 或 [QnaMaker.ai](https://qnamaker.ai)。 开发机器人时，需要能够跟踪所有这些服务。 可以使用各种方法，例如 appsettings.json、web.config 或 .env。 

> [!IMPORTANT]
> 在 Bot Framework SDK 4.3 版本之前，我们提供了 .bot 文件作为一种资源管理机制。 但是，在今后，建议你使用 appsettings.json 或 .env 文件来管理这些资源。 虽然 .bot 文件已**_弃用_**，但使用 .bot 文件的机器人目前仍可继续正常运行。 如果你一直在使用 .bot 文件管理资源，请按照适用的步骤来迁移设置。 

## <a name="migrating-settings-from-bot-file"></a>从 .bot 文件迁移设置
以下部分介绍如何从 .bot 文件迁移设置。 请根据相应的场景进行操作。

**场景 1：本地机器人有一个 .bot 文件**

在此场景中，你有一个使用 .bot 文件的本地机器人，但该机器人尚未迁移到 Azure 门户。 请按以下步骤将设置从 .bot 文件迁移到 appsettings.json 或 .env 文件。

- 如果 .bot 文件已加密，则需使用以下命令将其解密：

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear` command.
```

- 打开解密的 .bot 文件，复制值并将其添加到 appsettings.json 或 .env 文件。
- 更新从 appsettings.json 或 .env 文件读取设置的代码。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

在 `ConfigureServices` 方法中，使用 ASP.NET Core 提供的配置对象，例如： 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

在 JavaScript 中，引用 `process.env` 对象提供的 .env 变量，例如：
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

根据需要使用 appsettings.json 或 .env 文件预配资源并将其连接到机器人。

**场景 2：已使用 .bot 文件将机器人部署到 Azure**

在此场景中，你已使用 .bot 文件将机器人部署到 Azure 门户，现在需将设置从 .bot 文件迁移到 appsettings.json 或 .env 文件。

- 从 Azure 门户下载机器人代码。 下载代码时，系统会提示你包括 appsettings.json 或 .env 文件，其中会有你的 MicrosoftAppId 和 MicrosoftAppPassword 以及任何其他的设置。 
- 打开下载的 appsettings.json 或 .env 文件，将设置从其中复制到本地 appsettings.json 或 .env 文件中。 请勿忘记从本地 appsettings.json 或 .env 文件中删除 botSecret 和 botFilePath 条目。
- 更新从 appsettings.json 或 .env 文件读取设置的代码。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
在 `ConfigureServices` 方法中，使用 ASP.NET Core 提供的配置对象，例如： 

**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
在 JavaScript 中，引用 `process.env` 对象提供的 .env 变量，例如：
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

此外还需从 **Azure 门户**的“应用程序设置”部分删除 `botFilePath` 和 `botFileSecret`。

根据需要使用 appsettings.json 或 .env 文件预配资源并将其连接到机器人。

**场景 3：机器人使用 appsettings.json 或 .env 文件**

此场景涵盖这样的情况：你从头开始使用 SDK 4.3 来开发机器人，没有可供迁移的现有 .bot 文件。 需要在机器人中使用的所有设置均位于 appsettings.json 或 .env 文件中，如下所示：

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

若要在 C# 代码中读取上述设置，需使用 ASP.NET Core 提供的配置对象，例如：**Startup.cs**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
在 JavaScript 中，引用 `process.env` 对象提供的 .env 变量，例如：**index.js**
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

根据需要使用 appsettings.json 或 .env 文件预配资源并将其连接到机器人。


## <a name="faq"></a>常见问题解答
**问：** 我想要在 Azure 门户中创建一个新的 V4 机器人。 在删除 .bot 文件后，Azure 门户的体验有何变化？

**答:** 在 Azure 门户中创建机器人时，不会创建 .bot 文件。 可以在 **Azure 门户**中使用“应用程序设置”部分查找 ID/密钥。 当你下载代码时，系统会为你将这些设置存储在 appsettings.json 或 .env 文件中。 可以在机器人中更新代码，以便在调用单项服务之前读取设置。 在更新机器人代码之后，即可使用 az bot publish 来部署机器人。

**问：** V3 机器人的情况如何？

**答:** V3 机器人的场景类似于没有 .bot 文件的 V4 机器人的场景。 部署过程仍然有效。 

## <a name="additional-resources"></a>其他资源
- 若要了解机器人部署步骤，请参阅[部署](../bot-builder-deploy-az-cli.md)主题。
- 为了保护密钥和机密，建议使用 Azure Key Vault。 Azure Key Vault 是一个用于安全地存储和访问机密（例如机器人的终结点和创作密钥）的工具。 它提供[密钥管理](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis)解决方案，方便你创建和控制加密密钥，使你能够严格控制机密。


<!--

# Manage resources with a .bot file

Bots usually consume lots of different services, such as [LUIS.ai](https://luis.ai) or [QnaMaker.ai](https://qnamaker.ai). When you are developing a bot, there is no uniform place to store the metadata about the services that are in use.  This prevents us from building tooling that looks at a bot as a whole.

To address this problem, we have created a **.bot file** to act as the place to bring all service references together in one place to 
enable tooling.  For example, the Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) uses a  .bot file to create a unified view over the connected services your bot consumes.  

With a .bot file, you can register services like:

* **Localhost** local debugger endpoints
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service registrations.
* [**LUIS.AI**](https://www.luis.ai/) LUIS gives your bot the ability to communicate with people using natural language.. 
* [**QnA Maker**](https://qnamaker.ai/) Build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) models for dispatching across multiple services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) for insights and bot analytics.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) for bot state persistence. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - globally distributed, multi-model database service to persist bot state.

Apart from these, your bot might rely on other custom services. You can leverage the [generic service](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) capability to connect a generic service configuration.

## When is a .bot file created? 
- If you create a bot using [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), a .bot file is automatically created for you with list of connected services provisioned. The .bot is encrypted by default.
- If you create a bot using Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio or using Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder), a .bot file is automatically created. No connected services are provisioned in this flow and the bot file is not encrypted.
- If you are starting with [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), every sample for Bot Framework V4 SDK includes a .bot file and the .bot file is not encrypted. 
- You can also create a bot file using the [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) tool.

## What does a bot file look like? 
Take a look at a sample [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) file.
To learn about encrypting and decrypting the .bot file, see [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## Why do I need a .bot file?

A .bot file is **not** a requirement to build bots with Bot Framework SDK. You can continue to use appsettings.json, web.config, env, 
keyvault or any mechanism you see fit to keep track of service references and keys that your bot depends on. However, to test
the bot using the Emulator, you'll need a .bot file. The good news is that Emulator can create a .bot file for testing. To do that, 
start the Emulator, click on the **create a new bot configuration** link on the Welcome page. In the dialog box that appears, type a **Bot name** and an **Endpoint URL**. Then connect.

The advantages of using .bot file are:
- Provides a standard way of storing resources regardless of the language/platform you use.   
- Bot Framework Emulator and CLI tools rely on and work great with tracking connected services in a consistent format (in a .bot file) 
- Elegant tooling solutions around services creation and management is harder without a well defined schema (.bot file).  


## Using .bot file in your Bot Framework SDK bot

You can use the .bot file to get service configuration information in your bot's code. The BotFramework-Configuration library available 
for [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) and [JS](https://www.npmjs.com/package/botframework-config) helps you load a bot file and supports several methods to query and get the appropriate service configuration information.

## Additional resources
Refer to [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) readme file for more information on using a bot file.

-->

