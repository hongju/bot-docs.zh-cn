---
title: 将 C# 机器人从消耗计划迁移到应用服务计划 | Microsoft Docs
description: 将机器人服务 C# 机器人从消耗托管计划迁移到应用服务托管计划。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/13/2017
ms.openlocfilehash: aafbfb2a38e2d5370cb2db5721dd7bc130497d74
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999214"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>更改机器人服务的托管计划

本主题说明如何将采用消耗计划的 C# 脚本机器人迁移到采用应用服务计划的 C# 机器人中。 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>机器人采用应用服务计划所带来的优势

应用服务计划上的机器人作为 Azure Web 应用运行。 Web 应用机器人可完成消耗计划机器人无法完成的操作：

- Web 应用机器人可添加自定义路由定义。
- Web 应用机器人可启用 Websocket 服务器。 
- 使用消耗托管计划的机器人与 Azure Functions 上运行的所有代码具有相同的限制。 有关详细信息，请参阅 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 消耗计划和应用服务计划</a>。

## <a name="download-your-existing-bot-source"></a>下载现有的机器人源

请按照下列步骤下载现有机器人的源代码：

1. 在 Azure 机器人中，单击“设置”选项卡并展开“持续部署”部分。  
2. 单击蓝色按钮以下载包含机器人源代码的 zip 文件。  
    ![下载机器人 zip 文件](~/media/continuous-deployment-consumption-download.png)
3. 将下载的 zip 文件的内容提取到本地文件夹。 


## <a name="create-a-bot-template"></a>创建机器人模板

机器人服务向消耗计划和应用服务计划机器人提供相同的模板。 要从消耗计划机器人迁移，请基于同一模板在机器人服务中创建新的应用服务计划。 同一模板的两种托管类型之间的基础代码可能不同，但是新的 Web 应用的结构和配置功能与现有机器人所使用的类似。

## <a name="download-the-new-bot-source"></a>下载新的机器人源

请按照下列步骤下载新机器人的源代码：

1. 在 Azure 机器人中，单击“BUILD”选项卡，找到“下载源代码”部分，然后单击“下载 zip 文件”。 
2. 将下载的 zip 文件的内容提取到本地文件夹。

## <a name="add-source-files-to-new-solution"></a>将源文件添加到新的解决方案

某些 .csx 文件可能会在新的解决方案中编译并作为 .cs 文件运行。 在解决方案中为每个 .csx 文件创建一个 .cs 文件（`run.csx` 除外）。 要手动迁移 `run.csx` 逻辑。 在 .cs 文件中，可能需要添加类声明和可选的命名空间声明。

## <a name="migrate-runcsx-logic-into-your-project"></a>将 run.csx 逻辑迁移到项目中

C# 脚本项目具有 `Run` 方法，其中处理不同的 `ActivityTypes` 值。 在 `MessageController.cs` 中，将活动处理逻辑导入到 `MessageController.Post` 方法。

## <a name="remove-compiler-keywords"></a>删除编译器关键字

C# 脚本文件可包含使用 `#r` 关键字的引用模块。 删除这些行，并将它们作为引用添加到 Visual Studio 项目。 同时删除 `#load` 关键字，它们在文件编译中插入其他源代码文件。 此外，将所有 `.csx` 文件作为 `.cs` 源代码添加到项目中。

## <a name="add-references-from-projectjson"></a>从 project.json 添加引用

如果消耗计划机器人在其 `project.json` 文件中添加 NuGet 引用，则将这些引用添加到新的 Visual Studio 解决方案，方法是右键单击“解决方案资源管理器”窗格中的项目，然后单击“添加引用”。

### <a name="add-references-that-were-implicit"></a>添加隐式引用

消耗计划上的机器人服务机器人将这些引用隐式包含在所有 .csx 源文件中。 迁移到 .cs 源文件的源可能需要添加以下类的显式引用：

- 提供 `Microsoft.Azure.WebJobs.Host``TraceWriter` 命名空间类型的 NuGet 包中的 `Microsoft.Azure.WebJobs`。 
- NuGet 包 `Microsoft.Azure.WebJobs.Extensions` 中的计时器触发器
- `Newtonsoft.Json`、`Microsoft.ServiceBus` 和其他自动引用的程序集
- `System.Threading.Tasks` 和其他自动导入的命名空间

有关其他指南，请参阅<a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>发布 .NET 类库作为 Function App</a> 中的“转换为类文件”。

## <a name="debug-your-new-bot"></a>调试新机器人

与采用消耗计划的机器人相比，可在本地更轻松地调试采用应用服务计划的机器人。 可使用[模拟器](bot-service-debug-emulator.md)调试本地迁移的代码。

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>通过 Visual Studio 发布或设置持续部署

最后，将迁移的源代码发布到机器人服务，方法是导入其 `.PublishSettings` 文件并单击“发布”，或者[设置持续部署](bot-service-debug-bot.md)。
