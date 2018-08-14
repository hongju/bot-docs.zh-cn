---
title: 请求支付 | Microsoft Docs
description: 了解如何使用 Bot Builder SDK for .NET 发送支付请求。
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ce58c210123219c14f580d7164c759307ac2237b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/18/2018
ms.locfileid: "39298419"
---
# <a name="request-payment"></a>请求支付
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

如果机器人支持用户购买物品，它可通过在[资讯卡](bot-builder-dotnet-add-rich-card-attachments.md)内添加特殊类型的按钮来请求支付。 本文介绍如何使用 Bot Builder SDK for .NET 发送支付请求。

## <a name="prerequisites"></a>先决条件

必须先完成以下先决条件任务才能使用 Bot Builder SDK for .NET 发送支付请求。

### <a name="update-webconfig"></a>更新 Web.config

更新机器人的 Web.config 文件，以将 `MicrosoftAppId` 和 `MicrosoftAppPassword` 设置为[注册](~/bot-service-quickstart-registration.md)过程中为机器人生成的应用 ID 和密码值。 

> [!NOTE]
> 要查找机器人的 AppID 和 AppPassword，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="create-and-configure-merchant-account"></a>创建和配置商家帐户

1. <a href="https://dashboard.stripe.com/register" target="_blank">如果还没有存储帐户，请创建并激活一个 Stripe 帐户。</a>

2. <a href="https://seller.microsoft.com/en-us/dashboard/registration/seller/?accountprogram=botframework" target="_blank">使用 Microsoft 帐户登录到卖家中心。</a>

3. 在卖家中心内，使用 Stripe 连接帐户。

4. 在卖方中心内，导航到“仪表板”和并复制值 MerchantID。

5. 更新机器人的 Web.config 文件，以将 `MerchantId` 设置为从卖家中心仪表板中复制的值。 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>支付机器人示例

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支付机器人</a>示例提供了使用 .NET 发送支付请求的机器人示例。 若要查看实际操作中的此机器人，可以<a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">在网上聊天中试用一下</a>，<a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">将其添加为 Skype 联系人</a>或下载支付机器人示例并使用 Bot Framework Emulator 在本地运行。 

> [!NOTE]
> 要使用网上聊天或 Skype 中的支付机器人示例完成端到端支付过程，必须在 Microsoft 帐户内指定有效的信用卡或借记卡（即来自美国发卡机构的有效卡片）。 系统不会收取卡费用，并且不会验证卡的 CVV，因为支付机器人在测试模式下运行（即 `LiveMode` 在 Web.config 中设置为 `false`）。

本文下面几节结合支付机器人示例，介绍支付过程的三个部分。

## <a id="request-payment"></a>请求支付

机器人通过使用指定“支付” `CardAction.Type` 的按钮发送包含[资讯卡附件](bot-builder-dotnet-add-rich-card-attachments.md)的消息来请求用户支付。 支付机器人示例中的代码片段创建一条包含英雄卡的消息，用户可通过单击（或点击）卡片中的“购买”按钮启动支付过程。 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#requestPayment)]

在此示例中，该按钮的类型被指定为 `PaymentRequest.PaymentActionType`，Bot Builder 库将其定义为“支付”。 按钮的值由 `BuildPaymentRequest` 方法填充，该方法将返回包含支持的支付方式、详细信息和选项的信息的 `PaymentRequest` 对象。 有关实现详细信息的更多信息，请参阅支付机器人示例内的 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Dialogs/RootDialog.cs</a>。

此屏幕截图显示上述代码片段生成的英雄卡（含“购买”按钮）。 
 
![支付机器人示例](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> 有权访问“购买”按钮的任何用户都能使用它启动支付过程。 在群组聊天的上下文中，指定某个按钮仅供某个特定用户使用是不可能的。 

## <a id="user-experience"></a>用户体验

用户单击“购买”按钮时，他/她将被定向到网页版支付，以通过其 Microsoft 帐户提供所有必需的支付、送货和联系信息。 

![Microsoft 支付](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>HTTP 回调

HTTP 回调将发送给机器人，以指示它应执行某些操作。 每个回调都是一个[活动](bot-builder-dotnet-activities.md)并包含以下属性值： 

| 属性 | 值 |
|----|----|
| `Activity.Type` | invoke | 
| `Activity.Name` | 指示机器人应执行的操作类型（例如，送货地址更新、送货选项更新和支付完成）。 | 
| `Activity.Value` | JSON 格式的请求负载。 | 
| `Activity.RelatesTo` |  描述与支付请求关联的通道和用户。 | 

> [!NOTE]
> `invoke` 是保留供 Microsoft Bot Framework 使用的特殊活动类型。 `invoke` 活动的发件人期望机器人通过发送 HTTP 响应确认回调。

## <a id="process-callbacks"></a>处理回调

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>送货地址更新和送货选项更新回调

[!INCLUDE [Process shipping address and shipping option callbacks](../includes/snippet-payment-process-callbacks-1.md)]

### <a name="payment-complete-callbacks"></a>支付完成回调

收到支付完成回调时，将向机器人提供最初的、未经修改的支付请求副本以及 `Activity.Value` 属性中的支付响应对象。 支付响应对象将包含客户所做的最终选择以及支付令牌。 机器人应利用这个机会，根据最初的支付请求和客户的最终选择，重新计算最终支付请求。 假设客户的选择确定有效，机器人将验证支付令牌标头中的金额和货币，以确保它们匹配最终支付请求。  如果机器人决定向客户收费，应仅收取支付令牌标头中的金额，因为这是客户所确认的价格。 如果机器人预期值和支付完成回调中的值不匹配，可通过发送 HTTP 状态代码 `200 OK` 并将结果字段设为 `failure`，使支付请求失效。   

除了验证支付详细信息，机器人还应先验证能够完成订单，然后再启动支付过程。 例如，它可能想要验证所购买的物品在库存中仍然可用。 如果值正确无误并且支付处理器已成功对支付令牌收费，机器人将通过 HTTP 状态代码 `200 OK` 响应，并将结果字段设为 `success`，以使网页版支付显示支付确认。 机器人收到的支付令牌仅供请求的商家使用一次，并且必须提交到 Stripe（Bot Framework 目前支持的唯一支付处理器）。 发送 `400` 或 `500` 范围内的任何 HTTP 状态代码会导致客户出现一般错误。

支付机器人示例中的 `OnInvoke` 方法将处理机器人收到的回调。 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#processCallback)]

在此示例中，机器人检查传入活动的 `Name` 属性，以确定需要执行的操作类型，然后调用相应的方法处理回调。 有关实现详细信息的更多信息，请参阅支付机器人示例内的 <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Controllers/MessagesControllers.cs</a>。

## <a name="testing-a-payment-bot"></a>测支付机器人

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

在<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支付机器人</a>示例中，Web.config 中的 `LiveMode`配置设置确定支付完成回调中是否会包含模拟的支付令牌或真实的支付令牌。 如果 `LiveMode` 设置为 `false`，将向机器人的出站支付请求添加标头，以指示该机器人处理测试模式，并且支付完成回调将包含无法进行收费的模拟支付令牌。 如果 `LiveMode` 设置为 `true`，将从机器人的出站支付请求中省略指示机器人处于测试模式的标头，并且支付完成回调将包含机器人将提交到 Stripe 进行支付处理的真实支付令牌。 这将是真正的交易，可导致对指定支付方式收取费用。 

## <a name="additional-resources"></a>其他资源

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">支付机器人示例</a>
- [活动概述](bot-builder-dotnet-activities.md)
- [向消息添加资讯卡](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="http://www.w3.org/Payments/" target="_blank">W3C 上的网页支付</a> 
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Bot Builder SDK for .NET 参考</a>