{% import "views/_helper.njk" as docs %}

{{ docs.defaultLang('js') }}

{{ docs.useIMLangSpec()}}

# 2. Advanced Messaging Features, Push Notifications, Synchronization, and Multi Device Sign-on

## Introduction

In the previous chapter [Basic Conversations and Messages](realtime-guide-beginner.html), we introduced how you can create components in your app that support one-on-one chatting and group chats, as well as how you can handle events triggered by the cloud. In this chapter, we will show you how to implement advanced features like:

- Getting receipts when messages are delivered and read
- Mentioning people with "@"
- Recalling and editing messages
- Push notifications and message synchronization
- Single device or multi device sign-on
- Sending messages of your custom types

## Advanced Messaging Features

If you are building an app for team collaboration or social networking, you may want more features to be included beside basic messaging. For example:

- To mention someone with "@" so that they can easily find out what messages are important to them.
- To edit or recall a piece of message that has been sent out.
- To send status messages like "Someone is typing".
- To allow the sender of a message to know if it's delivered or read.
- To synchronize messages sent to a receiver that has been offline.

With LeanMessage, you can easily implement the functions mentioned above.

### Mentioning People

Some group chats have a lot of messages going on and people may easily overlook the information that's important to them. That's why we need a way for senders to get people's attention.

The most commonly used way to mention someone is to type "@ + name" when composing a message. But if we break it down, we'll notice that the "name" here is something determined by the app (it could be the real name or the nickname of the user) and could be totally different from the `clientId` identifying the user (since one is for people to see and one is for computers to read). A problem could be caused if someone changes their name at the moment another user sends a message with the old name mentioned. Beside this, we also need to consider the way to mention all the members in a conversation. Is it "@all"? "@group"? Or "@everyone"? Maybe all of them will be used, which totally depends on how the UI of the app is designed.

So we cannot mention people by simply adding "@ + name" into a message. To walk through that, LeanMessage allows you to specify two properties with `AVIMMessage`:

- `mentionList`, an array of strings containing the list of `clientId`s being mentioned;
- `mentionAll`, a `Bool` indicating whether all the members are mentioned.

Based on the logic of your app, it's possible for you to have both `mentionAll` to be set and `mentionList` to contain a list of members. Your app shall provide the UI that allows users to type in and select the members they want to mention. The only thing you need to do with the SDK is to call the setters of `mentionList` and `mentionAll` to set the members being mentioned. Here is a code example:

```js
const message = new TextMessage(`@Tom Come back early.`).setMentionList(['Tom']);
conversation.send(message).then(function(message) {
  console.log('Sent!');
}).catch(console.error);
```
```swift
do {
    let message = IMTextMessage(text: "@Tom Come back early.")
    message.mentionedMembers = ["Tom"]
    try conversation.send(message: message, completion: { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
AVIMMessage *message = [AVIMTextMessage messageWithText:@"@Tom Come back early." attributes:nil];
message.mentionList = @[@"Tom"];
[conversation sendMessage:message callback:^(BOOL succeeded, NSError * _Nullable error) {
    /* A message mentioning Tom has been sent */
}];
```
```java
String content = "@Tom Come back early.";
AVIMTextMessage  message = new AVIMTextMessage();
message.setText(content);
List<String> list = new ArrayList<>(); // A list for holding the members being mentioned; you can add members into the list with the code below
list.add("Tom");
message.setMentionList(list);
imConversation.sendMessage(message, new AVIMConversationCallback() {
   @Override
   public void done(AVIMException e) {
   }
});
```
```cs
var textMessage = new AVIMTextMessage("@Tom Come back early.")
{
    MentionList = new List<string>() { "Tom" }
};
await conversation.SendMessageAsync(textMessage);
```

You can also mention everyone by setting `mentionAll`:

```js
const message = new TextMessage(`@all`).mentionAll();
conversation.send(message).then(function(message) {
  console.log('Sent!');
}).catch(console.error);
```
```swift
do {
    let message = IMTextMessage(text: "@all")
    message.isAllMembersMentioned = true
    try conversation.send(message: message, completion: { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
AVIMMessage *message = [AVIMTextMessage messageWithText:@"@all" attributes:nil];
message.mentionAll = YES;
[conversation sendMessage:message callback:^(BOOL succeeded, NSError * _Nullable error) {
    /* A message mentioning everyone has been sent */
}];
```
```java
String content = "@all";
AVIMTextMessage  message = new AVIMTextMessage();
message.setText(content);

boolean mentionAll = true; // Indicates if everyone is mentioned
message.mentionAll(mentionAll);

imConversation.sendMessage(message, new AVIMConversationCallback() {
   @Override
   public void done(AVIMException e) {
   }
});
```
```cs
var textMessage = new AVIMTextMessage("@all")
{
    MentionAll = true
};
await conv.SendMessageAsync(textMessage);
```

The receiver of the message can call the getters of `mentionList` and `mentionAll` to see the members being mentioned:

```js
client.on(Event.MESSAGE, function messageEventHandler(message, conversation) {
  var mentionList = receivedMessage.getMentionList();
});
```
```swift
func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .message(event: let messageEvent):
        switch messageEvent {
        case .received(message: let message):
            if let mentionedMembers = message.mentionedMembers {
                print(mentionedMembers)
            }
            if let isAllMembersMentioned = message.isAllMembersMentioned {
                print(isAllMembersMentioned)
            }
        default:
            break
        }
    default:
        break
    }
}
```
```objc
// The code below shows how you can get a list of clientIds being mentioned in an AVIMTypedMessage; the code can be modified to serve other types inherited from AVIMMessage
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    // Get a list of clientIds being mentioned
    NSArray *mentionList = message.mentionList;
}
```
```java
@Override
public void onMessage(AVIMAudioMessage msg, AVIMConversation conv, AVIMClient client) {
  // Get a list of clientIds being mentioned
  List<String> currentMsgMentionUserList = message.getMentionList();
}
```
```cs
private void OnMessageReceived(object sender, AVIMMessageEventArgs e)
{
    if (e.Message is AVIMImageMessage imageMessage)
    {
        var mentionedList = e.Message.MentionList;
    }
}
```

To make it easier to display things on the UI, the following two flags are offered by `AVIMMessage` to indicate the status of mentioning:

- `mentionedAll`: Whether all the members in the conversation are mentioned. Becomes `true` only when `mentionAll` is true, otherwise it remains `false`.
- `mentioned`: Whether the current user is mentioned. Becomes `true` when `mentionList` contains the `clientId` of the current user or when `mentionAll` is `true`, otherwise it remains `false`.

Here is a code example:

```js
client.on(Event.MESSAGE, function messageEventHandler(message, conversation) {
  var mentionedAll = receivedMessage.mentionedAll;
  var mentionedMe = receivedMessage.mentioned;
});
```
```swift
func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .message(event: let messageEvent):
        switch messageEvent {
        case .received(message: let message):
            print(message.isCurrentClientMentioned)
        default:
            break
        }
    default:
        break
    }
}
```
```objc
// The code below shows how you can check if all the members in the conversation or the current user is mentioned in an AVIMTypedMessage; the code can be modified to serve other types inherited from AVIMMessage
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    // Check if all the members in the conversation are mentioned
    BOOL mentionAll = message.mentionAll;
    // Check if the current user is mentioned
    BOOL mentionedMe = message.mentioned;
}
```
```java
@Override
public void onMessage(AVIMAudioMessage msg, AVIMConversation conv, AVIMClient client) {
  // Check if all the members in the conversation are mentioned
  boolean currentMsgMentionAllUsers = message.isMentionAll();
  // Check if the current user is mentioned
  boolean currentMsgMentionedMe = message.mentioned();
}
```
```cs
private void OnMessageReceived(object sender, AVIMMessageEventArgs e)
{
    if (e.Message is AVIMImageMessage imageMessage)
    {
         var mentionedAll = e.Message.MentionAll;
         // Check if the current user is mentioned; requires additional comparison in .NET SDK
         var mentioned = e.Message.MentionAll || e.Message.MentionList.Contains("Tom");
    }
}
```

### Recalling and Editing Messages

> Before implementing these functions, go to your app's [Dashboard > Messaging > LeanMessage > Settings > LeanMessage Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/realtime/conf) and enable **Allow editing messages with SDK** and **Allow recalling messages with SDK**.

A user has the ability to edit or recall messages they have sent out with `Conversation#updateMessage` or `Conversation#recallMessage` methods. There are no limits on the time within which they can perform these operations. However, users are only allowed to edit or recall the messages they have sent out, not the ones sent by others.

The code below lets Tom **recall a message he has sent out**:

```js
conversation.recall(oldMessage).then(function(recalledMessage) {
  // The message is recalled
  // recalledMessage is a RecalledMessage
}).catch(function(error) {
  // Handle error
});
```
```swift
do {
    try conversation.recall(message: oldMessage, completion: { (result) in
        switch result {
        case .success(value: let recalledMessage):
            print(recalledMessage)
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
AVIMMessage *oldMessage = <#MessageYouWantToRecall#>;

[conversation recallMessage:oldMessage callback:^(BOOL succeeded, NSError * _Nullable error, AVIMRecalledMessage * _Nullable recalledMessage) {
    if (succeeded) {
        NSLog(@"The message is recalled.");
    }
}];
```
```java
conversation.recallMessage(message, new AVIMMessageRecalledCallback() {
    @Override
    public void done(AVIMRecalledMessage recalledMessage, AVException e) {
        if (null == e) {
            // The message is recalled; the UI may be updated now
        }
    }
});
```
```cs
await conversation.RecallAsync(message);
```

After Tom calls `recallMessage`, other members in the conversation will receive the `MESSAGE_RECALL` event:

```js
var { Event } = require('leancloud-realtime');
conversation.on(Event.MESSAGE_RECALL, function(recalledMessage, reason) {
  // recalledMessage is the message being recalled
  // Look for the original message with its ID and replace it with recalledMessage
  // reason (optional) is the reason the message is recalled; see the part for editing messages for more details
});
```
```swift
func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .message(event: let messageEvent):
        switch messageEvent {
        case let .updated(updatedMessage: updatedMessage, reason: _):
            if let recalledMessage = updatedMessage as? IMRecalledMessage {
                print(recalledMessage)
            }
        default:
            break
        }
    default:
        break
    }
}
```
```objc
/* A delegate method for handling events of editing and recalling messages */
- (void)conversation:(AVIMConversation *)conversation messageHasBeenUpdated:(AVIMMessage *)message {
    /* A message is updated or recalled */

    switch (message.mediaType) {
    case kAVIMMessageMediaTypeRecalled:
        NSLog(@"message is a message being recalled.");
        break;
    default:
        NSLog(@"message is a message being updated.");
        break;
    }
}
```
```java
void onMessageRecalled(AVIMClient client, AVIMConversation conversation, AVIMMessage message) {
  // message is the message being recalled
}
```
```cs
tom.OnMessageRecalled += Tom_OnMessageRecalled;
private void Tom_OnMessageRecalled(object sender, AVIMMessagePatchEventArgs e)
{
    // e.Messages contains the messages being edited; it is a collection of messages since the SDK may combine multiple operations into a single request
}
```

For Android and iOS SDKs, if caching is enabled (enabled by default), the SDKs will first delete the recalled message from the cache and then trigger an event to the app. This ensures the consistency of data internally. When you receive such event, simply refresh the chatting page to reflect the latest collection of messages. Based on your implementation, either the total amount of messages would become less or a message indicating message being recalled would be displayed.

Beside recalling the message, Tom can also **edit the message directly**. When this happens, what you would do is not to update the original message instance, but to create a new one and call `Conversation#updateMessage(oldMessage, newMessage)` to submit the request to the cloud. Here is a code example:

```js
var newMessage = new TextMessage('The new message.');
conversation.update(oldMessage, newMessage).then(function() {
  // The message is updated
}).catch(function(error) {
  // Handle error
});
```
```swift
do {
    let newMessage = IMTextMessage(text: "Just a new message")
    try conversation.update(oldMessage: oldMessage, to: newMessage, completion: { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
AVIMMessage *oldMessage = <#MessageYouWantToUpdate#>;
AVIMMessage *newMessage = [AVIMTextMessage messageWithText:@"The new message." attributes:nil];

[conversation updateMessage:oldMessage
              toNewMessage:newMessage
                  callback:^(BOOL succeeded, NSError * _Nullable error) {
                      if (succeeded) {
                          NSLog(@"The message is updated.");
                      }
}];
```
```java
AVIMTextMessage textMessage = new AVIMTextMessage();
textMessage.setContent("The new message.");
imConversation.updateMessage(oldMessage, textMessage, new AVIMMessageUpdatedCallback() {
  @Override
  public void done(AVIMMessage avimMessage, AVException e) {
    if (null == e) {
      // The message is updated; avimMessage is the updated message
    }
  }
});
```
```cs
var newMessage = new AVIMTextMessage("The new message.");
await conversation.UpdateAsync(oldMessage, newMessage);
```

After Tom updates the message, other members in the conversation will receive the `MESSAGE_UPDATE` event:

```js
var { Event } = require('leancloud-realtime');
conversation.on(Event.MESSAGE_UPDATE, function(newMessage, reason) {
  // newMessage is the message being updated
  // Look for the original message with its ID and replace it with newMessage
  // reason (optional) is the reason the message is edited
  // If reason is not specified, it means the sender edited the message
  // If the code of reason is positive, it means a hook on LeanEngine caused the edit
  // (the code value can be specified when defining hooks)
  // If the code of reason is negative, it means a built-in mechanism of the system caused the edit
  // For example, -4408 means the message is edited due to keyword filtering
  // The detail of reason is a string containing explanation
});
```
```swift
func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .message(event: let messageEvent):
        switch messageEvent {
        case let .updated(updatedMessage: updatedMessage, reason: _):
            print(updatedMessage)
        default:
            break
        }
    default:
        break
    }
}
```
```objc
/* A delegate method for handling events of editing and recalling messages */
- (void)conversation:(AVIMConversation *)conversation messageHasBeenUpdated:(AVIMMessage *)message {
    /* A message is updated or recalled */

    switch (message.mediaType) {
    case kAVIMMessageMediaTypeRecalled:
        NSLog(@"message is a message being recalled.");
        break;
    default:
        NSLog(@"message is a message being updated.");
        break;
    }
}
```
```java
void onMessageUpdated(AVIMClient client, AVIMConversation conversation, AVIMMessage message) {
  // message is the message being updated
}
```
```cs
tom.OnMessageUpdated += (sender, e) => {
  var message = (AVIMTextMessage) e.Message; // e.Messages contains the messages being updated; it is a collection of messages since the SDK may combine multiple operations into a single request
  Debug.Log(string.Format("Message with ID {1} is updated to {0}", message.TextContent, message.Id));
};
```

For Android and iOS SDKs, if caching is enabled (enabled by default), the SDKs will first update the edited message in the cache and then trigger an event to the app. When you receive such event, simply refresh the chatting page to reflect the latest collection of messages.

If a message is edited by the system (for example, due to keyword filtering or by a hook on LeanEngine), all the members in the conversation (including the sender) will receive a `MESSAGE_UPDATE` event.

### Transient Messages

Sometimes we need to send status updates like "Someone is typing…" or "Someone changed the group name to XX". Different from messages sent by users, these messages don't need to be stored to the history, nor do they need to be guaranteed to be delivered (if members are offline or there is a network error, it would be okay if these messages are not delivered). Such messages are best sent as transient messages.

Transient message is a special type of message. It has the following differences comparing to a basic message:

- It won't be stored to the cloud so it couldn't be retrieved from history messages.
- It's only delivered to those who are online. Offline members cannot receive it later or get push notifications about it.
- It's not guaranteed to be delivered. If there's a network error preventing the message from being delivered, the server won't make a second attempt.

Therefore, transient messages are best for communicating real-time updates of statuses that are changing frequently or implementing simple control protocols.

The way to construct a transient message is the same as that for a basic message. The only difference is the way it is being sent. So far we have shown the following way for sending messages with `AVIMConversation`:

```js
/**
 * Send a message
 * @param  {Message} message The message itself; an instance of Message or its subtype
 * @return {Promise.<Message>} The message being sent
 */
async send(message)
```
```swift
/// Send Message.
///
/// - Parameters:
///   - message: The message to be sent.
///   - options: @see `MessageSendOptions`.
///   - priority: @see `IMChatRoom.MessagePriority`.
///   - pushData: The push data of APNs.
///   - progress: The file uploading progress.
///   - completion: callback.
public func send(message: IMMessage, options: MessageSendOptions = .default, priority: IMChatRoom.MessagePriority? = nil, pushData: [String : Any]? = nil, progress: ((Double) -> Void)? = nil, completion: @escaping (LCBooleanResult) -> Void) throws
```
```objc
/*!
 Send a message
 */
- (void)sendMessage:(AVIMMessage *)message
           callback:(void (^)(BOOL succeeded, NSError * _Nullable error))callback;
```
```java
/**
 * Send a message
 */
public void sendMessage(AVIMMessage message, final AVIMConversationCallback callback)
```
```cs
public static Task<T> SendAsync<T>(this AVIMConversation conversation, T message)
            where T : IAVIMMessage
```

In fact, an additional parameter `AVIMMessageOption` can be provided when sending a message. Here is a complete list of interfaces offered by `AVIMConversation`:

```js
/**
 * Send a message
 * @param  {Message} message The message itself; an instance of Message or its subtype
 * @param {Object} [options] Options
 * @param {Boolean} [options.transient] Whether it is a transient message
 * @param {Boolean} [options.receipt] Whether receipts are needed
 * @param {Boolean} [options.will] Whether it is a will message
 * A will message will be sent when the user loses connection
 * @param {MessagePriority} [options.priority] The priority of the message; for chat rooms only
 * see: {@link module:leancloud-realtime.MessagePriority MessagePriority}
 * @param {Object} [options.pushData] The content for push notification; if the receiver is offline, a push notification with this content will be triggered
 * @return {Promise.<Message>} The message being sent
 */
async send(message, options)
```
```swift
/// Message Sending Option
public struct MessageSendOptions: OptionSet {
    /// Get Receipt when other client received message or read message.
    public static let needReceipt = MessageSendOptions(rawValue: 1 << 0)
    
    /// Indicates whether this message is transient.
    public static let isTransient = MessageSendOptions(rawValue: 1 << 1)
    
    /// Indicates whether this message will be auto delivering to other client when this client disconnected.
    public static let isAutoDeliveringWhenOffline = MessageSendOptions(rawValue: 1 << 2)
}

/// Send Message.
///
/// - Parameters:
///   - message: The message to be sent.
///   - options: @see `MessageSendOptions`.
///   - priority: @see `IMChatRoom.MessagePriority`.
///   - pushData: The push data of APNs.
///   - progress: The file uploading progress.
///   - completion: callback.
public func send(message: IMMessage, options: MessageSendOptions = .default, priority: IMChatRoom.MessagePriority? = nil, pushData: [String : Any]? = nil, progress: ((Double) -> Void)? = nil, completion: @escaping (LCBooleanResult) -> Void) throws
```
```objc
/*!
 Send a message
 @param message － The message itself
 @param option － Options
 @param callback － Callback
 */
- (void)sendMessage:(AVIMMessage *)message
             option:(nullable AVIMMessageOption *)option
           callback:(void (^)(BOOL succeeded, NSError * _Nullable error))callback;
```
```java
/**
 * Send a message
 * @param message
 * @param messageOption
 * @param callback
 */
public void sendMessage(final AVIMMessage message, final AVIMMessageOption messageOption, final AVIMConversationCallback callback)；
```
```cs
/// <summary>
/// Send a message
/// </summary>
/// <param name="avMessage">The message itself</param>
/// <param name="options">Options<see cref="AVIMSendOptions"/></param>
/// <returns></returns>
public Task<IAVIMMessage> SendMessageAsync(IAVIMMessage avMessage, AVIMSendOptions options);
```

With `AVIMMessageOption`, we can specify:

- Whether it is a transient message (field `transient`).
- Whether receipts are needed (field `receipt`; more details will be covered later).
- The priority of the message (field `priority`; more details will be covered later).
- Whether it is a will message (field `will`; more details will be covered later).
- The content for push notification (field `pushData`; more details will be covered later); if the receiver is offline, a push notification with this content will be triggered.

The code below sends a transient message saying "Tom is typing…" to the conversation when Tom's input box gets focused:

```js
const message = new TextMessage('Tom is typing…');
conversation.send(message, {transient: true});
```
```swift
do {
    let message = IMTextMessage(text: "Tom is typing…")
    try conversation.send(message: message, options: [.isTransient], completion: { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
AVIMMessage *message = [AVIMTextMessage messageWithText:@"Tom is typing…" attributes:nil];
AVIMMessageOption *option = [[AVIMMessageOption alloc] init];
option.transient = true;
[conversation sendMessage:message option:option callback:^(BOOL succeeded, NSError * _Nullable error) {
    /* A transient message is sent */
}];
```
```java
String content = "Tom is typing…";
AVIMTextMessage  message = new AVIMTextMessage();
message.setText(content);

AVIMMessageOption option = new AVIMMessageOption();
option.setTransient(true);

imConversation.sendMessage(message, option, new AVIMConversationCallback() {
   @Override
   public void done(AVIMException e) {
   }
});
```
```cs
var textMessage = new AVIMTextMessage("Tom is typing…")
{
    MentionAll = true
};
var option = new AVIMSendOptions(){Transient = true};
await conv.SendAsync(textMessage, option);
```

The procedure for receiving transient messages is also the same as that for basic messages. You can run different logic based on the types of messages. The example above sets the type of the message to be text message, but it would be better if you assign a distinct type to it. Our SDK doesn't offer a type for transient messages, so you may build your own depending on what you need. See [Custom Message Types](#custom-message-types) for more details.

### Receipts

When the cloud is delivering messages, it follows the sequence the messages are pushed to the cloud and delivers the former messages in advance of the latter ones (FIFO). Our internal protocol also requires that the SDK sends an acknowledgement (ack) back to the cloud for every single message received by it. If a message is received by the SDK but the ack is not received by the cloud due to a packet loss, the cloud would assume that the message is not successfully delivered and will keep redelivering it until an ack is received. Correspondingly, the SDK also does it work to make duplicate messages insensible by the app. The entire mechanism ensures that no message will be lost in the entire delivery process.

However, in certain scenarios, functionality beyond the one mentioned above is demanded. For example, a sender may want to know when the receiver got the message and when they opened the message. In a product for team collaboration or private communication, a sender may even want to monitor the real-time status of every message sent out by them. Such requirements can be satisfied with the help of receipts.

Similar to the way of sending transient messages, if you want receipts to be given back, you need to specify an option in `AVIMMessageOption`:

```js
var message = new TextMessage('A very important message.');
conversation.send(message, {
  receipt: true,
});
```
```swift
do {
    let message = IMTextMessage(text: "A very important message.")
    try conversation.send(message: message, options: [.needReceipt], completion: { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
[conversation sendMessage:message options:AVIMMessageSendOptionRequestReceipt callback:^(BOOL succeeded, NSError *error) {
  if (succeeded) {
    NSLog(@"Message sent with receipts requested.");
  }
}];
```
```java
AVIMMessageOption messageOption = new AVIMMessageOption();
messageOption.setReceipt(true);
imConversation.sendMessage(message, messageOption, new AVIMConversationCallback() {
   @Override
   public void done(AVIMException e) {
   }
});
```
```cs
var textMessage = new AVIMTextMessage("A very important message.");
var option = new AVIMSendOptions(){Receipt = true};
await conv.SendAsync(textMessage, option);
```

> Note:
>
> Receipts are not enabled by default. You need to manually turn that on when sending each message. Receipts are only available for conversations with no more than 2 members.

So how do senders handle the receipts they get?

#### Delivery Receipts

When a message is delivered to the receiver, the cloud will give a delivery receipt to the sender. **Keep in mind that this is not the same as a read receipt.**

```js
var { Event } = require('leancloud-realtime');
conversation.on(Event.LAST_DELIVERED_AT_UPDATE, function() {
  console.log(conversation.lastDeliveredAt);
  // Update the UI to mark all the messages before lastDeliveredAt to be "delivered"
});
```
```swift
func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .message(event: let messageEvent):
        switch messageEvent {
        case let .delivered(toClientID: toClientID, messageID: messageID, deliveredTimestamp: deliveredTimestamp):
            if messageID == message.ID {
                message.deliveredTimestamp = deliveredTimestamp
            }
        default:
            break
        }
    default:
        break
    }
}
```
```objc
// `conversation:messageDelivered` listens if there are messages delivered
- (void)conversation:(AVIMConversation *)conversation messageDelivered:(AVIMMessage *)message{
    NSLog(@"%@", @"Message delivered."); // Print log
}
```
```java
public class CustomConversationEventHandler extends AVIMConversationEventHandler {
  /**
   * Handle notifications for messages being delivered
   */
  public void onLastDeliveredAtUpdated(AVIMClient client, AVIMConversation conversation) {
    ;
  }
}

// Set up global event handler
AVIMMessageManager.setConversationEventHandler(new CustomConversationEventHandler());
```
```cs
// Tom creates an AVIMClient with his name as clientId
AVIMClient client = new AVIMClient("Tom");

// Tom logs in
await client.ConnectAsync();

// Enable delivery receipt
conversaion.OnMessageDeliverd += (s, e) =>
{
// Things to do after messages are delivered
};
// Send message
await conversaion.SendTextMessageAsync("Wanna go to bakery tonight?");
```

The content included in the receipt will not be a specific message. Instead, it will be the time the messages in the current conversation are last delivered (`lastDeliveredAt`). We have mentioned earlier that messages are delivered according to the sequence they are pushed to the cloud. Therefore, given the time of the last delivery, we can infer that all the messages sent before it are delivered. On the UI of the app, you can mark all the messages sent before `lastDeliveredAt` to be "delivered".

#### Read Receipts

When we say a message is delivered, what we mean is that the message is received by the client from the cloud. At this time, the actual user might not have the conversation page or even the app open (Android apps can receive messages in the background). So we cannot assume that a message is read just because it is delivered.

Therefore, we offer another kind of receipt showing if a receiver has actually seen a message.

Again, since messages are delivered in time order, we don't have to check if every single message is being read. Think about a scenario like this:

<img src="images/realtime_read_confirm.png" width="400" class="img-responsive" alt="A pop-up with the title &quot;Welcome Back&quot; says &quot;You have 5002 unread messages. Would you like to skip them all? (Select 'Yes' to mark them as read)&quot;. There are two buttons on the bottom: &quot;Yes&quot; and &quot;No&quot;.">

When a user opens a conversation, we can say that the user has read all the messages in it. You can use the following interface of `Conversation` to mark all the messages in it as read:

```js
/**
 * Mark as read
 * @return {Promise.<this>} self
 */
async read();
```
```swift
/// Clear unread messages that its sent timestamp less than the sent timestamp of the parameter message.
///
/// - Parameter message: The default is the last message.
public func read(message: IMMessage? = nil)
```
```objc
/*!
 Mark as read
 This method marks the latest message sent into a conversation by the other member as read. The sender of the message will get a read receipt.
 */
- (void)readInBackground;
```
```java
/**
 * Mark as read
 */
public void read();
```
```cs
// Not supported yet
```

After the receiver has read the latest messages, the sender will get a receipt indicating that the messages they have sent out are read.

So if Tom is chatting with Jerry and wants to know if Jerry has read the messages, the following procedure would apply:

1. Tom sends a message to Jerry and requests receipts on it:
  
    ```js
    var message = new TextMessage('A very important message.');
    conversation.send(message, {
      receipt: true,
    });
    ```
    ```swift
    do {
        let message = IMTextMessage(text: "Hello, Jerry!")
        try conversation.send(message: message, options: [.needReceipt], completion: { (result) in
            switch result {
            case .success:
                break
            case .failure(error: let error):
                print(error)
            }
        })
    } catch {
        print(error)
    }
    ```
    ```objc
    AVIMMessageOption *option = [[AVIMMessageOption alloc] init];
    option.receipt = YES; /* Request receipts */

    AVIMTextMessage *message = [AVIMTextMessage messageWithText:@"Hello, Jerry!" attributes:nil];

    [conversaiton sendMessage:message option:option callback:^(BOOL succeeded, NSError * _Nullable error) {
        if (!error) {
            /* Sent */
        }
    }];
    ```
    ```java
    AVIMClient tom = AVIMClient.getInstance("Tom");
    AVIMConversation conv = client.getConversation("551260efe4b01608686c3e0f");
    
    AVIMTextMessage textMessage = new AVIMTextMessage();
    textMessage.setText("Hello, Jerry!");

    AVIMMessageOption option = new AVIMMessageOption();
    option.setReceipt(true); /* Request receipts */

    conv.sendMessage(textMessage, option, new AVIMConversationCallback() {
      @Override
      public void done(AVIMException e) {
        if (e == null) {
          /* Sent */
        }
      }
    });
    ```
    ```cs
    // Not supported yet
    ```

2. Jerry reads Tom's message and call `read` on the conversation to mark the latest messages as read:
  
    ```js
    conversation.read().then(function(conversation) {
      ;
    }).catch(console.error.bind(console));
    ```
    ```swift
    conversation.read()
    ```
    ```objc
    [conversation readInBackground];
    ```
    ```java
    conversation.read();
    ```
    ```cs
    // Not supported yet
    ```

3. Tom gets a read receipt with the conversation's `lastReadAt` updated. The UI can be updated to mark all messages sent before `lastReadAt` to be read:
  
    ```js
    var { Event } = require('leancloud-realtime');
    conversation.on(Event.LAST_READ_AT_UPDATE, function() {
      console.log(conversation.lastReadAt);
      // Update the UI to mark all the messages before lastReadAt to be "read"
    });
    ```
    ```swift
    func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
        switch event {
        case .message(event: let messageEvent):
            switch messageEvent {
            case let .read(byClientID: byClientID, messageID: messageID, readTimestamp: readTimestamp):
                if messageID == message.ID {
                    message.readTimestamp = readTimestamp
                }
            default:
                break
            }
        default:
            break
        }
    }
    ```
    ```objc
    // Tom can get the update of lastReadAt within the delegate method of client
    - (void)conversation:(AVIMConversation *)conversation didUpdateForKey:(NSString *)key {
        if ([key isEqualToString:@"lastReadAt"]) {
            NSDate *lastReadAt = conversation.lastReadAt;
            /* The message is read by Jerry; lastReadAt can be used to update UI; for example, to mark all the messages before lastReadAt to be "read" */
        }
    }
    ```
    ```java
    public class CustomConversationEventHandler extends AVIMConversationEventHandler {
      /**
       * Handle notifications for messages being read
       */
      public void onLastReadAtUpdated(AVIMClient client, AVIMConversation conversation) {
        /* The message is read by Jerry; the time Jerry read messages for the last time can be retrieved by calling conversation.getLastReadAt() */
      }
    }

    // Set up global event handler
    AVIMMessageManager.setConversationEventHandler(new CustomConversationEventHandler());
    ```
    ```cs
    // Not supported yet
    ```

> Note:
>
> To use read receipts, turn on [notifications on updates of unread message count](#notifications-on-updates-of-unread-message-count) when initializing your app.

### Muting Conversations

If a user doesn't want to receive notifications from a conversation but still wants to stay in it, they can mute the conversation. See [Muting Conversations](realtime-guide-senior.html#muting-conversations) in the next chapter for more details.

### Will Messages

Will message can be used to automatically notify other members in a conversation when a user goes offline unexpectedly. It gets its name from the wills filed by testators, giving people a feeling that the last messages of a person should always be heard. It looks like the message saying "Tom is offline and cannot receive messages" in this image:

<img src="images/lastwill-message.png" width="400" class="img-responsive" alt="In a conversation named &quot;Tom & Jerry&quot;, Jerry receives a will message saying &quot;Tom is offline and cannot receive messages&quot;. The will message looks like a system notification and shares a different style with other messages.">

A will message needs to be composed ahead of time and cached on the cloud. The cloud doesn't send it out immediately after receiving it. Instead, it waits until the sender of it goes offline unexpectedly. You can implement your own logic to handle such event.

```js
var message = new TextMessage('I am a will message. I will be sent out to other members in the conversation when the sender goes offline unexpectedly.');
conversation.send(message, { will: true }).then(function() {
  // Sent; other members will see the message once the current client goes offline unexpectedly
}).catch(function(error) {
  // Handle error
});
```
```swift
do {
    let message = IMTextMessage(text: "I am a will message. I will be sent out to other members in the conversation when the sender goes offline unexpectedly.")
    try conversation.send(message: message, options: [.isAutoDeliveringWhenOffline], completion: { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    })
} catch {
    print(error)
}
```
```objc
AVIMMessageOption *option = [[AVIMMessageOption alloc] init];
option.will = YES;

AVIMMessage *willMessage = [AVIMTextMessage messageWithText:@"I am a will message. I will be sent out to other members in the conversation when the sender goes offline unexpectedly." attributes:nil];

[conversaiton sendMessage:willMessage option:option callback:^(BOOL succeeded, NSError * _Nullable error) {
    if (succeeded) {
        NSLog(@"Sent!");
    }
}];
```
```java
AVIMTextMessage message = new AVIMTextMessage();
message.setText("I am a will message. I will be sent out to other members in the conversation when the sender goes offline unexpectedly.");

AVIMMessageOption option = new AVIMMessageOption();
option.setWill(true);

conversation.sendMessage(message, option, new AVIMConversationCallback() {
  @Override
  public void done(AVIMException e) {
    if (e == null) {
      // Sent
    }
  }
});
```
```cs
var message = new AVIMTextMessage()
{
    TextContent = "I am a will message. I will be sent out to other members in the conversation when the sender goes offline unexpectedly."
};
var sendOptions = new AVIMSendOptions()
{
    Will = true
};
await conversation.SendAsync(message, sendOptions);
```

Once the sender goes offline unexpectedly, other members will immediately receive the will message. You can design your own way to display it on the UI.

Will message has the **following restrictions**:

- Each user can only have one will message set up at a time. This means that if a user sets will messages for multiple conversations or multiple will messages for the same conversation, only the last one will take effect.
- Will messages don't get stored to the history.
- If a user logs out proactively, the will message set by this user will not be sent out (if there is one).

### Keyword Filtering

If you app allows users to create group chats, you might consider filtering cuss words out from the messages sent by users. LeanMessage offers a built-in component that helps you easily implement such function. See [Keyword Filtering](realtime-guide-senior.html#keyword-filtering) in the next chapter for more details.

### Handling Undelivered Messages

Sometimes you may need to store the messages that are not successfully sent out into a local cache and handle them later. For example, if a client's connection to the server is lost and a message cannot be sent out due to this, you may still keep the message locally. Perhaps you can add an error icon and a button for retrying next to the message displayed on the UI. The user may tap on the button when the connection is recovered to make another attempt to send the message.

By default, both Android and iOS SDKs enable a local cache for storing messages. The cache stores all the messages that are already sent to the cloud and keeps itself updated with the data in the cloud. To make things easier, undelivered messages can also be stored in the same cache.

The code below adds a message to the cache:

```js
// Not supported yet
```
```swift
// Not supported yet
```
```objc
[conversation addMessageToCache:message];
```
```java
conversation.addToLocalCache(message);
```
```cs
// Not supported yet
```

The code below removes a message from the cache:

```js
// Not supported yet
```
```swift
// Not supported yet
```
```objc
[conversation removeMessageFromCache:message];
```
```java
conversation.removeFromLocalCache(message);
```
```cs
// Not supported yet
```

When reading messages from the cache, you can make messages look different on the UI based on the property `message.status`. If the `status` of a message is `AVIMMessageStatusFailed`, it means the message cannot be sent out, so you can add a button for retrying on the UI. An additional benefit for using the local cache is that the SDK will make sure the same message only gets sent out once. This ensures that there won't be any duplicate messages on the cloud.

## Push Notifications

If your users are using your app on mobile devices, they might close the app at anytime, which prevents you from delivering new messages to them in the ordinary way. At this time, using push notifications becomes a good alternative to get users notified when new messages are coming in.

If you are building an iOS or Android app, you can utilize the built-in push notification services offered by these operating systems, as long as you have your certificates configured for iOS or have the function enabled for Android. Check the following documentation for more details:

1. [Push Notification Overview](push_guide.html)
2. [Android Push Notification Guide](android_push_guide.html) / [iOS Push Notification Guide](ios_push_guide.html)

LeanCloud offers a complete set of push notification services that works perfectly with LeanMessage. When being implemented, it links the `clientId`s of users with the `_Installation` table of your app that keeps track of the device information. When a user sends a message to a conversation, the cloud will automatically convert the message to a push notification and send it to those who are offline but are using iOS devices or using Android devices with push notification services enabled. We also allow you to connect third-party push notification services to your app.

The highlight of this feature is that you can **customize the contents of push notifications**. You have the following three ways to specify the contents:

1. Setting up a static message

  You can fill in a global static JSON string in your app's [Dashboard > Messaging > LeanMessage > Settings > Push Notification Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/realtime/conf) for delivering push notifications with a static message. For example, if you put:

  ```json
  { "alert": "New message received", "badge": "Increment" }
  ```

  Then whenever there is a new message going to an offline user, the user will receive a push notification saying "New message received".

  Keep in mind that `badge` is for iOS devices only which means to increase the number displayed on the badge of the app. Its value `Increment` is case-sensitive. [iOS Push Notification Guide · Customizing Push Notifications](ios_push_guide.html#customizing-push-notifications) introduces how you can clear the badge. Besides, you can also customize the sounds of push notifications for iOS devices. See [Push Notification Guide · Message Contents](push_guide.html#message-contents) for more details.

2. Specifying contents when sending messages from a client

  When using the first way introduced above, the content included in each push notification is the same regardless of the message being sent out. Is it possible to dynamically generate these contents to make them relevant to the actual messages?

  Remember how we specified the `AVIMMessageOption` parameter when sending transient messages? The same parameter takes in a `pushData` property which allows you to specify the contents of push notifications. Here is a code example:

  ```js
  const message = new TextMessage('Hey Jerry, me and Kate are going to a bar to watch a game tonight. Do you wanna come?');
  conversation.send(message), {
      pushData: {
          "alert": "New message received",
          "category": "Message",
          "badge": 1,
          "sound": "message.mp3", // The name of the sound file; has to be present in the app
          "custom-key": "This is a custom attribute. custom-key is just an example. You can use your own names for keys."
      }
  });
  ```
  ```swift
  do {
      let message = IMTextMessage(text: "Hey Jerry, me and Kate are going to a bar to watch a game tonight. Do you wanna come?")
      let pushData: [String: Any] = [
          "alert": "New message received",
          "category": "Message",
          "badge": 1,
          "sound": "message.mp3",
          "custom-key": "This is a custom attribute. custom-key is just an example. You can use your own names for keys."
      ]
      try conversation.send(message: message, pushData: pushData, completion: { (result) in
          switch result {
          case .success:
              break
          case .failure(error: let error):
              print(error)
          }
      })
  } catch {
      print(error)
  }
  ```
  ```objc
  AVIMMessageOption *option = [[AVIMMessageOption alloc] init];
  option.pushData = @{@"alert" : @"New message received", @"sound" : @"message.mp3", @"badge" : @1, @"custom-key" : @"This is a custom attribute. custom-key is just an example. You can use your own names for keys."};
  [conversation sendMessage:[AVIMTextMessage messageWithText:@"Hey Jerry, me and Kate are going to a bar to watch a game tonight. Do you wanna come?" attributes:nil] option:option callback:^(BOOL succeeded, NSError * _Nullable error) {
      // Handle result and error
  }];
  ```
  ```java
  AVIMTextMessage msg = new AVIMTextMessage();
  msg.setText("Hey Jerry, me and Kate are going to a bar to watch a game tonight. Do you wanna come?");

  AVIMMessageOption messageOption = new AVIMMessageOption();
  messageOption.setPushData("Custom message for push notification.");
  conv.sendMessage(msg, messageOption, new AVIMConversationCallback() {
      @Override
      public void done(AVIMException e) {
          if (e == null) {
              // Sent
          }
      }
  });
  ```
  ```cs
  var message = new AVIMTextMessage()
  {
      TextContent = "Hey Jerry, me and Kate are going to a bar to watch a game tonight. Do you wanna come?"
  };

  AVIMSendOptions sendOptions = new AVIMSendOptions()
  {
      PushData = new Dictionary<string, object>()
      {
          { "alert", "New message received"},
          { "category", "Message"},
          { "badge", 1},
          { "sound", "message.mp3"}, // The name of the sound file; has to be present in the app
          { "custom-key", "This is a custom attribute. custom-key is just an example. You can use your own names for keys."}
      }
  };
  ```

3. Generating contents dynamically on the server side

  The second way introduced above allows you to compose the contents of push notifications based on the messages being sent out, but the logic needs to be predefined on the client side, which makes things less flexible.

  So we offer a third way that allows you to define the logic of push notifications on the server side with the help of [hooks](realtime-guide-systemconv.html#hooks). Check the following documentation for more details:

  - [Hooks and System Conversations](realtime-guide-systemconv.html#_receiversOffline)
  - [Node.js Cloud Function Guide · Hooks for LeanMessage](leanengine_cloudfunction_guide-node.html#onIMReceiversOffline)
  - [Python Cloud Function Guide · Hooks for LeanMessage](leanengine_cloudfunction_guide-python.html#_receiversOffline)
  - [PHP Cloud Function Guide · Hooks for LeanMessage](leanengine_cloudfunction_guide-php.html#_receiversOffline)
  - [Java Cloud Function Guide · Hooks for LeanMessage](leanengine_cloudfunction_guide-java.html#receiversOffline)

Here is a comparison of the priorities of the three methods mentioned above: **Generating contents dynamically on the server side > Specifying contents when sending messages from a client > Setting up a static message**.

If more than one of these methods are implemented at the same time, the push notifications generated on the server side will always get the highest priority. Those sent from clients will get the secondary priority, and the static message set up on the dashboard will get the lowest priority.

### Implementations and Restrictions

If your app is using push notification services together with LeanMessage, when a client is logging in, the SDK will automatically associate the `clientId` with the device information (stored in the `Installation` table) by having the device **subscribe** to the channel with `clientId` as its name. Such association can be seen in the `channels` field of the `_Installation` table. By doing so, when the cloud wants to send a push notification to a client, the client's device can be targeted with the `clientId` associated with it.

Since LeanMessage generates way more push notifications than other sources, the cloud will not keep any records for it, nor can you find them in your app's **Dashboard** > **Messaging** > **Push Notifications** > **Push Notification History**.

Each push notification is only valid for 7 days. This means that if a device doesn't connect to the service for more than 7 days, it will not receive the push notification anymore.

### Advanced Settings for Push Notifications

By default, **production certificate** is used for sending push notifications. You can specify the certificate you want to use by adding a `_profile` property into the JSON string:

```json
{
  "alert":    "New message received",
  "_profile": "dev"
}
```

Apple doesn't allow a single request to include push notifications sent to devices belonging to different Team IDs. If your app has private keys for multiple Team IDs, please confirm the one that should be used for you target devices and fill it into the `_apns_team_id` parameter:

```json
{
  "alert":         "New message received",
  "_apns_team_id": "my_fancy_team_id"
}
```

`_profile` and `_apns_team_id` will not be included in the actual contents of push notifications.

For the message set up in your app's [Dashboard > Messaging > LeanMessage > Settings > Push Notification Settings](https://console.leancloud.app/messaging.html?appid={{appid}}#/message/realtime/conf), you can include the following variables:

* `${convId}` The ID of the conversation
* `${timestamp}` The Unix timestamp when the push notification is triggered
* `${fromClientId}` The `clientId` of the sender

## Message Synchronization

Push notification seems to be a good way to remind users of new messages, but the actual messages won't get delivered until the user goes online. If a user hasn't been online for an extremely long time, there will be tons of messages piled up on the cloud. How can we make sure that all these messages will be perfectly delivered once the user goes online?

LeanMessage offers two methods for synchronizing messages:

- One is to have the cloud push messages to the client. The cloud keeps track of the last message each user receives from each conversation. When a user goes online, the cloud will automatically push new messages generated in each conversation to the client, with one push notification for each message (the client can handle them in the same way as receiving new messages). For each conversation, the cloud will deliver at most 20 messages and the rest of them will not be delivered.
- Another one is to have the client pull messages from the cloud. The cloud keeps track of the last message each user receives from each conversation. When a user goes online, the conversations containing new messages as well as the number of unread messages in each of them will be computed and the client will receive a notification indicating that there is an update on the total number of unread messages. The client can then proactively fetch these messages.

The first method is relatively easier to implement, but since the cloud only pushes no more than 20 messages for each conversation, this method won't work for many of the scenarios, including those that demand displaying the progress of each user reading messages or the exact number of unread messages. Therefore, we highly recommend that you use the second method to have the client pull messages from the cloud.

For historical reasons, different SDKs have different supports for the two methods introduced above:
1. Android and iOS SDKs support both of them and use the first method (pushing) by default.
2. JavaScript SDK supports the second method (pulling) by default.
3. .NET SDK doesn't support the second method yet.

> Please don't use different methods for different platforms at the same time (for example, to use the first method for iOS and the second one for Android). This will make your app unable to fetch messages correctly.

### Notifications on Updates of Unread Message Count

To know when to pull messages from the cloud, the client relies on the notifications sent from the cloud regarding updates of the numbers of unread messages in all the conversations the current user is in. These numbers are computed at the moment the client goes online.

To receive such notifications, the client needs to indicate that it is using the method of pulling messages from the cloud. As mentioned earlier, JavaScript SDK uses this method by default, so there isn't any configuration that needs to be done. For Android and iOS SDKs, the following line needs to be added when initializing `AVOSCloud` to indicate that this method will be used:

```js
// Supported by default; no extra configuration required
```
```swift
// Supported by default; no extra configuration required

// Turn off notifications
do {
    var options = IMClient.Options.default
    options.remove(.receiveUnreadMessageCountAfterSessionDidOpen)
    let client = try IMClient(ID: "CLIENT_ID", options: options)
} catch {
    print(error)
}
```
```objc
[AVIMClient setUnreadNotificationEnabled:YES];
```
```java
AVIMClient.setUnreadNotificationEnabled(true);
```
```cs
// Not supported yet
```

The SDK will maintain an `unreadMessagesCount` field on each `AVIMConversation` to track the number of unread messages in the conversation.

When the client goes online, the cloud will drop in a series of `<Conversation, UnreadMessageCount, LastMessage>` in the format of events indicating updates on the numbers of unread messages. Each of them matches a conversation containing new messages and serves as the initial value of a `<Conversation, UnreadMessageCount>` maintained on the client side. After this, whenever a message is received by the SDK, the corresponding `unreadMessageCount` will be automatically increased. When the number of unread messages of a conversation is cleared, both `<Conversation, UnreadMessageCount>` on the cloud and maintained by the SDK will be reset.

> If the count of unread messages is enabled, it will keep increasing even though a message is received when the client is online. Make sure to reset the count whenever needed.

When the number of `<Conversation, UnreadMessageCount>` changes, the SDK will send an `UNREAD_MESSAGES_COUNT_UPDATE` event to the app through `IMClient`. You can listen to this event and make corresponding changes to the number of unread messages on the UI. However, since this event gets triggered very frequently and there are often other events coming together with it, there is **no need for you to implement anything else in response to it**.

```js
var { Event } = require('leancloud-realtime');
client.on(Event.UNREAD_MESSAGES_COUNT_UPDATE, function(conversations) {
  for(let conv of conversations) {
    console.log(conv.id, conv.name, conv.unreadMessagesCount);
  }
});
```
```swift
func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .unreadMessageCountUpdated:
        print(conversation.unreadMessageCount)
    default:
        break
    }
}
```
```objc
// Use delegate method conversation:didUpdateForKey: to observe the unreadMessagesCount property of the conversation
- (void)conversation:(AVIMConversation *)conversation didUpdateForKey:(NSString *)key {
    if ([key isEqualToString:@"unreadMessagesCount"]) {
        NSUInteger unreadMessagesCount = conversation.unreadMessagesCount;
        /* New messages exist; update UI or fetch messages */
    }
}
```
```java
// Receive notifications on updates of unread message count
onUnreadMessagesCountUpdated(AVIMClient client, AVIMConversation conversation) {
    // conversation.getUnreadMessagesCount() is the number of unread messages in conversation
}
```
```cs
// Not supported yet
```

When responding to an `UNREAD_MESSAGES_COUNT_UPDATE` event, you get a `Conversation` object containing the `lastMessage` property which is the last message received by the current user from the conversation. To display the actual unread messages, [fetch the messages](realtime-guide-beginner.html#retrieving-messages) that come after it.

The only way to clear the number of unread messages is to mark the messages as read with `Conversation#read`. You may do so when:

- The user opens a conversation
- The user is already in a conversation and a new message comes in

## Multi Device Sign-on and Single Device Sign-on

In some scenarios, a user can stay logged in on multiple devices at the same time. In other ones, a user can be logged in on only one device at a time. With LeanMessage, you can easily implement both **multi device sign-on** and **single device sign-on** depending on your needs.

When creating an `IMClient` instance, you can provide a `tag` parameter beside `clientId` to have the cloud check the uniqueness of `<ClientId, Tag>` when logging the user in. If the user is already logged in on another device with the same `tag`, the cloud will log the user out from that device, otherwise the user will stay logged in on all their devices. When a user is logged in on multiple devices, the messages coming to this user will be delivered to all these devices and the numbers of unread messages will be synchronized. If the user sends a message from one of these devices, the message will appear on other devices as well.

You can assign a unique `tag` for each type of device. For example, if you have `Mobile` for phones, `Pad` for tablets, and `Web` for desktop computers, a user will be able to stay logged in on three devices with different types, but not two desktop computers.

> If `tag` is not specified when logging in, a user will be able to have any number of devices logged in at the same time. If all the clients share the same `tag`, a user will be able to stay logged in on only one device at a time.

### Setting Tags

The code below sets a `tag` called `Mobile` when creating `IMClient`, which can be used for the mobile client of your app:

```js
realtime.createIMClient('Tom', { tag: 'Mobile' }).then(function(tom) {
  console.log('Tom is logged in.');
});
```
```swift
do {
    let client = try IMClient(ID: "CLIENT_ID", tag: "Mobile")
    client.open { (result) in
        switch result {
        case .success:
            break
        case .failure(error: let error):
            print(error)
        }
    }
} catch {
    print(error)
}
```
```objc
AVIMClient *currentClient = [[AVIMClient alloc] initWithClientId:@"Tom" tag:@"Mobile"];
[currentClient openWithCallback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        // Successfully logged in
    }
}];
```
```java
// Provide tag as a second parameter
AVIMClient currentClient = AVIMClient.getInstance(clientId, "Mobile");
currentClient.open(new AVIMClientCallback() {
  @Override
  public void done(AVIMClient avimClient, AVIMException e) {
    if(e == null){
      // Successfully logged in
    }
  }
});
```
```cs
AVIMClient tom = await realtime.CreateClientAsync("Tom", tag: "Mobile", deviceId: "your-device-id");
```

With the code above, if a user logs in on one mobile device and then logs in on another one (with the same `tag`), the user will be logged out from the former one.

### Handling Conflicts

When the cloud encounters the same `<ClientId, Tag>` for a second time, the device used for the earlier one will be logged out and receive a `CONFLICT` event:

```js
var { Event } = require('leancloud-realtime');
tom.on(Event.CONFLICT, function() {
  // Tell the user that the same clientId is logged in on another device
});
```
```swift
func client(_ client: IMClient, event: IMClientEvent) {
    switch event {
    case .sessionDidClose(error: let error):
        if error.code == 4111 {
            // Tell the user that the same clientId is logged in on another device
        }
    default:
        break
    }
}
```
```objc
-(void)client:(AVIMClient *)client didOfflineWithError:(NSError *)error{
    if ([error code]  == 4111) {
        // Tell the user that the same clientId is logged in on another device
    }
};
```
```java
public class AVImClientManager extends AVIMClientEventHandler {
  /**
   * Handle the event of being logged out
   *
   * 
   * @param client
   * @param code The status code indicating the reason of being logged out
   */
  @Override
  public void onClientOffline(AVIMClient avimClient, int i) {
    if(i == 4111){
      // Tell the user that the same clientId is logged in on another device
    }
  }
}

// Need to register the custom AVIMClientEventHandler to receive onClientOffline notifications
AVIMClient.setClientEventHandler(new AVImClientManager());
```
```cs
tom.OnSessionClosed += Tom_OnSessionClosed;
private void Tom_OnSessionClosed(object sender, AVIMSessionClosedEventArgs e)
{
}
```

The reason a device gets logged out will be included in the event so that you can display a message to the user with that.

## Custom Message Types

Although LeanMessage already support a number of common message types by default, you can still define your own types as you need. For example, if you want your users to send messages containing payments and contacts, you can implement that with custom message types.

### Creating Your Own Message Types

{{ docs.langSpecStart('js') }}

By inheriting from `TypedMessage`, you can define your own types of messages. The basic steps include:

* Declare the new message type by inheriting from `TypedMessage` or its subclass and then:
  * Apply the `messageType(123)` decorator to the class. The value assigned to the type (`123` here) can be defined by yourself (negative numbers are for [types offered by default](realtime-guide-beginner.html#default-message-types) and positive numbers are for those defined by you).
  * Apply the `messageField(['fieldName'])` decorator to the class to declare the fields that should be sent.
* Call `Realtime#register()` to register the type.

For example, to implement the `OperationMessage` introduced in [Transient Messages](#transient-messages):

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('swift') }}

By inheriting from `IMCategorizedMessage`, you can define your own types of messages. The basic steps include:

* Implement `IMMessageCategorizing` protocol;
* Register the subclass. This is often done by calling `try CustomMessage.register()` in the `application(_:didFinishLaunchingWithOptions:)` method of `AppDelegate`.

{{ docs.langSpecEnd('swift') }}

{{ docs.langSpecStart('objc') }}

By inheriting from `AVIMTypedMessage`, you can define your own types of messages. The basic steps include:

* Implement `AVIMTypedMessageSubclassing` protocol;
* Register the subclass. This is often done by calling `[YourClass registerSubclass]` in the `+load` method of the subclass or `-application:didFinishLaunchingWithOptions:` in `UIApplication`.

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

By inheriting from `AVIMTypedMessage`, you can define your own types of messages. The basic steps include:

* Implement the new message type inherited from `AVIMTypedMessage`. Make sure to:
  * Add the `@AVIMMessageType(type=123)` annotation to the class<br/>The value assigned to the type (`123` here) can be defined by yourself. Negative numbers are for types offered by default and positive numbers are for those defined by you.
  * Add the `@AVIMMessageField(name="")` annotation when declaring custom fields<br/>`name` is optional. Custom fields need to have their getters and setters.
  * **Include an empty constructor and a Creator** (see the sample below), otherwise there will be an error with type conversion.
* Call `AVIMMessageManager.registerAVIMMessageType()` to register the class.
* Call `AVIMMessageManager.registerMessageHandler()` to register the handler for messages.

For your reference, here is the source code of `AVIMTextMessage`:

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

By inheriting from `AVIMTypedMessage`, you can define your own types of messages. The basic steps include:

* Define a subclass inherited from `AVIMTypedMessage`.
* Register the subclass when initializing.

{{ docs.langSpecEnd('cs') }}

```js
// TypedMessage, messageType, messageField are provided by leancloud-realtime
// Use `var { TypedMessage, messageType, messageField } = AV;` in browser
var { TypedMessage, messageType, messageField } = require('leancloud-realtime');
var inherit = require('inherit');
// Define OperationMessage for sending and receiving messages regarding user operations
export const OperationMessage = inherit(TypedMessage);
// Specify the type; can be other positive integers
messageType(1)(OperationMessage);
// The `op` field needs to be sent with the message
messageField('op')(OperationMessage);
// Register the class, otherwise an incoming OperationMessage cannot be automatically resolved
realtime.register(OperationMessage);
```
```swift
class CustomMessage: IMCategorizedMessage {
    
    class override var messageType: MessageType {
        return 1
    }
}

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    
    do {
        try CustomMessage.register()
    } catch {
        print(error)
        return false
    }
    
    return true
}

func client(_ client: IMClient, conversation: IMConversation, event: IMConversationEvent) {
    switch event {
    case .message(event: let messageEvent):
        switch messageEvent {
        case .received(message: let message):
            if let customMessage = message as? CustomMessage {
                print(customMessage)
            }
        default:
            break
        }
    default:
        break
    }
}
```
```objc
// Definition

@interface CustomMessage : AVIMTypedMessage <AVIMTypedMessageSubclassing>

+ (AVIMMessageMediaType)classMediaType;

@end

@implementation CustomMessage

+ (AVIMMessageMediaType)classMediaType {
    return 123;
}

@end

// 1. Register subclass
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [CustomMessage registerSubclass];
}
// 2. Receive message
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    if (message.mediaType == 123) {
        CustomMessage *imageMessage = (CustomMessage *)message;
        // Handle custom message
    }
}
```
```java
@AVIMMessageType(type = AVIMMessageType.TEXT_MESSAGE_TYPE)
public class AVIMTextMessage extends AVIMTypedMessage {
  // An empty constructor has to be present
  public AVIMTextMessage() {

  }

  @AVIMMessageField(name = "_lctext")
  String text;
  @AVIMMessageField(name = "_lcattrs")
  Map<String, Object> attrs;

  public String getText() {
    return this.text;
  }

  public void setText(String text) {
    this.text = text;
  }

  public Map<String, Object> getAttrs() {
    return this.attrs;
  }

  public void setAttrs(Map<String, Object> attr) {
    this.attrs = attr;
  }
  // Creator has to be present
  public static final Creator<AVIMTextMessage> CREATOR = new AVIMMessageCreator<AVIMTextMessage>(AVIMTextMessage.class);
}
```
```cs
// Provide a class name
[AVIMMessageClassName("InputtingMessage")]
// Assign a value to the type; can only be positive integers; negative numbers are for types offered by default
[AVIMTypedMessageTypeIntAttribute(2)]
public class InputtingMessage : AVIMTypedMessage
{
    public InputtingMessage() { }
    // We can include an Emoji when sending messages; here we use Ecode for the code of the Emoji
    [AVIMMessageFieldName("Ecode")]
    public string Ecode { get; set; }
}

// Register subclass
realtime.RegisterMessageType<InputtingMessage>();

// Send a message with the custom type:
var inputtingMessage = new InputtingMessage();
// TextContent is inherited from AVIMTypedMessage
inputtingMessage.TextContent = "Typing…";
// Ecode is the property we just defined
inputtingMessage.Ecode = "#e001";
        
// At this point, the user is already logged in and the conversation object for the current conversation is already obtained
await conversation.SendMessageAsync(inputtingMessage);

For other members in the same `conversation`, they can receive the message with the following code:
async void Start() 
{   
    var jerry = await realtime.CreateClientAsync("jerry");
    // Event for message received
    jerry.OnMessageReceived += Jerry_OnMessageReceived;
}
void Jerry_OnMessageReceived(object sender, AVIMMessageEventArgs e)
{
    if (e.Message is InputtingMessage)
    {
        var inputtingMessage = (InputtingMessage)e.Message;
        Debug.Log(string.Format("Message with custom type received: {0} {1}", inputtingMessage.TextContent, inputtingMessage.Ecode));
    }
}

// Here we introduced how you can build your own message type to support the function of displaying typing status. However, to fully implement such function, you have to make the message into a [transient message](#transient-messages).
```

See [Back to Receiving Messages](realtime-guide-beginner.html#back-to-receiving-messages) in the previous chapter for more details on how to receive messages with custom types.

### When to Use Custom Message Types

The following message types are offered by default:

- `TextMessage` Text message
- `ImageMessage` Image message
- `AudioMessage` Audio message
- `VideoMessage` Video message
- `FileMessage` File message (.txt, .doc, .md, etc.)
- `LocationMessage` Location message

When composing messages with these types, you can include additional information by attaching custom attributes in the format of key-value pairs. For example, if your are sending an image message and need to include some location information beside text, you can put it into `attributes` of the message rather than create a new type of message.

Therefore, we suggest that you create your own message types only when your requirements cannot be fulfilled with the built-in types.

## Continue Reading

[3. Security, Permission Management, Chat Rooms, and Temporary Conversations](realtime-guide-senior.html)

[4. Hooks and System Conversations](realtime-guide-systemconv.html)
