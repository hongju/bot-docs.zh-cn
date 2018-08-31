---
title: 下载并重新部署机器人源代码 | Microsoft Docs
description: 了解如何下载和发布机器人服务。
keywords: 下载源代码, 重新部署, 部署, zip 文件, 发布
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: b77e096d28f51f605db9c49d36e796553f9293ef
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756479"
---
# <a name="download-and-redeploy-bot-source-code"></a>下载并重新部署机器人源代码

使用机器人服务可以下载用于机器人的整个源项目。 这样，你可以使用所选的 IDE 在本地处理机器人。 一旦完成更改后，就可以将所做的更改发布回 Azure。 

本主题将演示如何下载机器人的源代码和将所做的更改发布回 Azure。 

## <a name="download-bot-source-code"></a>下载机器人源代码

要本地开发机器人，请执行以下操作：

1. 在 Azure 门户中打开机器人的边栏选项卡。
2. 在“机器人管理”部分下，单击“生成”。
3. 单击“下载 zip 文件”。 

   ![下载源代码](~/media/azure-bot-build/download-zip-file.png)

4. 将 zip 文件解压到本地目录。
5. 导航到解压文件夹，并在收藏夹的 IDE 中打开源文件。
6. 对你的源进行更改。 编辑现有的源文件，或将新的源文件添加到项目中。

准备就绪后，可以将源发布回 Azure。

## <a name="publish-node-bot-source-code-to-azure"></a>将 Node 机器人源代码发布到 Azure

要安装这些包，请从命令提示符处浏览到项目目录，然后运行以下 NPM 命令。

**注意：** 这些包只需添加一次。

```console
npm install --save fs
npm install --save path
npm install --save request
npm install --save zip-folder
```

现在你已准备好将项目发布到 Microsoft Azure。 要将项目发布到 Microsoft Azure，请在命令提示符中运行以下 NPM 命令：

```console
npm run azure-publish
```

> [!NOTE]
> 如果运行 NPM 命令后遇到错误，则可能需要将 `"scripts": {"azure-publish": "node publish.js"}` 添加到 `package.json` 文件，然后再次运行。

## <a name="publish-c-bot-source-code-to-azure"></a>将 C# 机器人源代码发布到 Azure

使用 Visual Studio 将 C# 代码发布到 Azure 需要个步骤：首先配置发布设置。 然后发布你的更改。

要通过 Visual Studio 配置发布，请执行以下操作：

1. 在 Visual Studio 中，单击“解决方案资源管理器”。
2. 右键单击项目名，并单击 **发布...** \。“发布”窗口随即打开。
3. 单击“新建配置文件”，单击“导入配置文件”，再单击“确定”。
4. 导航到项目文件夹，然后导航到“PostDeployScripts”文件夹，选择以“PublishSettings”结尾的文件。 单击“打开”。

你的项目现在已配置为将更改发布到 Azure。

配置项目后，可以通过执行以下操作将机器人源代码发布回 Azure：

1. 在 Visual Studio 中，单击“解决方案资源管理器”。
2. 右键单击项目名，并单击“发布...”。
3. 单击“发布”按钮将所做的更改发布到 Azure。

## <a name="next-steps"></a>后续步骤
你已了解如何在本地构建机器人，现在可以为机器人设置持续部署。

> [!div class="nextstepaction"]
> [设置持续部署](bot-service-build-continuous-deployment.md)
