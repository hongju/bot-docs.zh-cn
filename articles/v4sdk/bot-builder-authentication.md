---
title: 通过 Azure 机器人服务向机器人添加身份验证 | Microsoft Docs
description: 了解如何使用 Azure 机器人服务身份验证功能向机器人添加 SSO。
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 06/07/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e708f6b556c832ed7f8858a893cc5fb0a8406ab2
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404323"
---
<!-- Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
-->

<!-- General TODO: (Feedback from CSE (Nafis))
- Add note that: token management is based on user ID
- Explain why/how to share existing website authentication with a bot.
- Risk: Even people who use a DirectLine token can be vulnerable to user ID impersonation.
    Docs/samples that show exchange of secret for token don't specify a user ID, so an attacker can impersonate a different user by modifying the ID client side. There's a [blog post](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fblog.botframework.com%2F2018%2F09%2F01%2Fusing-webchat-with-azure-bot-services-authentication%2F&data=02%7C01%7Cv-jofing%40microsoft.com%7Cee005e1c9d2c4f4e7ea508d6b231b422%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636892323874079713&sdata=b0DWMxHzmwQvg5EJtlqKFDzR7fYKmg10fXma%2B8zGqEI%3D&reserved=0) that shows how to do this properly.
"Major issues":
- This doc is a sample walkthrough, but there's no deeper documentation explaining how the Azure Bot Service is handling tokens. How does the OAuth flow work? Where is it storing my users' access tokens? What's the security and best practices around using it?

"Minor issues":
- AAD v2 steps tell you to add delegated permission scopes during registration, but this shouldn't be necessary in AAD v2 due to dynamic scopes. (Ming, "This is currently necessary because scopes are not exposed through our runtime API. We don’t currently have a way for the developer to specify which scope he wants at runtime.")

- "The scope of the connection setting needs to have both openid and a resource in the Azure AD graph, such as Mail.Read." Unclear if I need to take some action at this point to make happen. Kind of out of context. I'm registering an AAD application in the portal, there's no connection setting
- Does the bot need all of these scopes for the samples? (e.g. "Read all users' basic profiles")
-->

# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>通过 Azure 机器人服务向机器人添加身份验证

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Azure 机器人服务和 v4 SDK 包含新的机器人身份验证功能，并提供各种功能来帮助轻松开发用于向 Azure AD (Azure Active Directory)、GitHub、Uber 等各种标识提供者验证用户身份的机器人。 这些功能可通过免去某些客户端的幻码验证来改善用户体验  。

在此之前，机器人需要添加 OAuth 控制器和登录链接、存储目标客户端 ID 和密码，以及执行用户令牌管理。 机器人将要求用户在网站上登录，然后生成一个幻码，供用户用来验证其身份  。

现在，机器人开发人员不再需要托管 OAuth 控制器或管理令牌生存期，因为现在 Azure 机器人服务就可以处理这些事项。

具体功能包括：

- 改进了通道，支持新的身份验证功能（如新的 WebChat 和 DirectLineJS 库），无需 6 位数幻码验证。
- 改进了 Azure 门户，可添加、删除和配置各种 OAuth 标识提供者的连接设置。
- 支持各种现成的标识提供者，包括 Azure AD（v1 和 v2 终结点）、GitHub 等。
- 更新了 C# 和 Node.js Bot Framework SDK，使之可检索令牌、创建 OAuthCard 并处理 TokenResponse 事件。
- 提供用于向 Azure AD 进行身份验证的机器人的开发方法示例。

若要详细了解 Azure 机器人服务如何处理身份验证，请参阅 [User authentication within a conversation](bot-builder-concept-authentication.md)（聊天中的用户身份验证）。

你可以根据本文中的步骤进行推断，将此类功能添加到现有机器人。 这些示例机器人演示了新的身份验证功能。

> [!NOTE]
> 身份验证功能也适用于 BotBuilder v3。 但本文只介绍 v4 示例代码。

### <a name="about-this-sample"></a>关于此示例

需要创建 Azure 机器人资源，并且需要：

1. 一项 Azure AD 应用注册，以便机器人能够访问外部资源，例如 Office 365。
1. 一项单独的机器人资源。 该机器人资源将注册机器人的凭据；即使在本地运行机器人代码，也需要使用这些凭据来测试身份验证功能。

> [!IMPORTANT]
> 每当在 Azure 中注册机器人时，系统都会为它分配一个 Azure AD 应用。 但是，此应用为通道到机器人的访问提供保护。 对于你希望机器人能够代表用户验证的每个应用程序，还需要额外分配一个 AAD 应用。

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

## <a name="prerequisites"></a>先决条件

- 了解[机器人基础知识][concept-basics]、, [managing state][concept-state]、[对话库][概念对话]、如何[实现有序对话流][simple-dialog]、, and how to [reuse dialogs][component-dialogs]。
- 具备 Azure 和 OAuth 2.0 开发方面的知识。
- Visual Studio 2017 或更高版本、Node.js、NPM 和 Git。
- 以下示例之一。

| 示例 | BotBuilder 版本 | 演示 |
|:---|:---:|:---|
|  [**CSharp**][cs-auth-sample] or [**JavaScript**][js-auth-sample] 中的机器人身份验证 | v4 | OAuthCard 支持 |
|  [**CSharp**][cs-msgraph-sample] or [**JavaScript**][js-msgraph-sample] 中的机器人身份验证 MSGraph | v4 |  使用 OAuth 2 的 Microsoft Graph API 支持 |

## <a name="create-your-bot-resource-on-azure"></a>在 Azure 中创建机器人资源

使用 [Azure 门户](https://portal.azure.com/)创建“机器人通道注册”  。

记录机器人的应用 ID 和密码。 若要收集该信息，请参阅[管理机器人](../bot-service-manage-overview.md)。

## <a name="create-and-register-an-azure-ad-application"></a>创建并注册 Azure AD 应用程序

需要提供一个 Azure AD 应用程序让机器人用来连接到 Microsoft Graph API。

对于这种机器人，可使用 Azure AD v1 或 v2 终结点。
有关 v1 和 v2 终结点之间差异的信息，请参阅 [v1-v2 比较](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare)和 [Azure AD v2.0 终结点概述](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview)。

### <a name="create-your-azure-ad-application"></a>创建 Azure AD 应用程序

使用这些步骤创建新的 Azure AD 应用程序。 可在创建的应用中使用 v1 或 v2 终结点。

> [!TIP]
> 你需要在租户中创建和注册 Azure AD 应用程序，在租户中你可以同意应用程序请求的委托权限。

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

### <a name="test-your-connection"></a>测试连接

1. 单击连接项打开刚刚创建的连接。
1. 单击“服务提供程序连接设置”窗格顶部的“测试连接”   。
1. 第一次进行此操作时，应该会打开一个新的浏览器选项卡（其中列出了应用请求的权限）并提示你接受。
1. 单击“接受”  。
1. 然后，应会重定向到“\<连接名称> 的连接测试成功”页  。

现在可以在机器人代码中使用此连接名称检索用户令牌。

## <a name="prepare-the-bot-code"></a>准备机器人代码

需要机器人的应用 ID 和密码才能完成此过程。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. 从 github 存储库克隆要使用的示例：[机器人身份验证][cs-auth-sample]or [**Bot authentication MSGraph**][cs-msgraph-sample]  。
1. 更新 **appsettings.json**：

    - 将 `ConnectionName` 设置为要添加到机器人的 OAuth 连接设置的名称。
    - 将 `MicrosoftAppId` 和 `MicrosoftAppPassword` 设置为机器人的应用 ID 和应用机密。

      根据机器人机密中的字符，可能需要 XML 转义密码。 例如，任何连字符 (&) 都需要编码为 `&amp;`。

    [!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. 从要使用的 github 存储库进行克隆：[机器人身份验证][js-auth-sample]or [**Bot authentication MSGraph**][js-msgraph-sample]  。
1. 更新 **.env**：

    - 将 `connectionName` 设置为要添加到机器人的 OAuth 连接设置的名称。
    - 将 `MicrosoftAppId` 和 `MicrosoftAppPassword` 值设置为机器人的应用 ID 和应用机密。

      根据机器人机密中的字符，可能需要 XML 转义密码。 例如，任何连字符 (&) 都需要编码为 `&amp;`。

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

---

如果不知道如何获取 **Microsoft 应用 ID** 值和 **Microsoft 应用密码**值，可以[按照此处的说明](../bot-service-quickstart-registration.md#bot-channels-registration-password)创建新密码

> [!NOTE]
> 现在即可将此机器人代码发布到 Azure 订阅（右键单击该项目并选择“发布”），但本文不需要这样做  。 你需要设置一个发布配置，该配置使用在 Azure 门户中配置机器人时使用的应用程序和托管计划。

## <a name="test-the-bot"></a>测试机器人

1. 安装 [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme)（如果尚未安装）。
1. 在计算机本地运行示例。
1. 启动模拟器，连接到机器人，然后发送消息。

    - 在连接到机器人时，需提供机器人的应用 ID 和密码。

        - 如果以前需要对机器人代码中的密码进行 XML 转义，则还需在此处进行该操作。

    - 键入 `help` 查看机器人的可用命令列表，然后测试身份验证功能。
    - 登录后，在注销前都无需再次提供凭据。
    - 要注销并取消身份验证，请键入 `logout`。

> [!NOTE]
> 机器人身份验证需要使用 Bot Connector 服务。 该服务会访问机器人的机器人通道注册信息。

# <a name="bot-authenticationtabbot-oauth"></a>[机器人身份验证](#tab/bot-oauth)

在**机器人身份验证**示例中，此对话被设计为在用户登录后检索用户令牌。

![示例输出](media/how-to-auth/auth-bot-test.png)

# <a name="bot-authentication-msgraphtabbot-msgraph-auth"></a>[机器人身份验证 MSGraph](#tab/bot-msgraph-auth)

在**机器人身份验证 MSGraph** 示例中，此对话被设计为在用户登录后接受有限的一组命令。

![示例输出](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>其他信息

如果用户要求机器人执行某项操作，而该操作要求机器人让用户登录，则机器人可以使用 `OAuthPrompt` 来开始检索给定连接的令牌。 `OAuthPrompt` 创建一个令牌检索流，其中包含：

1. 查看 Azure 机器人服务是否已经有一个适用于当前用户和连接的令牌。 如果有令牌，则会返回令牌。
1. 如果 Azure 机器人服务没有缓存的令牌，则会创建 `OAuthCard`，这是一个可供用户单击的登录按钮。
1. 用户单击 `OAuthCard` 登录按钮以后，Azure 机器人服务会向机器人直接发送用户的令牌，或者会向用户提供一个可在聊天窗口中输入的 6 位数身份验证代码。
1. 如果 Azure 机器人服务向用户提供身份验证代码，则机器人会用此身份验证代码来交换获取用户的令牌。

以下部分描述示例如何执行某些常见的身份验证任务。

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>使用 OAuth 提示来登录用户并获取令牌

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![机器人体系架构](media/how-to-auth/architecture.png)

<!-- The two authentication samples have nearly identical architecture. Using 18.bot-authentication for the sample code. -->

**Dialogs\MainDialog.cs**

将 OAuth 提示添加到其构造函数中的 **MainDialog**。 在这里，连接名称的值已从 **appsettings.json** 文件中检索。

[!code-csharp[Add OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=23-31)]

在对话步骤中，请使用 `BeginDialogAsync` 来启动 OAuth 提示，该提示要求用户登录。

- 如果用户已登录，则会在不提示用户的情况下生成一个令牌响应事件，
- 否则会提示用户登录。 Azure 机器人服务会在用户尝试登录后发送令牌响应事件。

[!code-csharp[Use the OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=49)]

在以下对话步骤中，检查上一步的结果中是否存在令牌。 如果该令牌不为 null，则表明用户已成功登录。

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-58)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![机器人体系架构](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

将 OAuth 提示添加到其构造函数中的 **MainDialog**。 在这里，连接名称的值已从 **.env** 文件中检索。

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=23-28)]

在对话步骤中，请使用 `beginDialog` 来启动 OAuth 提示，该提示要求用户登录。

- 如果用户已登录，则会在不提示用户的情况下生成一个令牌响应事件，
- 否则会提示用户登录。 Azure 机器人服务会在用户尝试登录后发送令牌响应事件。

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

在以下对话步骤中，检查上一步的结果中是否存在令牌。 如果该令牌不为 null，则表明用户已成功登录。

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=61-64)]

---

### <a name="wait-for-a-tokenresponseevent"></a>等待 TokenResponseEvent

启动 OAuth 提示时，它会等待令牌响应事件，目的是从中检索用户的令牌。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot** 派生自 `ActivityHandler`，会显式处理令牌响应事件活动。 在这里，我们会继续活动对话，这样 OAuth 提示就能处理事件并检索令牌。

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot** 派生自 `ActivityHandler`，会显式处理令牌响应事件活动。 在这里，我们会继续活动对话，这样 OAuth 提示就能处理事件并检索令牌。

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=28-33)]

---

### <a name="log-the-user-out"></a>注销用户

最佳做法是让用户显式注销，而不是等待连接超时。

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=20-61&highlight=35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=13-42&highlight=25)]

---

### <a name="further-reading"></a>延伸阅读

- [其他 Bot Framework 资源](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help)包含其他支持资源的链接。
- [Bot Framework SDK](https://github.com/microsoft/botbuilder) 存储库包含 Bot Builder SDK 相关的存储库、示例、工具和规范的详细信息。

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview

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
