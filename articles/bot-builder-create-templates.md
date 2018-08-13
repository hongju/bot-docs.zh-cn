---
title: 使用 Botbuilder 模板创建机器人
description: 可以使用 Botbuilder 工具直接从命令行管理机器人资源
keywords: botbuilder 模板, node.js, python, java, .net, ludown, yeoman
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
ms.openlocfilehash: 60cdc3de200336b00173749a553205a47a32457e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39297061"
---
# <a name="create-bots-with-botbuilder-templates"></a>使用 Botbuilder 模板创建机器人

现在可以使用模板在每个 Botbuilder SDK 平台中创建机器人： 

- Node.js
- Python
- Java
- .NET

## <a name="prerequisites"></a>先决条件

Node.js、Java 和 Python 模板均创建简单的回显机器人，且都是通过 [Yeoman](http://yeoman.io/) 提供的。

- [Node.js](https://nodejs.org/en/) v 8.5 或更高版本
- [Yeoman](http://yeoman.io/)

如果尚未安装 Yeoman，则需对其进行全局安装。

```shell
npm install -g yo
```

## <a name="nodejs-python-java-templates"></a>Node.js、Python、Java 模板
在命令行中通过 cd 命令转到所选新文件夹。 Botbuilder Node.js 模板有两个可用的版本，分别对应于 SDK 的 **v3** 和 **v4** 版本。 Python 和 Java 模板仅在 v4 中提供。 

若要安装 **Node.js v3 Botbuilder** 模板生成器，请执行以下命令：

```shell
npm install generator-botbuilder
```

若要安装 **Node.js v4 Botbuilder** 模板生成器，请执行以下命令：
```shell
npm install generator-botbuilder@preview
```

Python 和 Java 模板**仅在 v4 中提供**。 

对于 **Python**，请执行以下命令：
```shell
npm install generator-botbuilder-python
```

对于 **Java**，请执行以下命令：
```shell
npm install generator-botbuilder-java
```

安装生成器以后，可以直接在 CLI 中运行 **yo** 命令，以便查看在 Yeoman 中提供的生成器的列表。

![Yeoman 接口](media/botbuilder-templates/yeoman-generator-botbuilder.png)

切换到所选工作目录，然后选择要使用的生成器。 系统会提示你完成创建机器人所需的各种选项，例如名称和说明。 所有提示都完成以后，就会在同一工作文件夹中创建回显机器人模板。

![Node.js Yeoman 模板](media/botbuilder-templates/new-template-node.png)


## <a name="net"></a>.NET

.NET 有两个可用模板，分别对应于 SDK 的 **v3** 和 **v4** 版本。 二者均以 [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package) 包的形式提供。若要下载，请单击 [Visual Studio Marketplace](https://marketplace.visualstudio.com/) 上提供的下述链接之一。

- [BotBuilder V3 模板](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)
- [BotBuilder V4 模板](https://aka.ms/Ylcwxk)

#### <a name="prerequisites"></a>先决条件

- [Visual Studio 2015 或更高版本](https://www.visualstudio.com/downloads/)
- [Azure 帐户](https://azure.microsoft.com/en-us/free/)

### <a name="install-the-templates"></a>安装模板

从保存的目录直接打开 VSIX 包，机器人生成器模板就会安装到 Visual Studio，下次打开时就可以使用了。

![VSIX 包](media/botbuilder-templates/botbuilder-vsix-template.png)

若要使用模板创建新的机器人项目，请直接打开 Visual Studio 并选择“文件” > “新建” > “项目”，然后从 Visual C# 中选择“Bot Framework”>“简单的回显机器人应用程序”。 这样就会在本地创建新的回显机器人项目，可以根据需要对其进行编辑。 

![.NET 机器人模板](media/botbuilder-templates/new-template-dotnet.png)

## <a name="store-your-bot-information-with-msbot"></a>使用 MSBot 存储机器人信息

新 [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) 工具允许你创建 .bot 文件，该文件将机器人使用的不同服务的相关元数据存储在一个位置。 此文件还使机器人能够从 CLI 连接到这些服务。 该工具以 npm 模块形式提供。若要安装它，请运行：

```shell
npm install -g msbot 
```

若要创建机器人文件，请在 CLI 中输入“msbot init”后跟机器人的名称，并输入目标 URL 终结点，例如：

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
若要将机器人连接到服务，请在 CLI 中输入“msbot connect”后跟相应的服务：

```shell
msbot connect [Service]
```

| 服务 | Description |
| ------ | ----------- |
| azure  |将机器人连接到 Azure 机器人服务注册|
|localhost| 将机器人连接到 localhost 终结点|
|luis     | 将机器人连接到 LUIS 应用程序 |
| qna     |将机器人连接到 QnA 知识库|
|help [cmd]  |显示 [cmd] 帮助|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>使用 .bot 文件将机器人连接到 ABS

在安装了 MSBot 工具后，可以通过运行 az bot **show** 命令，轻松地将机器人连接到 Azure 机器人服务中的现有资源组。 

```azurecli
az bot show -n <botname> -g <resourcegroup> --msbot | msbot connect azure --stdin
```

这将使用目标资源组中的当前终结点、MSA appID 和密码，并在 .bot 文件中相应地更新信息。 


## <a name="manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>使用新 botbuilder-tools 管理、更新或创建 LUIS 和 QnA 服务

[Bot Builder 工具](https://github.com/microsoft/botbuilder-tools)是一个新的工具集，可用于直接从命令行管理机器人资源并与之进行交互。 

>[!TIP]
> 每个 Bot Builder 工具都包含全局帮助命令，可通过输入 -h 或 --help 从命令行进行访问。 此命令可随时在任何操作中使用，将显示对你可用的有用选项及其描述 

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) 允许你使用 **.lu** 文件为机器人描述和创建功能强大的语言组件。 新的 .lu 文件是 markdown 格式类型，LUDown 工具使用此类型并输出特定于目标服务的 .json 文件。 目前，可以使用 .lu 文件创建新的 [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) 应用程序或 [QnA](https://qnamaker.ai/Documentation/CreateKb) 知识库，对每个使用不同格式。 LUDown 可作为 npm 模块提供，以全局方式安装到计算机后即可使用：

```shell
npm install -g ludown
```
LUDown 工具可用于为 LUIS 和 QnA 创建新的 .json 模型。  


### <a name="creating-a-luis-application-with-ludown"></a>使用 LUDown 创建 LUIS 应用程序

可以为 LUIS 应用程序定义[意向](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents)和[实体](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities)，就像在 LUIS 门户中那样。 

'#\<intent-name\>' 介绍新的意向定义部分。 后续行列出用于描述该意向的[话语](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances)。

例如，可以在单个 .lu 文件中创建多个 LUIS 意向，如下所示： 

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="qna-pairs-with-ludown"></a>QnA 与 LUDown 配对

.lu 文件格式还支持使用以下表示法的 QnA 配对： 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

LUDown 工具会自动将问题和解答分离到 qnamaker JSON 文件，然后即可用其创建新的 [QnaMaker.ai](http://qnamaker.ai) 知识库。

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

也可将多个问题添加到相同答案，只需为单个答案添加新的问题变体行即可。 

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generating-json-models-with-ludown"></a>使用 LUDown 生成 .json 模型

在 .lu 格式中定义了 LUIS 或 QnA 语言组件后，即可将内容发布到 LUIS.json、QnA.json 或 QnA.tsv 文件。 运行时，LUDown 工具将在同一个工作目录中查找需分析的任何 .lu 文件。 由于 LUDown 工具可以使用 .lu 文件指向 LUIS 或 QnA，因此只需使用常规命令 **ludown parse <Service> -- in <luFile>** 指定要为哪个语言服务生成模块即可。 

在我们的示例工作目录中，我们有两个 .lu 文件可供分析，“1.lu”用于创建 LUIS 模型，“qna1.lu”用于创建 QnA 知识库。

#### <a name="generate-luis-json-models"></a>生成 LUIS.json 模型

若要使用 LUDown 生成 LUIS 模型，只需在当前工作目录中输入以下信息：

```shell
ludown parse ToLuis --in <luFile> 
```

#### <a name="generate-qna-knowledge-base-json"></a>生成 QnA 知识库 .json 文件

同样，若要创建 QnA 知识库，只需更改分析目标。 

```shell
ludown parse ToQna --in <luFile> 
```

生成的 JSON 文件可由 LUIS 和 QnA 通过各自的门户或新的 CLI 工具来使用。 

## <a name="connect-to-luis-an-qna-maker-services-from-the-cli"></a>从 CLI 连接到 LUIS（QnA Maker 服务）

### <a name="connect-to-luis-from-the-cli"></a>从 CLI 连接到 LUIS 

新工具集中包含的是 [LUIS 扩展](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS)，用于独立管理 LUIS 资源。 它以可下载的 npm 模块形式提供：

```shell
npm install -g luis-apis
```
CLI 中 LUIS 工具的基本命令用法是：

```shell
luis <action> <resource> <args...>
```
若要将机器人连接到 LUIS，则需创建 .luisrc 文件。 这是在应用程序执行出站调用时将 LUIS appID 和密码预配到服务终结点的配置文件。 可以通过运行 luis init 创建此文件，如下所示：

```shell
luis init
```
在此工具生成文件之前，系统会提示在终端中输入 LUIS 创作密钥、区域和 appID。  

![LUIS init](media/bot-builder-tools/luis-init.png) 


此文件生成后，应用程序就能够通过从 CLI 中运行以下命令来使用 LUIS.json 文件（从 LUDown 生成）。

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>从 CLI 连接到 QnA

新工具集中包含的是 [QnA 扩展](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker)，用于独立管理 LUIS 资源。 它以可下载的 npm 模块形式提供。

```shell
npm install -g qnamaker
```
使用 QnA Maker 工具，可以创建、更新、发布、删除和定型知识库。 若要开始，需要创建 .qnamakerrc 文件，以便启用面向服务的终结点。 可以运行 qnamaker init 并按提示操作来轻松创建此文件，然后预配 QnA Maker 知识库 ID。 

```shell
qnamaker init 
```
![QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

.qnamakerrc 文件生成后，即可通过以下命令连接到 QnA 知识库，以便使用知识库 .json/.tsv 文件（从 LUDown 生成）。

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="references"></a>参考
- [BotBuilder 工具源代码](https://github.com/Microsoft/botbuilder-tools)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
