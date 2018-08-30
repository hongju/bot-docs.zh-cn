---
title: 使用 Azure 搜索创建数据驱动体验 | Microsoft Docs
description: 了解如何使用 Azure 搜索创建数据驱动体验，并帮助用户在机器人中使用 Bot Builder SDK for Node.js 和 Azure 搜索导航大量内容。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e9f07cdd4616a2649dca31f096eca3377cd46b7d
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904231"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>使用 Azure 搜索创建数据驱动体验 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

可以将 [Azure 搜索][search]添加到机器人，帮助用户导航大量内容并为机器人用户创建数据驱动的探索体验。

Azure 搜索是一项 Azure 服务，提供关键字搜索、内置语言学、自定义评分、分面导航等。 Azure 搜索还可以为各种源（包括 Azure SQL DB、DocumentDB、Blob 存储和表存储）的内容编制索引。 它支持其他数据源的“推送”索引，并且可以打开包含非结构化数据的 PDF、Office 文档和其他格式的文件。 收集后，内容会进入 Azure 搜索索引，然后机器人可以进行查询。

## <a name="install-dependencies"></a>安装依赖项

从命令提示符导航到机器人的项目目录，并使用节点包管理器 (NPM) 安装以下模块：

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [请求](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>先决条件

以下条件是必需的： 
- 具有 Azure 订阅和 Azure 搜索主键。 可以在 Azure 门户中找到该值。
- 将 [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) 库复制到机器人的项目目录。 此库包含供用户搜索的常规对话框，但是可以根据需要进行自定义以适合你的机器人。 

- 将 [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) 库复制到机器人的项目目录。 此库包含创建请求并将其提交到 Azure 搜索所需的所有组件。

## <a name="connect-to-the-azure-service"></a>连接到 Azure 服务 

在机器人的主程序文件（例如：app.js）中，创建之前安装的两个库的引用路径。 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

将以下示例代码添加到机器人。 在 `AzureSearch` 对象中，将自己的 Azure 搜索设置传递给 `.create` 方法。 在运行时，这会将机器人绑定到 Azure 搜索服务并等待完成的 `Promise` 形式的用户查询。  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 `azureSearchClient` 引用创建 Azure 搜索模型，此模型从机器人的 `.env` 设置传递 Azure 服务的授权设置。 
 `ResultsMapper` 分析 Azure 响应对象并映射我们在 `ToSearchHit` 方法中定义的数据。 有关此方法的实现示例，请参阅 [Azure 搜索响应之后](#after-azure-search-responds)。

## <a name="register-the-search-library"></a>注册搜索库
可以直接在 `SearchLibrary` 模块本身中自定义搜索对话框。 `SearchLibrary` 执行大部分繁重工作，包括调用 Azure 搜索。 

在机器人的主程序文件中添加以下代码，以在机器人中注册“搜索对话框”库。 

```javascript
bot.library(SearchLibrary.create({
    multipleSelection: true,
    search: function (query) { return azureSearchClient.search(query).then(ResultsMapper); },
    refiners: ['refiner1', 'refiner2', 'refiner3'], // customize your own refiners 
    refineFormatter: function (refiners) {
        return _.zipObject(
            refiners.map(function (r) { return 'By ' + _.capitalize(r); }),
            refiners);
    }
}));
```
`SearchLibrary` 不仅存储所有与搜索相关的对话框，还将用户查询提交到 Azure 搜索。 你将需要在 `refiners` 数组中定义自己的精简条件，以指定想让用户缩小或过滤其搜索结果的实体。  

## <a name="create-a-search-dialog"></a>创建搜索对话框

可以选择根据需要组织对话框。 设置 Azure 搜索对话框的唯一要求是从 `SearchLibrary` 对象调用 `.begin` 方法，同时传入 Bot Builder SDK 生成的 `session` 对象。 

```javascript
function (session) {
        // Trigger Azure Search dialogs 
        SearchLibrary.begin(session);
    },
    function (session, args) {
        // Process selected search results
        session.send(
            'Search Completed!',
            args.selection.map(  ); // format your response 
    }
```
有关对话框的详细信息，请参阅[使用对话框管理会话](bot-builder-nodejs-dialog-manage-conversation.md)。

## <a name="after-azure-search-responds"></a>Azure 搜索响应之后 

Azure 搜索解析成功后，现在需要从响应对象中存储所需的数据，并以有意义的方式将其显示给用户。

> [!TIP]
> 请考虑包括 [util 模块][NodeUtil]。 它将帮助格式化和映射 Azure 搜索的响应。

在机器人的主程序文件中，创建 `ToSearchHit` 方法。 此方法返回一个对象，该对象从 Azure 响应中格式化需要的相关数据。 以下代码显示了如何在 `ToSearchHit` 方法中定义自己的参数。 
 
 ```javascript
 function ToSearchHit(azureResponse) {
     return {
         // define your own parameters 
         key: azureResponse.id,
         title: azureResponse.title,
         description: azureResponse.description,
         imageUrl: azureResponse.thumbnail
     };
 }
```
此操作完成后，接下来需要向用户显示数据。 

 在 SearchDialogLibrary 项目的 index.js 文件中，`searchHitAsCard` 方法分析 Azure 搜索的每个响应，并创建一个新的卡片对象以显示给用户。 在机器人的主程序文件中的 `ToSearchHit` 方法中定义的字段需要与 `searchHitAsCard` 方法中的属性同步。 

下面显示了来自 `ToSearchHit` 方法的定义参数用于构建呈现给用户的卡片附件 UI 的方法和位置。 

```javascript
function searchHitAsCard(showSave, searchHit) {
    var buttons = showSave
        ? [new builder.CardAction().type('imBack').title('Save').value(searchHit.key)]
        : [];

    var card = new builder.HeroCard()
        .title(searchHit.title) 
        .buttons(buttons);

    if (searchHit.description) {
        card.subtitle(searchHit.description);
    }

    if (searchHit.imageUrl) {
        card.images([new builder.CardImage().url(searchHit.imageUrl)]);
    }

    return card;
}
```

## <a name="sample-code"></a>代码示例

有关显示如何通过 Bot Builder SDK for Node.js 使用机器人支持 Azure 搜索的两个完整示例，请参阅 GitHub 中的[房地产机器人示例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/RealEstateBot)或[作业清单机器人示例](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-Search/JobListingBot)。 

## <a name="additional-resources"></a>其他资源

* [Azure 搜索][search]
* [Node Util][NodeUtil]
* [对话框](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search
