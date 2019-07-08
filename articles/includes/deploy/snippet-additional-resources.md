---
ms.openlocfilehash: 2c06c67099f44fe1df2eb0099a514a697ef0d1c9
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405595"
---
部署机器人时，通常会在 Azure 门户中创建以下资源：

| 资源      | 说明 |
|----------------|-------------|
| Web 应用机器人 | 部署到 Azure 应用服务的 Azure 机器人服务机器人。|
| [应用服务](https://docs.microsoft.com/azure/app-service/)| 用于生成和托管 Web 应用程序。|
| [应用服务计划](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| 为要运行的 Web 应用定义一组计算资源。|

如果通过 Microsoft Azure 门户创建机器人，则可以预配额外的资源（如[用于遥测的 Application Insights](~/v4sdk/bot-builder-telemetry.md)）。

若要查看有关 `az bot` 命令的文档，请参阅[参考](https://docs.microsoft.com/cli/azure/bot?view=azure-cli-latest)主题。

如果你不熟悉 Azure 资源组，请参阅此[术语](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#terminology)主题。