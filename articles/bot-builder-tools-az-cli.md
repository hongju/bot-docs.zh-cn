---
title: 使用 Azure CLI 创建机器人
description: 可以使用 Botbuilder 工具直接从命令行管理机器人资源
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/31/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 96660ecb8bf7a69115e517bfa8ec97a79a3e8c90
ms.sourcegitcommit: f0b22c6286e44578c11c9f15d22b542c199f0024
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404013"
---
# <a name="create-bots-with-azure-cli"></a>使用 Azure CLI 创建机器人

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

本教程介绍以下操作： 
- 使用 Azure CLI 创建新机器人 
- 下载用于开发的本地副本
- 使用新 MSBot 工具来存储所有机器人资源信息
- 使用 LUDown 管理、创建或更新 LUIS 和 QnA 模型
- 从 CLI 连接到 LUIS（QnA maker 服务）
- 将机器人从 CLI 部署到 Azure

## <a name="prerequisites"></a>先决条件

若要通过命令行使用这些工具，需要在计算机上安装 Node.js： 
- [Node.js（v8.5 或更高版本）](https://nodejs.org/en/)

## <a name="1-install-tools"></a>1.安装工具
1. [安装](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)最新版 Azure CLI。
2. [安装](https://aka.ms/botbuilder-tools-readme) Bot Builder 工具。

现在可以像使用任何其他 Azure 资源一样使用 Azure CLI 来管理机器人。

>[!TIP]
> Azure 机器人扩展目前只支持 v3 机器人。
  
3. 通过运行以下命令登录到 Azure CLI。

```azurecli
az login
```
此时会打开一个用于登录的浏览器窗口。 登录后，会看到以下消息：

![MS 设备登录](media/bot-builder-tools/az-browser-login.png)

在命令行窗口中，会看到以下信息：

![Azure 登录命令](media/bot-builder-tools/az-login-command.png)

## <a name="2-create-a-new-bot-from-azure-cli"></a>2.从 Azure CLI 创建新机器人

使用 Azure CLI 完全从命令行创建新机器人。 

```azurecli
az bot [command]
```
|命令|  |
|----|----|
| create      |添加资源|
| delete     |克隆资源|
| 下载   | 下载机器人源代码|
| 发布   |发布到现有机器人服务|
| show |显示现有机器人资源。|
| update| 更新现有机器人服务|

若要从 CLI 创建新机器人，需要选择一个现有[资源组](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)，或创建一个新的资源组。 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --version v3 --description "description-of-my-bot"
```
`--kind` 的允许值为 `function, registration, webapp`，`--version` 的允许值为 `v3, v4`。  请求成功后，将看到确认消息。
```
Obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> 如果收到错误消息，指示“找不到资源组”，可能需要在 Azure CLI 中设置[订阅](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer)。 Azure 订阅必须与创建资源组时输入的订阅匹配。 若要设置，请输入以下命令。
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> 若要查看帐户订阅列表，请输入以下命令。
> ```azurecli
> az account list
> ```

默认情况下，将创建新的 .NET 机器人。 可以通过使用 -- lang 参数指定语言来指定平台 SDK。 目前，机器人扩展包支持 C# 和 Node.js 机器人 SDK。 例如，若要创建 Node.js 机器人：

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
新的回显机器人将自动预配到 Azure 的资源组中，若要对其进行测试，只需选择 Web 应用机器人视图的机器人管理标头下的“在网上聊天中测试”。 

![Azure 回显机器人](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3.在本地下载机器人

有两种方法可以下载源代码：
- 通过 Azure 门户。
- 通过新的 Azure CLI。

若要从 [Azure 门户](http://portal.azure.com)下载机器人源代码，请直接选择机器人资源，然后选择机器人管理下的“生成”。 有几个不同的选项可用于在本地管理或检索机器人源代码。

![Azure 门户机器人下载](media/bot-builder-tools/az-portal-manage-code.png)

若要使用 CLI 下载机器人源代码，请输入以下命令。 机器人会下载到本地。

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```

![CLI 下载命令](media/bot-builder-tools/cli-bot-download-command.png)

## <a name="4-store-your-bot-information-with-msbot"></a>4.使用 MSBot 存储机器人信息

新 MSBot 工具用于创建 **.bot** 文件，该文件将机器人使用的不同服务的相关元数据存储在一个位置。 此文件还使机器人能够从 CLI 连接到这些服务。 MSBot 工具支持多个命令。有关详细信息，请参阅[自述](https://aka.ms/botbuilder-tools-msbot-readme)文件。 

若要安装 MSBot，请运行：

```shell
npm install -g msbot
```

若要创建机器人文件，请在 CLI 中输入“msbot init”后跟机器人的名称，并输入目标 URL 终结点，例如：

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```

若要将机器人连接到服务，请在 CLI 中输入“msbot connect”后跟相应的服务：

```shell
msbot connect service-type
```

| 服务类型 | Description |
| ------ | ----------- |
| azure  |将机器人连接到 Azure 机器人服务注册|
|endpoint| 将机器人连接到终结点，例如 localhost|
|luis     | 将机器人连接到 LUIS 应用程序 |
| qna     |将机器人连接到 QnA 知识库|
|help [cmd]  |显示 [cmd] 帮助|

如需支持的服务的完整列表，请参阅[自述](https://aka.ms/botbuilder-tools-msbot-readme)文件。

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>使用 .bot 文件将机器人连接到 ABS

在安装了 MSBot 工具后，可以通过运行 az 机器人 show 命令，轻松将机器人连接到 Azure 机器人服务中的现有资源组。

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

这将使用目标资源组中的当前终结点、MSA appID 和密码，并在 .bot 文件中相应地更新信息。


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5.使用新 botbuilder-tools 管理、更新或创建 LUIS 和 QnA 服务

[Bot Builder 工具](https://aka.ms/botbuilder-tools)是一个新的工具集，可用于直接从命令行管理机器人资源并与之进行交互。

>[!TIP]
> 每个 Bot Builder 工具都包含全局帮助命令，可通过输入 -h 或 --help 从命令行进行访问。 此命令可随时在任何操作中使用，将显示对你可用的有用选项及其描述。

### <a name="ludown"></a>LUDown

[LUDown](https://aka.ms/botbuilder-ludown) 允许你使用 **.lu** 文件为机器人描述和创建功能强大的语言组件。 新的 .lu 文件是 markdown 格式类型，LUDown 工具使用此类型并输出特定于目标服务的 .json 文件。 目前，可以使用 .lu 文件创建新的 [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) 应用程序或 [QnA](https://qnamaker.ai/Documentation/CreateKb) 知识库，对每个使用不同格式。 LUDown 可作为 npm 模块提供，以全局方式安装到计算机后即可使用：

```shell
npm install -g ludown
```

LUDown 工具可用于为 LUIS 和 QnA 创建新的 .json 模型。  

### <a name="creating-a-luis-application-with-ludown"></a>使用 LUDown 创建 LUIS 应用程序

可以为 LUIS 应用程序定义[意向](https://docs.microsoft.com/azure/cognitive-services/luis/add-intents)和[实体](https://docs.microsoft.com/azure/cognitive-services/luis/add-entities)，就像在 LUIS 门户中那样。

`# \<intent-name\>` 介绍新的意向定义部分。 后续行包含描述该意向的[表达](https://docs.microsoft.com/azure/cognitive-services/luis/add-example-utterances)。

例如，可以在单个 .lu 文件中创建多个 LUIS 意向，如下所示： 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>QnA 与 LUDown 配对

.lu 文件格式还支持使用以下表示法的 QnA 配对： 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
LUDown 工具会自动将问题和解答分离到 qnamaker JSON 文件，然后即可用其创建新的 [QnaMaker.ai](http://qnamaker.ai) 知识库。

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

此外可以将多个问题添加到相同答案，只需为单个答案新添几行问题变体。 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>使用 LUDown 生成 .json 模型

定义了 .lu 格式的 LUIS 或 QnA 语言组件后，可以将其发布到 LUIS.json、QnA.json 或 QnA.tsv 文件。 运行时，LUDown 工具将在同一个工作目录中查找任何 .lu 文件以便分析。 由于 LUDown 工具可以使用 .lu 文件指向 LUIS 或 QnA，我们只需使用常规命令 ludown parse <Service> --in <luFile> 指定生成的目标语言服务。 

在我们的示例工作目录中，我们有两个 .lu 文件可供分析，“luis-sample.lu”用于创建 LUIS 模型，“qna-sample.lu”用于创建 QnA 知识库。


#### <a name="generate-luis-json-models"></a>生成 LUIS.json 模型

**luis-sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

若要使用 LUDown 生成 LUIS 模型，只需在当前工作目录中输入以下信息：

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>生成 QnA 知识库 .json 文件

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

同样，若要创建 QnA 知识库，只需更改分析目标。 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

生成的 JSON 文件可由 LUIS 和 QnA 通过各自的门户或新的 CLI 工具来使用。 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6.从 CLI 连接到 LUIS（QnA Maker 服务）

### <a name="connect-to-luis-from-the-cli"></a>从 CLI 连接到 LUIS 

新工具集中包含的是 [LUIS 扩展](https://aka.ms/botbuilder-luis-cli)，用于独立管理 LUIS 资源。 它以可下载的 npm 模块形式提供：

```shell
npm install -g luis-apis
```
CLI 中 LUIS 工具的基本命令用法是：

```shell
luis action-name resource-name arguments-list
```
若要将机器人连接到 LUIS，则需创建 .luisrc 文件。 这是在应用程序执行出站调用时将 LUIS appID 和密码预配到服务终结点的配置文件。 可以通过运行 luis init 创建此文件，如下所示：

```shell
luis init
```
在此工具生成文件之前，系统会提示在终端中输入 LUIS 创作密钥、区域和 appID。  

![LUIS init](media/bot-builder-tools/luis-init.png) 


此文件生成后，应用程序将能够通过从 CLI 中运行以下命令使用 LUIS.json 文件（从 LUDown 生成）： 

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>从 CLI 连接到 QnA

新工具集中包含的是 [QnA 扩展](https://aka.ms/botbuilder-tools-qnaMaker)，允许独立管理 LUIS 资源。 它可以作为 npm 模块提供，可供下载：

```shell
npm install -g qnamaker
```
使用 QnA maker 工具，可以创建、更新、发布、删除和定型知识库。 若要开始，需要创建 .qnamakerrc 文件，以便启用面向服务的终结点。 可以通过运行 qnamaker init 并按照提示以及 QnA Maker 知识库 ID 轻松创建此文件。 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

.qnamakerrc 文件生成后，现在可以通过运行以下命令连接到 QnA 知识库来使用知识库 .json/.tsv 文件（从 LUDown 生成）：

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="7-publish-to-azure-from-the-cli"></a>7.从 CLI 发布到 Azure

对机器人源代码进行更改后，可以通过运行以下命令无缝发布所做的更改：

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>参考
- [Bot Builder 工具](https://aka.ms/botbuilder-tools-readme)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
