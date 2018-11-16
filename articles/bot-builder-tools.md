---
title: 使用 CLI 工具管理机器人
description: 可以使用 Bot Builder 工具直接从命令行管理机器人资源
keywords: botbuilder 模板, ludown, qna, luis, msbot, 管理, cli, .bot, 机器人
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5ffaf9a946e1a540b82819b7f745200f47384819
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645657"
---
# <a name="manage-bots-using-cli-tools"></a>使用 CLI 工具管理机器人

Bot Builder 工具涵盖端到端机器人开发工作流，其中包括规划、生成、测试、发布、连接和评估阶段。 让我们看看如何在开发周期的每个阶段使用这些工具。

## <a name="plan"></a>计划

### <a name="create-mock-conversations-using-chatdown"></a>使用 Chatdown 创建模拟聊天

Chatdown 是一个脚本生成器，使用 .chat 文件来生成模拟脚本。 生成的模拟脚本文件输出到 stdout。

好的机器人就像任何成功的应用程序或网站一样，一开始必须明确哪些方案是受支持的方案。 创建在机器人和用户之间进行的模拟聊天适用于以下情况：

- 确定机器人支持的方案的范围。
- 方便业务决策者查看和提供反馈。
- 定义用户和机器人 .chat 文件格式之间的聊天流的“适宜途径”（以及其他途径）有助于你创建用户和机器人之间的模拟聊天。 Chatdown CLI 工具将 .chat 文件转换成聊天脚本（.transcript 文件），方便在 [Bot Framework Emulator V4](https://github.com/microsoft/botframework-emulator) 中查看。

下面是一个示例 `.chat` 文件：

```markdown
user=Joe
bot=LulaBot

bot: Hi!
user: yo!
bot: [Typing][Delay=3000]
Greetings!
What would you like to do?
* update - You can update your account
* List - You can list your data
* help - you can get help

user: I need the bot framework logo.

bot:
Here you go.
[Attachment=bot-framework.png]
[Attachment=http://yahoo.com/bot-framework.png]
[AttachmentLayout=carousel]

user: thanks
bot:
Here's a form for you
[Attachment=card.json adaptivecard]

```

### <a name="create-a-transcript-file-from-chat-file"></a>从 .chat 文件创建脚本文件
Chatdown 命令如下所示：

```bash
chatdown sample.chat > sample.transcript
```

它将使用 `sample.chat` 并输出 `sample.transcript`。 有关详细信息，请参阅 [Chatdown CLI][chatdown]。

## <a name="build"></a>构建
### <a name="create-a-luis-application-with-ludown"></a>使用 LUDown 创建 LUIS 应用程序
LUDown 工具可用于为 LUIS 和 QnA 创建新的 .json 模型。  
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

### <a name="create-qna-pairs-with-ludown"></a>使用 LUDown 创建 QnA 对

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

此外可以将多个问题添加到相同答案，只需为单个答案新添几行问题变体。

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generate-json-models-with-ludown"></a>使用 LUDown 生成 .json 模型

定义了 .lu 格式的 LUIS 或 QnA 语言组件后，可以将其发布到 LUIS.json、QnA.json 或 QnA.tsv 文件。 运行时，LUDown 工具将在同一个工作目录中查找需分析的任何 .lu 文件。 由于 LUDown 工具可以使用 .lu 文件指向 LUIS 或 QnA，因此只需使用常规命令 **ludown parse <Service> -- in <luFile>** 指定要为哪个语言服务生成模块即可。 

在我们的示例工作目录中，我们有两个 .lu 文件可供分析，“1.lu”用于创建 LUIS 模型，“qna1.lu”用于创建 QnA 知识库。

#### <a name="generate-luis-json-models"></a>生成 LUIS.json 模型

若要使用 LUDown 生成 LUIS 模型，只需在当前工作目录中输入以下信息：

```shell
ludown parse ToLuis --in <luFile>
```

#### <a name="generate-qna-knowledge-base"></a>生成 QnA 知识库

同样，若要创建 QnA 知识库，只需更改分析目标。

```shell
ludown parse ToQna --in <luFile> 
```

生成的 JSON 文件可由 LUIS 和 QnA 通过各自的门户或新的 CLI 工具来使用。 若要了解详细信息，请参阅 [LUdown CLI][ludown] GitHub 存储库。

### <a name="track-service-references-using-bot-file"></a>使用 .bot 文件跟踪服务引用

新 [MSBot][msbotCli] 工具用于创建 **.bot** 文件，该文件将机器人使用的不同服务的相关元数据存储在一个位置。 此文件还使机器人能够从 CLI 连接到这些服务。 该工具以 npm 模块形式提供。若要安装它，请运行：

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

若要获取支持的服务的列表，请参阅[自述][msbotCli]文件。

### <a name="create-and-manage-luis-applications-using-luis-cli"></a>使用 LUIS CLI 创建和管理 LUIS 应用程序

新工具集中包含的是 [LUIS 扩展][luisCli]，用于独立管理 LUIS 资源。 它以可下载的 npm 模块形式提供：

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

此文件生成后，应用程序就能够通过从 CLI 中运行以下命令来使用 LUIS.json 文件（从 LUDown 生成）。

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```
若要了解详细信息，请参阅 [LUIS CLI][luisCli] GitHub 存储库。

### <a name="create-qna-maker-kb-using-qna-maker-cli"></a>使用 QnA Maker CLI 创建 QnA Maker KB

新工具集中包含的是 [QnA 扩展][qnaCli]，用于独立管理 LUIS 资源。 它以可下载的 npm 模块形式提供。

```shell
npm install -g qnamaker
```
使用 QnA Maker 工具，可以创建、更新、发布、删除和定型知识库。 可以使用通过 [ludown parse toqna](#generate-qna-knowledge-base) 命令生成的文件来创建/替换知识库。

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

若要了解详细信息，请参阅 [QnA Maker CLI][qnaCli] GitHub 存储库。

### <a name="create-dispatch-model-using-dispatch-cli"></a>使用 Dispatch CLI 创建调度模型

Dispatch 是一项创建和评估 LUIS 模型的工具，使用此类模型调度的意向可以跨多个机器人模块，例如 LUIS 模型、QnA 知识库等（作为一个文件类型添加到 Dispatch）。

以下情况请使用 Dispatch 模型：

- 机器人包含多个模块，而你需要借助该模型才能将用户的话语路由到这些模块并评估机器人集成情况。
- 评估单个 LUIS 模型的意向分类的质量。
- 根据文本文件创建文本分类模型。

将 .bot 文件与机器人依赖的 [LUIS 应用程序][msbotCli-luis]和 [QnA Maker 知识库][msbotCli-qna]组合起来以后，即可直接使用以下命令生成调度模型： 

```shell
dispatch create -b <YOUR-BOT-FILE> | msbot connect dispatch --stdin
```
若要了解详细信息，请参阅 [Disptach CLI][dispatchCli]。

## <a name="test"></a>测试

Bot Framework [Emulator](bot-service-debug-emulator.md) 是一个桌面应用程序，允许机器人开发人员对 localhost 上的或者通过隧道远程运行的机器人进行测试和调试。

## <a name="publish"></a>发布

可以使用 Azure CLI 来创建、下载机器人以及将其发布到 Azure 机器人服务。 通过以下命令安装机器人扩展： 
```shell
az extension add -n botservice
```

### <a name="create-azure-bot-service-bot"></a>创建 Azure 机器人服务机器人

注意：必须使用最新版本的 `az cli`。 请升级此服务，使 az cli 可与 MSBot 工具配合工作。 

通过以下命令登录到 Azure 帐户： 
```shell
az login
```

登录以后，即可使用以下命令创建新的 Azure 机器人服务机器人： 
```shell
az bot create [options]
```

若要使用机器人配置来创建机器人并更新 .bot 文件，请使用以下命令：  
```shell
az bot create [options] --msbot | msbot connect bot --stdin
```

如果有现有的机器人，请使用以下命令：  
```shell
az bot show [options] --msbot | msbot connect bot --stdin
```

| 选项                            | Description                                   |
|-----------------------------------|-----------------------------------------------|
| --kind -k [必需]              | 机器人的类型。  允许的值：function、registration、webapp。|
| --name -n [必需]              | 机器人的资源名称。 |
| --appid                           | 可以与机器人配合使用的 msa 帐户 ID。   |
| --location -l                     | 位置。 可以使用 `az configure --defaults location=<location>` 配置默认位置。  默认值：westus。|
| --msbot                           | 将输出显示为与 .bot 文件兼容的 json。  允许的值：false、true。|
| --password -p                     | 开发人员门户中的机器人的 msa 密码。 |
| --resource-group -g               | 资源组的名称。 可以使用 `az configure --defaults group=<name>` 配置默认组。  默认值：build2018。 |
| --tags                            | 要添加到机器人的标记集。 |


### <a name="configure-channels"></a>配置通道

可以使用 Azure CLI 管理机器人的通道。 
```shell
>az bot -h
Group
   az bot: Manage Bot Services.
    Subgroups:
        directline: Manage Directline Channel on a Bot.
        email     : Manage Email Channel on a Bot.
        facebook  : Manage Facebook Channel on a Bot.
        kik       : Manage Kik Channel on a Bot.
        msteams   : Manage Msteams Channel on a Bot.
        skype     : Manage Skype Channel on a Bot.
        slack     : Manage Slack Channel on a Bot.
        sms       : Manage Sms Channel on a Bot.
        telegram  : Manage Telegram Channel on a Bot.
        webchat   : Manage Webchat Channel on a Bot.

    Commands:
        create    : Create a new Bot Service.
        delete    : Delete an existing Bot Service.
        download  : Download an existing Bot Service.
        publish   : Publish to an existing Bot Service.
        show      : Get an existing Bot Service.
        update    : Update an existing Bot Service.

```

## <a name="additional-information"></a>其他信息
- [GitHub 上的 Bot Builder 工具][cliTools]

<!-- Footnote links -->

[cliTools]: https://aka.ms/botbuilder-tools-readme
[azureCli]: https://aka.ms/botbuilder-tools-azureCli
[msbotCli]: https://aka.ms/botbuilder-tools-msbot-readme
[msbotCli-luis]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-luis-application
[msbotCli-qna]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-qna-maker-knowledge-base
[chatdown]: https://aka.ms/botbuilder-tools-chatdown
[ludown]: https://aka.ms/botbuilder-ludown
[luisCli]: https://aka.ms/botbuilder-luis-cli
[qnaCli]: https://aka.ms/botbuilder-tools-qnaMaker
[dispatchCli]: https://aka.ms/botbuilder-tools-dispatch
