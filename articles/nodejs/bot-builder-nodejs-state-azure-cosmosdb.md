---
title: 使用 Azure Cosmos DB 管理自定义状态数据 | Microsoft Docs
description: 了解如何将 Azure Cosmos DB 与 Bot Builder SDK for Node.js 配合使用来保存和检索状态数据。
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9d3e1c315399ce3cadc6371ceb93055c836590a6
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298425"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>使用 Azure Cosmos DB for Node.js 管理自定义状态数据

在本文中，将实现 Cosmos DB 存储来存储和管理机器人的状态数据。 机器人使用的默认连接器状态服务不适用于生产环境。 应使用 GitHub 上提供的 [Azure 扩展](https://www.npmjs.com/package/botbuilder-azure)，或者使用所选的数据存储平台实现自定义状态客户端。 下面是使用自定义状态存储的一些原因：

- 状态 API 吞吐量更高（性能控制更强）
- 降低地理分布的延迟
- 控制数据的存储位置（例如美国西部与美国东部）
- 访问实际的状态数据
- 状态数据数据库不与其他机器人共享
- 存储超过 32kb

## <a name="prerequisites"></a>先决条件

- [Node.js](https://nodejs.org/en/)。
- [Bot Framework Emulator](~/bot-service-debug-emulator.md)
- 必须具备 Node.js 机器人。 如果没有，请[创建一个](bot-builder-nodejs-quickstart.md)。 

## <a name="create-azure-account"></a>创建 Azure 帐户
如果没有 Azure 帐户，请单击[此处](https://azure.microsoft.com/en-us/free/)注册免费帐户。

## <a name="set-up-the-azure-cosmos-db-database"></a>设置 Azure Cosmos DB 数据库
1. 登录 Azure 门户后，单击“新建”来新建一个 Azure Cosmos DB 数据库。 
2. 单击“数据库”。 
3. 找到“Azure Cosmos DB”，然后单击“创建”。
4. 填写字段。 对于 API 字段，选择“SQL (DocumentDB)”。 填写完所有字段后，单击屏幕底部的“创建”按钮来部署新的数据库。 
5. 部署新的数据库后，导航到新数据库。 单击“访问密钥”以查找密钥和连接字符串。 机器人将使用此信息来调用存储服务以保存状态数据。

## <a name="install-botbuilder-azure-module"></a>安装 botbuilder-azure 模块

要从命令提示符处安装 `botbuilder-azure` 模块，请导航到机器人的目录并运行以下 npm 命令：

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>修改机器人代码

要使用 Azure Cosmos DB 数据库，请在机器人的 app.js 文件中添加以下代码行。

1. 需使用新安装的模块。

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. 配置连接设置以连接到 Azure。
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   `host` 和 `masterKey` 值可在数据库的“密钥”菜单中找到。 如果 Azure 数据库中不存在 `database` 和 `collection` 条目，系统会为你创建它们。

3. 通过 `botbuilder-azure` 模块创建两个新的对象来连接到 Azure 数据库。 首先，创建传入连接配置设置的 `DocumentDBClient` 实例（根据上述内容定义为 `documentDbOptions`）。 接下来，创建传入 `DocumentDBClient` 对象的 `AzureBotStorage` 实例。 例如：
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. 指定希望使用自定义数据库，而不是内存中存储。 例如：

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
   ```

现在你已准备好使用模拟器测试机器人。

## <a name="run-your-bot-app"></a>运行机器人应用

从命令提示符导航到机器人的目录，并使用以下命令运行机器人：

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>将机器人连接到模拟器

此时，机器人在本地运行。 启动模拟器，然后从模拟器中连接到机器人：

1. 在模拟器的地址栏中键入 http://localhost:port-number/api/messages，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。 可暂时将“Microsoft 应用 ID”和“Microsoft 应用密码”字段留空。 稍后[注册机器人](~/bot-service-quickstart-registration.md)时将获取此信息。
2. 单击“连接”。
3. 通过向机器人发送消息测试机器人。 像往常一样与机器人进行交互。 完成后，转到存储资源管理器并查看所保存的状态数据。

## <a name="view-state-data-on-azure-portal"></a>在 Azure 门户中查看状态数据

要查看状态数据，请登录 Azure 门户并导航到你的数据库。 单击“数据资源管理器(预览版)”，验证是否正在保存来自机器人的状态信息。

## <a name="next-step"></a>后续步骤

现在你已完全掌控机器人的状态数据，接下来让我们看一下如何使用它来更好地管理聊天流。

> [!div class="nextstepaction"]
> [管理聊天流](bot-builder-nodejs-dialog-manage-conversation-flow.md)