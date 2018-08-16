---
title: 将机器人迁移到 Azure | Microsoft Docs
description: 了解如何将机器人从旧版 Bot Framework 门户迁移到 Azure 门户中的机器人服务。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 6/22/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e8c927ce49a2d8aecf0113a09cbedcf1135a8380
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298093"
---
# <a name="migrate-your-bot-to-azure"></a>将机器人迁移到 Azure

在 [Bot Framework 门户](http://dev.botframework.com)中创建的所有 **Azure 机器人服务（预览版）** 都必须迁移到 Azure 的新机器人服务。 该服务在 2017 年 12 月正式推出 (GA)。 

请注意，仅连接到以下通道的注册机器人不需要迁移：Teams、Skype 或 Cortana。 例如，连接到 Facebook 和 Skype 的注册机器人需要迁移，但连接到 Skype 和 Cortana 的注册机器人不需要迁移。

> [!IMPORTANT]
> 在迁移使用 Node.js 创建的 Functions 机器人之前，需要使用 Azure Functions 包将 node_modules 模块打包在一起。 这样做将提高迁移期间的性能和迁移后 Functions 机器人的执行力。 若要打包模块，请参阅[使用 Funcpack 包打包 Functions 机器人](#package-a-functions-bot-with-funcpack)。

若要迁移机器人，执行以下操作：

1. 登录到 [Bot Framework 门户](http://dev.botframework.com)，然后单击“我的机器人”。
2. 为要迁移的机器人单击“迁移”按钮。
3. 接受“条款”然后单击“迁移”以启动迁移过程，或者单击“取消”取消此操作。

> [!IMPORTANT]
> 正在进行迁移任务时，请勿导航离开页面或刷新页面。 执行此操作将导致过早地停止迁移任务，必须重新执行此操作。 若要确保迁移成功完成，请等待确认消息。

迁移过程成功完成后，“迁移状态”将指示已迁移机器人，“回滚迁移”按钮在迁移日期后的一周内可用，以免出现问题。

单击已迁移机器人的名称即可打开 [Azure 门户](http://portal.azure.com)中的机器人。

## <a name="package-a-functions-bot-with-funcpack"></a>使用 Funcpack 打包 Functions 机器人

通过 Node.js 创建的 Functions 机器人必须在迁移之前使用 [Funcpack](https://github.com/Azure/azure-functions-pack) 进行打包。 若要使用 Funcpack 打包项目，请执行以下步骤：

1.  如果还没有代码，请在本地[下载](bot-service-build-download-source-code.md#download-bot-source-code)。
2.  将 packages.json 中的 npm 包更新到最新版本，然后运行 `npm install`。
3.  打开“messages/index.js”并将 `module.exports = { default: connector.listen() }` 更改为 `module.exports = connector.listen();`
4.  通过 npm 安装 Funcpack：`npm install -g azure-functions-pack`
5.  若要打包“node_modules”目录，运行以下命令：`funcpack pack ./`
6.  通过运行 Functions 机器人使用 Bot Framework 模拟器在本地测试机器人。 有关如何运行 funcpack 机器人的详细信息，请参见[此处](https://github.com/Azure/azure-functions-pack#how-to-run)。 
7.  将代码上传回 Azure。 请确保上传 `.funcpack` 目录。 不需要上传“node_modules”目录。
8. 测试远程机器人，确保按预期方式进行响应。
9. 使用上述步骤[迁移机器人](#migrate-your-bot-to-azure)。

## <a name="migration-under-the-hood"></a>迁移后台

根据要迁移的机器人的类型，下面的列表可以帮助你更好地了解后台发生的事情。

* Web App 机器人或 Functions 机器人：对于这些类型的机器人，源代码项目从旧机器人复制到新机器人。 其他资源（例如机器人存储、Application Insights、LUIS 等）保持原样。 在这些情况下，新机器人包含一份这些现有资源的 ID/密钥/密码副本。 
* 机器人通道注册：对于此类型的机器人，迁移过程只需创建一个新机器人通道注册，并从旧机器人那里复制终结点。 
* 无论迁移的是哪种类型的机器人，迁移过程都不会改变现有机器人的状态。 如果需要的话，这可以让你安全回滚。
* 如果有[连续部署](bot-service-build-continuous-deployment.md)设置，将需要再次设置，以便源代码管理改为连接到新机器人。

## <a name="understanding-azure-resources-after-migration"></a>了解迁移后的 Azure 资源
完成迁移后，资源组将包含机器人运行所需的一些 Azure 资源。 资源的类型和数量取决于迁移的机器人类型。 请参阅以下各节以了解详细信息。

### <a name="registration-bot"></a>注册机器人

这是最简单的类型。 Azure 中的资源组将仅包含一项资源类型“机器人通道注册”。 这相当于 Bot Framework 开发人员门户中以前的机器人记录。

![Azure 中的机器人通道注册机器人列表](~/media/bot-service-migrate-bot/channel-registration-bot.png)

### <a name="web-app-bot"></a>Web 应用机器人
Web 应用机器人迁移将预配一个“Web 应用机器人”类型的机器人服务资源和一个新应用服务 Web 应用（以下屏幕截图中的绿色部分）。 以前的 Azure 机器人服务（预览版）机器人仍存在，可以删除（以下屏幕截图中的红色部分）。

![Azure 中的 Web 应用机器人列表](~/media/bot-service-migrate-bot/web-app-bot.png)

### <a name="functions-bot"></a>Functions 机器人
Functions 机器人迁移将预配一个“Functions 机器人”类型的机器人服务资源和一个新应用服务 Functions App（以下屏幕截图中的绿色部分）。 以前的 Azure 机器人服务（预览版）机器人仍存在，可以删除（以下屏幕截图中的红色部分）。

![Azure 中的 Functions 机器人列表](~/media/bot-service-migrate-bot/functions-bot.png)


## <a name="roll-back-migration"></a>回滚迁移

在迁移期间或在迁移后出现与机器人相关的问题，可以回滚迁移。 若要回滚迁移，请执行以下操作：

1. 登录到 [Bot Framework 门户](http://dev.botframework.com)，然后单击“我的机器人”。
2. 为要回滚的机器人单击“回滚迁移”按钮。 将显示一条提示。
3. 单击“是，请回滚”继续或“取消”以取消回滚操作。

> [!NOTE]
> 回滚功能将在迁移后一周内可用，因此应仅在遇到已迁移机器人问题时使用。

## <a name="migration-troubleshootingknown-issues"></a>迁移疑难解答/已知问题
我的 node.js/functions 机器人已成功迁移，但它无法响应：

* 检查终结点
  * 转到机器人资源的“设置”边栏选项卡，并验证机器人终结点是否有“代码”查询字符串参数，且其中包含值。 如果没有，则需要添加。
* 查看 kudu 中用于备份的机密文件夹
  * 在某些极少数情况下，可能会有几个造成冲突的备份机密文件。 转到 Kudu 中的 home\data\Functions\secrets 文件夹，并删除在其中找到的任何 host.snapshot（或 host.backup）文件。 应只有一个 host.json 和一个 messages.json。 最后重启应用服务，然后重试与机器人聊天。

有关任何其他问题，请向 Azure 支持提交 CRI 或在 [GitHub](https://github.com/MicrosoftDocs/bot-framework-docs/issues) 中提出问题。


## <a name="next-steps"></a>后续步骤

现已迁移机器人，接下来了解如何在 Azure 门户中管理机器人。

> [!div class="nextstepaction"]
> [管理机器人](bot-service-manage-overview.md)
