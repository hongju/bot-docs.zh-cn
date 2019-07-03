---
title: 使用 Azure 搜索创建数据驱动体验 | Microsoft Docs
description: 了解如何使用 Azure 搜索创建数据驱动体验，并帮助用户在机器人中使用 Bot Framework SDK for .NET 和 Azure 搜索导航大量内容。
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/28/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ee1e8c660eae27efae5c18b1392ff68d716f73da
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405644"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>使用 Azure 搜索创建数据驱动体验 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

你可向机器人添加 [Azure 搜索](https://azure.microsoft.com/services/search/)，帮助用户导航大量内容并创建数据驱动的探索体验。

Azure 搜索是一项 Azure 服务，提供关键字搜索、内置语言学、自定义评分、多面导航等。 Azure 搜索还可以为各种来源（包括 Azure SQL DB、DocumentDB、Blob 存储和表存储）的内容编制索引。 它支持其他数据源的“推送”索引，并且可以打开包含非结构化数据的 PDF、Office 文档和其他格式的文件。 内容一经收集即会进入 Azure 搜索索引，机器人随后可进行查询。

## <a name="prerequisites"></a>先决条件

在机器人项目中安装 [Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) Nuget 程序包。

机器人解决方案需要以下三个 C# 项目。 这些项目为机器人和 Azure 搜索提供附加功能。 从 [GitHub](https://aka.ms/v3-cs-search-demo) 创建项目分支或直接下载源代码。

- **Search.Azure** 项目定义 Azure 服务调用。
- **Search.Contracts** 项目定义用于处理数据的通用接口和数据模型。
- **Search.Dialogs** 项目包括用于查询 Azure 搜索的各种通用 Bot Builder 对话。

## <a name="configure-azure-search-settings"></a>配置 Azure 搜索设置

使用值字段中自己的 Azure 搜索凭据在项目的 **Web.config** 文件中配置 Azure 搜索设置。 `AzureSearchClient` 类中的构造函数将使用这些设置来注册并将机器人绑定到 Azure 服务。

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>创建搜索对话

在机器人的项目中，创建新的 `AzureSearchDialog` 类在机器人中调用 Azure 服务。 这个新类必须从 **Search.Dialogs** 项目继承 `SearchDialog` 类，其处理大部分繁重任务。 `GetTopRefiners()` 替代允许用户缩小搜索结果范围/筛选搜索结果，无需从头开始搜索，从而维护搜索对象的状态。 你可以在 `TopRefiners` 数组中添加自己的自定义精简程序，以便用户筛选搜索结果或缩小搜索结果范围。 

```cs
[Serializable]
public class AzureSearchDialog : SearchDialog
{
    private static readonly string[] TopRefiners = { "refiner1", "refiner2", "refiner3" }; // define your own custom refiners 

    public AzureSearchDialog(ISearchClient searchClient) : base(searchClient, multipleSelection: true)
    {
    }

    protected override string[] GetTopRefiners()
    {
        return TopRefiners;
    }
}
```

## <a name="define-the-response-data-model"></a>定义响应数据模型

`Search.Contracts` 项目中的 **SearchHit.cs** 类定义要从 Azure 搜索响应进行分析的相关数据。 对于你的机器人，唯一强制包含的内容是构造函数中的 `PropertyBag` IDictionary 声明和创建。 你可以根据机器人的需要定义此类中的所有其他属性。 

```cs
[Serializable]
public class SearchHit
{
    public SearchHit()
    {
        this.PropertyBag = new Dictionary<string, object>();
    }

    public IDictionary<string, object> PropertyBag { get; set; }

    // customize the fields below as needed 
    public string Key { get; set; }

    public string Title { get; set; }

    public string PictureUrl { get; set; }

    public string Description { get; set; }
}
```

## <a name="after-azure-search-responds"></a>Azure 搜索响应之后 

成功查询 Azure 服务后，需要分析搜索结果以检索机器人的相关数据，以便向用户显示。 若要启用此功能，需要创建 `SearchResultMapper` 类。 在构造函数中创建的 `GenericSearchResult` 对象定义一个列表和一个字典，可用于在针对相应机器人数据模型分析每个结果后分别存储结果和 Facet。 

同步 `ToSearchHit` 方法中的属性以匹配 **SearchHit.cs** 中的数据模型。 系统将执行 `ToSearchHit` 方法并为响应中找到的每个结果生成新的 `SearchHit`。  

```cs
public class SearchResultMapper : IMapper<DocumentSearchResult, GenericSearchResult>
{
    public GenericSearchResult Map(DocumentSearchResult documentSearchResult)
    {
        var searchResult = new GenericSearchResult();

        searchResult.Results = documentSearchResult.Results.Select(r => ToSearchHit(r)).ToList();
        searchResult.Facets = documentSearchResult.Facets?.ToDictionary(kv => kv.Key, kv => kv.Value.Select(f => ToFacet(f)));

        return searchResult;
    }

    private static GenericFacet ToFacet(FacetResult facetResult)
    {
        return new GenericFacet
        {
            Value = facetResult.Value,
            Count = facetResult.Count.Value
        };
    }

    private static SearchHit ToSearchHit(SearchResult hit)
    {
        return new SearchHit
        {
            // custom properties defined in SearchHit.cs 
            Key = (string)hit.Document["id"],
            Title = (string)hit.Document["title"],
            PictureUrl = (string)hit.Document["thumbnail"],
            Description = (string)hit.Document["description"]
        };
    }
}
```
分析和存储结果后，仍然需要向用户显示相关信息。 需要管理 `SearchHitStyler` 类以适应 `SearchHit` 类中的数据模型。 例如，示例代码使用 **SearchHit.cs** 类中的 `Title`、`PictureUrl` 和 `Description` 属性来创建新的卡附件。 以下代码为 `GenericSearchResult` 结果列表中的每个 `SearchHit` 对象创建一个新的卡附件，以便向用户显示。   

```cs
[Serializable]
public class SearchHitStyler : PromptStyler
{
    public override void Apply<T>(ref IMessageActivity message, string prompt, IReadOnlyList<T> options, IReadOnlyList<string> descriptions = null)
    {
        var hits = options as IList<SearchHit>;
        if (hits != null)
        {
            var cards = hits.Select(h => new ThumbnailCard
            {
                Title = h.Title,
                Images = new[] { new CardImage(h.PictureUrl) },
                Buttons = new[] { new CardAction(ActionTypes.ImBack, "Pick this one", value: h.Key) },
                Text = h.Description
            });

            message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
            message.Attachments = cards.Select(c => c.ToAttachment()).ToList();
            message.Text = prompt;
        }
        else
        {
            base.Apply<T>(ref message, prompt, options, descriptions);
        }
    }
}
```
搜索结果会向用户显示，并且你已成功向机器人添加 Azure 搜索。

## <a name="samples"></a>示例

有关展示如何通过 Bot Framework SDK for .NET 使用机器人支持 Azure 搜索的两个完整示例，请参阅 GitHub 中的[房地产机器人示例](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/RealEstateBot)或[作业清单机器人示例](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-Search/JobListingBot)。 

## <a name="additional-resources"></a>其他资源

- [Azure 搜索][search]
- [对话框概述](bot-builder-dotnet-dialogs.md)

[search]: /azure/search/search-what-is-azure-search
