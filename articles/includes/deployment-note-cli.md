---
ms.openlocfilehash: cc7e656d7c8a61e7bf784db579d0065c3bff331a
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035773"
---
如果使用 LUIS 等服务，则还需要传递 `luisAuthoringKey`。 若要使用 Azure 中的现有资源组，请在上述命令中使用 `groupName` 参数。

强烈建议使用 `verbose` 选项来帮助排查部署机器人期间可能发生的问题。 下面描述了可在 `msbot clone services` 命令中使用的其他选项：

| 参数    | 说明 |
|--------------|-------------|
| `folder`     | `bot.recipe` 文件的位置。 默认会在 `DeploymentsScript/MSBotClone` 中创建脚本文件。 切勿修改此文件。|
| `location`   | 用于创建机器人服务资源的地理位置。 例如 eastus、westus、westus2 等。|
| `proj-file`  | 对于 C# 机器人，它是 .csproj 文件。 对于 JS 机器人，它是本地机器人的启动项目文件名（例如 index.js）。|
| `name`       | 用于在 Azure 中部署机器人的唯一名称。 此名称可与本地机器人的名称相同。 切勿在此名称中包含空格或下划线。|
| `luisAuthoringKey` | LUIS 资源的相应 LUIS 创作区域的创作密钥。 |

在创建 Azure 资源之前，系统会提示你完成身份验证。 请遵照屏幕上的说明完成此步骤。

请注意，完成上述步骤需要花费几秒钟到几分钟的时间，Azure 中创建的资源会发生名称重整。 有关名称重整的详细信息，请参阅 GitHub 存储库中[问题 #796](https://github.com/Microsoft/botbuilder-tools/issues/796)。
