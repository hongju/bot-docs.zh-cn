---
title: 为机器人实现自定义存储 | Microsoft Docs
description: 如何在 Bot Framework SDK v4.0 中构建自定义存储
keywords: 自定义, 存储, 状态, 对话
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4503e2953543d2ec9c06e8cd60484a5c87d95987
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224002"
---
# <a name="implement-custom-storage-for-your-bot"></a>为机器人实现自定义存储

机器人的交互划分为三个方面：首先，与 Azure 机器人服务交换活动；其次，使用存储加载和保存对话状态；最后，机器人需要使用其他任何后端服务来完成其作业。

![横向扩展示意图](../media/scale-out/scale-out-interaction.png)

在本文中，我们将围绕机器人与 Azure 机器人服务和存储的交互来探索语义。

Bot Framework 包含默认的实现，此实现基本上能够满足许多应用程序的需求。若要利用此实现，只需通过少量的初始化代码行将各个片段连接到一起。 有许多示例演示了此操作。

不过，本文的目标是介绍当默认实现的语义不是十分适合应用程序时可以采取哪些措施。 基本要点在于，这只是一个框架，而不是具有固定行为的打包应用程序，换言之，该框架中许多机制的实现只是默认实现，而不是唯一的实现。

具体而言，该框架不会规定与 Azure 机器人服务的活动交换，以及加载和保存任何机器人状态这两者之间的关系，而只是提供默认实现。 为了进一步演示这一点，我们将开发一个具有不同语义的备用实现。 此备用解决方案同样合理地定位在框架中，甚至可能更适合开发中的应用程序， 这完全取决于具体的方案。

## <a name="behavior-of-the-default-botframeworkadapter-and-storage-providers"></a>默认 BotFrameworkAdapter 和存储提供程序的行为

首先，让我们了解框架包随附的默认实现，如以下序列图所示：

![横向扩展示意图](../media/scale-out/scale-out-default.png)

接收活动时，机器人加载对应于此聊天的状态。 然后，机器人使用此状态和刚刚抵达的活动运行对话逻辑。 在执行对话的过程中，将会创建并立即发送一个或多个出站活动。 对话处理完成后，机器人将保存更新的状态，并使用新状态覆盖旧状态。

应该考虑到可能会导致此行为出错的几个因素。

首先，如果 Save 操作出于某种原因失败，则状态会与通道中显示的状态失去同步，因为在查看响应的用户的印象中，状态已发展到下一步，但事实并非如此。 这种情形通常比状态为成功并且响应消息传送成功更糟糕。 这可能会对聊天设计造成影响：例如，对话可能包含与用户之间进行的附加或冗余确认交换。 

其次，如果以横向扩展的方式将该实现部署在多个节点之间，则可能会意外覆盖状态 - 这一点可能特别令人困惑，因为对话可能会将携带确认消息的活动发送到通道。 以披萨订购机器人为例，如果顾客要求提供佐料，而用户添加了蘑菇，随即又添加了奶酪，则在运行多个实例的横向扩展方案中，后续的活动可能会同时发送到运行机器人的不同计算机。 这种情况称为“竞争状态”，其中一台计算机可能会覆盖另一台计算机写入的状态。 但是，在我们的方案中，由于响应已发送，用户已收到添加了蘑菇和奶酪的确认。 遗憾的是，送达的披萨仅包含蘑菇或奶酪，而不是包含两者。

## <a name="optimistic-locking"></a>乐观锁定

解决方案是围绕状态引入某种锁定。 此处我们要使用的特定样式的锁定称为“乐观锁定”，因为我们要让运行的每个活动如同唯一要运行的活动，然后在完成处理后检测任何并发性冲突。 这听起来比较复杂，但使用云存储技术和机器人框架中适当的扩展点可以十分轻松地构建。

我们将使用基于实体标记标头 (ETag) 的标准 HTTP 机制。 了解此机制对于理解以下代码至关重要。 下图演示了序列。

![横向扩展示意图](../media/scale-out/scale-out-precondition-failed.png)

该示意图演示了对某个资源执行更新的两个客户端。 当客户端发出 GET 请求并从服务器返回资源时，该资源会附带一个 ETag 标头。 ETag 标头是代表资源状态的不透明值。 如果资源已更改，则 ETag 将会更新。 当客户端完成状态更新时，会通过 POST 将状态发回到服务器。发出此请求时，客户端会附加它先前在 If-Match 前提条件标头中收到的的 ETag 值。 如果此 ETag 与值不匹配，则上次（在任何响应中向任何客户端）返回前提条件检查的服务器将会失败并显示“412 不满足前提条件”。 这种失败向发出 POST 请求的客户端指示资源已更新。 如果看到这种失败，客户端的典型行为是通过 GET 再次获取资源、应用所需的更新，然后通过 POST 发回资源。 第二次 POST 将会成功，当然，前提是没有其他客户端更新资源；如果存在这种情况，则客户端只需重试即可。

此过程之所以称为“乐观锁定”，是因为获得了资源的客户端可继续执行处理，而资源本身并未“锁定”，因为其他客户端可以不受限制地访问它。 不管资源应处于哪种状态，客户端之间的任何争用只有在完成处理之后才能确定。 根据经验法则，在分布式系统中，此策略比对立的“悲观”方法更佳。

上面所述的乐观锁定机制假设程序逻辑可以安全重试，毋庸赘言，此处要考虑的重要问题是外部服务调用会发生什么情况。 理想的解决方案是这些服务可设为幂等。 在计算机科学中，幂等操作是指使用相同输入参数调用多次不会造成附加影响的操作。 实现 GET、PUT 和 DELETE 的单纯 HTTP REST 服务符合这种描述。 此处的推理非常直观：我们可以重试处理，因此，使任何所需的调用不会造成附加影响（因为是在重试过程中重新执行的）比较有利。 为方便讨论，我们假设我们身处一个理想的环境，本文开头提供的系统示意图右侧显示的后端服务都是幂等的 HTTP REST 服务。从下面开始，我们只重点讨论活动交换。

## <a name="buffering-outbound-activities"></a>缓冲出站活动

发送活动不属于幂等操作，并且也不明确此操作在端到端方案中有多大的意义。 毕竟，活动通常只是携带要追加到视图的消息，或者文本转语音代理所要讲述的消息。

在发送活动方面，我们必须避免多次将其发送。 但问题是，乐观锁定机制可能要求多次重新运行逻辑。 解决方案十分简单：必须缓冲来自对话的出站活动，直到我们确信不再需要重新运行逻辑为止。 即，成功完成 Save 操作为止。 我们正在寻求一个如下所示的流：

![横向扩展示意图](../media/scale-out/scale-out-buffer.png)

假设我们可以围绕对话执行构建一个重试循环，如果在执行 Save 操作时发生前提条件失败，我们将获得以下行为：

![横向扩展示意图](../media/scale-out/scale-out-save.png)

应用此机制后，如果重新访问前面的示例，则我们永远不会看到在订单中添加披萨佐料的确认误报。 事实上，尽管我们在多台计算机之间横向扩展部署，但仍可以有效使用乐观锁定方案来序列化状态更新。 在披萨订购方案中，现在甚至可以编写添加项后进行确认的代码，以准确反映完整状态。 例如，如果用户立即键入“奶酪”，在机器人有机会回复“蘑菇”之前，两条回复可能依次是“加奶酪的披萨”和“加奶酪和蘑菇的披萨”。

在序列图上可以看到，成功完成 Save 操作后回复可能会丢失，但是，它们可能是在端到端通信的任意环节中丢失的。 要点在于，状态管理基础结构无法解决此问题。 需要使用更高级别的协议来解决，也许还要涉及到通道的用户。 例如，如果在用户看来，机器人似乎未给出回复，则可以合理地预期用户最终需要重试或执行此类行为。 因此，方案偶尔出现这种暂时性中断是合理的，但预期用户能够筛选出确认误报或其他意外消息，是完全不合理的。 

概括而言，在新的自定义存储解决方案中，我们要完成框架中的默认实现所不能做的三件事。 首先，使用 ETag 来检测争用；其次，检测到 ETag 失败时重试处理；最后，在成功保存之前缓冲所有出站活动。 本文的余下内容将介绍这三个部分的实现。

## <a name="implementing-etag-support"></a>实现 ETag 支持

为了支持单元测试，我们首先为支持 ETag 的新存储定义一个接口。 使用接口意味着可以编写两个版本，其中一个版本用于无需访问网络即可在内存中运行的单元测试，另一个版本用于生产环境。 通过接口可以十分方便地利用 ASP.NET 中的依赖项注入机制。

接口由 Load 和 Save 方法组成。 这两个方法采用我们要对状态使用的键。 Load 返回数据和关联的 ETag。 Save 引入这些信息。 此外，Save 将返回布尔值。 此布尔值指示 ETag 是否匹配，以及 Save 是否成功。 此值并非旨在用作常规的错误指示器，而是前提条件失败的特定指示器。我们将此建模为返回代码而不是异常，因为我们将在重试循环的形状中围绕此机制编写控制流逻辑。

由于我们希望此最低级别存储部件可插入，因此，需确保不要对其施加任何序列化要求，但是，需要指定保存的内容为 JSON，这样，存储实现才能设置内容类型。 在 .NET 中，实现此目的的最简单且最自然的方式是使用参数类型，具体而言，键入 JObject 形式的内容参数。 在 JavaScript 或 TypeScript 中，这只是一个常规本机对象。  

最终的接口如下：

```csharp
public interface IStore
{
  Task<(JObject content, string eTag)> LoadAsync(string key);
  Task<bool> SaveAsync(string key, JObject content, string eTag);
}
```
针对 Azure Blob 存储实现此接口的过程非常直接。
```csharp
public class BlobStore : IStore
{
  private CloudBlobContainer _container;

  public BlobStore(string myAccountName, string myAccountKey, string containerName)
  {
    var storageCredentials = new StorageCredentials(myAccountName, myAccountKey);
    var cloudStorageAccount = new CloudStorageAccount(storageCredentials, useHttps: true);
    var client = cloudStorageAccount.CreateCloudBlobClient();
    _container = client.GetContainerReference(containerName);
  }

  public async Task<(JObject content, string eTag)> LoadAsync(string key)
  {
    var blob = _container.GetBlockBlobReference(key);
    try
    {
      var content = await blob.DownloadTextAsync();
      var obj = JObject.Parse(content);
      var eTag = blob.Properties.ETag;
      return (obj, eTag);
    }
    catch (StorageException e)
      when (e.RequestInformation.HttpStatusCode ==
        (int)HttpStatusCode.NotFound)
    {
      return (new JObject(), null);
    }
  }

  public async Task<bool> SaveAsync(string key, JObject obj, string eTag)
  {
    var blob = _container.GetBlockBlobReference(key);
    blob.Properties.ContentType = "application/json";
    var content = obj.ToString();
    if (eTag != null)
    {
      try
      {
        await blob.UploadTextAsync(content,
          new AccessCondition { IfMatchETag = eTag },
          new BlobRequestOptions(),
          new OperationContext());
      }
      catch (StorageException e)
        when (e.RequestInformation.HttpStatusCode ==
          (int)HttpStatusCode.PreconditionFailed)
      {
        return false;
      }
    }
    else
    {
      await blob.UploadTextAsync(content);
    }
    return true;
  }
}
```

此处可以看到，Azure Blob 存储正在执行实际工作。 请注意特定异常的捕获，以及如何根据调用代码的预期来转换结果。 即，在执行 Load 时，我们希望“找不到”异常返回 null；在执行 Save 时，“不满足前提条件”异常返回布尔值。

所有这些源代码将在相应的[示例](https://aka.ms/scale-out)中提供，该示例将包含内存存储实现。

## <a name="implementing-the-retry-loop"></a>实现重试循环
基本循环形状直接派生自序列图中所示的行为。

接收活动时，我们将为该聊天的相应状态创建一个键。 我们不会更改活动与聊天状态之间的关系，因此，将采用默认状态实现中完全相同的方式创建键。

创建相应的密钥后，我们将尝试加载相应的状态。 然后，运行机器人的对话并尝试执行 Save。 如果 Save 成功，则发送运行对话后生成的出站活动，并完成操作。 否则，返回到前面的步骤并重复 Load 之前的整个过程。 重新执行 Load 会创建新的 ETag，因此，下一次的 Save 有望成功。

生成的 OnTurn 实现如下所示：
```csharp
public async Task OnTurnAsync(ITurnContext turnContext,
  CancellationToken cancellationToken = default(CancellationToken))
{
  // Create the storage key for this conversation.
  string key = $"{turnContext.Activity.ChannelId}/conversations/{turnContext.Activity.Conversation?.Id}";

  // The execution sits in a loop because there might be a retry if the save operation fails.
  while (true)
  {
    // Load any existing state associated with this key
    var (oldState, etag) = await _store.LoadAsync(key);

    // Run the dialog system with the old state and inbound activity,
    // resulting in a new state and outbound activities.
    var (activities, newState) = await DialogHost.RunAsync(_rootDialog, turnContext.Activity, oldState);

    // Save the updated state associated with this key.
    bool success = await _store.SaveAsync(key, newState, etag);

    // Following a successful save, send any outbound Activities, otherwise retry everything.
    if (success)
    {
      if (activities.Any())
      {
        // This is an actual send on the TurnContext we were given and so will actual do a send this time.
        await turnContext.SendActivitiesAsync(activities);
      }
      break;
    }
  }
}
```
请注意，我们已将对话执行建模为函数调用。 也许某个更复杂的实现定义了接口并使此依赖项可注入，但根据我们的目的，将所有对话定位在静态函数的后面是为了强调方法的功能性。 一般而言，对实现进行适当的组织，使关键部件正常运行，将非常有利于该实现在网络中成功运作。


## <a name="implementing-outbound-activity-buffering"></a>实现出站活动缓冲 

下一项要求是缓冲出站活动，直到成功执行 Save 为止。 这需要自定义的 BotAdapter 实现。 在此代码中，我们将实现抽象 SendActivity 函数，以将活动添加到列表，而无需发送活动。 要托管的对话并不是很智能。
在此特定方案中，UpdateActivity 和 DeleteActivity 操作不受支持，因此这些方法只会引发“未实现”。 此外，我们不关心 SendActivity 的返回值。 例如，需要发送活动更新的方案中的某些通道会使用此方法，来禁用通道中显示的卡片上的按钮。 需要状态时，这些消息交换可能特别复杂，这不属于本文的讨论范畴。 自定义 BotAdapter 的完整实现如下所示：

```csharp
public class DialogHostAdapter : BotAdapter
{
  private List<Activity> _response = new List<Activity>();

  public IEnumerable<Activity> Activities => _response;

  public override Task<ResourceResponse[]> SendActivitiesAsync(ITurnContext turnContext,
    Activity[] activities, CancellationToken cancellationToken)
  {
    foreach (var activity in activities)
    {
      _response.Add(activity);
    }
    return Task.FromResult(new ResourceResponse[0]);
  }

  public override Task DeleteActivityAsync(ITurnContext turnContext,
    ConversationReference reference, CancellationToken cancellationToken)
  {
    throw new NotImplementedException();
  }

  public override Task<ResourceResponse> UpdateActivityAsync(ITurnContext turnContext,
    Activity activity, CancellationToken cancellationToken)
  {
    throw new NotImplementedException();
  }
}
```
剩余的所有集成工作是将这些不同的新片段组合在一起，并将其插入到现有的框架片段。 主要重试循环只位于 IBot OnTurn 函数中。 它包含自定义 IStore 实现，出于测试目的，我们已使依赖项可注入。 我们已将所有对话托管代码放到一个名为 DialogHost、可公开单个公共静态函数的类。 定义此函数的目的是提取入站活动和旧状态，然后返回最终的活动和新状态。

在此函数中，要做的第一件事是创建前面所述的自定义 BotAdapter。 然后，我们只需按照平时在创建 DialogSet 和 DialogContext 以及执行一般性“继续”或“开始”流时所用的相同方式来运行该对话。 现在，我们唯一没有提到的片段是自定义访问器。 访问器是一个非常简单的填充码，可以简化将对话状态传递到对话系统的过程。 与对话系统配合使用时，访问器使用引用语义，因此，需要执行的操作无非是传递句柄。 为使内容更明确，我们约束了用于引用语义的类模板。

我们在分层时非常小心，已将 JsonSerialization 内联代码放在托管代码中，因为我们不希望在不同的实现以不同的方式序列化时，该代码位于可插入的存储层中。

下面是驱动程序代码：
```csharp
public class DialogHost
{
  private static readonly JsonSerializer StateJsonSerializer = new JsonSerializer()
    { TypeNameHandling = TypeNameHandling.All };

  public static async Task<Tuple<Activity[], JObject>> RunAsync(Dialog rootDialog,
    Activity activity, JObject oldState)
  {
    // A custom adapter and corresponding TurnContext that buffers any messages sent.
    var adapter = new DialogHostAdapter();
    var turnContext = new TurnContext(adapter, activity);

    // Run the dialog using this TurnContext with the existing state.
    JObject newState = await RunTurnAsync(rootDialog, turnContext, oldState);

    // The result is a set of activities to send and a replacement state.
    return Tuple.Create(adapter.Activities.ToArray(), newState);
  }

  private static async Task<JObject> RunTurnAsync(Dialog rootDialog,
    TurnContext turnContext, JObject state)
  {
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
      // If we have some state, deserialize it. (This mimics the shape produced by BotState.cs.)
      var dialogState = state?[nameof(DialogState)]?.ToObject<DialogState>(StateJsonSerializer);

      // A custom accessor is used to pass a handle on the state to the dialog system.
      var accessor = new RefAccessor<DialogState>(dialogState);

      // The following is regular dialog driver code.
      var dialogs = new DialogSet(accessor);
      dialogs.Add(rootDialog);

      var dialogContext = await dialogs.CreateContextAsync(turnContext);
      var results = await dialogContext.ContinueDialogAsync();

      if (results.Status == DialogTurnStatus.Empty)
      {
        await dialogContext.BeginDialogAsync("root");
      }

      // Serialize the result, and put its value back into a new JObject.
      return new JObject
      {
        { nameof(DialogState), JObject.FromObject(accessor.Value, StateJsonSerializer) }
      };
    }

    return state;
  }
}
```
最后，对于自定义访问器，我们只需实现 Set，因为状态是按引用定义的：
```csharp
public class RefAccessor<T> : IStatePropertyAccessor<T> where T : class
{
  public RefAccessor(T value)
  {
    Value = value;
  }

  public T Value { get; private set; }

  public string Name => nameof(T);

  public Task<T> GetAsync(ITurnContext turnContext, Func<T> defaultValueFactory = null,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    if (Value == null)
    {
      if (defaultValueFactory == null)
      {
        throw new KeyNotFoundException();
      }
      else
      {
        Value = defaultValueFactory();
      }
    }
    return Task.FromResult(Value);
  }

  public Task DeleteAsync(ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    throw new NotImplementedException();
  }

  public Task SetAsync(ITurnContext turnContext, T value,
    CancellationToken cancellationToken = default(CancellationToken))
  {
    throw new NotImplementedException();
  }
}
```

## <a name="additional-resources"></a>其他资源
GitHub 上提供了本文中使用的 [C# ](http://aka.ms/scale-out) 示例代码。

