## <a name="application-settings-and-messaging-endpoint"></a>应用程序设置和消息传送终结点

### <a name="verify-application-settings"></a>验证应用程序设置

要使机器人在云环境中正常运行，必须确保其应用程序设置正确。 如果你有“appID”和“密码”，则在部署过程中更新应用程序配置设置中的 `Microsoft AppId` 和 `Microsoft App Password` 值。 要查找机器人的 AppID 和 AppPassword，请参阅 [MicrosoftAppID 和 MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword)。

> [!TIP]
> [!INCLUDE [Application configuration settings](~/includes/snippet-tip-bot-config-settings.md)]

如果尚未向 Bot Framework 注册机器人（因此还没有“appID”和“密码”），可以使用这些设置的临时占位符值部署机器人。
之后，在[注册机器人](~/bot-service-quickstart-registration.md)后，使用注册期间为机器人生成的“appID”和“密码”值更新已部署的应用程序设置。

### <a id="messagingEndpoint"></a> 验证消息传送终结点

部署的机器人必须具有消息传送终结点，用以从 Bot Framework Connector 服务接收消息。

> [!NOTE]
> 将机器人部署到 Azure 时，将为应用程序自动配置 SSL，从而启用 Bot Framework 需要的消息传送终结点。
> 如果部署到另一个云服务，请务必验证是否已针对 SSL 配置应用程序，以便机器人将具有消息传送终结点。
