---
title: 使用 Bot Builder SDK for Python 创建机器人 | Microsoft Docs
description: 使用 Bot Builder SDK for Python 快速创建机器人。
keywords: Bot Builder SDK, 创建机器人, 快速入门, python, 入门
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 08/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4536bef820bc1e6e21ba2905fb643fe5608b3788
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999014"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>使用 Bot Builder SDK for Python 创建机器人

>[!NOTE] 
> Python SDK 处于**预览**状态，请访问 Python [GitHub 存储库](https://github.com/Microsoft/botbuilder-python)以获取更多信息。 

本快速入门将指导你构建机器人，然后使用 Bot Framework Emulator 对其进行测试。 

## <a name="pre-requisite"></a>先决条件
- [Python 3.6.4](https://www.python.org/downloads/) 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

## <a name="create-a-bot"></a>创建机器人
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

## <a name="start-the-emulator-and-connect-your-bot"></a>启动模拟器并连接机器人

接下来，启动模拟器，然后在模拟器中连接到机器人：

1. 单击模拟器“欢迎”选项卡中的“打开机器人”链接。 
2. 选择创建项目时所在目录中的 .bot 文件。

## <a name="interact-with-your-bot"></a>与机器人交互

向机器人发送消息，机器人将回复消息。
![模拟器运行](../media/emulator-v4/emulator-running.png)


## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [机器人概念](../v4sdk/bot-builder-basics.md)
