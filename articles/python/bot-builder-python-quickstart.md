---
title: 使用 Bot Builder SDK for Python 创建机器人 | Microsoft Docs
description: 使用 Bot Builder SDK for Python 快速创建机器人。
keywords: Bot Builder SDK, 创建机器人, 快速入门, python, 入门
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6458bac5140fae14e8925e7af37aa8ac4ef1f1f5
ms.sourcegitcommit: 7b5675bbf7f1c2432bfc831ee5d627f6e5659e01
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/01/2018
ms.locfileid: "43380993"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>使用 Bot Builder SDK for Python 创建机器人
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Bot Builder SDK for Python 是一个易于使用的框架，用于开发机器人。 本快速入门将指导你构建机器人，然后使用 Bot Framework Emulator 对其进行测试。 SDK v4 处于预览状态，请访问 Python [GitHub 存储库](https://github.com/Microsoft/botbuilder-python)以获取更多信息。 

## <a name="pre-requisite"></a>先决条件
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

# <a name="create-a-bot"></a>创建机器人
在 main.py 文件中，导入以下标准模块：

```python
import http.server
import json
import asyncio
```

以及以下 SDK 模块：
```python
from botbuilder.schema import (Activity, ActivityTypes)
from botframework.connector import ConnectorClient
from botframework.connector.auth import (MicrosoftAppCredentials,
                                         JwtTokenValidation, SimpleCredentialProvider)
```
接下来，添加以下代码以使用 ConnectorClient 创建机器人：
```python
APP_ID = ''
APP_PASSWORD = ''


class BotRequestHandler(http.server.BaseHTTPRequestHandler):

    @staticmethod
    def __create_reply_activity(request_activity, text):
        return Activity(
            type=ActivityTypes.message,
            channel_id=request_activity.channel_id,
            conversation=request_activity.conversation,
            recipient=request_activity.from_property,
            from_property=request_activity.recipient,
            text=text,
            service_url=request_activity.service_url)

    def __handle_conversation_update_activity(self, activity):
        self.send_response(202)
        self.end_headers()
        if activity.members_added[0].id != activity.recipient.id:
            credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
            reply = BotRequestHandler.__create_reply_activity(activity, 'Hello and welcome to the echo bot!')
            connector = ConnectorClient(credentials, base_url=reply.service_url)
            connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_message_activity(self, activity):
        self.send_response(200)
        self.end_headers()
        credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
        connector = ConnectorClient(credentials, base_url=activity.service_url)
        reply = BotRequestHandler.__create_reply_activity(activity, 'You said: %s' % activity.text)
        connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_authentication(self, activity):
        credential_provider = SimpleCredentialProvider(APP_ID, APP_PASSWORD)
        loop = asyncio.new_event_loop()
        try:
            loop.run_until_complete(JwtTokenValidation.authenticate_request(
                activity, self.headers.get("Authorization"), credential_provider))
            return True
        except Exception as ex:
            self.send_response(401, ex)
            self.end_headers()
            return False
        finally:
            loop.close()

    def __unhandled_activity(self):
        self.send_response(404)
        self.end_headers()

    def do_POST(self):
        body = self.rfile.read(int(self.headers['Content-Length']))
        data = json.loads(str(body, 'utf-8'))
        activity = Activity.deserialize(data)

        if not self.__handle_authentication(activity):
            return

        if activity.type == ActivityTypes.conversation_update.value:
            self.__handle_conversation_update_activity(activity)
        elif activity.type == ActivityTypes.message.value:
            self.__handle_message_activity(activity)
        else:
            self.__unhandled_activity()


try:
    SERVER = http.server.HTTPServer(('localhost', 9000), BotRequestHandler)
    print('Started http server on localhost:9000')
    SERVER.serve_forever()
except KeyboardInterrupt:
    print('^C received, shutting down server')
    SERVER.socket.close()
```


保存 main.py。 若要在 Windows 上运行该示例，请在命令行窗口中输入以下内容：
```
python main.py
```
在本地终端中，应看到消息“已在 localhost:9000 上启动了 http 服务器”

### <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

接下来，启动模拟器，然后在模拟器中连接到机器人：


1. 单击模拟器“欢迎使用”选项卡中的“新建机器人配置”链接。 

2. 输入“机器人名称”并输入机器人代码的目录路径。 机器人配置文件将保存到此路径。

3. 在“终结点 URL”字段中键入 `http://localhost:port-number/api/messages`，其中 port-number 与运行应用程序的浏览器中显示的端口号相匹配。

4. 单击“连接”以连接到机器人。 无需指定“Microsoft 应用 ID”和“Microsoft 应用密码”。 可以暂时将这些字段留空。 以后当你注册机器人时，将获得此信息。

在模拟器中输入 **Hello**，机器人将回显**你说了“Hello”**。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [基本机器人概念](../v4sdk/bot-builder-basics.md)
