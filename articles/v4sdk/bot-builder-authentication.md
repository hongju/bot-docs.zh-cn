---
title: 通过 Azure 机器人服务向机器人添加身份验证 | Microsoft Docs
description: 了解如何使用 Azure 机器人服务身份验证功能向机器人添加 SSO。
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2f15817abe087650bc3f2bb998a32f177848cf50
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904530"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>通过 Azure 机器人服务向机器人添加身份验证

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure 机器人服务和 v4 SDK 包含新的机器人身份验证功能，并提供各种功能来帮助轻松开发用于向 Azure AD (Azure Active Directory)、GitHub、Uber 等各种标识提供者验证用户身份的机器人。 这些功能可通过免去某些客户端的幻码验证来改善用户体验。

在此之前，机器人需要添加 OAuth 控制器和登录链接、存储目标客户端 ID 和密码，以及执行用户令牌管理。 机器人将要求用户在网站上登录，然后生成一个幻码，供用户用来验证其身份。

现在，机器人开发人员不再需要托管 OAuth 控制器或管理令牌生存期，因为现在 Azure 机器人服务就可以处理这些事项。

具体功能包括：

- 改进了通道，支持新的身份验证功能（如新的 WebChat 和 DirectLineJS 库），无需 6 位数幻码验证。
- 改进了 Azure 门户，可添加、删除和配置各种 OAuth 标识提供者的连接设置。
- 支持各种现成的标识提供者，包括 Azure AD（v1 和 v2 终结点）、GitHub 等。
- 更新了 C# 和 Node.js Bot Framework SDK，使之可检索令牌、创建 OAuthCard 并处理 TokenResponse 事件。
- 提供用于向 Azure AD 进行身份验证的机器人的开发方法示例。

你可以根据本文中的步骤进行推断，将此类功能添加到现有机器人。 这些示例机器人演示了新的身份验证功能。

> [!NOTE]
> 身份验证功能也适用于 BotBuilder v3。 但本文只介绍 v4 示例代码。

### <a name="about-this-sample"></a>关于此示例

需要创建一个 Azure 机器人资源，并需要创建新的 Azure AD（v1 或 v2）应用程序来让机器人访问 Office 365。 该机器人资源将注册机器人的凭据；即使在本地运行机器人代码，也需要使用这些凭据来测试身份验证功能。

> [!IMPORTANT]
> 每当在 Azure 中注册机器人时，系统都会为它分配一个 Azure AD 应用。 但是，此应用为通道到机器人的访问提供保护。
对于你希望机器人能够代表用户验证的每个应用程序，还需要额外分配一个 AAD 应用。

本文将会介绍一个使用 Azure AD v1 或 v2 令牌连接到 Microsoft Graph 的示例机器人。 此外，介绍如何创建和注册关联的 Azure AD 应用。 在此过程中，你将使用 [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) GitHub 存储库中的代码。 本文将介绍这些过程。

- **创建机器人资源**
- **创建 Azure AD 应用程序**
- **在机器人中注册 Azure AD 应用程序**
- **准备机器人示例代码**

完成这些过程后，将获得一个在本地运行的机器人，它可以对 Azure AD 应用程序响应某些简单任务，例如检查和发送电子邮件，或者显示你是谁以及你的经理是谁。 为此，机器人需要对 Microsoft.Graph 库使用 Azure AD 应用程序中的令牌。 无需发布机器人即可测试 OAuth 登录功能；但是，机器人需要有效的 Azure 应用 ID 和密码。

这些身份验证功能也适用于其他类型的机器人。 但是，本文将使用仅限注册的机器人。

### <a name="web-chat-and-direct-line-considerations"></a>网络聊天和 Direct Line 注意事项

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

将 Azure 机器人服务身份验证用于网络聊天时，需要注意几个重要的安全问题。

1. 防止模拟，否则攻击者可能会导致机器人认为它们不是自己。 在网络聊天中，攻击者可能会通过更改网络聊天实例的用户 ID 来模拟其他某人。

    若要防止这种情况，请使用不可猜出的用户 ID。 在 Direct Line 通道中启用增强的身份验证选项时，Azure 机器人服务可以检测并拒绝任何用户 ID 更改。 从 Direct Line 发送到机器人的消息中的用户 ID 始终与初始化网络聊天时所用的用户 ID 相同。 请注意，此功能要求用户 ID 以 `dl_` 开头。

1. 确保正确的用户已登录。 该用户具有两个标识：通道中的标识，以及在标识提供者中的标识。 在网络聊天中，Azure 机器人服务可以保证登录过程在网络聊天本身所在的同一个浏览器会话中完成。

    若要启用此项保护，请使用一个 Direct Line 令牌启动网络聊天，该令牌包含可以托管机器人网络聊天客户端的受信任域的列表。 然后，在 Direct Line 配置页中以静态方式指定受信任域（来源）列表。

使用 Direct Line 的 `/v3/directline/tokens/generate` REST 终结点生成聊天令牌，并在请求有效负载中指定用户 ID。 有关代码示例，请参阅 [Enhanced Direct Line Authentication Features](https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/)（增强的 Direct Line 身份验证功能）博客文章。

<!-- The eventual article about this should talk about the tokens/generate endpoint and its parameters: user, trustedOrigins, and [maybe] eTag.
Sample payload
{
  "user": {
    "id": "string",
    "name": "string",
    "aadObjectId": "string",
    "role": "string"
  },
  "trustedOrigins": [
    "string"
  ],
  "eTag": "string"
}
 -->

## <a name="prerequisites"></a>先决条件

- 具备[机器人基础知识][concept-basics]和[管理状态][concept-state]方面的知识。
- 具备 Azure 和 OAuth 2.0 开发方面的知识。
- Visual Studio 2017 或更高版本、Node.js、NPM 和 Git。
- 以下示例之一。

| 示例 | BotBuilder 版本 | 演示 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] 或 [**JavaScript**][js-auth-sample] 中的**机器人身份验证** | v4 | OAuthCard 支持 |
| [**CSharp**][cs-msgraph-sample] 或 [**JavaScript**][js-msgraph-sample] 中的**机器人身份验证 MSGraph** | v4 |  使用 OAuth 2 的 Microsoft Graph API 支持 |

## <a name="create-your-bot-resource-on-azure"></a>在 Azure 中创建机器人资源

使用 [Azure 门户](https://portal.azure.com/)创建“机器人通道注册”。

## <a name="create-and-register-an-azure-ad-application"></a>创建并注册 Azure AD 应用程序

需要提供一个 Azure AD 应用程序让机器人用来连接到 Microsoft Graph API。

对于这种机器人，可使用 Azure AD v1 或 v2 终结点。
有关 v1 和 v2 终结点之间差异的信息，请参阅 [v1-v2 比较](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)和 [Azure AD v2.0 终结点概述](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)。

### <a name="create-your-azure-ad-application"></a>创建 Azure AD 应用程序

使用这些步骤创建新的 Azure AD 应用程序。 可在创建的应用中使用 v1 或 v2 终结点。

> [!TIP]
> 需要在你对其拥有管理权限的租户中创建并注册 Azure AD 应用程序。

1. 在 Azure 门户中打开 [Azure Active Directory][azure-aad-blade] 面板。
    如果进入了错误的租户，请单击“切换目录”切换到正确的租户。 （有关创建租户的说明，请参阅[访问门户并创建租户](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)。）
1. 打开“应用注册”面板。
1. 在“应用注册”面板中，单击“新建应用程序注册”。
1. 填写必填字段并创建应用注册。

   1. 命名应用程序。
   1. 将“应用程序类型”设置为“Web 应用/API”。
   1. 将“登录 URL”设置为 `https://token.botframework.com/.auth/web/redirect`。
   1. 单击“创建”。

      - 创建完成后，它会显示在“已注册的应用”窗格中。
      - 记录“应用程序 ID”值。 稍后在将 Azure AD 应用程序注册到机器人时，需将此值用作“客户端 ID”。

1. 单击“设置”，配置应用程序。
1. 单击“密钥”打开“密钥”面板。

   1. 在“密码”下，创建 `BotLogin` 密钥。
   1. 将“持续时间”设置为“永不过期”。
   1. 单击“保存”并记录密钥值。 稍后在将 Azure AD 应用程序注册到机器人时，需将此值用作“客户端机密”。
   1. 关闭“密钥”面板。

1. 单击“所需权限”打开“所需权限”面板。

   1. 单击“添加”。
   1. 单击“选择 API”，然后选择“Microsoft Graph”并单击“选择”。
   1. 单击“选择权限”。 选择应用程序要使用的委托权限。

      > [!NOTE]
      > 标记为“需要管理员”的任何权限都需要用户和租户管理员登录，因此对于机器人来说，请尽量避免这些操作。

      选择以下 Microsoft Graph 委托权限：
      - 读取所有用户的基本配置文件
      - 读取用户邮件
      - 以用户身份发送邮件
      - 登录并读取用户配置文件
      - 查看用户的基本配置文件
      - 查看用户的电子邮件地址

   1. 依次单击“选择”、“完成”。
   1. 关闭“所需权限”面板。

现在已完成 Azure AD v1 应用程序的配置。

### <a name="register-your-azure-ad-application-with-your-bot"></a>在机器人中注册 Azure AD 应用程序

下一步是在机器人中注册创建的 Azure AD 应用程序。

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. 在 [Azure 门户](http://portal.azure.com/)中，导航到机器人的资源页。
1. 单击“设置”。
1. 在页面底部附近的“OAuth 连接设置”下，单击“添加设置”。
1. 按如下所示填写表单：

    1. 对于“名称”，输入连接的名称。 稍后将在机器人代码中使用此名称。
    1. 对于“服务提供程序”，选择“Azure Active Directory”。 选择该选项后，将显示特定于 Azure AD 的字段。
    1. 对于“客户端 ID”，输入为 Azure AD v1 应用程序记录的应用程序 ID。
    1. 对于“客户端密码”，输入为应用程序的 `BotLogin` 密钥记录的密钥。
    1. 对于“授权类型”，输入 `authorization_code`。
    1. 对于“登录 URL”，输入 `https://login.microsoftonline.com`。
    1. 对于“租户 ID”，输入 Azure Active Directory 的租户 ID，例如 `microsoft.com` 或 `common`。

       此租户将与可进行身份验证的用户相关联。 要允许任何人通过机器人对自己进行身份验证，请使用 `common` 租户。

    1. 对于“资源 URL”，输入 `https://graph.microsoft.com/`。
    1. 将“作用域”留空。

1. 单击“ **保存**”。

> [!NOTE]
> 这些值让应用程序可以通过 Microsoft Graph API 访问 Office 365 数据。

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. 在 [Azure 门户](http://portal.azure.com/)上导航到机器人的“机器人通道注册”页。
1. 单击“设置”。
1. 在页面底部附近的“OAuth 连接设置”下，单击“添加设置”。
1. 按如下所示填写表单：

    1. 对于“名称”，输入连接的名称。 在机器人代码中会用到。
    1. 对于“服务提供程序”，选择“Azure Active Directory v2”。 选择该选项后，将显示特定于 Azure AD 的字段。
    1. 对于“客户端 ID”，输入来自应用程序注册的 Azure AD v2 应用程序 ID。
    1. 对于“客户端密码”，输入来自应用程序注册的 Azure AD v2 应用程序密码。
    1. 对于“租户 ID”，输入 Azure Active Directory 的租户 ID，例如 `microsoft.com` 或 `common`。

        此租户将与可进行身份验证的用户相关联。 要允许任何人通过机器人对自己进行身份验证，请使用 `common` 租户。

    1. 对于“作用域”，输入从应用程序注册中选择的权限的名称：`Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`。

        > [!NOTE]
        > 对于 Azure AD v2，“作用域”值区分大小写，以空格分隔。

1. 单击“ **保存**”。

> [!NOTE]
> 这些值让应用程序可以通过 Microsoft Graph API 访问 Office 365 数据。

---

### <a name="test-your-connection"></a>测试连接

1. 单击连接项打开刚刚创建的连接。
1. 单击“服务提供程序连接设置”窗格顶部的“测试连接”。
1. 第一次进行此操作时，应该会打开一个新的浏览器选项卡（其中列出了应用请求的权限）并提示你接受。
1. 单击“接受”。
1. 然后，应会重定向到“\<连接名称> 的连接测试成功”页。

现在可以在机器人代码中使用此连接名称检索用户令牌。

## <a name="prepare-the-bot-sample-code"></a>准备机器人示例代码

你将使用 C# 或 Node，具体取决于所选示例。

| 示例 | BotBuilder 版本 | 演示 |
|:---|:---:|:---|
| [**CSharp**][cs-auth-sample] 或 [**JavaScript**][js-auth-sample] 中的**机器人身份验证** | v4 | OAuthCard 支持 |
| [**CSharp**][cs-msgraph-sample] 或 [**JavaScript**][js-msgraph-sample] 中的**机器人身份验证 MSGraph** | v4 |  使用 OAuth 2 的 Microsoft Graph API 支持 |

1. 单击上面的某个示例链接，克隆 github 存储库。
1. 按照 GitHub 自述文件页上的说明操作，此页介绍了如何运行该特定的机器人（C# 或 Node）。
1. 如果使用 C# Bot-Authentication 示例，请执行以下操作：

    1. 将 `AuthenticationBot.cs` 文件中的 `ConnectionName` 变量设置为配置机器人的 OAuth 2.0 连接设置时使用过的值。
    1. 将 `BotConfiguration.bot` 文件中的 `appId` 值设置为机器人的应用 ID。
    1. 将 `BotConfiguration.bot` 文件中的 `appPassword` 值设置为机器人的机密。

1. 如果使用 Node/JS Bot-Authentication 示例，请执行以下操作：

    1. 将 `bot.js` 文件中的 `CONNECTION_NAME` 变量设置为配置机器人的 OAuth 2.0 连接设置时使用过的值。
    1. 将 `bot-authentication.bot` 文件中的 `appId` 值设置为机器人的应用 ID。
    1. 将 `bot-authentication.bot` 文件中的 `appPassword` 值设置为机器人的机密。

    > [!IMPORTANT]
    > 根据密码中的字符，可能需要 XML 转义密码。 例如，任何连字符 (&) 都需要编码为 `&amp;`。

    ```json
    {
        "name": "BotAuthentication",
        "secretKey": "",
        "services": [
            {
            "appId": "",
            "id": "http://localhost:3978/api/messages",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "BotAuthentication"
            }
        ]
    }
    ```

如果不知道如何获取 **Microsoft 应用 ID** 值和 **Microsoft 应用密码**值，可以根据

[机器人通道注册密码](../bot-service-quickstart-registration.md#bot-channels-registration-password)中的说明创建新密码，
  
也可以根据 [Find Your Azure Bot’s AppID and AppSecret](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret)（查找 Azure 机器人的 AppID 和 AppSecret）中的说明，从部署中检索通过“机器人通道注册”预配的 **Microsoft 应用 ID** 和 **Microsoft 应用密码**

    > [!NOTE]
    > You could now publish this bot code to your Azure subscription (right-click on the project and choose **Publish**), but it is not necessary for this tutorial. You would need to set up a publishing configuration that uses the application and hosting plan that you used when configuration the bot in the Azure Portal.

## <a name="use-the-emulator-to-test-your-bot"></a>使用模拟器测试机器人

在本地测试机器人需要安装[机器人模拟器](https://github.com/Microsoft/BotFramework-Emulator)。 可以使用 v3 或 v4 模拟器。

1. 启动机器人（无论有无调试）。
1. 请注意页面的 localhost 端口号。 与机器人交互时将需要此信息。
1. 启动模拟器。
1. 连接到机器人。 确保机器人配置在使用身份验证时使用“Microsoft 应用 ID”和“Microsoft 应用密码”
1. 确保在模拟器设置中勾选“使用登录验证代码作为 OAuthCard”并启用“ngrok”，以便 Azure 机器人服务能够在令牌可用时将其返回给模拟器。

   如果尚未配置连接，请提供地址和机器人的 Microsoft 应用 ID 和密码。 将 `/api/messages` 添加到机器人的 URL。 URL 应类似于 `http://localhost:portNumber/api/messages`。

1. 键入 `help` 查看机器人的可用命令列表，然后测试身份验证功能。
1. 登录后，在注销前都无需再次提供凭据。
1. 要注销并取消身份验证，请键入 `signout`。

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> 机器人身份验证需要使用 Bot Connector 服务。 该服务会访问机器人的机器人通道注册信息，因此你需要在门户上设置机器人的消息传递终结点。 身份验证还需要使用 HTTPS，因此你需要为本地运行的机器人创建 HTTPS 转发地址。

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>有关令牌检索流的说明

如果用户要求机器人执行某项操作，而该操作要求机器人让用户登录，则机器人可以使用 `OAuthPrompt` 来开始检索给定连接的令牌。 `OAuthPrompt` 创建一个令牌检索流，其中包含：

1. 查看 Azure 机器人服务是否已经有一个适用于当前用户和连接的令牌。 如果有令牌，则会返回令牌。
1. 如果 Azure 机器人服务没有缓存的令牌，则会创建 `OAuthCard`，这是一个可供用户单击的登录按钮。
1. 用户单击 `OAuthCard` 登录按钮以后，Azure 机器人服务会向机器人直接发送用户的令牌，或者会向用户提供一个可在聊天窗口中输入的 6 位数身份验证代码。
1. 如果 Azure 机器人服务向用户提供身份验证代码，则机器人会用此身份验证代码来交换获取用户的令牌。

接下来的几个代码片段取自 `OAuthPrompt`，后者演示这些步骤如何在提示中运行。

### <a name="check-for-a-cached-token"></a>检查缓存的令牌

在此代码中，机器人首先会进行快速检查，确定 Azure 机器人服务是否已有用户（由当前活动发件人标识）和给定 ConnectionName（配置中使用的连接名）的令牌。 Azure 机器人服务要么已缓存令牌，要么没有缓存。 调用 GetUserTokenAsync 即可执行此快速检查。 如果 Azure 机器人服务有令牌并将其返回，则可以立即使用该令牌。 如果 Azure 机器人服务没有令牌，则此方法将返回 NULL。 在这种情况下，机器人可以发送自定义的 OAuthCard 供用户登录。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// First ask Bot Service if it already has a token for this user
var token = await adapter.GetUserTokenAsync(turnContext, connectionName, null, cancellationToken).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
public async getUserToken(context: TurnContext, code?: string): Promise<TokenResponse|undefined> {
    // Get the token and call validator
    const adapter: any = context.adapter as any; // cast to BotFrameworkAdapter
    return await adapter.getUserToken(context, this.settings.connectionName, code);
}
```

---

### <a name="send-an-oauthcard-to-the-user"></a>向用户发送 OAuthCard

你可以使用想要的任何文本或按钮文本自定义 OAuthCard。 重要部分：

- 将 `ContentType` 设置为 `OAuthCard.ContentType`。
- 将 `ConnectionName` 属性置为要使用的连接的名称。
- 在一个按钮中加入 `CardAction` `Type` 的 `ActionTypes.Signin`；注意，无需为登录链接指定任何值。

在此调用结束时，机器人需要“等待令牌”返回。 这种等待发生在主活动流上，因为用户登录时可能需要执行很多操作。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task SendOAuthCardAsync(ITurnContext turnContext, IMessageActivity message, CancellationToken cancellationToken = default(CancellationToken))
{
    if (message.Attachments == null)
    {
        message.Attachments = new List<Attachment>();
    }

    message.Attachments.Add(new Attachment
    {
        ContentType = OAuthCard.ContentType,
        Content = new OAuthCard
        {
            Text = "Please sign in",
            ConnectionName = connectionName,
            Buttons = new[]
            {
                new CardAction
                {
                    Title = "Sign In",
                    Text = "Sign In",
                    Type = ActionTypes.Signin,
                },
            },
        },
    });

    await turnContext.SendActivityAsync(message, cancellationToken).ConfigureAwait(false);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
private async sendOAuthCardAsync(context: TurnContext, prompt?: string|Partial<Activity>): Promise<void> {
    // Initialize outgoing message
    const msg: Partial<Activity> =
        typeof prompt === 'object' ? {...prompt} : MessageFactory.text(prompt, undefined, InputHints.ExpectingInput);
    if (!Array.isArray(msg.attachments)) { msg.attachments = []; }

    const cards: Attachment[] = msg.attachments.filter((a: Attachment) => a.contentType === CardFactory.contentTypes.oauthCard);
    if (cards.length === 0) {
        // Append oauth card
        msg.attachments.push(CardFactory.oauthCard(
            this.settings.connectionName,
            this.settings.title,
            this.settings.text
        ));
    }

    // Send prompt
    await context.sendActivity(msg);
}
```

---

### <a name="wait-for-a-tokenresponseevent"></a>等待 TokenResponseEvent

此代码中的机器人在等待 `TokenResponseEvent`（请参阅下文，详细了解如何路由到对话框堆栈）。 `WaitForToken` 方法首先确定是否已发送此事件。 如果已发送，那么它就可以供机器人使用。 如果没有发送，则 `RecognizeTokenAsync` 方法会接受发送到机器人的任何文本并将其传递给 `GetUserTokenAsync`。 原因是某些客户端（如 WebChat）不需要幻码验证码，可以直接在 `TokenResponseEvent` 中发送令牌。 其他客户端（如 Facebook 或 Slack）仍然需要幻码。 Azure 机器人服务将为这些客户端提供一个六位数的幻码，并要求用户在聊天窗口中键入此代码。 虽然不理想，但这是“回退”行为，因此如果 `RecognizeTokenAsync` 收到代码，机器人可以将此代码发送到 Azure 机器人服务并取回一个令牌。 如果此调用也失败了，则可以决定报告错误或执行其他操作。 但大多数情况下，机器人此时将有一个用户令牌。

如果查看每个示例的机器人代码，则会看到 `Event` 和 `Invoke` 活动也被路由到对话框堆栈。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// This can be called when the bot receives an Activity after sending an OAuthCard
private async Task<TokenResponse> RecognizeTokenAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (IsTokenResponseEvent(turnContext))
    {
        // The bot received the token directly
        var tokenResponseObject = turnContext.Activity.Value as JObject;
        var token = tokenResponseObject?.ToObject<TokenResponse>();
        return token;
    }
    else if (IsTeamsVerificationInvoke(turnContext))
    {
        var magicCodeObject = turnContext.Activity.Value as JObject;
        var magicCode = magicCodeObject.GetValue("state")?.ToString();

        var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, magicCode, cancellationToken).ConfigureAwait(false);
        return token;
    }
    else if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // make sure it's a 6-digit code
        var matched = _magicCodeRegex.Match(turnContext.Activity.Text);
        if (matched.Success)
        {
            var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, matched.Value, cancellationToken).ConfigureAwait(false);
            return token;
        }
    }

    return null;
}

private bool IsTokenResponseEvent(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Event && activity.Name == "tokens/response";
}

private bool IsTeamsVerificationInvoke(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Invoke && activity.Name == "signin/verifyState";
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
private async recognizeToken(context: TurnContext): Promise<PromptRecognizerResult<TokenResponse>> {
    let token: TokenResponse|undefined;
    if (this.isTokenResponseEvent(context)) {
        token = context.activity.value as TokenResponse;
    } else if (this.isTeamsVerificationInvoke(context)) {
        const code: any = context.activity.value.state;
        await context.sendActivity({ type: 'invokeResponse', value: { status: 200 }});
        token = await this.getUserToken(context, code);
    } else if (context.activity.type === ActivityTypes.Message) {
        const matched: RegExpExecArray = /(\d{6})/.exec(context.activity.text);
        if (matched && matched.length > 1) {
            token = await this.getUserToken(context, matched[1]);
        }
    }

    return token !== undefined ? { succeeded: true, value: token } : { succeeded: false };
}

private isTokenResponseEvent(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Event && activity.name === 'tokens/response';
}

private isTeamsVerificationInvoke(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Invoke && activity.name === 'signin/verifyState';
}
```

---

### <a name="message-controller"></a>消息控制器

在之后调用机器人时，注意该示例机器人绝不会缓存令牌。 这是因为机器人始终可以向 Azure 机器人服务询问令牌。 这样机器人就无需管理令牌生存期、刷新令牌等，因为 Azure 机器人服务会为你完成所有这些操作。

### <a name="further-reading"></a>延伸阅读

- [其他 Bot Framework 资源](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)包含其他支持资源的链接。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) 存储库包含 Bot Builder SDK 相关的存储库、示例、工具和规范的详细信息。

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample
