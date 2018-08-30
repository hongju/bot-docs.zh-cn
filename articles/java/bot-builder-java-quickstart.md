---
title: 使用 Bot Builder SDK for Java 创建机器人 | Microsoft Docs
description: 使用 Bot Builder SDK for Java 快速创建机器人。
keywords: Bot Builder SDK, 创建机器人, 快速入门, java, 入门
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3b618bfb7cd1a462390aee4d564778c8ec0a7247
ms.sourcegitcommit: d486dd088b87a44fc8142f7a08877ff993861a42
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928426"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-java"></a>通过 Bot Builder SDK for Java 创建机器人
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot Builder SDK for Java 为 Java 开发人员提供了一种熟悉的方法来编写机器人。 SDK v4 处于预览状态，请访问 Java [GitHub 存储库](https://github.com/Microsoft/botbuilder-java)以获取更多信息。

> [!NOTE]
> 我们的代码示例和文档当前面向 Java 版本 1.8。

## <a name="getting-started"></a>入门

v4 SDK 包含一系列的[库](https://github.com/Microsoft/botbuilder-java/tree/master/libraries)。 若要在本地构建，请参阅[构建 SDK](https://github.com/Microsoft/botbuilder-java/wiki/building-the-sdk)。

- 安装 [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

### <a name="create-echobot"></a>创建 EchoBot

在 App.java 文件中，添加以下内容：

```Java
package com.microsoft.bot.connector.sample;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.microsoft.aad.adal4j.AuthenticationException;
import com.microsoft.bot.connector.customizations.CredentialProvider;
import com.microsoft.bot.connector.customizations.CredentialProviderImpl;
import com.microsoft.bot.connector.customizations.JwtTokenValidation;
import com.microsoft.bot.connector.customizations.MicrosoftAppCredentials;
import com.microsoft.bot.connector.implementation.ConnectorClientImpl;
import com.microsoft.bot.schema.models.Activity;
import com.microsoft.bot.schema.models.ActivityTypes;
import com.microsoft.bot.schema.models.ResourceResponse;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.URLDecoder;
import java.util.logging.Level;
import java.util.logging.Logger;

public class App {
    private static final Logger LOGGER = Logger.getLogger( App.class.getName() );
    private static String appId = "";       // <-- app id -->
    private static String appPassword = ""; // <-- app password -->

    public static void main( String[] args ) throws IOException {
        CredentialProvider credentialProvider = new CredentialProviderImpl(appId, appPassword);
        HttpServer server = HttpServer.create(new InetSocketAddress(3978), 0);
        server.createContext("/api/messages", new MessageHandle(credentialProvider));
        server.setExecutor(null);
        server.start();
    }

    static class MessageHandle implements HttpHandler {
        private ObjectMapper objectMapper;
        private CredentialProvider credentialProvider;
        private MicrosoftAppCredentials credentials;

        MessageHandle(CredentialProvider credentialProvider) {
            this.objectMapper = new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                    .findAndRegisterModules();
            this.credentialProvider = credentialProvider;
            this.credentials = new MicrosoftAppCredentials(appId, appPassword);
        }

        public void handle(HttpExchange httpExchange) throws IOException {
            if (httpExchange.getRequestMethod().equalsIgnoreCase("POST")) {
                Activity activity = getActivity(httpExchange);
                String authHeader = httpExchange.getRequestHeaders().getFirst("Authorization");
                try {
                    JwtTokenValidation.assertValidActivity(activity, authHeader, credentialProvider);

                    // send ack to user activity
                    httpExchange.sendResponseHeaders(202, 0);
                    httpExchange.getResponseBody().close();

                    if (activity.type().equals(ActivityTypes.MESSAGE)) {
                        // reply activity with the same text
                        ConnectorClientImpl connector = new ConnectorClientImpl(activity.serviceUrl(), this.credentials);
                        ResourceResponse response = connector.conversations().sendToConversation(activity.conversation().id(),
                                new Activity()
                                        .withType(ActivityTypes.MESSAGE)
                                        .withText("Echo: " + activity.text())
                                        .withRecipient(activity.from())
                                        .withFrom(activity.recipient())
                        );
                    }
                } catch (AuthenticationException ex) {
                    httpExchange.sendResponseHeaders(401, 0);
                    httpExchange.getResponseBody().close();
                    LOGGER.log(Level.WARNING, "Auth failed!", ex);
                } catch (Exception ex) {
                    LOGGER.log(Level.WARNING, "Execution failed", ex);
                }
            }
        }

        private String getRequestBody(HttpExchange httpExchange) throws IOException {
            StringBuilder buffer = new StringBuilder();
            InputStream stream = httpExchange.getRequestBody();
            int rByte;
            while ((rByte = stream.read()) != -1) {
                buffer.append((char)rByte);
            }
            stream.close();
            if (buffer.length() > 0) {
                return URLDecoder.decode(buffer.toString(), "UTF-8");
            }
            return "";
        }

        private Activity getActivity(HttpExchange httpExchange) {
            try {
                String body = getRequestBody(httpExchange);
                LOGGER.log(Level.INFO, body);
                return objectMapper.readValue(body, Activity.class);
            } catch (Exception ex) {
                LOGGER.log(Level.WARNING, "Failed to get activity", ex);
                return null;
            }

        }
    }
}
```

如果使用的是 Maven，则可以从此存储库中的示例文件夹中复制 pom.xml 文件。 开始运行可执行文件后，启动 Bot Framework Emulator。

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

此时，机器人在本地运行。
接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎使用”选项卡中的“新建机器人配置”链接。 

2. 输入“机器人名称”并输入机器人代码的目录路径。 机器人配置文件将保存到此路径。

3. 在“终结点 URL”字段中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。

4. 单击“连接”以连接到机器人。 无需指定“Microsoft 应用 ID”和“Microsoft 应用密码”。 可以暂时将这些字段留空。 以后当你注册机器人时，将获得此信息。

### <a name="interact-with-your-bot"></a>与机器人交互
发送“Hi”到机器人，机器人将回应消息。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [基本机器人概念](../v4sdk/bot-builder-basics.md)
