---
title: 对 Bot Framework 身份验证进行故障排除 | Microsoft Docs
description: 了解如何对机器人的身份验证错误进行故障排除。
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/17
ms.openlocfilehash: 0fdd196716c0fffb36583c0df894481b032dd83e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999404"
---
# <a name="troubleshooting-bot-framework-authentication"></a>对 Bot Framework 身份验证进行故障排除

本指南通过评估一系列方案来确定问题所在，从而帮助对机器人的身份验证问题进行故障排除。 

> [!NOTE]
> 若要完成本指南中的所有步骤，需下载和使用 [Bot Framework Emulator][Emulator]，并且必须有权在 <a href="https://dev.botframework.com" target="_blank">Bot Framework 门户</a> 中访问机器人的注册设置。

## <a id="PW"></a>应用 ID 和密码

需使用向 Bot Framework 注册机器人时获取的 Microsoft 应用 ID 和 Microsoft 应用密码配置机器人安全性。 这些值通常在机器人的配置文件中指定，用于从 Microsoft 帐户服务检索访问令牌。 

如果尚未执行此操作，请[将机器人部署到 azure](~/bot-builder-howto-deploy-azure.md)，以获取可用于身份验证的 Microsoft 应用 ID 和 Microsoft 应用密码。 

> [!NOTE]
> 若要查找已部署机器人的机器人 **AppID** 和 **AppPassword**，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

## <a name="step-1-disable-security-and-test-on-localhost"></a>步骤 1：禁用安全性，并在 localhost 上进行测试

在此步骤中，将验证在禁用安全性的情况下，机器人在 localhost 上是否处于可访问状态并能够正常运行。 

> [!WARNING]
> 禁用机器人安全性可能会允许未知攻击者模拟用户。 只有在受保护的调试环境中运行时，才能实施以下过程。

### <a id="disable-security-localhost"></a>禁用安全性

若要禁用机器人的安全性，请编辑其配置设置，删除应用 ID 和密码值。 

::: moniker range="azure-bot-service-3.0"

如果使用 Bot Builder SDK for .NET，请在 Web.config 文件中编辑这些设置： 

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

如果使用 Bot Builder SDK for Node.js，请编辑这些值（或更新相应的环境变量）：

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

::: moniker-end

::: moniker range="azure-bot-service-4.0"

如果使用 Bot Builder SDK for .NET，请在 `appsettings.config` 文件中编辑设置：

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

如果使用 Bot Builder SDK for Node.js，请编辑这些值（或更新相应的环境变量）：

```javascript
const adapter = new BotFrameworkAdapter({
    appId: null,
    appPassword: null
});
```

如果使用 `.bot` 文件进行配置，则可将 `appId` 和 `appPassword` 更新为 `""`。

::: moniker-end

### <a name="test-your-bot-on-localhost"></a>在 localhost 上测试机器人 

接下来，使用 Bot Framework Emulator 在 localhost 上测试机器人。

1. 在 localhost 上启动机器人。
2. 启动 Bot Framework Emulator。
3. 使用模拟器连接到机器人。
    - 在模拟器的地址栏中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。 
    - 确保“Microsoft 应用 ID”和“Microsoft 应用密码”字段均为空。
    - 单击“连接”。
4. 若要测试与机器人的连接性，请在模拟器中键入一些文本，然后按 Enter。

如果机器人对输入做出响应，且聊天窗口中未显示任何错误，则可确认在禁用安全性的情况下，机器人在 localhost 上处于可访问状态并能正常运行。 继续执行[步骤 2](#step-2)。

如果聊天窗口中显示一个或多个错误，请单击错误，了解详细信息。 常见问题包括：

* 模拟器设置指定的机器人终结点不正确。 请确保 URL 中的端口号正确，URL 末尾处的路径正确（例如 `/api/messages`）。
* 模拟器设置指定以 `https` 开头的机器人终结点。 在 localhost 上，终结点应以 `http` 开始。
* 模拟器设置指定了“Microsoft 应用 ID”字段和/或“Microsoft 应用密码”字段的值。 这两个字段应为空。
* 尚未禁用机器人的安全性。 [验证](#disable-security-localhost)机器人未指定应用 ID 或密码的值。

## <a id="step-2"></a>步骤 2：验证机器人的应用 ID 和密码

在此步骤中，将验证机器人用于身份验证的应用 ID 和密码是否有效。 （如果不知道这些值，请立即[获取这些值](#PW)。） 

> [!WARNING]
> 以下说明禁用 `login.microsoftonline.com` 的 SSL 验证。 仅在安全网络上执行此过程，完成后请考虑更改应用程序密码。

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>向 Microsoft 登录服务发出 HTTP 请求

这些说明介绍如何使用 [cURL](https://curl.haxx.se/download.html) 向 Microsoft 登录服务发出 HTTP 请求。 可能会使用 Postman 等备选工具，只要确保请求符合 Bot Framework [身份验证协议](~/rest-api/bot-framework-rest-connector-authentication.md)即可。

若要验证机器人的应用 ID 和密码是否有效，请使用 cURL（将 `APP_ID` 和 `APP_PASSWORD` 替换为机器人的应用 ID 和密码）发出以下请求。

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

此请求尝试以机器人的应用 ID 和密码换取访问令牌。 如果请求成功，将收到包含 `access_token` 属性及其他属性的 JSON 有效负载。 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

如果请求成功，则可确认请求中指定的应用 ID 和密码有效。 继续执行[步骤 3](#step-3)。

如果请求响应中收到错误，请检查该响应，确定错误原因。 如果响应指示应用 ID 或密码无效，请从 Bot Framework 门户[获取正确的值](#PW)，并使用新值重新发出请求，确认其是否有效。 

## 步骤 3：启用安全性，并在 localhost 上进行测试<a id="step-3"></a>

此时，已验证在禁用安全性的情况下，机器人在 localhost 上处于可访问状态并能够正常运行，并已确认机器人用于身份验证的应用 ID 和密码有效。 在此步骤中，将验证在启用安全性的情况下，机器人在 localhost 上是否处于可访问状态并能够正常运行。

### <a id="enable-security-localhost"></a>启用安全性

机器人的安全性依赖于 Microsoft 服务，即使机器人仅在 localhost 上运行也是如此。 若要启用机器人的安全性，请编辑其配置设置，使用[步骤 2](#step-2) 中验证的值填充应用 ID 和密码。

如果使用 Bot Builder SDK for .NET，请在 `.bot` 或 `appsettings.config` 文件中填充这些设置：

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

如果使用 Bot Builder SDK for Node.js，请填充这些设置（或更新相应的环境变量）：

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> 要查找机器人的 AppID 和 AppPassword，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

### <a name="test-your-bot-on-localhost"></a>在 localhost 上测试机器人 

接下来，使用 Bot Framework Emulator 在 localhost 上测试机器人。

1. 在 localhost 上启动机器人。
2. 启动 Bot Framework Emulator。
3. 使用模拟器连接到机器人。
    - 在模拟器的地址栏中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。 
    - 在“Microsoft 应用 ID”字段输入机器人的应用 ID。
    - 在“Microsoft 应用密码”字段输入机器人的密码。
    - 单击“连接”。
4. 若要测试与机器人的连接性，请在模拟器中键入一些文本，然后按 Enter。

如果机器人对输入做出响应，且聊天窗口中未显示任何错误，则可确认在启用安全性的情况下，机器人在 localhost 上处于可访问状态并能正常运行。  继续执行[步骤 4](#step-4)。

如果聊天窗口中显示一个或多个错误，请单击错误，了解详细信息。 常见问题包括：

* 模拟器设置指定的机器人终结点不正确。 请确保 URL 中的端口号正确，URL 末尾处的路径正确（例如 `/api/messages`）。
* 模拟器设置指定以 `https` 开头的机器人终结点。 在 localhost 上，终结点应以 `http` 开始。
* 在模拟器设置中，“Microsoft 应用 ID”字段和/或“Microsoft 应用密码”不包含有效值。 应填充这两个字段，每个字段应包含[步骤 2](#step-2) 中验证的相应值。
* 尚未启用机器人的安全性。 [验证](#enable-security-localhost)机器人配置设置是否指定应用 ID 和密码的值。

## 步骤 4：在云端测试机器人<a id="step-4"></a>

此时，已验证在禁用安全性的情况下，机器人在 localhost 上处于可访问状态并能够正常运行，已确认机器人的应用 ID 和密码有效，并已验证在启用安全性的情况下，机器人在 localhost 上处于可访问状态并能够正常运行。 在此步骤中，将机器人部署到云，并验证在启用安全性的情况下，机器人是否处于可访问状态并能够正常运行。 

### <a name="deploy-your-bot-to-the-cloud"></a>将机器人部署到云

Bot Framework 要求必须可从 Internet 访问机器人，因此必须将机器人部署到 Azure 等云托管平台。 请务必在部署之前启用机器人的安全性，如[步骤 3](#step-3) 中所述。

> [!NOTE]
> 如果尚无云托管提供程序，可注册<a href="https://azure.microsoft.com/en-us/free/" target="_blank">免费帐户</a>。 

如果将机器人部署到 Azure，将为应用程序自动配置 SSL，从而启用 Bot Framework 需要的 HTTPS 终结点。 如果部署到其他云托管提供程序，请务必验证应用程序是否配置 SSL，以便机器人具有 HTTPS 终结点。

### <a name="test-your-bot"></a>测试机器人 

若要在启用安全性的情况下在云端测试机器人，请完成以下步骤。

1. 请确保机器人已成功部署并正在运行。 
2. 登录 <a href="https://dev.botframework.com" target="_blank">Bot Framework 门户</a>。
3. 单击“我的机器人”。
4. 选择要测试的机器人。
5. 单击“测试”，在嵌入式 Web 聊天控件中打开机器人。
6. 若要测试与机器人的连接性，请在 Web 聊天控件中键入一些文本，然后按 Enter。

如果聊天窗口中指示存在错误，请使用错误消息确定错误原因。 常见问题包括： 

* 在 Bot Framework 门户的“设置”页上为机器人指定的“消息终结点”不正确。 请确保 URL 末尾处包含的路径正确（例如 `/api/messages`）。
* 在 Bot Framework 门户的“设置”页上为机器人指定的“消息终结点”不以 `https` 开头，或者不受 Bot Framework 信任。 机器人必须具有一个有效的链信任证书。
* 机器人的应用 ID 值或密码值未配置或配置不正确。 [验证](#enable-security-localhost)机器人配置设置是否指定有效的应用 ID 值和密码值。

如果机器人对输入作出相应响应，则可确认在启用安全性的情况下，机器人在云中处于可访问状态并能够正常运行。 此时，可将机器人安全地[连接到通道](~/bot-service-manage-channels.md)（如 Skype、Facebook Messenger、Direct Line 等）。

## <a name="additional-resources"></a>其他资源

如果完成上述步骤后仍然遇到问题，可执行以下操作：

* 使用 Bot Framework Emulator 和 <a href="https://ngrok.com/" target="_blank">ngrok</a> [在云中调试机器人](~/bot-service-debug-emulator.md)。
* 使用 [Fiddler](https://www.telerik.com/fiddler) 等代理工具检查传入和传出机器人的 HTTPS 流量。 Fiddler 不是 Microsoft 产品。
* 查阅 [Bot Connector 身份验证指南][BotConnectorAuthGuide]，了解 Bot Framework 使用的身份验证技术。
* 使用 Bot Framework [支持][Support]资源，寻求他人帮助。 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md
