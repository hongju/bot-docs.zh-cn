---
title: 通过 Azure 表存储管理自定义状态数据 | Microsoft Docs
description: 了解如何将 Azure 表存储与 Bot Builder SDK for Node.js 配合使用来保存和检索状态数据。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c77b07801b8eb0168ac3e09d7b271ddfb17a04ac
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297968"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>通过适用于 Node.js 的 Azure 表存储来管理自定义状态数据

在本文中，将实现 Azure 表存储来存储和管理机器人的状态数据。 机器人使用的默认连接器状态服务不适用于生产环境。 应使用 GitHub 上提供的 [Azure 扩展](https://www.npmjs.com/package/botbuilder-azure)，或者使用所选的数据存储平台实现自定义状态客户端。 下面是使用自定义状态存储的一些原因：

- 更高的状态 API 吞吐量（更多地控制性能）
- 降低地理分布的延迟
- 控制数据的存储位置（例如：美国西部和美国东部）
- 访问实际的状态数据
- 状态数据 db 不与其他机器人共享
- 存储超过 32 kb

## <a name="prerequisites"></a>先决条件

- [Node.js](https://nodejs.org/en/)。
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)。
- 必须具有 Node.js 机器人。 如果没有，请[创建机器人](bot-builder-nodejs-quickstart.md)。 
- [存储资源管理器](http://storageexplorer.com/)。

## <a name="create-azure-account"></a>创建 Azure 帐户
如果没有 Azure 帐户，请单击[此处](https://azure.microsoft.com/en-us/free/)注册免费帐户。

## <a name="set-up-the-azure-table-storage-service"></a>设置 Azure 表存储服务
1. 登录 Azure 门户后，单击“新建”创建新的 Azure 表存储。 
2. 搜索实现 Azure 表的“存储帐户”。 单击“创建”开始创建存储资源。 
3. 填写字段，单击屏幕底部的“创建”按钮以部署新存储服务。 
4. 部署新存储服务后，导航到刚刚创建的存储帐户。 可以在“存储帐户”边栏选项卡列表中找到。
4. 选择“访问密钥”，并复制密钥供将来使用。 机器人将使用存储帐户名称和密钥来调用存储服务以保存状态数据。

## <a name="install-botbuilder-azure-module"></a>安装 botbuilder-azure 模块

若要从命令提示符安装 `botbuilder-azure` 模块，请导航到机器人的目录并运行以下 npm 命令：

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>修改机器人代码

若要使用 Azure 表存储，将以下代码行添加到机器人的 app.js 文件。

1. 需要新安装的模块。

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. 配置连接设置以连接到 Azure。
   ```javascript
   // Table storage
   var tableName = "Table-Name"; // You define
   var storageName = "Table-Storage-Name"; // Obtain from Azure Portal
   var storageKey = "Azure-Table-Key"; // Obtain from Azure Portal
   ```
   `storageName` 和 `storageKay` 值可在 Azure 表的“访问密钥”菜单中找到。 如果 Azure 表中不存在 `tableName`，将为你创建一个。

3. 使用 `botbuilder-azure` 模块，创建两个新对象来连接到 Azure 表。 首先，创建传入连接配置设置的 `AzureTableClient` 实例。 接下来，创建传入 `AzureTableClient` 对象的 `AzureBotStorage` 实例。 例如：

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. 指定希望使用自定义数据库（而不是内存中存储）并向数据库添加会话信息。 例如：

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...

        // capture session user information
        session.userData = {"userId": session.message.user.id, "jobTitle": "Senior Developer"};

        // capture conversation information  
        session.conversationData[timestamp.toISOString().replace(/:/g,"-")] = session.message.text;

        // save data
        session.save();
   })
   .set('storage', tableStorage);
   ```
现在你已准备好使用模拟器测试机器人。

## <a name="run-your-bot-app"></a>运行机器人应用

从命令提示符导航到机器人的目录，并使用以下命令运行机器人：

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>将机器人连接到模拟器

此时，机器人在本地运行。 启动模拟器，然后从模拟器连接到机器人：

1. 在模拟器的地址栏中键入 <strong>http://localhost:port-number/api/messages</strong>，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。 现在可以将“Microsoft 应用 ID”和“Microsoft 应用密码”字段留空。 以后当你[注册机器人](~/bot-service-quickstart-registration.md)时，将获得此信息。
2. 单击“连接”。
3. 通过向机器人发送消息测试机器人。 像往常那样与机器人进行交互。 完成后，请转到“存储资源管理器”并查看保存的状态数据。

## <a name="view-data-in-storage-explorer"></a>在存储资源管理器中查看数据

若要查看状态数据，请打开“存储资源管理器”并使用 Azure 门户凭据连接到 Azure，或使用 `storageName` 和 `storageKey` 直接连接到表，然后导航到 `tableName`。 

![带 botdata 表行的存储资源管理器屏幕截图](~/media/bot-builder-nodejs-state-azure-table-storage/bot-builder-nodejs-state-azure-table-storage-query.png)

“数据”列中的一条会话记录如下所示：

```JSON
{
    "2018-05-15T18-23-48.780Z": "I'm the second user",
    "2018-05-15T18-23-55.120Z": "Do you know what time it is?",
    "2018-05-15T18-24-12.214Z": "I'm looking for information about the new process."
}
```

## <a name="next-step"></a>后续步骤

现在你可以完全控制机器人状态数据，接下来让我们看一下如何使用它来更好地管理会话流。

> [!div class="nextstepaction"]
> [管理会话流](bot-builder-nodejs-dialog-manage-conversation-flow.md)