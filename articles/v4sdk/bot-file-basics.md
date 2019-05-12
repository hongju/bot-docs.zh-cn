---
title: 管理机器人资源 | Microsoft Docs
description: 介绍机器人文件的用途和用法。
keywords: 机器人文件, .bot, .bot 文件, msbot, 机器人资源, 管理机器人资源
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 20b434c4fe5106ffe953c1a9ba9a282254511c9c
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032321"
---
# <a name="manage-bot-resources"></a>管理机器人资源

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

机器人往往使用不同的服务，例如 [LUIS.ai](https://luis.ai) 或 [QnaMaker.ai](https://qnamaker.ai)。 开发机器人时，需要能够跟踪所有这些服务。 可以使用各种方法，例如 appsettings.json、web.config 或 .env。 

> [!IMPORTANT]
> 在 Bot Framework SDK 4.3 版本之前，我们提供了 .bot 文件作为一种资源管理机制。 但是，在今后，建议你使用 appsettings.json 或 .env 文件来管理这些资源。 虽然 .bot 文件已**_弃用_**，但使用 .bot 文件的机器人目前仍可继续正常运行。 如果你一直在使用 .bot 文件管理资源，请按照适用的步骤来迁移设置。 

## <a name="migrating-settings-from-bot-file"></a>从 .bot 文件迁移设置
以下部分介绍如何从 .bot 文件迁移设置。 请根据相应的场景进行操作。

**场景 1：本地机器人有一个 .bot 文件**

在此场景中，你有一个使用 .bot 文件的本地机器人，但该机器人尚未迁移到 Azure 门户。 请按以下步骤将设置从 .bot 文件迁移到 appsettings.json 或 .env 文件。

- 如果 .bot 文件已加密，则需使用以下命令将其解密：

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
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

## <a name="additional-resources"></a>其他资源
- 若要了解机器人部署步骤，请参阅[部署](../bot-builder-deploy-az-cli.md)主题。
- 了解如何使用 [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-overview) 来保护及管理云应用程序与服务使用的加密密钥和机密。
