**临时主动消息**是最简单的主动消息类型。
只要触发了消息，机器人就会简单地将消息插入到聊天中，而不考虑用户当前是否正在致力于与机器人进行的聊天的单独主题，并且不会尝试以任何方式更改聊天。

若要更顺畅地处理通知，请考虑将通知集成到聊天流中的其他方法，例如在聊天状态中设置标志或将通知添加到队列。

<!--Snip
A **dialog-based proactive message** is more complex than an ad hoc proactive message. 
Before it can inject this type of proactive message into the conversation, 
the bot must identify the context of the existing conversation and decide how (or if)
it will resume that conversation after the message interrupts. 

For example, consider a bot that needs to initiate a survey at a given point in time. 
When that time arrives, the bot stops the existing conversation with the user and 
redirects the user to a `SurveyDialog`. 
The `SurveyDialog` is added to the top of the dialog stack and takes control of the conversation. 
When the user finishes all required tasks at the `SurveyDialog`, the `SurveyDialog` closes,
 returning control to the previous dialog, where the user can continue with the prior topic of conversation.

A dialog-based proactive message is more than just simple notification. 
In sending the notification, the bot changes the topic of the existing conversation. 
It then must decide whether to resume that conversation later, or to abandon that conversation altogether by resetting the dialog stack. 
/Snip-->