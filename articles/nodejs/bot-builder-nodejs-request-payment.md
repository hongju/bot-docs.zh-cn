---
title: 请求付款 | Microsoft Docs
description: 了解如何使用 Bot Framework SDK for Node.js 发送支付请求。
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 88fd1ee1eb46fa056ecadae1f0cc1b19eb7c908f
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404843"
---
# <a name="request-payment"></a>请求付款

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

如果机器人支持用户购买商品，它可通过在[资讯卡](bot-builder-nodejs-send-rich-cards.md)内添加特殊类型的按钮来请求付款。 本文介绍了如何使用 Bot Framework SDK for Node.js 发送支付请求。

## <a name="prerequisites"></a>先决条件

必须先完成以下先决条件任务才能使用 Bot Framework SDK for Node.js 发送支付请求。

### <a name="register-and-configure-your-bot"></a>注册并配置机器人

将机器人的 `MicrosoftAppId` 和 `MicrosoftAppPassword` 环境变量更新为在[注册](~/bot-service-quickstart-registration.md)过程中为机器人生成的应用 ID 和密码值。 

> [!NOTE]
> 若要查找机器人的“AppID”  和“AppPassword”  ，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="create-and-configure-merchant-account"></a>创建并配置商家帐户

1. <a href="https://dashboard.stripe.com/register" target="_blank">如果还没有 Stripe 帐户，请创建并激活一个 Stripe 帐户。</a>

2. <a href="https://seller.microsoft.com/dashboard/registration/seller/?accountprogram=botframework" target="_blank">使用 Microsoft 帐户登录到卖家中心。</a>

3. 在卖家中心，将帐户与 Stripe 相关联。

4. 在卖家中心，导航到仪表板并复制 **MerchantID** 的值。

5. 将 `PAYMENTS_MERCHANT_ID` 环境变量更新为从卖家中心仪表板复制的值。 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>付款机器人示例

<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款机器人</a>示例提供了使用 Node.js 发送付款请求的机器人示例。 若要查看此示例机器人的实际操作，可以<a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">在网上聊天中试用一下</a>，<a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">将其添加为 Skype 联系人</a>，或下载付款机器人示例并使用 Bot Framework Emulator 在本地运行该示例。 

> [!NOTE]
> 若要在网上聊天或 Skype 中使用**付款机器人**示例完成端到端付款流程，必须在 Microsoft 帐户中指定有效的信用卡或借记卡（即，美国发卡机构发行的有效卡片）。 系统不会向你的卡收取费用，并且不会验证卡的 CVV，因为**付款机器人**示例在测试模式下运行（即， **.env** 中的 `PAYMENTS_LIVEMODE` 设置为 `false`）。

本文下面几节结合**付款机器人**示例，介绍付款流程的三个部分。

## <a id="request-payment"></a> 请求付款

机器人可以通过发送包含[资讯卡](bot-builder-nodejs-send-rich-cards.md)（具有指定“付款”`type` 的按钮）的消息来请求用户付款。 **付款机器人**示例中的此代码片段会创建一条包含英雄卡的消息，用户可通过单击（或点击）卡片中的“购买”按钮来启动付款流程  。 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#requestPayment)]

在此示例中，该按钮的类型指定为 `payments.PaymentActionType`，而应用将其定义为“付款”。 该按钮的值由 `createPaymentRequest` 函数填充，该函数返回一个 `PaymentRequest` 对象，其中包含有关支持的付款方式、详细信息和选项的信息。 有关实现细节的详细信息，请参阅<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款机器人</a>示例中的 **app.js**。

此屏幕截图显示上述代码片段生成的英雄卡（含“购买”按钮）  。 
 
![付款示例机器人](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> 有权访问“购买”按钮的任何用户都能用它来启动付款流程  。 在群组聊天的上下文中，无法指定某个按钮仅供某个特定用户使用。 

## <a id="user-experience"></a> 用户体验

当用户单击“购买”按钮时，他或她将定向至网页版支付，以通过其 Microsoft 帐户提供所有必需的付款、送货和联系信息  。 

![Microsoft 付款](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>HTTP 回调

HTTP 回调将发送给机器人，以指示它应执行某些操作。 每个回调都是一个包含以下属性值的事件： 

| 属性 | 值 |
|----|----|
| `type` | invoke | 
| `name` | 指示机器人应执行的操作类型（例如，送货地址更新、送货选项更新、付款完成）。 | 
| `value` | JSON 格式的请求有效负载。 | 
| `relatesTo` |  介绍与付款请求关联的通道和用户。 | 

> [!NOTE]
> `invoke` 是保留供 Microsoft Bot Framework 使用的特殊事件类型。 `invoke` 事件的发送者期望机器人通过发送 HTTP 响应来确认回调。

## <a id="process-callbacks"></a>处理回调

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>送货地址更新和送货选项更新回调

收到“发货地址更新”或“发货选项更新”回调时，机器人将从该事件的 `value` 属性中的客户端获取付款详细信息的当前状态。
作为商家，你应该将这些回调视为静态，根据给定的输入付款详细信息计算某些输出付款详细信息。如果客户端提供的输入状态因故无效，则操作会失败。 
如果机器人确定给定信息按原样有效，则只需发送 HTTP 状态代码 `200 OK` 以及未修改的付款详细信息。 或者，机器人可以发送 HTTP 状态代码 `200 OK` 以及在处理订单之前应该应用的更新付款详细信息。 在某些情况下，机器人可能会确定更新的信息无效，并且无法按原样处理订单。 例如，用户的送货地址可能会指定产品供应商不发货的国家/地区。 在这种情况下，机器人可能会发送 HTTP 状态代码 `200 OK` 以及一个填充了付款详细信息对象的错误属性的消息。 发送 `400` 或 `500` 范围内的任何 HTTP 状态代码会导致客户出现一般错误。

### <a name="payment-complete-callbacks"></a>支付完成回调

收到“付款完成”回调时，机器人将获取未修改的初始付款请求的副本以及该事件的 `value` 属性中的付款响应对象。 支付响应对象将包含客户所做的最终选择以及支付令牌。 机器人应借此机会，根据初始付款请求和客户的最终选择重新计算最终付款请求。 假设客户的选择确定有效，机器人将验证支付令牌标头中的金额和货币，以确保它们匹配最终支付请求。  如果机器人决定向客户收费，应仅收取支付令牌标头中的金额，因为这是客户所确认的价格。 如果机器人预期值和支付完成回调中的值不匹配，可通过发送 HTTP 状态代码 `200 OK` 并将结果字段设为 `failure`，使支付请求失效。   

除了验证支付详细信息，机器人还应先验证能够完成订单，然后再启动支付过程。 例如，它可能想要验证所购买的物品在库存中仍然可用。 如果值正确无误并且支付处理器已成功对支付令牌收费，机器人将通过 HTTP 状态代码 `200 OK` 响应，并将结果字段设为 `success`，以使网页版支付显示支付确认。 机器人收到的支付令牌仅供请求的商家使用一次，并且必须提交到 Stripe（Bot Framework 目前支持的唯一支付处理器）。 发送 `400` 或 `500` 范围内的任何 HTTP 状态代码会导致客户出现一般错误。

**付款机器人**示例中的此代码片段将处理机器人收到的回调。 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#processCallback)]

在此示例中，机器人检查传入事件的 `name` 属性以确定它需要执行的操作类型，然后调用适当的方法来处理回调。 有关实现细节的详细信息，请参阅<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款机器人</a>示例中的 **app.js**。

## <a name="testing-a-payment-bot"></a>测试付款机器人

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

在<a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">付款机器人</a>示例中， **.env** 中的 `PAYMENTS_LIVEMODE` 环境变量用于确定“付款完成”回调是包含模拟付款令牌还是真实付款令牌。 如果 `PAYMENTS_LIVEMODE` 设置为 `false`，将向机器人的出站支付请求添加标头，以指示该机器人处理测试模式，并且支付完成回调将包含无法进行收费的模拟支付令牌。 如果 `PAYMENTS_LIVEMODE` 设置为 `true`，将从机器人的出站支付请求中省略指示机器人处于测试模式的标头，并且支付完成回调将包含机器人将提交到 Stripe 进行支付处理的真实支付令牌。 这将是真正的交易，可导致对指定付款方式收取费用。 

## <a name="additional-resources"></a>其他资源

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">支付机器人示例</a>
- [向消息添加资讯卡附件](bot-builder-nodejs-send-rich-cards.md)
- <a href="http://www.w3.org/Payments/" target="_blank">W3C 上的网页支付</a> 
