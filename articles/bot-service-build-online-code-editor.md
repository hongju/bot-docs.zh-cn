---
title: 使用 Azure 联机代码编辑器生成机器人 | Microsoft Docs
description: 了解如何在机器人服务中使用联机代码编辑器生成机器人。
keywords: 联机代码编辑器, azure 门户, functions 机器人
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: bdb287e26c31a784bf6f53ad1601d586781c5ef3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298031"
---
# <a name="edit-a-bot-with-online-code-editor"></a>使用联机代码编辑器编辑机器人

可以使用联机代码编辑器生成机器人，无需使用 IDE。 本主题介绍如何在联机代码编辑器中打开机器人代码。 

## <a name="edit-bot-source-code-in-online-code-editor"></a>在联机代码编辑器中编辑机器人源代码

要在联机代码编辑器中编辑机器人的源代码，请为已有类型的特定类型执行以下步骤。

### <a name="web-app-bot"></a>Web 应用机器人
1. 登录到 [Azure 门户](http://portal.azure.com)，并打开机器人的边栏选项卡。
2. 在“机器人管理”部分下，单击“生成”。
3. 单击“打开联机代码编辑器”。 此操作将在新的浏览器窗口中打开机器人应用的代码。 

   ![打开联机代码编辑器](~/media/azure-bot-build/open-online-code-editor.png)

   根据机器人的语言，WWWRoot 目录下的文件结构将有所不同。 例如，如果你有 C# 机器人，那么 WWWRoot 可能如下所示：

   ![C# 文件结构](~/media/azure-bot-build/cs-wwwroot-structure.png)

   例如，如果你有 Node.js 机器人，那么 WWWRoot 可能如下所示：

   ![Node.js 文件结构](~/media/azure-bot-build/node-wwwroot-structure.png)

4. 更改代码。 例如，对于 C# 机器人，可以从 Dialogs/EchoDialog.cs 文件开始。 对于 Node.js 机器人，可以从 App.js 文件开始。

   > [!NOTE]
   > 虽然可以对项目中的当前源文件进行代码更改，但无法使用联机代码编辑器创建新的源文件。 若要向机器人添加新的源文件，需要[下载源](bot-service-build-download-source-code.md)项目，添加文件并将更改发布回 Azure.

5. 保存所做更改。 对于消耗计划中的 C# 机器人及所有 Node.js 机器人，在单击“保存”按钮保存源代码后，机器人会自动更新。 

   ![Node.js 文件结构](~/media/azure-bot-build/node-save-file.png)

   对于应用服务计划中的 C# 机器人，打开“控制台”边栏选项卡并发送 build.cmd 命令。 

   ![在控制台边栏选项卡中生成项目](~/media/azure-bot-build/cs-console-build-cmd.png)
 
   > [!NOTE]
   > 如果此命令无法生成，请尝试重启机器人的应用服务并尝试重新生成。 要重启应用服务，从机器人的边栏选项卡中单击“所有应用服务设置”，然后单击“重启”按钮。
   > ![重启 Web 应用](~/media/azure-bot-build/open-online-code-editor-restart-appservice.png)

6. 切换回 Azure 门户，并单击“在网上聊天中测试”测试更改。 如果已为此机器人打开了网上聊天，请单击“重新开始”，查看新的更改。

### <a name="functions-bot"></a>Functions 机器人

1. 登录到 [Azure 门户](http://portal.azure.com)，并打开机器人的边栏选项卡。
2. 在“机器人管理”部分下，单击“生成”。
3. 单击“在 Azure Functions 中打开此机器人”。 将通过 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> UI 打开机器人。 
4. 更改代码。 例如，更新功能的消息代码。 下面的屏幕截图显示 Node.js Functions 机器人的消息代码。

   ![Functions 机器人消息代码编辑器](~/media/azure-bot-build/functions-messages-code.png)

5. 保存代码更改。
6. 切换回 Azure 门户，并单击“在网上聊天中测试”测试更改。 如果已为此机器人打开了网上聊天，请单击“重新开始”，查看新的更改。

## <a name="next-steps"></a>后续步骤
现在你已了解如何使用联机代码编辑器编辑机器人代码，还可以使用自己喜欢的 IDE 在本地生成机器人。

> [!div class="nextstepaction"]
> [下载机器人源代码](bot-service-build-download-source-code.md)
