---
ms.openlocfilehash: 8eb125b2d0f0c9981cafb763ba809c7d042d8f77
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225982"
---
机器人服务为机器人提供两种不同的托管计划。 将机器人源代码从一个计划转换为另一个计划是一个手动过程。   

## <a name="app-service-plan"></a>应用服务计划

使用应用服务计划的机器人是一个标准的 Azure Web 应用，你可以将其设置为以可预测成本和缩放来分配预定义的容量。 借助使用此托管计划的机器人，你可以：

* 使用高级浏览器内代码编辑器联机编辑机器人源代码。
* 使用 Visual Studio 下载、调试和重新发布 C# 机器人。
* 为 Visual Studio Online 和 Github 轻松设置持续部署。
* 使用为 Bot Framework SDK 准备的示例代码。

## <a name="consumption-plan"></a>消耗量计划

使用消耗计划的机器人是在 <a href="http://go.microsoft.com/fwlink/?linkID=747839" target="_blank">Azure Functions</a> 上运行的无服务器机器人，并使用按运行付费的 Azure Functions 定价。 使用此托管计划的机器人可以扩展以处理巨大的流量峰值。 可以使用基本的浏览器内代码编辑器联机编辑机器人源代码。 有关消耗计划机器人的运行时环境的更多信息，请参阅 <a target='_blank' href='/azure/azure-functions/functions-scale'>Azure Functions 消耗计划和应用服务计划</a>。
