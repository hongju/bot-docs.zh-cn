---
title: 通过 Azure 表存储管理自定义状态数据 | Microsoft Docs
description: 了解如何将 Azure 表存储与 Bot Builder SDK for .NET 配合使用来保存和检索状态数据
author: kaiqb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e5ff23caa1bdb1158ab19fa7c66e1fe4f6899f49
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905110"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>通过适用于 .NET 的 Azure 表存储来管理自定义状态数据

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

在本文中，将实施 Azure 表存储来存储和管理机器人的状态数据。 机器人使用的默认连接器状态服务不适用于生产环境。 应使用 GitHub 上提供的 [Azure 扩展](https://github.com/Microsoft/BotBuilder-Azure)，或者使用所选的数据存储平台实现自定义状态客户端。 下面是使用自定义状态存储的一些原因：
 - 状态 API 吞吐量更高（性能控制更强）
 - 降低地理分布的延迟
 - 控制数据的存储位置
 - 访问实际的状态数据
 - 存储超过 32kb 的数据

## <a name="prerequisites"></a>先决条件
需要：
 - [Microsoft Azure 帐户](https://azure.microsoft.com/en-us/free/)
 - [Visual Studio 2015 或更高版本](https://www.visualstudio.com/)
 - [Bot Builder Azure NuGet 包](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/)
 - [Autofac Web Api2 NuGet 包](https://www.nuget.org/packages/Autofac.WebApi2/)
 - [Bot Framework Emulator](https://emulator.botframework.com/)
 - [Azure 存储资源管理器](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>创建 Azure 帐户
如果没有 Azure 帐户，请单击[此处](https://azure.microsoft.com/en-us/free/)注册免费帐户。

## <a name="set-up-the-azure-table-storage-service"></a>设置 Azure 表存储服务
1. 登录 Azure 门户后，单击“新建”创建新的 Azure 表存储。 
2. 搜索实施 Azure 表的“存储账户”。 
3. 填写字段，单击屏幕底部的“创建”按钮以部署新存储服务。 部署新存储服务后，它将显示可用的功能和选项。
4. 选择左侧的“访问密钥”选项卡，然后复制连接字符串供稍后使用。 机器人将使用此连接字符串来调用存储服务以保存状态数据。

## <a name="install-nuget-packages"></a>安装 NuGet 包
1. 打开现有的 C# 机器人项目，或在 Visual Studio 中使用 C# 机器人模板创建一个新项目。 
2. 安装以下 NuGet 包：
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>添加连接字符串 
将以下条目添加到 Web.config 文件中： 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
将“YourConnectionString”替换为先前保存的 Azure 表存储的连接字符串。 保存 Web.config 文件。

## <a name="modify-your-bot-code"></a>修改机器人代码
在 Global.asax.cs 文件中，添加以下 `using` 语句：
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
在 `Application_Start()` 方法中，创建 `TableBotDataStore` 类的实例。 `TableBotDataStore` 类实现 `IBotDataStore<BotData>` 接口。 通过 `IBotDataStore` 接口，可替代默认“连接器状态服务”连接。
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
注册该服务，如下所示：
 ```cs
 Conversation.UpdateContainer(
            builder =>
            {
                builder.Register(c => store)
                          .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                          .AsSelf()
                          .SingleInstance();

                builder.Register(c => new CachingBotDataStore(store,
                           CachingBotDataStoreConsistencyPolicy
                           .ETagBasedConsistency))
                           .As<IBotDataStore<BotData>>()
                           .AsSelf()
                           .InstancePerLifetimeScope();

                
            });
 ```
保存 Global.asax.cs 文件。

## <a name="run-your-bot-app"></a>运行机器人应用
在 Visual Studio 中运行机器人，你添加的代码将在 Azure 中创建自定义 **botdata** 表。

## <a name="connect-your-bot-to-the-emulator"></a>将机器人连接到模拟器
此时，机器人在本地运行。 接下来，启动模拟器，然后在模拟器中连接到机器人：
1. 在地址栏中键入 http://localhost:port-number/api/messages，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。 可暂时将“Microsoft 应用 ID”和“Microsoft 应用密码”字段留空。 稍后[注册机器人](~/bot-service-quickstart-registration.md)时将获取此信息。
2. 单击“连接”。 
3. 通过在模拟器中键入一些消息来测试机器人。 

## <a name="view-data-in-azure-table-storage"></a>在 Azure 表存储中查看数据
若要有查看状态数据，请打开“存储资源管理器”并使用 Azure 门户凭据连接到 Azure，或使用存储名称和存储密钥直接连接到表，然后导航到表名。  

## <a name="next-steps"></a>后续步骤
在本文中，实施了 Azure 表存储来保存和管理机器人的数据。 接下来，学习如何使用对话为聊天流建模。

> [!div class="nextstepaction"]
> [管理会话流](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>其他资源

如果不熟悉上面代码中使用的控制反转容器和依赖注入模式，请转到 [Autofac](http://autofac.readthedocs.io/en/latest/) 站点了解相关信息。 

也可从 GitHub 下载完整的 [Azure 表存储](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable)示例。
