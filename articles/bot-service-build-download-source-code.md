---
title: 下载并重新部署机器人源代码 | Microsoft Docs
description: 了解如何下载和发布机器人服务。
keywords: 下载源代码, 重新部署, 部署, zip 文件, 发布
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/26/2018
ms.openlocfilehash: 19dd474c16224cc811a214acea6e9cb51da95b3f
ms.sourcegitcommit: cb0b70d7cf1081b08eaf1fddb69f7db3b95b1b09
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/09/2018
ms.locfileid: "51332761"
---
# <a name="download-and-redeploy-bot-code"></a>下载并重新部署机器人代码
可以通过 Azure 机器人服务下载适用于机器人的整个源项目，以便使用所选 IDE 在本地进行工作。 更新完代码以后，即可将所做的更改发布回 Azure 门户。 我们会介绍如何使用 Azure 门户和 `az` cli 来下载代码。 我们还会介绍如何使用 Visual Studio 和 `az` cli 工具来重新部署更新的机器人代码。 你可以选择最适合自己的方法。

## <a name="prerequisites"></a>先决条件
- 安装 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- 使用 `az extension add -n botservice` 命令安装 az botservice 扩展

### <a name="download-code-using-the-azure-portal"></a>使用 Azure 门户下载代码
若要从 [Azure 门户](https://portal.azure.com)下载代码，请执行以下操作：
1. 打开机器人的边栏选项卡。
1. 在“机器人管理”部分下，单击“生成”。
1. 在“下载源代码”下，单击“下载 zip 文件”。
1. 等待 Azure 准备下载 URI，然后单击通知中的“下载 zip 文件”。
1. 将 zip 文件保存并解压到本地目录。

如果有 C# 机器人，请更新 `appsettings.json` 文件，使之包含 .bot 文件信息，如下所示：

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

如果有一个 node.js 机器人，请添加一个包含以下条目的 `.env` 文件：
```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

`botFilePath` 引用机器人的名称，请直接将“yourbasicBot.bot”替换为自己的机器人名称。 若要获取 `botFileSecret` 密钥，请参阅 [Bot 文件加密](https://aka.ms/bot-file-encryption)一文，了解如何为机器人生成密钥。

接下来，请编辑现有的源文件或将新的源文件添加到项目中，以便对源进行更改。 使用模拟器测试代码。 准备好将修改的代码重新部署到 Azure 门户以后，请按以下说明操作。

### <a name="publish-code-using-visual-studio"></a>使用 Visual Studio 发布代码
1. 在 Visual Studio 中右键单击项目名称，然后单击“发布...”。“发布”窗口随即打开。

![Azure 发布](~/media/azure-bot-build/azure-csharp-publish.png)

2. 选择项目的配置文件。
3. 复制在项目的 _publish.cmd_ 文件中列出的密码。
4. 单击“发布” 。
5. 出现提示时，输入在步骤 3 中复制的密码。   

配置项目以后，项目更改会发布到 Azure。 

接下来介绍如何使用 `az` cli 来下载并重新部署代码。

### <a name="download-code-using-azure-cli"></a>使用 Azure CLI 下载代码

首先使用 az cli 工具登录到 Azure 门户。

```azcli
az login
```

系统将提示你提供唯一的临时身份验证代码。 若要登录，请使用 Web 浏览器访问 Microsoft [设备登录](https://microsoft.com/devicelogin)，然后粘贴 CLI 提供的代码，以便继续操作。

若要通过 `az` cli 下载代码，请使用以下命令：
```azcli
az bot download --name "my-bot-name" --resource-group "my-resource-group"`
```
在下载代码以后，请执行以下操作：
- 对于 C# 机器人，请更新 appsettings.json 文件，使之包含 .bot 文件信息，如下所示：

```
{
  "botFilePath": "yourbasicBot.bot",
  "botFileSecret": "ukxxxxxxxxxxxs="
}
```

- 对于 node.js 机器人，请添加一个包含以下条目的 .env 文件：

```
botFilePath=yourbasicBot.bot
botFileSecret=ukxxxxxxxxxxxxs=
```

接下来，请编辑现有的源文件或将新的源文件添加到项目中，以便对源进行更改。 使用模拟器测试代码。 准备好将修改的代码重新部署到 Azure 门户以后，请按以下说明操作。

### <a name="login-to-azure-cli-by-running-the-following-command"></a>通过运行以下命令登录到 Azure CLI。
如果已登录，则可跳过此步骤。

```azcli
az login
```
系统将提示你提供唯一的临时身份验证代码。 若要登录，请使用 Web 浏览器访问 Microsoft [设备登录](https://microsoft.com/devicelogin)，然后粘贴 CLI 提供的代码，以便继续操作。

### <a name="publish-code-using-azure-cli"></a>使用 Azure CLI 发布代码
若要通过 `az` cli 将代码发布回 Azure，请使用以下命令：
```azcli
az bot publish --name "my-bot-name" --resource-group "my-resource-group" --code-dir <path to directory> 
```

可以使用 `code-dir` 选项来指示要使用的目录。 如果未提供该选项，`az bot publish` 命令会使用本地目录进行发布。

## <a name="next-steps"></a>后续步骤
了解如何将更改上传回 Azure 后，就可以为机器人设置持续部署了。

> [!div class="nextstepaction"]
> [设置持续部署](bot-service-build-continuous-deployment.md)
