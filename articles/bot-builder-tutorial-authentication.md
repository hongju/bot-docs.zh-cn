---
title: 通过 Azure 机器人服务向机器人添加身份验证 | Microsoft Docs
description: 了解如何使用 Azure 机器人服务身份验证功能向机器人添加 SSO。
author: JonathanFingold
ms.author: JonathanFingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ROBOTS: NOINDEX
ms.date: 10/04/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d1b14d405b4df19db81269fc1f588305840485bd
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405962"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>通过 Azure 机器人服务向机器人添加身份验证

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]  

本教程使用 Azure 机器人服务中的新机器人身份验证功能，提供各种功能以帮助轻松开发用于向 Azure AD (Azure Active Directory)、GitHub、Uber 等各种标识提供者验证用户身份的机器人。 这些更新还采取了一些措施，通过免去某些客户端的“幻码验证”来改善用户体验  。

在此之前，机器人需要添加 OAuth 控制器和登录链接、存储目标客户端 ID 和密码，以及执行用户令牌管理。
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

现在，机器人开发人员不再需要托管 OAuth 控制器或管理令牌生存期，因为现在 Azure 机器人服务就可以处理这些事项。

具体功能包括：

- 改进了通道，支持新的身份验证功能（如新的 WebChat 和 DirectLineJS 库），无需 6 位数幻码验证。
- 改进了 Azure 门户，可添加、删除和配置各种 OAuth 标识提供者的连接设置。
- 支持各种现成的标识提供者，包括 Azure AD（v1 和 v2 终结点）、GitHub 等。
- 更新了 C# 和 Node.js Bot Framework SDK，使之可检索令牌、创建 OAuthCard 并处理 TokenResponse 事件。
- 提供用于向 Azure AD（v1 和 v2 终结点）以及 GitHub 进行身份验证的机器人的开发方法示例。

你可以根据本文中的步骤进行推断，将此类功能添加到现有机器人。 以下是演示新身份验证功能的示例机器人

| 示例 | BotBuilder 版本 | 说明 |
|:---|:---:|:---|
| [AadV1Bot](https://aka.ms/AadV1Bot) | v3 | 演示 v3 C# SDK 中的 OAuthCard 支持（使用 Azure AD v1 终结点） |
| [AadV2Bot](https://aka.ms/AadV2Bot) | v3 |  演示 v3 C# SDK 中的 OAuthCard 支持（使用 Azure AD v2 终结点） |
| [GitHubBot](https://aka.ms/GitHubBot) | v3 |  演示 v3 C# SDK 中的 OAuthCard 支持（使用 GitHub） |
| [BasicOAuth](https://aka.ms/BasicOAuth) | v3 |  演示 v3 C# SDK 中的 OAuth 2.0 支持 |

> [!NOTE]
> 身份验证功能还适用于使用 BotBuilder v3 的 Node.js。 但本文只介绍了 C# 代码示例。

有关其他信息和支持，请参阅 [Bot Framework 的其他资源](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)。

## <a name="overview"></a>概述

本教程创建一个机器人示例，它使用 Azure AD v1 或 v2 令牌连接到 Microsoft Graph。 <!--verify this info and fix wording--> 在此过程中，你将使用 GitHub 存储库中的代码。本教程介绍如何进行此方面的设置，包括机器人应用程序的设置。

- [创建机器人和身份验证应用程序](#create-your-bot-and-an-authentication-application)
- [准备机器人示例代码](#prepare-the-bot-sample-code)
- [使用模拟器测试机器人](#use-the-emulator-to-test-your-bot)

要完成这些步骤，需要安装 Visual Studio 2017、npm、节点和 git。 还应该熟悉 Azure、OAuth 2.0 和机器人开发。

完成操作后，可获得一个机器人，它可以对 Azure AD 应用程序响应某些简单任务，例如检查和发送电子邮件，或者显示你是谁以及你的经理是谁。 为此，机器人需要对 Microsoft.Graph 库使用 Azure AD 应用程序中的令牌。

最后一节会详细列出部分机器人代码

- [有关令牌检索流的说明](#notes-on-the-token-retrieval-flow)

## <a name="create-your-bot-and-an-authentication-application"></a>创建机器人和身份验证应用程序

你需要创建一个注册机器人，以便在其中将消息传送终结点设置为已部署机器人的代码，还需要创建 Azure AD（v1 或 v2）应用程序，以便机器人访问 Office 365。

> [!NOTE]
> 这些身份验证功能适用于其他类型的机器人。 但本教程仅使用注册机器人。

### <a name="register-an-application-in-azure-ad"></a>在 Azure AD 中注册应用程序

需要提供一个 Azure AD 应用程序让机器人用来连接到 Microsoft Graph API。

对于这种机器人，可使用 Azure AD v1 或 v2 终结点。
有关 v1 和 v2 终结点之间差异的信息，请参阅 [v1-v2 比较](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)和 [Azure AD v2.0 终结点概述](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)。

#### <a name="to-create-an-azure-ad-application"></a>创建 Azure AD 应用程序

使用这些步骤创建新的 Azure AD 应用程序。 可在创建的应用中使用 v1 或 v2 终结点。

> [!TIP]
> 你需要在租户中创建和注册 Azure AD 应用程序，在该租户中，你可以同意委托应用程序请求的权限。

1. 在 Microsoft Azure 门户中打开 [Azure Active Directory][azure-aad-blade] 面板。
    如果进入了错误的租户，请单击“切换目录”切换到正确的租户。  （有关创建租户的说明，请参阅[访问门户并创建租户](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)。）
1. 打开“应用注册”面板。 
1. 在“应用注册”面板中，单击“新建注册”   。
1. 填写必填字段并创建应用注册。

   1. 命名应用程序。
   1. 为应用程序选择“支持的帐户类型”。  （这些选项中的任何一个都可以用于此示例。）
   1. 对于“重定向 URI” 
       1. 选择“Web”。 
       1. 将 URL 设置为 `https://token.botframework.com/.auth/web/redirect`。
   1. 单击“注册”  。

      - 创建应用以后，Azure 会显示应用的“概览”页。 
      - 记录“应用程序(客户端) ID”值  。 稍后在将 Azure AD 应用程序注册到机器人时，需将此值用作“客户端 ID”。 
      - 另请记录“目录(租户) ID”值  。 也可以用它将此应用程序注册到机器人。

1. 在导航窗格中单击“证书和机密”，为应用程序创建机密  。

   1. 在“客户端机密”下，单击“新建客户端机密”。  
   1. 添加一项说明，用于将此机密与可能需要为此应用创建的其他机密（例如 `bot login`）区别开来。
   1. 将“过期”设置为“永不”。  
   1. 单击“添加”  。
   1. 在离开此页面之前，记录该机密。 稍后在将 Azure AD 应用程序注册到机器人时，需将此值用作“客户端机密”。 

1. 在导航窗格中，单击“API 权限”打开“API 权限”面板   。 最佳做法是为应用显式设置 API 权限。

   1. 单击“添加权限”，显示“请求 API 权限”窗格。  
   1. 对于此示例，请选择“Microsoft API”和“Microsoft Graph”。  
   1. 选择“委托的权限”，确保选中所需权限。  本示例需要这些权限。

      > [!NOTE]
      > 标记为“需要管理员同意”的任何权限都需要用户和租户管理员登录，因此对于机器人来说，请尽量避免这些操作  。

      - **openid**
      - **profile**
      - **Mail.Read**
      - **Mail.Send**
      - **User.Read**
      - **User.ReadBasic.All**

   1. 单击“添加权限”。  （用户首次通过机器人访问此应用时，需授予许可。）

现在已完成 Azure AD 应用程序的配置。

### <a name="create-your-bot-on-azure"></a>在 Azure 上创建机器人

使用 [Azure 门户](https://portal.azure.com/)创建“机器人通道注册”  。

### <a name="register-your-azure-ad-application-with-your-bot"></a>在机器人中注册 Azure AD 应用程序

下一步是在机器人中注册刚创建的 Azure AD 应用程序。

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. 在 [Azure 门户](http://portal.azure.com/)中，导航到机器人的资源页。
1. 单击“设置”  。
1. 在页面底部附近的“OAuth 连接设置”下，单击“添加设置”   。
1. 按如下所示填写表单：

    1. 对于“名称”，输入连接的名称  。 稍后将在机器人代码中使用此名称。
    1. 对于“服务提供程序”，选择“Azure Active Directory”   。 选择该选项后，将显示特定于 Azure AD 的字段。
    1. 对于“客户端 ID”，请输入为 Azure AD v1 应用程序记录的应用程序（客户端）ID  。
    1. 对于“客户端机密”，请输入所创建的机密，以便为机器人授予访问 Azure AD 应用的权限。 
    1. 对于“授权类型”，输入 `authorization_code`  。
    1. 对于“登录 URL”，输入 `https://login.microsoftonline.com`  。
    1. 对于“租户 ID”，请输入此前为 Azure AD 应用记录的目录（租户）ID。 

       此租户将与可进行身份验证的用户相关联。

    1. 对于“资源 URL”  ，输入 `https://graph.microsoft.com/`。
    1. 将“作用域”留空  。

1. 单击“ **保存**”。

> [!NOTE]
> 这些值让应用程序可以通过 Microsoft Graph API 访问 Office 365 数据。

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. 在 [Azure 门户](http://portal.azure.com/)上导航到机器人的“机器人通道注册”页。
1. 单击“设置”  。
1. 在页面底部附近的“OAuth 连接设置”下，单击“添加设置”   。
1. 按如下所示填写表单：

    1. 对于“名称”，输入连接的名称  。 在机器人代码中会用到。
    1. 对于“服务提供程序”，选择“Azure Active Directory v2”   。 选择该选项后，将显示特定于 Azure AD 的字段。
    1. 对于“客户端 ID”，请输入为 Azure AD v1 应用程序记录的应用程序（客户端）ID  。
    1. 对于“客户端机密”，请输入所创建的机密，以便为机器人授予访问 Azure AD 应用的权限。 
    1. 对于“租户 ID”，请输入此前为 Azure AD 应用记录的目录（租户）ID。 

       此租户将与可进行身份验证的用户相关联。

    1. 对于“作用域”，输入从应用程序注册中选择的权限的名称：`Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`  。

        > [!NOTE]
        > 对于 Azure AD v2，“作用域”值区分大小写，以空格分隔  。

1. 单击“ **保存**”。

> [!NOTE]
> 这些值让应用程序可以通过 Microsoft Graph API 访问 Office 365 数据。

---

#### <a name="to-test-your-connection"></a>测试连接

1. 打开刚才创建的连接。
1. 单击“服务提供程序连接设置”窗格顶部的“测试连接”   。
1. 第一次进行此操作时，应该会打开一个新的浏览器选项卡（其中列出了应用请求的权限）并提示你接受。
1. 单击“接受”  。
1. 然后，此操作应该会将你重定向到“‘<your-connection-name>’的连接测试成功”页面  。

## <a name="prepare-the-bot-sample-code"></a>准备机器人示例代码

1. 前往 https://github.com/Microsoft/BotBuilder 克隆 github 存储库。
1. 打开并生成解决方案 `BotBuilder\CSharp\Microsoft.Bot.Builder.sln`。
1. 关闭该解决方案并打开 `BotBuilder\CSharp\Samples\Microsoft.Bot.Builder.Samples.sln`。
1. 设置启动项目。
    - 对于使用 v1 Azure AD 应用程序的机器人，请使用 `Microsoft.Bot.Sample.AadV1Bot` 项目。
    - 对于使用 v2 Azure AD 应用程序的机器人，请使用 `Microsoft.Bot.Sample.AadV2Bot` 项目。
1. 打开 `Web.config` 文件，按如下所示修改应用设置：
    1. 将 `ConnectionName` 设置为配置机器人的 OAuth 2.0 连接设置时使用的值。
    1. 将 `MicrosoftAppId` 值设置为机器人的应用 ID。
    1. 将 `MicosoftAppPassword` 值设置为机器人的密码。

    > [!IMPORTANT]
    > 根据密码中的字符，可能需要 XML 转义密码。 例如，任何连字符 (&) 都需要编码为 `&amp;`。

    ```xml
    <appSettings>
        <add key="ConnectionName" value="<your-AAD-connection-name>"/>
        <add key="MicrosoftAppId" value="<your-bot-appId>" />
        <add key="MicrosoftAppPassword" value="<your-bot-password>" />
    </appSettings>
    ```

    如果不知道如何获取 **Microsoft 应用 ID** 值和 **Microsoft 应用密码**值，可以根据[机器人通道注册密码](bot-service-quickstart-registration.md#bot-channels-registration-password)中的说明创建新密码，也可以根据 [Find Your Azure Bot’s AppID and AppSecret](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret)（查找 Azure 机器人的 AppID 和 AppSecret）中的说明从部署中检索通过“机器人通道注册”  预配的 **Microsoft 应用 ID** 和 **Microsoft 应用密码**

    > [!NOTE]
    > 现在即可将此机器人代码发布到 Azure 订阅（右键单击该项目并选择“发布”），但本教程不需要这样做  。 你需要设置一个发布配置，该配置使用在 Azure 门户中配置机器人时使用的应用程序和托管计划。

## <a name="use-the-emulator-to-test-your-bot"></a>使用模拟器测试机器人

在本地测试机器人需要安装[机器人模拟器](https://github.com/Microsoft/BotFramework-Emulator)。 可以使用 v3 或 v4 模拟器。

1. 启动机器人（无论有无调试）。
1. 请注意页面的 localhost 端口号。 与机器人交互时将需要此信息。
1. 启动模拟器。
1. 连接到机器人。

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

当用户要求机器人执行某些需要让用户登录的操作时，机器人可以使用 `Microsoft.Bot.Builder.Dialogs.GetTokenDialog` 来启动检索给定连接的令牌。 接下来的几个代码片段取自 `GetTokenDialog` 类。

### <a name="check-for-a-cached-token"></a>检查缓存的令牌

在此代码中，机器人首先会进行快速检查，确定 Azure 机器人服务是否已有用户（由当前活动发件人标识）和给定 ConnectionName（配置中使用的连接名）的令牌。 Azure 机器人服务要么已缓存令牌，要么没有缓存。 对 GetUserTokenAsync 的调用会执行此“快速检查”操作。 如果 Azure 机器人服务有令牌并将其返回，则可以立即使用该令牌。 如果 Azure 机器人服务没有令牌，则此方法将返回 NULL。 在这种情况下，机器人可以发送自定义的 OAuthCard 供用户登录。

```csharp
// First ask Bot Service if it already has a token for this user
var token = await context.GetUserTokenAsync(ConnectionName).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
    await SendOAuthCardAsync(context, (Activity)context.Activity);
}
```

### <a name="send-an-oauthcard-to-the-user"></a>向用户发送 OAuthCard

你可以使用想要的任何文本或按钮文本自定义 OAuthCard。 重要部分：

- 将 `ContentType` 设置为 `OAuthCard.ContentType`。
- 将 `ConnectionName` 属性置为要使用的连接的名称。
- 在一个按钮中加入 `CardAction` `Type` 的 `ActionTypes.Signin`；注意，无需为登录链接指定任何值。

在此调用结束时，机器人需要“等待令牌”返回。 这种等待发生在主活动流上，因为用户登录时可能需要执行很多操作。

```csharp
private async Task SendOAuthCardAsync(IDialogContext context, Activity activity)
{
    await context.PostAsync($"To do this, you'll first need to sign in.");

    var reply = await context.Activity.CreateOAuthReplyAsync(_connectionName, _signInMessage, _buttonLabel).ConfigureAwait(false);
    await context.PostAsync(reply);

    context.Wait(WaitForToken);
}
```

### <a name="wait-for-a-tokenresponseevent"></a>等待 TokenResponseEvent

在此代码中，机器人的 dialog 类在等待 `TokenResponseEvent`（请参阅下文，详细了解如何路由到 Dialog 堆栈）。 `WaitForToken` 方法首先确定是否已发送此事件。 如果已发送，那么它就可以供机器人使用。 如果没有发送，则 `WaitForToken` 方法会接受发送到机器人的任何文本并将其传递给 `GetUserTokenAsync`。 原因是某些客户端（如 WebChat）不需要幻码验证码，可以直接在 `TokenResponseEvent` 中发送令牌。 其他客户端（如 Facebook 或 Slack）仍然需要幻码。 Azure 机器人服务将为这些客户端提供一个六位数的幻码，并要求用户在聊天窗口中键入此代码。 虽然不理想，但这是“回退”行为，因此如果 `WaitForToken` 收到代码，机器人可以将此代码发送到 Azure 机器人服务并取回一个令牌。 如果此调用也失败了，则可以决定报告错误或执行其他操作。 但大多数情况下，机器人此时将有一个用户令牌。

如果查看 MessageController.cs 文件，将看到此类型的 `Event` 活动也被路由到对话堆栈  。

```csharp
private async Task WaitForToken(IDialogContext context, IAwaitable<object> result)
{
    var activity = await result as Activity;

    var tokenResponse = activity.ReadTokenResponseContent();
    if (tokenResponse != null)
    {
        // Use the token to do exciting things!
    }
    else
    {
        if (!string.IsNullOrEmpty(activity.Text))
        {
            tokenResponse = await context.GetUserTokenAsync(ConnectionName,
                                                               activity.Text);
            if (tokenResponse != null)
            {
                // Use the token to do exciting things!
                return;
            }
        }
        await context.PostAsync($"Hmm. Something went wrong. Let's try again.");
        await SendOAuthCardAsync(context, activity);
    }
}
```

### <a name="message-controller"></a>消息控制器

在之后调用机器人时，注意该示例机器人绝不会缓存令牌。 这是因为机器人始终可以向 Azure 机器人服务询问令牌。 这样机器人就无需管理令牌生存期、刷新令牌等，因为 Azure 机器人服务会为你完成所有这些操作。

```csharp
else if(message.Type == ActivityTypes.Event)
{
    if(message.IsTokenResponseEvent())
    {
        await Conversation.SendAsync(message, () => new Dialogs.RootDialog());
    }
}
```
## <a name="additional-resources"></a>其他资源
[Bot Framework SDK](https://github.com/microsoft/botbuilder)
