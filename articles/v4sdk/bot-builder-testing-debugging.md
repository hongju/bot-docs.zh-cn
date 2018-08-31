---
title: 测试和调试指南 | Microsoft Docs
description: 了解如何测试和调试机器人。
keywords: 测试原则, 模拟元素, 常见问题解答, 测试级别
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/09/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: caa424ed0ea0944805836739ed48a7a61f78d21c
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905256"
---
# <a name="testing-and-debugging-guidelines"></a>测试和调试指南

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

机器人是复杂的应用，有大量不同的协同工作的部件。 就像其他复杂应用一样，这可能会导致出现一些需要关注的 Bug，或者会导致机器人的行为异常。

有时候，机器人的测试和后续调试是一项困难的任务。 每位开发人员在完成该任务时都有自己的首选方式；下面提供的指南是供你使用的适用于大多数机器人的建议。

## <a name="testing-your-bot"></a>测试机器人

下面提供的指南分三个不同的**级别**。  提高级别时，测试的复杂性和测试的功能都会增加，因此建议你在对某个级别的结果满意之后再进行下一级别的测试。 这样一来，就可以先厘清并解决最低级别的问题，然后再增加复杂性。

测试最佳做法会尽可能涵盖不同的角度， 其中可能包括安全性、集成、URL 格式错误、验证攻击、HTTP 状态代码、JSON 有效负载、null 值，等等。 如果机器人处理的信息会影响用户隐私，这就尤其重要。

### <a name="level-1-use-mock-elements"></a>级别 1：使用模拟元素

第一级测试是确保应用（在本示例中为机器人）的每个小组件运行起来完全符合预期。 为此，可以对当前不测试的项目使用模拟元素。 通常可以将此级别视为单元和集成测试（供参考）。

**使用模拟元素测试单个部分**

尽可能多模拟一些元素，这样可以更好地隔离要测试的组件。 模拟元素的候选对象包括存储、适配器、中间件、活动管道、通道，以及任何其他不直接归属于机器人的部件。 这种测试也可能会临时去除某些方面的内容（例如，不让中间件加入要测试的机器人），以便隔离每个组件。 但是，若要测试中间件，则可改为对机器人进行模拟。

模拟元素可以采用多种形式，例如使用另一已知对象来替换某个元素，或者实现单纯的 hello world 功能。 也可直接删除该元素（如果不是必需的元素），或者直接强制它不执行任何操作。 

此级别应该在机器人内部演示单个方法和函数。 测试单个方法时，可以使用建议的内置单元测试，使用你自己的测试应用或测试套件，或者在 IDE 中手动这样做。 

**使用模拟元素测试更大型的功能**

对每个方法的行为满意以后，请使用这些模拟元素在机器人中测试更完整的功能。 这样可以演示在与你的用户聊天时，多个层是如何协调工作的。 

为此，可以使用多种提供的工具。 例如，[Azure Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) 提供的模拟通道可以与机器人通信。 使用模拟器时，遇到的情况会比单纯的单元和集成测试更复杂，因此还需进行下一级测试。

### <a name="level-2-use-a-direct-line-client"></a>级别 2：使用 Direct Line 客户端

验证机器人的运行在表面上是否符合预期以后，下一步是将其连接到某个通道。 为此，可以将机器人部署到过渡服务器，并创建你自己的 [Direct Line 客户端](bot-builder-howto-direct-line.md)，供机器人进行连接。

创建自己的客户端以后，即可定义通道的内部运作，并对机器人对某些活动交换操作的响应进行具体测试。 连接到客户端以后，请运行测试，以便设置机器人状态并验证功能。 如果机器人使用语音之类的功能，则可通过这些通道来验证该功能。

在这里，通过 Azure 门户来使用 Emulator 和网上聊天可以进一步了解机器人在与不同通道交互时的表现情况。

### <a name="level-3-channel-tests"></a>级别 3：通道测试

对机器人的独立表现有把握以后，必须了解它在不同通道中的运行情况，因为它是在这些通道中提供给用户使用的。 

实现这一点的方式可能差异很大，既可以单独使用不同的通道和浏览器，也可以使用 [Selenium](https://docs.seleniumhq.org/) 之类的第三方工具通过通道以及来自机器人的擦除响应进行交互。

### <a name="other-testing"></a>其他测试

不同类型的测试可以与上述级别的测试配合使用，也可以从不同的角度来进行，例如：压力测试、性能测试或机器人活动分析。 Visual studio 提供用于在本地执行此类操作的方法，并提供进行应用测试的[工具套件](https://www.visualstudio.com/team-services/testing-tools/)，而 [Azure 门户](https://portal.azure.com)则可用于了解机器人的表现情况。

## <a name="debugging"></a>调试

调试机器人与调试其他多线程应用类似，可以设置断点，也可以使用即时窗口之类的功能。 

机器人遵循事件驱动型编程范例。如果不熟悉此方面的内容，则可能难以理解。 如果想要让机器人在无状态且使用多个线程的情况下处理异步/等待调用，则可能导致意外 Bug。 虽然调试机器人的方式与调试其他多线程应用类似，但我们还是会介绍一些有用的建议、工具和资源。

**通过模拟器了解机器人活动**

除了正常的消息活动以外，机器人还处理不同类型的[活动](bot-builder-concept-activity-processing.md)。 可以通过[模拟器](../bot-service-debug-emulator.md)来了解这些活动是什么、其发生时间以及其包含的具体信息。 了解这些活动有助于你提高机器人代码编写效率，并可验证机器人发送和接收的活动是否符合预期。

**中间件工作原理**

在首次尝试使用[中间件](bot-builder-concept-middleware.md)时，可能会觉得它不直观，尤其是在涉及到持续执行或短路执行的情况下。 中间件可以在某个轮次的前缘或后缘执行，可以调用 `next()` 委托来指示何时将执行传递给机器人逻辑。 

如果使用多个中间件，委托可能会根据管道的指引将执行传递给不同的中间件。 若要更清楚地了解该理念，可查看[机器人中间件管道](bot-builder-concept-middleware.md#the-bot-middleware-pipeline)的详细信息。

如果未调用 `next()` 委托，则会将其称为[短路路由](bot-builder-concept-middleware.md#short-circuiting)。 当中间件符合当前活动情况且确定不需传递执行时，则会发生这种情况。 

了解中间件何时会短路以及短路的原因后，即可指示哪个中间件应该先进入管道。 另外，了解预期结果对于 SDK 或其他开发者提供的内置中间件来说特别重要。 可以试着先[创建你自己的中间件](bot-builder-create-middleware.md)试验一下，然后再使用内置的中间件，这样也许会有用。

例如，[QnA Maker](bot-builder-howto-qna.md) 旨在处理特定的交互并会在执行此类操作时造成管道短路，这对于首次学习其使用的用户来说可能难以理解。

**了解状态**

记录状态是机器人必须做的，尤其对于复杂任务来说。 通常情况下，最佳做法是尽快处理活动并让处理操作完成，这样状态就能持久保存。 活动可能会在几乎同一时间发送至机器人，这可能会引入令人极为困惑的 Bug，因为体系结构是异步的。

最重要的，请确保在持久保存状态时，所用方式与预期一致。 可以根据持久保存的状态的存储位置，使用 [Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator) 和 [Azure 表存储](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator)的存储模拟器来验证该状态，然后再使用生产性存储。

**如何使用活动处理程序**

活动处理程序可能会引入另一层复杂性，尤其是每个活动都在独立线程（或 Web 辅助角色，具体取决于语言）上运行。 这可能会导致当前状态不符合预期的问题，具体取决于处理程序的操作。

内置状态在一个轮次结束时写入，但是，由该轮次生成的任何活动在执行时均独立于轮次管道。 通常这不会影响我们，但如果活动处理程序更改状态，则写入的状态必须包含所做的更改。 在这种情况下，轮次管道在完成操作之前，会先等待活动处理完毕，目的是确保记录该轮次的正确状态。

如果需要向用户输出某些内容，则“发送活动”方法及其处理程序会造成一个独特的问题，因为在“发送活动”方法中直接调用其自身会导致线程无限分叉。 可以通过多种方式解决该问题，例如，可以在传出信息中追加调试消息，或者将内容写出到另一位置（例如控制台或文件），以免机器人崩溃。


## <a name="additional-resources"></a>其他资源
* [在 Visual Studio 中进行调试](https://docs.microsoft.com/en-us/visualstudio/debugger/index)
* 适用于 Bot Framework 的[调试、跟踪和分析](https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/)
* 将 [ConditionalAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.conditionalattribute?view=netcore-2.0) 用于不希望包括在生产代码中的方法 
* 使用 [Fiddler](https://www.telerik.com/fiddler) 之类的工具来查看网络流量 
* [机器人工具存储库](https://github.com/Microsoft/botbuilder-tools)
* 可以借助 [Moq](https://github.com/moq/moq4) 之类的框架进行测试
