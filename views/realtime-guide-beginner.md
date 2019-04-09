{% import "views/_helper.njk" as docs %}

{{ docs.defaultLang('js') }}

{{ docs.useIMLangSpec()}}

# 1. Basic Conversations and Messages

## Introduction

A lot of products today have the needs to offer instant messaging functions to their users. For example:

- To have the staff behind the product talk to the users.
- To have the workers in a company communicate with each other.
- To have the audience of live-streamed contents interact with each other.
- To have the users of an app or players of a game chat with each other.

Based on the hierarchy of needs and the difficulty of implementation, we wrote the following documentations for you to learn how you can embed LeanMessage into your app:

- In this documentation, we will introduce how you can implement one-on-one chatting and group chats, how you can create and join conversations, and how you can send and receive rich media messages. We will also introduce how history messages are kept on the cloud and how you can retrieve them. By the end of this documentation, you should be able to build a simple chatting page in your app.
- [In the second documentation](realtime-guide-intermediate.html), we will introduce some advanced features built around messaging, including mentioning people with @, deleting messages, editing messages, getting reports when messages are delivered and read, sending push notifications, and synchronizing messages. The implementation of multi device sign-on and custom message types will also be covered. By the end of this documentation, you should be able to integrate a chatting component into your app with these features.
- [In the third documentation](realtime-guide-senior.html), we will introduce the security features offered by our services, including third-party signing mechanism, permission management of members, and blacklisting. We will also go over the usage of chat rooms and temporary conversations. By the end of this documentation, you will get a set of skills to improve the security and usability of your app, as well as to build conversations that serve different purposes.
- [In the last documentation](realtime-guide-systemconv.html), we will introduce the usage of hooks and system conversations, plus how you can build your own chatbots based on them. By the end of this documentation, you will learn how you can have your app spontaneously chat with users.

We aim these documentations to not only help you complete the functions you are currently building but also give you a better understanding of all the things LeanMessage can do (which you will find helpful when you plan to add more features into your app).

> Before you continue:
>
> Take a look at [LeanMessage Overview](realtime_v2.html) if you haven’t done yet. Also make sure you have already followed [SDK Installation](start.html) to install and initialize the SDK for the platform (language) you are using.

## One-on-One Chatting

Before diving into the main topic, let’s see what an `IMClient` object is in LeanMessage SDK:

> An `IMClient` refers to an actual user, meaning that the user logged in to the system as a client.

See [LeanMessage Overview](realtime_v2.html#clientId,-user,-and-log-in) for more details.

### Creating `IMClient`

Assuming that there is a user named “Tom”. Now let’s create an `IMClient` instance for him:

```js
var realtime = new Realtime({
  appId: 'your-app-id',
  appKey: 'your-app-key',
  plugins: [TypedMessagesPlugin], // Register plugins for rich media messages
});
// Tom logs in with his name as clientId
realtime.createIMClient('Tom').then(function(tom) {
  // Successfully logged in
}).catch(console.error);
```
```objc
@property (nonatomic, strong) AVIMClient *tom;
// clientId is Tom
tom = [[AVIMClient alloc] initWithClientId:@"Tom"]
```
```java
// clientId is Tom
AVIMClient tom = AVIMClient.getInstance("Tom");
```
```cs
var realtime = new AVRealtime('your-app-id','your-app-key');
var tom = await realtime.CreateClientAsync('Tom');
```

Keep in mind that an `IMClient` refers to an actual user. It should be stored globally since all the further actions done by this user will have to access it.

### Logging in to the LeanMessage Server

After creating the `IMClient` instance for Tom, we will need to have this instance log in to the LeanMessage server. Only clients that are logged in can chat with other users and receive notifications from the cloud.

For JavaScript and C# (Unity3D) SDKs, clients will be automatically logged in when `IMClient` instances are created; for iOS (both Objective-C and Swift) and Android (including Java) SDKs, clients need to be logged in manually with the `open` method:

```js
// Tom logs in with his name as clientId and gets the IMClient instance
realtime.createIMClient('Tom').then(function(tom) {
  // Successfully logged in
}).catch(console.error);
```
```objc
// Tom creates a client and logs in with his name as clientId
AVIMClient *tom = [[AVIMClient alloc] initWithClientId:@"Tom"];
// Tom logs in
[tom openWithCallback:^(BOOL succeeded, NSError *error) {
  if(succeeded) {
    // Successfully connected
  }
}];
```
```java
// Tom creates a client and logs in with his name as clientId
AVIMClient tom = AVIMClient.getInstance("Tom");
// Tom logs in
tom.open(new AVIMClientCallback() {
  @Override
  public void done(AVIMClient client, AVIMException e) {
    if (e == null) {
      // Successfully connected
    }
  }
}
```
```cs
var realtime = new AVRealtime('your-app-id','your-app-key');
var tom = await realtime.CreateClientAsync('Tom');
```

### Logging in with `_User`

Beside specifying a `clientId` within the app, you can also log in directly with a `_User` object after an `IMClient` is created. By doing so, the [signing process for logging in](realtime-guide-senior.html#signatures-for-logging-in) can be skipped which helps you easily integrate LeanStorage with LeanMessage:

```js
var AV = require('leancloud-storage');
// Log in to LeanMessage with the username and password of an AVUser
AV.User.logIn('username', 'password').then(function(user) {
  return realtime.createIMClient(user);
}).catch(console.error.bind(console));
```
```objc
// Log in to LeanMessage with the username and password of an AVUser
[AVUser logInWithUsernameInBackground:username password:password block:^(AVUser * _Nullable user, NSError * _Nullable error) {
    // Create a client with AVUser instance
    AVIMClient *client = [[AVIMClient alloc] initWithUser:user];
    // Open the client to connect to the cloud
    [client openWithCallback:^(BOOL succeeded, NSError * _Nullable error) {
        // Do something you like.
    }];
}];
```
```java
// Log in to LeanMessage with the username and password of an AVUser
AVUser.logInInBackground("username", "password", new LogInCallback<AVUser>() {
    @Override
    public void done(AVUser user, AVException e) {
        if (null != e) {
          return;
        }
        // Connect to the server
        AVIMClient client = AVIMClient.getInstance(user);
        client.open(new AVIMClientCallback() {
          @Override
          public void done(final AVIMClient avimClient, AVIMException e) {
            // do something as you need.
          }
       });
    }
});
```
```cs
// Not supported yet
```

### Creating Conversations

A `Conversation` needs to be created before a user can chat with others.

> [Conversations](realtime_v2.html#conversation) are the carriers of messages. All the messages are sent to conversations to be delivered to the members in them.

Since Tom is already logged in, he can start chatting with other users now. If he wants to chat with Jerry, he can create a `Conversation` containing Jerry and himself:

```js
// Create a conversation with Jerry
return tom.CreateConversationAsync({
  // Members of the conversation include Tom (the SDK will automatically add the current user into the conversation) and Jerry
  members: ['Jerry'],
  // Name of the conversation
  name: 'Tom & Jerry',
  unique: true
});.catch(console.error);
```
```objc
// Create a conversation with Jerry
[tom createConversationWithName:@"Tom & Jerry" clientIds:@[@"Jerry"] attributes:nil options:AVIMConversationOptionUnique
                       callback:^(AVIMConversation *conversation, NSError *error) {

}];
```
```java
tom.createConversation(Arrays.asList("Jerry"), "Tom & Jerry", null, false, true,
    new AVIMConversationCreatedCallback() {
        @Override
        public void done(AVIMConversation conversation, AVIMException e) {
          if(e == null) {
            // Successfullly created
          }
        }
});
```
```cs
var tom = await realtime.CreateClientAsync('Tom');
var conversation = await tom.CreateConversationAsync("Jerry", name:"Tom & Jerry", isUnique:true);
```

`createConversation` creates a new conversation and stores it into the `_Conversation` table which can be found in your app’s [Dashboard > LeanStorage > Data](/data.html?appid={{appid}}#/). Below are the interfaces offered by different SDKs for creating conversations:

```js
/**
 * Create a conversation
 * @param {Object} options The fields beside the following ones will be treated as custom attributes
 * @param {String[]} options.members The members of the conversation; required; include the current client by default
 * @param {String} [options.name] The name of the conversation; optional; defaults to null
 * @param {Boolean} [options.transient=false] Whether the conversation is a chat room; optional
 * @param {Boolean} [options.unique=false] Whether the conversation is unique; if it is true and an existing conversation contains the same composition of members, the existing conversation will be reused, otherwise a new conversation will be created
 * @param {Boolean} [options.tempConv=false] Whether the conversation is temporary; optional
 * @param {Integer} [options.tempConvTTL=0] Optional; if tempConv is true, the TTL of the conversation can be specified here
 * @return {Promise.<Conversation>}
 */
async createConversation({
  members: m,
  name,
  transient,
  unique,
  tempConv,
  tempConvTTL,
  ...properties
});
```
```objc
/*!
 Create a conversation
 For one-on-one chatting, pass in a single clientId; for group chats, pass in a list of clientIds
 @param name - The name of the conversation
 @param clientIds - The list of clientIds of participants in the conversation (except the creator)
 @param callback － The callback after the conversation is created
 */
- (void)createConversationWithName:(NSString * _Nullable)name
                         clientIds:(NSArray<NSString *> *)clientIds
                          callback:(void (^)(AVIMConversation * _Nullable conversation, NSError * _Nullable error))callback;
/*!
 Create a conversation
 For one-on-one chatting, pass in a single clientId; for group chats, pass in a list of clientIds
 @param name - The name of the conversation
 @param clientIds - The list of clientIds of participants in the conversation (except the creator)
 @param attributes - Custom attributes
 @param options － Optional; multiple options can be divided by “|”
 @param callback － The callback after the conversation is created
 */
- (void)createConversationWithName:(NSString * _Nullable)name
                         clientIds:(NSArray<NSString *> *)clientIds
                        attributes:(NSDictionary * _Nullable)attributes
                           options:(AVIMConversationOption)options
                          callback:(void (^)(AVIMConversation * _Nullable conversation, NSError * _Nullable error))callback;

```
```java
/**
 * Create or find an existing conversation
 *
 * @param members The members in the conversation
 * @param name The name of the conversation
 * @param attributes Custom attributes
 * @param isTransient Whether the conversation is a chat room
 * @param isUnique Whether return the existing conversation satisfying conditions
 *                 If false, create a new conversation
 *                 If true, find if there is an existing conversation satisfying conditions; if so, return the conversation, otherwise create a new conversation
 *                 If true, only members is the valid query condition
 * @param callback The callback after the conversation is created
 */
public void createConversation(final List<String> members, final String name,
    final Map<String, Object> attributes, final boolean isTransient, final boolean isUnique,
    final AVIMConversationCreatedCallback callback);
/**
 * Create a conversation
 *
 * @param members The members in the conversation
 * @param attributes Custom attributes
 * @param isTransient Whether the conversation is a chat room
 * @param callback The callback after the conversation is created
 */
public void createConversation(final List<String> members, final String name,
                               final Map<String, Object> attributes, final boolean isTransient,
                               final AVIMConversationCreatedCallback callback);
/**
 * Create a conversation
 *
 * @param conversationMembers The members in the conversation
 * @param name       The name of the conversation
 * @param attributes Custom attributes
 * @param callback   The callback after the conversation is created
 * @since 3.0
 */
public void createConversation(final List<String> conversationMembers, String name,
    final Map<String, Object> attributes, final AVIMConversationCreatedCallback callback);
/**
 * Create a conversation
 * 
 * @param conversationMembers The members in the conversation
 * @param attributes Custom attributes
 * @param callback   The callback after the conversation is created
 * @since 3.0
 */
public void createConversation(final List<String> conversationMembers,
    final Map<String, Object> attributes, final AVIMConversationCreatedCallback callback);
```
```cs
/// <summary>
/// Create a conversation with members.
/// </summary>
/// <returns>Return the conversation.</returns>
/// <param name="member">The member in the conversation.</param>
/// <param name="members">The list of members in the conversation.</param>
/// <param name="name">The name of the conversation.</param>
/// <param name="isSystem">Whether it is a system conversation. Note that system conversations cannot be created by clients, so the conversation will not be created if it is true here.</param>
/// <param name="isTransient">Whether the conversation is a chat room.</param>
/// <param name="isUnique">Whether the conversation is unique.</param>
/// <param name="options">Custom attributes.</param>
public Task<AVIMConversation> CreateConversationAsync(string member = null,
    IEnumerable<string> members = null,
    string name = "",
    bool isSystem = false,
    bool isTransient = false,
    bool isUnique = true,
    IDictionary<string, object> options = null);
```

Although SDKs for different languages/platforms share different interfaces, they take in the similar set of parameters when creating a conversation:


1. `members`: Required; includes the initial list of members in the conversation. The initiator of the conversation is included by default, so `members` does not have to include the `clientId` of the current user.
2. `name`: The name of the conversation; optional. The code above puts “Tom & Jerry” for it.
3. `attributes`: The custom attributes of the conversation; optional. The code above does not specify any attributes. If you ever specify them for your conversations, you can retrieve them later with `AVIMConversation`. Such attributes will be stored in the `attr` field of the `_Conversation` table.
4. `unique`/`isUnique` or `AVIMConversationOptionUnique`: Markes if the conversation is unique; optional.
   - If true, the cloud will perform a query on conversations with the list of members specified. If an existing conversation contains the same members, the conversation will be returned, otherwise a new conversation will be created.
   - If false, a new conversation will be created each time `createConversation` is called.
   - If not specified, it defaults to true for Java SDK and false for others (will be changed to true in the future).
   - In general, it is more reasonable that there is only one conversation existing for the same composition of members, otherwise it could be messy since multiple sets of message histories are available for the same group of people. We **strongly recommend that you set `unique` to be `true`** when creating conversations.
5. Other parameters specifying the type of the conversation; optional. For example, `transient`/`isTransient` specifies if it is a chat room, and `tempConv`/`tempConvTTL` or `AVIMConversationOptionTemporary` specifies if it is a temporary conversation. If nothing is specified, it will be a basic conversation. We will talk more about them later.

The built-in properties of a conversation can be retrieved once the conversation is created. For example, a globally unique ID will be created for each conversation which can be retrieved with `Conversation.id`. This is the field often used for querying conversations.

### Sending Messages

Now that the conversation is created, Tom can start sending messages to it:

```js
var { TextMessage } = require('leancloud-realtime');
conversation.send(new TextMessage('Get up, Jerry!')).then(function(message) {
  console.log('Tom & Jerry', 'Message sent!');
}).catch(console.error);
```
```objc
AVIMTextMessage *message = [AVIMTextMessage messageWithText:@"Get up, Jerry!" attributes:nil];
[conversation sendMessage:message callback:^(BOOL succeeded, NSError *error) {
  if (succeeded) {
    NSLog(@"Message sent!");
  }
}];
```
```java
AVIMTextMessage msg = new AVIMTextMessage();
msg.setText("Get up, Jerry!");
// Send the message
conversation.sendMessage(msg, new AVIMConversationCallback() {
  @Override
  public void done(AVIMException e) {
    if (e == null) {
      Log.d("Tom & Jerry", "Message sent!");
    }
  }
});
```
```cs
var textMessage = new AVIMTextMessage("Get up, Jerry!");
await conversation.SendMessageAsync(textMessage);
```

`Conversation#send` sends a message to the conversation specified. All the other members who are online will immediately receive the message.

So how would Jerry see the message on his device?

### Receiving Messages

On another device, we create an `AVIMClient` with `Jerry` as `clientId` and log in to the server (just as how we did for Tom):

```js
var { Event } = require('leancloud-realtime');
// Jerry logs in
realtime.createIMClient('Jerry').then(function(jerry) {
}).catch(console.error);
```
```objc
jerry = [[AVIMClient alloc] initWithClientId:@"Jerry"];
[jerry openWithCallback:^(BOOL succeeded, NSError *error) {

}];
```
```java
// Jerry logs in
AVIMClient jerry = AVIMClient.getInstance("Jerry");
jerry.open(new AVIMClientCallback(){
  @Override
  public void done(AVIMClient client,AVIMException e){
      if(e==null){
        ... // Things to do after logging in
      }
  }
});
```
```cs
var realtime = new AVRealtime('your-app-id','your-app-key');
var jerry = await realtime.CreateClientAsync('Jerry');
```

As the receiver of the message, Jerry doesn’t have to create a conversation with Tom and may as well not know that Tom created a conversation with him. Jerry needs to set up a callback function to get notified for the things Tom did.

By setting up callbacks, clients will be able to handle notifications sent from the cloud. Here we focus on the following two events:
- The user is invited to a conversation. At the moment Tom creates a new conversation with Jerry, Jerry will receive a notification saying something like “Tom invited you to a conversation”.
- A new message is delivered to a conversation the user is already in. At the moment Tom sends out the message “Get up, Jerry!”, Jerry will receive a notification including the message itself as well as the context information like the conversation the message is sent to and the sender of the message.

Now let’s see how clients should handle such notifications. The code below handles both “joining conversation” and “getting new message” events for Jerry:

```js
// JS SDK responds to notifications by binding events on IMClient with callbacks

// The current user is added to a conversation
jerry.on(Event.INVITED, function invitedEventHandler(payload, conversation) {
    console.log(payload.invitedBy, conversation.id);
});

// The current user receives a message
jerry.on(Event.MESSAGE, function(message, conversation) {
    console.log('Message received: ' + message.text);
});
```
```objc
// Objective-C SDK responds to notifications with AVIMClientDelegate

/*!
 The current user is added to a conversation
 @param conversation － The conversation
 @param clientId - The ID of the inviter
 */
-(void)conversation:(AVIMConversation *)conversation invitedByClientId:(NSString *)clientId{
    NSLog(@"%@", [NSString stringWithFormat:@"Current clientId (Jerry) is invited by %@ to join the conversation.",clientId]);
}

/*!
 The current user receives a message
 @param conversation － The conversation
 @param message - The content of the message
 */
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    NSLog(@"%@", message.text); // Get up, Jerry!
}
```
```java
// Java/Android SDK responds to notifications with custom event handlers
public class CustomConversationEventHandler extends AVIMConversationEventHandler {
  /**
   * The current user is added to a conversation
   *
   * @param client
   * @param conversation The conversation
   * @param operator The inviter
   * @since 3.0
   */
  @Override
  public void onInvited(AVIMClient client, AVIMConversation conversation, String invitedBy) {
    // Things to do after the current clientId (Jerry) is invited to the conversation
  }
}
// Set up global conversation event handler
AVIMMessageManager.setConversationEventHandler(new CustomConversationEventHandler());

// Java/Android SDK responds to notifications with custom event handlers
public static class CustomMessageHandler extends AVIMMessageHandler{
  /**
   * Reload this method to handle message receiving
   * 
   * @param message
   * @param conversation
   * @param client
   */
   @Override
   public void onMessage(AVIMMessage message,AVIMConversation conversation,AVIMClient client){
     if(message instanceof AVIMTextMessage){
       Log.d(((AVIMTextMessage)message).getText());// Get up, Jerry!
     }
   }
 }
// Set up global message handling handler
AVIMMessageManager.registerDefaultMessageHandler(new CustomMessageHandler());
```
```cs
// SDK responds to notifications by binding events on IMClient with callbacks
var jerry = await realtime.CreateClientAsync("Jerry");
jerry.OnInvited += (sender, args) =>
{
  var invitedBy = args.InvitedBy;
  var conversationId = args.ConversationId;
};

private void Jerry_OnMessageReceived(object sender, AVIMMessageEventArgs e)
{
    if (e.Message is AVIMTextMessage)
    {
        var textMessage = (AVIMTextMessage)e.Message;
        // textMessage.ConversationId is the ID of the conversation
        // textMessage.TextContent is the text content of the message
        // textMessage.FromClientId is the clientId of the sender
    }
}
jerry.OnMessageReceived += Jerry_OnMessageReceived;
```

With the two event handling functions above, Jerry will be able to receive messages from Tom. Jerry can send messages to Tom as well, as long as Tom has the same functions on his side.

Now let’s take a look at the sequence diagram showing how the first message sent from Tom to Jerry is processed:

```seq
Tom->Cloud: 1. Tom adds Jerry into the conversation
Cloud-->Jerry: 2. Sends notification: you are invited to the conversation
Jerry-->UI: 3. Loads UI
Tom->Cloud: 4. Sends message
Cloud-->Jerry: 5. Sends notification: you have a new message
Jerry-->UI: 6. Shows the message
```

Beside responding to notifications about new messages, clients also need to respond to those indicating the change of members in a conversation, like “XX invited XX into the conversation”, “XX left the conversation”, and “XX is removed by the admin”. Such notifications will be delivered to clients in real time. See [Summary of Event Notifications Regarding Changes of Members](#summary-of-event-notifications-regarding-changes-of-members) for more details.

## Group Chats

We just discussed how we can create a conversation between two users. Now let’s see how we can create a group chat with more people.

There aren’t many differences between the two types of conversations and a major one would be the amount of members in them. You can either specify all the members of a group chat when creating it, or add them later after the conversation is created.

### Creating Group Chats

In the previous conversation between Tom and Jerry (assuming conversation ID to be `CONVERSATION_ID`), if Tom wants to add Mary into the conversation, the following code can be used:

```js
// Get the conversation with ID
tom.getConversation(‘CONVERSATION_ID’).then(function(conversation) {
  // Invite Mary
  return conversation.add(['Mary']);
}).then(function(conversation) {
  console.log('Member added!', conversation.members);
  // The conversation now contains ['Mary', 'Tom', 'Jerry']
}).catch(console.error.bind(console));
```
```objc
// Get the conversation with ID
AVIMConversationQuery *query = [self.client conversationQuery];
[query getConversationById:@"CONVERSATION_ID" callback:^(AVIMConversation *conversation, NSError *error) {
    // Invite Mary
    [conversation addMembersWithClientIds:@[@"Mary"] callback:^(BOOL succeeded, NSError *error) {
        if (succeeded) {
            NSLog(@"Member added!");
        }
    }];
}];
```
```java
// Get the conversation with ID
final AVIMConversation conv = client.getConversation("CONVERSATION_ID");
// Invite Mary
conv.addMembers(Arrays.asList("Mary"), new AVIMConversationCallback() {
    @Override
    public void done(AVIMException e) {
      // Member added
    }
});
```
```cs
// Get the conversation with ID
var conversation = await tom.GetConversationAsync("CONVERSATION_ID");
// Invite Mary
await tom.InviteAsync(conversation, "Mary");
```

On Jerry’s side, he can add a listener for handling events regarding “new members being added”. With the code below, he will be notified once Tom invites Mary to the conversation:

```js
// A user is added to the conversation
jerry.on(Event.MEMBERS_JOINED, function membersjoinedEventHandler(payload, conversation) {
    console.log(payload.members, payload.invitedBy, conversation.id);
});
```
```objc
jerry.delegate = self;

#pragma mark - AVIMClientDelegate
/*!
 All members will receive a notification when a new member joins the conversation
 @param conversation － The conversation
 @param clientIds - The list of new members
 @param clientId - The ID of the inviter
 */
- (void)conversation:(AVIMConversation *)conversation membersAdded:(NSArray *)clientIds byClientId:(NSString *)clientId {
    NSLog(@"%@", [NSString stringWithFormat:@"%@ is added to the conversation by %@",[clientIds objectAtIndex:0],clientId]);
}
```
```java
public class CustomConversationEventHandler extends AVIMConversationEventHandler {
  /**
   * All members will receive a notification when a new member joins the conversation
   *
   * @param client
   * @param conversation
   * @param members The list of new members
   * @param invitedBy The ID of the inviter; could be the new member itself
   * @since 3.0
   */
    @Override
    public void onMemberJoined(AVIMClient client, AVIMConversation conversation,
        List<String> members, String invitedBy) {
        // Shows that Mary is added to 551260efe4b01608686c3e0f by Tom
        Toast.makeText(AVOSCloud.applicationContext,
          members + " is added to " + conversation.getConversationId() + " by "
              + invitedBy, Toast.LENGTH_SHORT).show();
    }
}
// Set up global event handler
AVIMMessageManager.setConversationEventHandler(new CustomConversationEventHandler());
```
```cs
private void OnMembersJoined(object sender, AVIMOnInvitedEventArgs e)
{
    // e.InvitedBy is the inviter, e.ConversationId is the ID of the conversation
    Debug.Log(string.Format("{0} invited {1} to the conversation {2}", e.InvitedBy,e.JoinedMembers, e.ConversationId));
}
jerry.OnMembersJoined += OnMembersJoined;
```

{{ docs.langSpecStart('js') }}

`payload` contains the following fields:

1. `members`: Array of strings; the list of `clientId`s of the members being added
2. `invitedBy`: String; the `clientId` of the inviter

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('cs') }}

`AVIMOnInvitedEventArgs` contains the following fields:

1. `InvitedBy`: The inviter
2. `JoinedMembers`: The list of members being added
3. `ConversationId`: The conversation

{{ docs.langSpecEnd('cs') }}

Here is the sequence diagram of the operation:

```seq
Tom->Cloud: 1. Adds Mary
Cloud->Tom: 2. Sends notification: you invited Mary to the conversation
Cloud-->Mary: 2. Sends notification: you are added to the conversation by Tom
Cloud-->Jerry: 2. Sends notification: Mary is added to the conversation by Tom
```

On Mary’s side, to know that she is added to the conversation between Tom and Jerry, she can follow the way Jerry listens to the `INVITED` event, which can be found in [## One-on-One Chatting](one-on-one-chatting).

If Tom wants to **create a new conversation with all the members included**, the following code can be used:

```js
tom.createConversation({
  // Add Jerry and Mary to the conversation when creating it; more members can be added later as well
  members: ['Jerry','Mary'],
  // The name of the conversation
  name: 'Tom & Jerry & friends',
  unique: true,
}).catch(console.error);
```
```objc
// Tom creates a conversation with his friends
NSArray *friends = @[@"Jerry", @"Mary"];
[tom createConversationWithName:@"Tom & Jerry & friends" clientIds:friends
  options:AVIMConversationOptionUnique
  callback:^(AVIMConversation *conversation, NSError *error) {
    if (!error) {
        NSLog(@"Conversation created!");
    }
}];
```
```java
tom.createConversation(Arrays.asList("Jerry","Mary"), "Tom & Jerry & friends", null,
   new AVIMConversationCreatedCallback() {
      @Override
      public void done(AVIMConversation conversation, AVIMException e) {
           if (e == null) {
              // Conversation created
           }
      }
   });
```
```cs
var conversation = await tom.CreateConversationAsync(new string[]{ "Jerry","Mary" }, name:"Tom & Jerry & friends", isUnique:true);
```

### Sending Group Messages

In a group chat, if a member sends a message, the message will be delivered to all the online members in the group. The process is the same as how Jerry receives the message from Tom.

For example, if Tom sends a welcoming message to the group:

```js
conversation.send(new TextMessage('Welcome everyone!'));
```
```objc
[conversation sendMessage:[AVIMTextMessage messageWithText:@"Welcome everyone!" attributes:nil] callback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Message sent!");
    }
}];
```
```java
AVIMTextMessage msg = new AVIMTextMessage();
msg.setText("Welcome everyone!");
// Send the message
conversation.sendMessage(msg, new AVIMConversationCallback() {
  @Override
  public void done(AVIMException e) {
    if (e == null) {
      Log.d("Group chat", "Message sent!");
    }
  }
});
```
```cs
var textMessage = new AVIMTextMessage("Welcome everyone!");
await conversation.SendMessageAsync(textMessage);
```

Both Jerry and Mary will have `Event.MESSAGE` event triggered which can be used to retrieve the message and have it displayed on the UI.

### Removing Members

One day Mary spoke something that made Tom angry and Tom wants to kick her out of the group chat. How would Tom do that?

```js
conversation.remove(['Mary']).then(function(conversation) {
  console.log('Member removed!', conversation.members);
}).catch(console.error.bind(console));
```
```objc
[conversation removeMembersWithClientIds:@[@"Mary"] callback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Member removed!");
    }
}];
```
```java
conv.kickMembers(Arrays.asList("Mary"),new AVIMConversationCallback(){
    @Override
    public void done(AVIMException e){
    }
});
```
```cs
await conversation.RemoveMembersAsync("Mary");
```

The following process will be triggered:

```seq
Tom->Cloud: 1. Removes Mary
Cloud-->Mary: 2. Send notification: You are removed by Tom
Cloud-->Jerry: 2. Send notification: Mary is removed by Tom
Cloud-->Tom: 2. Send notification: Mary is removed
```

Here we see that Mary receives `KICKED` which indicates that she (the current user) is removed. Other members (Jerry and Tom) will receive `MEMBERS_LEFT` which indicates that someone else in the conversation is removed. Such events can be handled with the following code:

```js
// Someone else is removed
jerry.on(Event.MEMBERS_LEFT, function membersjoinedEventHandler(payload, conversation) {
    console.log(payload.members, payload.kickedBy, conversation.id);
});
// The current user is removed
jerry.on(Event.KICKED, function membersjoinedEventHandler(payload, conversation) {
    console.log(payload.kickedBy, conversation.id);
});
```
```objc
jerry.delegate = self;

#pragma mark - AVIMClientDelegate
/*!
 Someone else is removed
 @param conversation － The conversation
 @param clientIds - The list of members being removed
 @param clientId - The ID of the operator
 */
- (void)conversation:(AVIMConversation *)conversation membersRemoved:(NSArray<NSString *> * _Nullable)clientIds byClientId:(NSString * _Nullable)clientId {
  ;
}
/*!
 The current user is removed
 @param conversation － The conversation
 @param clientId - The ID of the operator
 */
- (void)conversation:(AVIMConversation *)conversation kickedByClientId:(NSString * _Nullable)clientId {
  ;
}
```
```java
public class CustomConversationEventHandler extends AVIMConversationEventHandler {
  /**
   * Someone else is removed
   *
   * @param client
   * @param conversation
   * @param members The members being removed
   * @param kickedBy The ID of the operator; could be the current user itself
   * @since 3.0
   */
  @Override
  public abstract void onMemberLeft(AVIMClient client,
    AVIMConversation conversation, List<String> members, String kickedBy) {
    Toast.makeText(AVOSCloud.applicationContext,
      members + "  are removed from " + conversation.getConversationId() + " by "
          + kickedBy, Toast.LENGTH_SHORT).show();
  }
  /**
   * The current user is removed
   *
   * @param client
   * @param conversation
   * @param kickedBy The person who removed you
   * @since 3.0
   */
  @Override
  public abstract void onKicked(AVIMClient client, AVIMConversation conversation,
    String kickedBy) {
    Toast.makeText(AVOSCloud.applicationContext,
      "You are removed from " + conversation.getConversationId() + " by "
          + kickedBy, Toast.LENGTH_SHORT).show();
  }
}
// Set up global event handler
AVIMMessageManager.setConversationEventHandler(new CustomConversationEventHandler());
```
```cs
private void OnMembersLeft(object sender, AVIMOnInvitedEventArgs e)
{
    Debug.Log(string.Format("{0} removed {1} from {2}", e.KickedBy, e.JoinedMembers, e.ConversationId));
}
private void OnKicked(object sender, AVIMOnInvitedEventArgs e)
{
    Debug.Log(string.Format("You are removed from {2} by {1}", e.KickedBy, e.ConversationId));
}
jerry.OnMembersLeft += OnMembersLeft;
jerry.OnKicked += OnKicked;
```

### Joining Conversations

Tom is feeling bored after removing Mary. He goes to William and tells him that there is a group chat that Jerry and himself are in. He gives the ID (or name) of the group chat to William which makes him curious about what’s going on in it. William then adds himself to the group:

```js
william.getConversation('CONVERSATION_ID').then(function(conversation) {
  return conversation.join();
}).then(function(conversation) {
  console.log('Successfully joined!', conversation.members);
  // The conversation now contains ['William', 'Tom', 'Jerry']
}).catch(console.error.bind(console));
```
```objc
AVIMConversationQuery *query = [william conversationQuery];
[query getConversationById:@"CONVERSATION_ID" callback:^(AVIMConversation *conversation, NSError *error) {
    [conversation joinWithCallback:^(BOOL succeeded, NSError *error) {
        if (succeeded) {
            NSLog(@"Successfully joined!");
        }
    }];
}];
```
```java
AVIMConversation conv = william.getConversation("CONVERSATION_ID");
conv.join(new AVIMConversationCallback(){
    @Override
    public void done(AVIMException e){
        if(e==null){
          // Successfully joined
        }
    }
});
```
```cs
await william.JoinAsync("CONVERSATION_ID");
```

The following process will be triggered:

```seq
William->Cloud: 1. Joins the conversations
Cloud-->William: 2. Sends notification: you joined the conversation
Cloud-->Tom: 2. Sends notification: William joined the conversation
Cloud-->Jerry: 2. Sends notification: William joined the conversation
```

Other members can listen to `MEMBERS_JOINED` to know that William joined the conversation:

```js
jerry.on(Event.MEMBERS_JOINED, function membersJoinedEventHandler(payload, conversation) {
    console.log(payload.members, payload.invitedBy, conversation.id);
});
```
```objc
- (void)conversation:(AVIMConversation *)conversation membersAdded:(NSArray *)clientIds byClientId:(NSString *)clientId {
    NSLog(@"%@", [NSString stringWithFormat:@"%@ joined the convesation; operated by %@",[clientIds objectAtIndex:0],clientId]);
}
```
```java
public class CustomConversationEventHandler extends AVIMConversationEventHandler {
  @Override
  public void onMemberJoined(AVIMClient client, AVIMConversation conversation,
      List<String> members, String invitedBy) {
      // Shows that William joined 551260efe4b01608686c3e0f; operated by William
      Toast.makeText(AVOSCloud.applicationContext,
        members + " joined " + conversation.getConversationId() + "; operated by "
            + invitedBy, Toast.LENGTH_SHORT).show();
  }
}
```
```cs
private void OnMembersJoined(object sender, AVIMOnInvitedEventArgs e)
{
    // e.InvitedBy is the operator; e.ConversationId is the ID of the conversation
    Debug.Log(string.Format("{0} joined {1}; operated by {2}",e.JoinedMembers, e.ConversationId, e.InvitedBy));
}
jerry.OnMembersJoined += OnMembersJoined;
```

### Leaving Conversations

With more and more people being invited by Tom, Jerry feels that he doesn’t like most of them and wants to leave the conversation. He can do that with `Conversation#quit`:

```js
conversation.quit().then(function(conversation) {
  console.log('You left the conversation!', conversation.members);
}).catch(console.error.bind(console));
```
```objc
[conversation quitWithCallback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"You left the conversation!");
    }
}];
```
```java
conversation.quit(new AVIMConversationCallback(){
    @Override
    public void done(AVIMException e){
      if(e==null){
        // You left the conversation
      }
    }
});
```
```cs
await jerry.LeaveAsync(conversation);
```

After leaving the conversation, Jerry will no longer receive messages from it. Here is the sequence diagram of the operation:

```seq
Jerry->Cloud: 1. Leaves the conversation
Cloud-->Jerry: 2. Sends notification: You left the conversation
Cloud-->Mary: 2. Sends notification: Jerry left the conversation
Cloud-->Tom: 2. Sends notification: Jerry left the conversation
```

Other members can listen to `MEMBERS_LEFT` to know that Jerry left the conversation:

```js
mary.on(Event.MEMBERS_LEFT, function membersLeftEventHandler(payload, conversation) {
    console.log(payload.members, payload.invitedBy, conversation.id);
});
```
```objc
// If Mary is logged in, the following callback will be triggered when Jerry leaves the conversation
-(void)conversation:(AVIMConversation *)conversation membersRemoved:(NSArray *)clientIds byClientId:(NSString *)clientId{
    NSLog(@"%@", [NSString stringWithFormat:@"%@ left the conversation; operated by %@",[clientIds objectAtIndex:0],clientId]);
}
```
```java
public class CustomConversationEventHandler extends AVIMConversationEventHandler {
  @Override
  public void onMemberLeft(AVIMClient client, AVIMConversation conversation, List<String> members,
      String kickedBy) {
      // Things to do after someone left
  }
}
```
```cs
mary.OnMembersLeft += OnMembersLeft;
private void OnMembersLeft(object sender, AVIMOnMembersLeftEventArgs e)
{
    // e.InvitedBy is the operator; e.ConversationId is the ID of the conversation
    Debug.Log(string.Format("{0} left {1}; operated by {2}",e.JoinedMembers, e.ConversationId, e.InvitedBy));
}

# Summary of Event Notifications Regarding Changes of Members

The sequence diagrams displayed earlier already described what would happen when certain events are triggered. The table below serves as a summary of them.

Assuming that Tom and Jerry are already in the conversation:

Operation | Tom | Jerry | Mary | William
--- | --- | --- | ---
Tom invites Mary | `MEMBERS_JOINED` | `MEMBERS_JOINED` | `INVITED` | /
Tom removes Mary | `MEMBERS_LEFT` | `MEMBERS_LEFT` | `KICKED` | /
William joins | `MEMBERS_JOINED` | `MEMBERS_JOINED` | / | `MEMBERS_JOINED`
Jerry leaves | `MEMBERS_LEFT` | `MEMBERS_LEFT` | / | `MEMBERS_LEFT`

## Rich Media Messages

We’ve seen how we can send messages containing plain text. Now let’s see how we can send rich media messages like images, videos, and locations.

By default LeanCloud supports text messages, files, images, audios, videos, locations, and binary data. All of them, except binary data, are sent as strings, though there are some slight differences between text messages and rich media messages (files, images, audios, and videos):

- When sending text messages, the messages themselves are sent directly as strings.
- When sending rich media messages (like images), the SDK will first upload the binary files to the cloud with LeanStorage’s `AVFile` interface, then embed the URLs of them into the messages being sent. We can say that **the essence of an image message is a text message holding the URL of the image**.

> Files stored on LeanStorage have CDN enabled by default. Therefore, binary data (like images) are not directly encoded as part of text messages. This helps users access them faster and the cost on you can be lowered at the same time.

### Default Message Types

The following message types are offered by default:

- `TextMessage` Text message
- `ImageMessage` Image message
- `AudioMessage` Audio message
- `VideoMessage` Video message
- `FileMessage` File message (.txt, .doc, .md, etc.)
- `LocationMessage` Location message

All of them are derived from `AVIMMessage`, with the following properties available for each:

{{ docs.langSpecStart('js') }}

| Name | Type | Description |
| --- | --- | --- |
| `from`        | `String` | The `clientId` of the sender. |
| `cid`         | `String` | The ID of the conversation. |
| `id`          | `String` | A unique ID for each message. Assigned by the cloud automatically. |
| `timestamp`   | `Date`   | The time the message is sent. Assigned by the cloud automatically. |
| `deliveredAt` | `Date`   | The time the message is delivered. Assigned by the cloud automatically. |
| `status`      | `Symbol` | The status of the message. Could be one of the members of [`MessageStatus`](https://leancloud.github.io/js-realtime-sdk/docs/module-leancloud-realtime.html#.MessageStatus:<br/><br/>`MessageStatus.NONE` (unknown)<br/>`MessageStatus.SENDING` (sending)<br/>`MessageStatus.SENT` (sent)<br/>`MessageStatus.DELIVERED` (delivered)<br/>`MessageStatus.FAILED` (failed) |

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('objc') }}

| Name | Type | Description |
| --- | --- | --- |
| `content`            | `NSString`             | The content of the message. |
| `clientId`           | `NSString`             | The `clientId` of the sender. |
| `conversationId`     | `NSString`             | The ID of the conversation. |
| `messageId`          | `NSString`             | A unique ID for each message. Assigned by the cloud automatically. |
| `sendTimestamp`      | `int64_t`              | The time the message is sent. Assigned by the cloud automatically. |
| `deliveredTimestamp` | `int64_t`              | The time the message is delivered. Assigned by the cloud automatically. |
| `status`             | A member of `AVIMMessageStatus` | The status of the message. Could be one of:<br/><br/>`AVIMMessageStatusNone` (unknown)<br/>`AVIMMessageStatusSending` (sending)<br/>`AVIMMessageStatusSent` (sent)<br/>`AVIMMessageStatusDelivered` (delivered)<br/>`AVIMMessageStatusFailed` (failed) |
| `ioType`             | A member of `AVIMMessageIOType` | The direction of the message. Could be one of:<br/><br/>`AVIMMessageIOTypeIn` (sent to the current user)<br/>`AVIMMessageIOTypeOut` (sent by the current user) |

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

| Name | Type | Description |
| --- | --- | --- |
| `content`          | `String`               | The content of the message. |
| `clientId`         | `String`               | The `clientId` of the sender. |
| `conversationId`   | `String`               | The ID of the conversation. |
| `messageId`        | `String`               | A unique ID for each message. Assigned by the cloud automatically. |
| `timestamp`        | `long`                 | The time the message is sent. Assigned by the cloud automatically. |
| `receiptTimestamp` | `long`                 | The time the message is delivered. Assigned by the cloud automatically. |
| `status`           | A member of `AVIMMessageStatus` | The status of the message. Could be one of:<br/><br/>`AVIMMessageStatusNone` (unknown)<br/>`AVIMMessageStatusSending` (sending)<br/>`AVIMMessageStatusSent` (sent)<br/>`AVIMMessageStatusReceipt` (delivered)<br/>`AVIMMessageStatusFailed` (failed) |
| `ioType`           | A member of `AVIMMessageIOType` | The direction of the message. Could be one of:<br/><br/>`AVIMMessageIOTypeIn` (sent to the current user)<br/>`AVIMMessageIOTypeOut` (sent by the current user) |

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

| Name | Type | Description |
| --- | --- | --- |
| `content`          | `String`               | The content of the message. |
| `clientId`         | `String`               | The `clientId` of the sender. |
| `conversationId`   | `String`               | The ID of the conversation. |
| `messageId`        | `String`               | A unique ID for each message. Assigned by the cloud automatically. |
| `timestamp`        | `long`                 | The time the message is sent. Assigned by the cloud automatically. |
| `receiptTimestamp` | `long`                 | The time the message is delivered. Assigned by the cloud automatically. |
| `status`           | A member of `AVIMMessageStatus` | The status of the message. Could be one of:<br/><br/>`AVIMMessageStatusNone` (unknown)<br/>`AVIMMessageStatusSending` (sending)<br/>`AVIMMessageStatusSent` (sent)<br/>`AVIMMessageStatusReceipt` (delivered)<br/>`AVIMMessageStatusFailed` (failed) |
| `ioType`           | A member of `AVIMMessageIOType` | The direction of the message. Could be one of:<br/><br/>`AVIMMessageIOTypeIn` (sent to the current user)<br/>`AVIMMessageIOTypeOut` (sent by the current user) |

{{ docs.langSpecEnd('cs') }}

A number is assigned to each message type which can be used by your app to identify it. Negative numbers are for those defined by the SDK (see the table below) and positive ones are for your own types. `0` is reserved for untyped messages.

Message Type | Number
--- | ---
Text messages | `-1`
Image messages | `-2`
Audio messages | `-3`
Video messages | `-4`
Location messages | `-5`
File messages | `-6`

### Image Messages

#### Sending Image Files

A image message can be constructed from either binary data or a local path. The diagram below shows the sequence of it:

```seq
Tom-->Local: 1. Get the content of the image
Tom-->Storage: 2. The SDK uploads the file (AVFile) to the cloud
Storage-->Tom: 3. Return the URL of the image
Tom-->Cloud: 4. The SDK sends the image message to the cloud
Cloud->Jerry: 5. Receive the image message and display that in UI
```

Notes:
1. The “Local” in the diagram could be `localStorage` or `camera`, meaning that the image could be either from the local storage of the phone (like iPhone’s Camera Roll) or taken in real time with camera API.
2. `AVFile` is the file object used by LeanStorage. See [AVFile](storage_overview.html#avfile) for more details.

The diagram above may look complicated, but the code itself is quite simple since the image gets automatically uploaded when being sent with `send` method:

```js
var AV = require('leancloud-storage');
var { ImageMessage } = require('leancloud-realtime-plugin-typed-messages');

var fileUploadControl = $('#photoFileUpload')[0];
var file = new AV.File('avatar.jpg', fileUploadControl.files[0]);
file.save().then(function() {
  var message = new ImageMessage(file);
  message.setText('Sent via Ins.');
  message.setAttributes({ location: 'San Francisco' });
  return conversation.send(message);
}).then(function() {
  console.log('Sent!');
}).catch(console.error.bind(console));
```
```objc
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *documentsDirectory = [paths objectAtIndex:0];
NSString *imagePath = [documentsDirectory stringByAppendingPathComponent:@"LeanCloud.png"];
NSError *error;
AVFile *file = [AVFile fileWithLocalPath:imagePath error:&error];
AVIMImageMessage *message = [AVIMImageMessage messageWithText:@"She is sweet." file:file attributes:nil];
[conversation sendMessage:message callback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Sent!");
    }
}];
```
```java
AVFile file = AVFile.withAbsoluteLocalPath("San_Francisco.png", Environment.getExternalStorageDirectory() + "/San_Francisco.png");
// Create an image message
AVIMImageMessage m = new AVIMImageMessage(file);
m.setText("Sent via Android.");
conv.sendMessage(m, new AVIMConversationCallback() {
  @Override
  public void done(AVIMException e) {
    if (e == null) {
      // Sent
    }
  }
});
```
```cs
var image = new AVFile("screenshot.png", Path.Combine(Application.persistentDataPath, "screenshot.PNG"));
var imageMessage = new AVIMImageMessage();
imageMessage.File = image;
imageMessage.TextContent = "Sent via Windows.";
await conversation.SendMessageAsync(imageMessage);
```

#### Sending Image URLs

Beside sending an image directly, a user may also copy the URL of an image from somewhere else and send it to a conversation:

```js
var AV = require('leancloud-storage');
var { ImageMessage } = require('leancloud-realtime-plugin-typed-messages');
// Create an image message from URL
var file = new AV.File.withURL('cute-girl', 'http://pic2.zhimg.com/6c10e6053c739ed0ce676a0aff15cf1c.gif');
file.save().then(function() {
  var message = new ImageMessage(file);
  message.setText('She is sweet.');
  return conversation.send(message);
}).then(function() {
  console.log('Sent!');
}).catch(console.error.bind(console));
```
```objc
// Tom sends an image to Jerry
AVFile *file = [AVFile fileWithURL:[self @"http://ww3.sinaimg.cn/bmiddle/596b0666gw1ed70eavm5tg20bq06m7wi.gif"]];
AVIMImageMessage *message = [AVIMImageMessage messageWithText:@"She is sweet." file:file attributes:nil];
[conversation sendMessage:message callback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Sent!");
    }
}];
```
```java
AVFile file =new AVFile("cute-girl","http://ww3.sinaimg.cn/bmiddle/596b0666gw1ed70eavm5tg20bq06m7wi.gif", null);
AVIMImageMessage m = new AVIMImageMessage(file);
m.setText("She is sweet.");
// Create an image message
conv.sendMessage(m, new AVIMConversationCallback() {
    @Override
    public void done(AVIMException e) {
      if (e == null) {
        // Sent
      }
    }
});
```
```cs
var image = new AVFile("Satomi_Ishihara.gif", "http://ww3.sinaimg.cn/bmiddle/596b0666gw1ed70eavm5tg20bq06m7wi.gif");
var imageMessage = new AVIMImageMessage();
imageMessage.File = image;
imageMessage.TextContent = "Sent via Windows.";
await conversation.SendMessageAsync(imageMessage);
```

#### Receiving Image Messages

The way to receive image messages is similar to that for basic messages. The only thing that needs to be added is to have the callback function retrieve the image and render it on the UI. For example:

```js
var { Event, TextMessage } = require('leancloud-realtime');
var { ImageMessage } = require('leancloud-realtime-plugin-typed-messages');

client.on(Event.MESSAGE, function messageEventHandler(message, conversation) {
   var file;
   switch (message.type) {
      case ImageMessage.TYPE:
        file = message.getFile();
        console.log('Image received. URL: ' + file.url());
        break;
   }
}
```
```objc
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    AVIMImageMessage *imageMessage = (AVIMImageMessage *)message;

    // The ID of the message
    NSString *messageId = imageMessage.messageId;
    // The URL of the image file
    NSString *imageUrl = imageMessage.file.url;
    // The clientId of the sender
    NSString *fromClientId = message.clientId;
}
```
```java
AVIMMessageManager.registerMessageHandler(AVIMImageMessage.class,
    new AVIMTypedMessageHandler<AVIMImageMessage>() {
        @Override
        public void onMessage(AVIMImageMessage msg, AVIMConversation conv, AVIMClient client) {
            // Only handle messages from Jerry
            // sent to the conversation with conversationId 55117292e4b065f7ee9edd29
            if ("Jerry".equals(client.getClientId()) && "55117292e4b065f7ee9edd29".equals(conv.getConversationId())) {
                String fromClientId = msg.getFrom();
                String messageId = msg.getMessageId();
                String url = msg.getFileUrl();
                Map<String, Object> metaData = msg.getFileMetaData();
                if (metaData.containsKey("size")) {
                  int size = (Integer) metaData.get("size");
                }
                if (metaData.containsKey("width")) {
                  int width = (Integer) metaData.get("width");
                }
                if (metaData.containsKey("height")) {
                  int height = (Integer) metaData.get("height");
                }
                if (metaData.containsKey("format")) {
                  String format = (String) metaData.get("format");
                }
            }
        }
});
```
```cs
private void OnMessageReceived(object sender, AVIMMessageEventArgs e)
{
    if (e.Message is AVIMImageMessage imageMessage)
    {
        AVFile file = imageMessage.File;
        Debug.Log(file.Url);
    }
}
```

### Sending Audios, Videos, and Files

#### The Flow

The SDK follows the steps below to send images, audios, videos, and files:

When **constructing files from data streams using client API**:

1. Construct a local `AVFile`
2. Upload the `AVFile` to the cloud and retrieve its `metaData`
3. Embed the `objectId`, URL, and metadata of the file into the message
4. Send the message

When **constructing files with URLs**:

1. Embed the URL into the message without metadata (like the length of audio) or `objectId`
2. Send the message

For example, when sending an audio message, the basic flow would be: read the audio file (or record a new one) > construct an audio message > send the message.

```js
var AV = require('leancloud-storage');
var { AudioMessage } = require('leancloud-realtime-plugin-typed-messages');

var fileUploadControl = $('#photoFileUpload')[0];
var file = new AV.File('never-gonna-give-you-up.mp3', fileUploadControl.files[0]);
file.save().then(function() {
  var message = new AudioMessage(file);
  message.setText('I heard this song became a meme.');
  return conversation.send(message);
}).then(function() {
  console.log('Sent!');
}).catch(console.error.bind(console));
```
```objc
NSError *error = nil;
AVFile *file = [AVFile fileWithLocalPath:localPath error:&error];
if (!error) {
    AVIMAudioMessage *message = [AVIMAudioMessage messageWithText:@"I heard this song became a meme." file:file attributes:nil];
    [conversation sendMessage:message callback:^(BOOL succeeded, NSError *error) {
        if (succeeded) {
            NSLog(@"Sent!");
        }
    }];
}
```
```java
AVFile file = AVFile.withAbsoluteLocalPath("never-gonna-give-you-up.mp3",localFilePath);
AVIMAudioMessage m = new AVIMAudioMessage(file);
m.setText("I heard this song became a meme.");
// Create an audio message
conv.sendMessage(m, new AVIMConversationCallback() {
    @Override
    public void done(AVIMException e) {
      if (e == null) {
        // Sent
      }
    }
});
```
```cs
var audio = new AVFile("tante.mp3", Path.Combine(Application.persistentDataPath, "tante.mp3"));
var audioMessage = new AVIMAudioMessage();
audioMessage.File = audio;
audioMessage.TextContent = "I heard this song became a meme.";
await conversation.SendMessageAsync(audioMessage);
```

Similar to image messages, you can construct audio messages from URLs as well:

```js
var AV = require('leancloud-storage');
var { AudioMessage } = require('leancloud-realtime-plugin-typed-messages');

var file = new AV.File.withURL('apple.acc', 'https://some.website.com/apple.acc');
file.save().then(function() {
  var message = new AudioMessage(file);
  message.setText('Here is the recording from Apple Special Event.');
  return conversation.send(message);
}).then(function() {
  console.log('Sent!');
}).catch(console.error.bind(console));
```
```objc
AVFile *file = [AVFile fileWithRemoteURL:[NSURL URLWithString:@"https://some.website.com/apple.acc"]];
AVIMAudioMessage *message = [AVIMAudioMessage messageWithText:@"Here is the recording from Apple Special Event." file:file attributes:nil];
[conversation sendMessage:message callback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Sent!");
    }
}];
```
```java
AVFile file =new AVFile("apple.acc", "https://some.website.com/apple.acc", null);
AVIMAudioMessage m = new AVIMAudioMessage(file);
m.setText("Here is the recording from Apple Special Event.");
conv.sendMessage(m, new AVIMConversationCallback() {
    @Override
    public void done(AVIMException e) {
      if (e == null) {
        // Sent
      }
    }
});
```
```cs
var audio = new AVFile("apple.acc", "https://some.website.com/apple.acc");
var audioMessage = new AVIMAudioMessage();
audioMessage.File = audio;
audioMessage.TextContent = "Here is the recording from Apple Special Event.";
await conversation.SendMessageAsync(audioMessage);
```

### Sending Location Messages

The code below sends a message containing a location:

```js
var AV = require('leancloud-storage');
var { LocationMessage } = require('leancloud-realtime-plugin-typed-messages');

var location = new AV.GeoPoint(31.3753285,120.9664658);
var message = new LocationMessage(location);
message.setText('Here is the location of the bakery.');
conversation.send(message).then(function() {
  console.log('Sent!');
}).catch(console.error.bind(console));
```
```objc
AVIMLocationMessage *message = [AVIMLocationMessage messageWithText:@"Here is the location of the bakery." latitude:31.3753285 longitude:120.9664658 attributes:nil];
[conversation sendMessage:message callback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Sent!");
    }
}];
```
```java
final AVIMLocationMessage locationMessage=new AVIMLocationMessage();
// The location here is hardcoded for demonstration; you can get actual locations with the API offered by the device
locationMessage.setLocation(new AVGeoPoint(31.3753285,120.9664658));
locationMessage.setText("Here is the location of the bakery.");
conversation.sendMessage(locationMessage, new AVIMConversationCallback() {
    @Override
    public void done(AVIMException e) {
        if (null != e) {
          e.printStackTrace();
        } else {
          // Sent
        }
    }
});
```
```cs
var locationMessage = new AVIMLocationMessage();
locationMessage.Location = new AVGeoPoint(31.3753285, 120.9664658);
await conversation.SendMessageAsync(locationMessage);
```

### Back to Receiving Messages

{{ docs.langSpecStart('js') }}

When a new message comes in, no matter what type of message it is, the JavaScript SDK would always trigger the callback set for the event `Event.MESSAGE` on `IMClient`. You can address different types of messages in different ways within the callback function.

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('objc') }}

The Objective-C SDK handles new messages with `AVIMClientDelegate` and uses two separate methods to handle basic messages (`AVIMMessage`) and rich media messages (`AVIMTypedMessage`; including messages with custom types):

```
/*!
 New basic message received.
 @param conversation － The conversation.
 @param message - The content of the message.
 */
- (void)conversation:(AVIMConversation *)conversation didReceiveCommonMessage:(AVIMMessage *)message;

/*!
 New rich media message received.
 @param conversation － The conversation.
 @param message - The content of the message.
 */
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message;
```

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

The Java/Android SDK handles new messages with `AVIMMessageHandler`. You can register your own message handlers by calling `AVIMMessageManager#registerDefaultMessageHandler`. `AVIMMessageManager` offers two different methods for you to register default message handlers and handlers for specific message types:

```
/**
 * Register default message handler.
 *
 * @param handler
 */
public static void registerDefaultMessageHandler(AVIMMessageHandler handler);
/**
 * Register handler for specific message type.
 *
 * @param clazz The message type
 * @param handler
 */
public static void registerMessageHandler(Class<? extends AVIMMessage> clazz, MessageHandler<?> handler);
/**
 * Deregister handler for specific message type.
 *
 * @param clazz
 * @param handler
 */
public static void unregisterMessageHandler(Class<? extends AVIMMessage> clazz, MessageHandler<?> handler);
```

Different handlers can be registered or deregistered for different message types (including those defined by yourself). These handles should be set up when initializing the app.

If you register `defaultMessageHandler` directly in `AVIMMessageManager` for multiple times, only the last one would work. However, if you register `AVIMMessageHandler` through `registerMessageHandler`, different handlers could coexist with each other.

When a message is received by the client, the SDK would:

- Detect the type of the message, look for all the handlers registered for this type, and call the `onMessage` functions within all these handlers.
- If no handler is found for this type, `defaultHandler` will be triggered.

So when handlers are specified for `AVIMTypedMessage` (and its subtypes) and a global `defaultHandler` is also specified, if the sender sends a general `AVIMMessage` message, the receiver will have its handler in `AVIMMessageManager#registerDefaultMessageHandler()` triggered; if the sender sends a message of `AVIMTypedMessage` (or its subtype), the receiver will have its handler in `AVIMMessageManager#registerMessageHandler()` triggered.

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

The C# SDK handles new messages with `OnMessageReceived`. There are **two levels** here:

* The first level is applied to `AVIMClient` which serves to get you notified that there are messages coming in. This is especially helpful when there is no local `AVIMConversation` (like when a user just logged in) and a new message is sent to a conversation involving the current user. `AVIMClient.OnMessageReceived` would take in the messages, but has nothing to do with the type of the message.
* The second level is applied to `AVIMConversation` which serves to receive the actual messages and perform different operations for different types of messages.

The table below illustrates how these two levels work. Assuming that we are receiving an `AVIMTextMessage`:

Receiver’s `AVIMClient` | Condition 1 | Condition 2 | Condition 3 | Condition 4 | Condition 5
--- | --- | --- | --- | --- | ---
`AVIMClient.OnMessageReceived` | × | √ | √ | √ | √
`AVIMConversation.OnMessageReceived` | × | × | √ | × | ×
`AVIMConversation.OnTypedMessageReceived` | × | × | × | √ | ×
`AVIMConversation.OnTextMessageReceived` | × | × | × | × | √

Below are the conditions:

Condition 1:

```c#
AVIMClient.Status != Online
``` 

Condition 2:

```c#
   AVIMClient.Status == Online 
&& AVIMClient.OnMessageReceived != null
```

Condition 3:

```c#
   AVIMClient.Status == Online 
&& AVIMClient.OnMessageReceived != null 
&& AVIMConversation.OnMessageReceived != null
```

Condition 4:

```c#
   AVIMClient.Status == Online 
&& AVIMClient.OnMessageReceived != null 
&& AVIMConversation.OnMessageReceived != null
&& AVIMConversation.OnTypedMessageReceived != null
&& AVIMConversation.OnTextMessageReceived == null
```

Condition 5:

```c#
   AVIMClient.Status == Online 
&& AVIMClient.OnMessageReceived != null 
&& AVIMConversation.OnMessageReceived != null
&& AVIMConversation.OnTypedMessageReceived != null
&& AVIMConversation.OnTextMessageReceived != null
```

In `AVIMConversation`, the process of receiving a message is:

`OnTextMessageReceived` > `OnTypedMessageReceived` > `OnMessageReceived`

Such design allows you to assign different operations to different layers. It works for other types of rich media messages as well.

{{ docs.langSpecEnd('cs') }}

Below is an example:

```js
// Load TypedMessagesPlugin when initializing Realtime
// var realtime = new Realtime({
//   appId: appId,
//   plugins: [TypedMessagesPlugin]
// });
var { Event, TextMessage } = require('leancloud-realtime');
var { FileMessage, ImageMessage, AudioMessage, VideoMessage, LocationMessage } = require('leancloud-realtime-plugin-typed-messages');
// Register handler for MESSAGE event
client.on(Event.MESSAGE, function messageEventHandler(message, conversation) {
  // Your logic here
  var file;
  switch (message.type) {
    case TextMessage.TYPE:
      console.log('Text message received. Text: ' + message.getText() + ', msgId: ' + message.id);
      break;
    case FileMessage.TYPE:
      file = message.getFile(); // file is an AV.File instance
      console.log('File message received. URL: ' + file.url() + ', size: ' + file.metaData('size'));
      break;
    case ImageMessage.TYPE:
      file = message.getFile();
      console.log('Image message received. URL: ' + file.url() + ', width: ' + file.metaData('width'));
      break;
    case AudioMessage.TYPE:
      file = message.getFile();
      console.log('Audio message received. URL: ' + file.url() + ', width: ' + file.metaData('duration'));
      break;
    case VideoMessage.TYPE:
      file = message.getFile();
      console.log('Video message received. URL: ' + file.url() + ', width: ' + file.metaData('duration'));
      break;
    case LocationMessage.TYPE:
      var location = message.getLocation();
      console.log('Location message received. Latitude: ' + location.latitude + ', Longitude: ' + location.longitude);
      break;
    default:
      console.warn('Unknown message received.');
  }
});

// `MESSAGE` event will be triggered on conversation as well
conversation.on(Event.MESSAGE, function messageEventHandler(message) {
  // Your logic here
});
```
```objc
// Handle built-in typed message
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    // For example: received image message
    if (message.mediaType == kAVIMMessageMediaTypeImage) {
        AVIMImageMessage *imageMessage = (AVIMImageMessage *)message;
        // Handle image message
    } else if(message.mediaType == kAVIMMessageMediaTypeAudio){
        // Handle audio message
    } else if(message.mediaType == kAVIMMessageMediaTypeVideo){
        // Handle video message
    } else if(message.mediaType == kAVIMMessageMediaTypeLocation){
        // Handle location message
    } else if(message.mediaType == kAVIMMessageMediaTypeFile){
        // Handle file message
    } else if(message.mediaType == kAVIMMessageMediaTypeText){
        // Handle text message
    }
}

// Handle customize typed message

// 1. Register subclass
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [AVIMCustomMessage registerSubclass];
}
// 2. Received message
- (void)conversation:(AVIMConversation *)conversation didReceiveTypedMessage:(AVIMTypedMessage *)message {
    if (message.mediaType == 1) {
        AVIMCustomMessage *imageMessage = (AVIMCustomMessage *)message;
        // Handle image message
    }
}
```
```java
// 1. Register default handler
AVIMMessageManager.registerDefaultMessageHandler(new AVIMMessageHandler(){
    public void onMessage(AVIMMessage message, AVIMConversation conversation, AVIMClient client) {
      // Receive new-coming message
    }

    public void onMessageReceipt(AVIMMessage message, AVIMConversation conversation, AVIMClient client) {
      // Do something responding of message receipt event
    }
});
// 2. Register typed message handler
AVIMMessageManager.registerMessageHandler(AVIMTypedMessage.class, new AVIMTypedMessageHandler<AVIMTypedMessage>(){
    public void onMessage(AVIMTypedMessage message, AVIMConversation conversation, AVIMClient client) {
    switch (message.getMessageType()) {
        case AVIMMessageType.TEXT_MESSAGE_TYPE:
        // Do something
        AVIMTextMessage textMessage = (AVIMTextMessage)message;
        break;
        case AVIMMessageType.IMAGE_MESSAGE_TYPE:
        // Do something
        AVIMImageMessage imageMessage = (AVIMImageMessage)message;
        break;
        case AVIMMessageType.AUDIO_MESSAGE_TYPE:
        // Do something
        AVIMAudioMessage audioMessage = (AVIMAudioMessage)message;
        break;
        case AVIMMessageType.VIDEO_MESSAGE_TYPE:
        // Do something
        AVIMVideoMessage videoMessage = (AVIMVideoMessage)message;
        break;
        case AVIMMessageType.LOCATION_MESSAGE_TYPE:
        // Do something
        AVIMLocationMessage locationMessage = (AVIMLocationMessage)message;
        break;
        case AVIMMessageType.FILE_MESSAGE_TYPE:
        // Do something
        AVIMFileMessage fileMessage = (AVIMFileMessage)message;
        break;
        case AVIMMessageType.RECALLED_MESSAGE_TYPE:
        // Do something
        AVIMRecalledMessage recalledMessage = (AVIMRecalledMessage)message;
        break;
        default:
        // UnsupportedMessageType
        break;
    }
    }

    public void onMessageReceipt(AVIMTypedMessage message, AVIMConversation conversation, AVIMClient client) {
    // Do something responding of message receipt event
    }
});

// Handle customize typed message
public class CustomMessage extends AVIMMessage {
  
}

AVIMMessageManager.registerMessageHandler(CustomMessage.class, new MessageHandler<CustomMessage>(){
  public void onMessage(CustomMessage message, AVIMConversation conversation, AVIMClient client) {
    // Receive new-coming message
  }

  public void onMessageReceipt(CustomMessage message, AVIMConversation conversation, AVIMClient client){
    // Do something responding of message receipt event
  }
});
```
```cs
// Here is a simple demo; you may use switch/case instead
private void OnMessageReceived(object sender, AVIMMessageEventArgs e)
{
    if (e.Message is AVIMImageMessage imageMessage)
    {

    }
    else if (e.Message is AVIMAudioMessage audioMessage)
    {

    }
    else if (e.Message is AVIMVideoMessage videoMessage)
    {

    }
    else if (e.Message is AVIMFileMessage fileMessage)
    {

    }
    else if (e.Message is AVIMLocationMessage locationMessage)
    {

    }
    else if (e.Message is AVIMTypedMessage baseTypedMessage)
    {

    } // Attach more conditions for custom types
}
```

## Custom Attributes

A `Conversation` object holds some built-in properties which match the fields in the `_Conversation` table. The table below shows these **built-in** properties:

{{ docs.langSpecStart('js') }}

| Property of `Conversation` | Field in `_Conversation` | Description |
| --- | --- | --- |
| `id` | `objectId` | A globally unique ID. |
| `name` | `name` | The name of the conversation. Shared by all members. |
| `members` | `m` | The list of members. |
| `creator` | `c` | The creator of the conversation. |
| `transient` | `tr` | Whether it is a chat room. |
| `system` | `sys` | Whether it is a system conversation. |
| `mutedMembers` | `mu` | The list of members that muted the conversation. |
| `muted` | N/A | Whether the current user muted the conversation. |
| `createdAt` | `createdAt` | The time the conversation is created. |
| `updatedAt` | `updatedAt` | The time the conversation is updated. |
| `lastMessageAt` | `lm` | The time the last message is sent. |
| `lastMessage` | N/A | The last message. Could be empty. |
| `unreadMessagesCount` | N/A | The number of unread messages. |
| `lastDeliveredAt` | N/A | The time the last message being delivered is sent (for one-on-one chatting only). |
| `lastReadAt` | N/A | The time the last message being read is sent (for one-on-one chatting only). |

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('objc') }}

| Property of `AVIMConversation` | Field in `_Conversation` | Description |
| --- | --- | --- |
| `conversationId` | `objectId` | A globally unique ID. |
| `name` | `name` | The name of the conversation. Shared by all members. |
| `members` | `m` | The list of members. |
| `creator` | `c` | The creator of the conversation. |
| `attributes` | `attr` | Custom attributes. |
| `transient` | `tr` | Whether it is a chat room. |
| `createdAt` | `createdAt` | The time the conversation is created. |
| `updatedAt` | `updatedAt` | The time the conversation is updated. |
| `system` | `sys` | Whether it is a system conversation. |
| `lastMessageAt` | `lm` | The time the last message is sent. |
| `lastMessage` | N/A | The last message. Could be empty. |
| `muted` | N/A | Whether the current user muted the conversation. |
| `unreadMessagesCount` | N/A | The number of unread messages. |
| `lastDeliveredAt` | N/A | The time the last message being delivered is sent (for one-on-one chatting only). |
| `lastReadAt` | N/A | The time the last message being read is sent (for one-on-one chatting only). |

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

| Property of `AVIMConversation` | Field in `_Conversation` | Description |
| --- | --- | --- |
| `conversationId` | `objectId` | A globally unique ID. |
| `name` | `name` | The name of the conversation. Shared by all members. |
| `members` | `m` | The list of members. |
| `creator` | `c` | The creator of the conversation. |
| `attributes` | `attr` | Custom attributes. |
| `isTransient` | `tr` | Whether it is a chat room. |
| `lastMessageAt` | `lm` | The time the last message is sent. |
| `lastMessage` | N/A | The last message. Could be empty. |
| `muted` | N/A | Whether the current user muted the conversation. |
| `unreadMessagesCount` | N/A | The number of unread messages. |
| `lastDeliveredAt` | N/A | The time the last message being delivered is sent (for one-on-one chatting only). |
| `lastReadAt` | N/A | The time the last message being read is sent (for one-on-one chatting only). |
| `createdAt` | `createdAt` | The time the conversation is created. |
| `updatedAt` | `updatedAt` | The time the conversation is updated. |
| `system` | `sys` | Whether it is a system conversation. |

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

| Property of `AVIMConversation` | Field in `_Conversation` | Description |
| --- | --- | --- |
| `Id` | `objectId` | A globally unique ID. |
| `Name` | `name` | The name of the conversation. Shared by all members. |
| `MemberIds` | `m` | The list of members. |
| `MuteMemberIds` | `mu` | The list of members that muted the conversation. |
| `Creator` | `c` | The creator of the conversation. |
| `IsTransient` | `tr` | Whether it is a chat room. |
| `IsUnique` | `unique` | If this is `true`, the same conversation will be reused when a new conversation is created with the same composition of members and `unique` to be `true`. |
| `IsSystem` | `sys` | Whether it is a system conversation. |
| `LastMessageAt` | `lm` | The time the last message is sent. |
| `LastMessage` | N/A | The last message. Could be empty. |
| `LastDeliveredAt` | N/A | The time the last message being delivered is sent (for one-on-one chatting only). |
| `LastReadAt` | N/A | The time the last message being read is sent (for one-on-one chatting only). |
| `CreatedAt` | `createdAt` | The time the conversation is created. |
| `UpdatedAt` | `updatedAt` | The time the conversation is updated. |

{{ docs.langSpecEnd('cs') }}

Beside these built-in properties, you can also define your custom attributes to store more data with each conversation.

### Creating Custom Attributes

When introducing [one-on-one conversations](#creating-conversations), we mentioned that `IMClient#createConversation` allows you to attach custom attributes to a conversation. Now let’s see how we can do that.

Assume that we need to add two properties `{ "type": "private", "pinned": true }` to a conversation we are creating. We can do so by passing in the properties when calling `IMClient#createConversation`:

```js
tom.createConversation({
  members: ['Jerry'],
  name: 'Tom & Jerry',
  unique: true,
  type: 'private',
  pinned: true,
}).then(function(conversation) {
  console.log('Conversation created! ID: ' + conversation.id);
}).catch(console.error.bind(console));
```
```objc
// Tom creates a conversation named “Tom & Jerry” and attaches custom attributes to it
NSDictionary *attributes = @{ 
    @"type": @"private",
    @"pinned": @(YES) 
};
[tom createConversationWithName:@"Tom & Jerry" clientIds:@[@"Jerry"] attributes:attributes options:AVIMConversationOptionUnique callback:^(AVIMConversation *conversation, NSError *error) {
    if (succeeded) {
        NSLog(@"Conversation created!");
    }
}];
```
```java
HashMap<String,Object> attr = new HashMap<String,Object>();
attr.put("type","private");
attr.put("pinned",true);
client.createConversation(Arrays.asList("Jerry"),"Tom & Jerry", attr, false, true,
    new AVIMConversationCreatedCallback(){
        @Override
        public void done(AVIMConversation conv,AVIMException e){
          if(e==null){
            // Conversation created
          }
        }
    });
```
```cs
vars options = new Dictionary<string, object>();
options.Add("type", "private");
options.Add("pinned",true);
var conversation = await tom.CreateConversationAsync("Jerry", name:"Tom & Jerry", isUnique:true, options:options);
```

**The SDK allows everyone in a conversation to access its custom attributes.** You can also query conversations by custom attributes. See [Querying Conversations with Advanced Conditions](#querying-conversations-with-advanced-conditions).

### Updating and Retrieving Properties

The built-in properties (like `name`) of a `Conversation` object can be updated by all the members unless you set restrictions in your app:

```js
conversation.name = 'Tom is Smart';
conversation.save();
```
```objc
conversation[@"name"] = @"Tom is Smart";
[conversation updateWithCallback:^(BOOL succeeded, NSError * _Nullable error) {
    if (succeeded) {
        NSLog(@"Updated!");
    }
}];
```
```java
AVIMConversation conversation = client.getConversation("55117292e4b065f7ee9edd29");
conversation.setName("Tom is Smart");
conversation.updateInfoInBackground(new AVIMConversationCallback(){
  @Override
  public void done(AVIMException e){        
    if(e==null){
    // Updated
    }
  }
});
```
```cs
conversation.Name = "Tom is Smart";
await conversation.SaveAsync();
```

Custom attributes can also be retrieved or updated by all the members:

```js
// Retrieve custom attribute
var type = conversation.get('attr.type');
// Set new value for pinned
conversation.set('attr.pinned',false);
// Save
conversation.save();
```
```objc
// Retrieve custom attribute
NSString *type = [conversation objectForKey:@"attr.type"];
// Set new value for pinned
[conversation setObject:@(NO) forKey:@"attr.pinned"];
// Save
[conversation updateWithCallback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"Saved!");
    }
}];
```
```java
// Retrieve custom attribute
String type = conversation.get("attr.type");
// Set new value for pinned
conversation.set("attr.pinned",false);
// Save
conversation.updateInfoInBackground(new AVIMConversationCallback(){
  @Override
  public void done(AVIMException e){        
    if(e==null){
    // Saved
    }
  }
});
```
```cs
// Retrieve custom attribute
var type = conversation["attr.type"];
// Set new value for pinned
conversation["attr.pinned"] = false;
// Save
await conversation.SaveAsync();
```

> Notes about custom attributes:
>
> The custom attributes specified with `IMClient#createConversation` will be stored in the field `attr` of the `_Conversation` table. If you need to retrieve or update them later, the full path needs to be specified, like `attr.type`.

### Synchronization of Properties

The properties of a conversation (like name) are shared by everyone in it. If someone ever changes a property, other members need to get updated on it. In the example we used earlier, a user changed the name of a conversation to “Tom is Smart”. How would other members get to know about it?

LeanMessage offers the mechanism that automatically delivers the change made by a user to a conversation to all the members in it (for those who are offline, they will receive updates once they get online):


```js
/**
 * The properties of a conversation are updated
 * @event IMClient#CONVERSATION_INFO_UPDATED
 * @param {Object} payload
 * @param {Object} payload.attributes The properties being updated
 * @param {String} payload.updatedBy The ID of the operator
 */
var { Event } = require('leancloud-realtime');
client.on(Event.CONVERSATION_INFO_UPDATED, function(payload) {
});
```
```objc
/**
 The properties of a conversation are updated
 
 @param conversation The conversation
 @param date The time of the update
 @param clientId The ID of the operator
 @param data The data being updated
 */
- (void)conversation:(AVIMConversation *)conversation didUpdateAt:(NSDate * _Nullable)date byClientId:(NSString * _Nullable)clientId updatedData:(NSDictionary * _Nullable)data;
```
```java
// The following definition exists in AVIMConversationEventHandler
/**
 * The properties of a conversation are updated
 *
 * @param client
 * @param conversation
 * @param attr      The properties being updated.
 * @param operator  The ID of the operator
 */
public void onInfoChanged(AVIMClient client, AVIMConversation conversation, JSONObject attr,
                          String operator)
```
```cs
// Not supported yet
```

> Notes:
>
> You can either retrieve the properties being updated from the callback function or directly read the latest values from the `Conversation` object.

### Retrieving Member Lists

To get the list of members in a conversation, we can call the method for fetching on a `Conversation` object and then get the result from it:

```js
conversation.fetch().then(function(conversation) {
  console.log('members: ', conversation.members);
).catch(console.error.bind(console));
```
```objc
[conversation fetchWithCallback:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        NSLog(@"", conversation.members);
    }
}];
```
```java
conversation.fetchInfoInBackground(new AVIMConversationCallback() {
  @Override
  public void done(AVIMException e) {
    if (e == null) {
      conversation.getMembers();
    }
  }
});
```
```cs
// Not supported yet
```

> Notes:
>
> You can only get member lists of **basic conversations**. Chat rooms and system conversations don’t have member lists.

## Querying Conversations with Advanced Conditions

There are more ways to get a `Conversation` beside listening to incoming events. You might want your users to search chat rooms by the names or locations of them, or to look for conversations that has certain members in them. All these requirements can be satisfied with the help of queries.

### Queries on ID

Here ID refers to the `objectId` in the `_Conversation` table. Since IDs are indexed, querying by ID is the easiest and most efficient way to look for a conversation:

```js
tom.getConversation('551260efe4b01608686c3e0f').then(function(conversation) {
  console.log(conversation.id);
}).catch(console.error.bind(console));
```
```objc
AVIMConversationQuery *query = [tom conversationQuery];
[query getConversationById:@"551260efe4b01608686c3e0f" callback:^(AVIMConversation *conversation, NSError *error) {
    if (succeeded) {
        NSLog(@"Query completed!");
    }
}];
```
```java
AVIMConversationsQuery query = tom.getConversationsQuery();
query.whereEqualTo("objectId","551260efe4b01608686c3e0f");
query.findInBackground(new AVIMConversationQueryCallback(){
    @Override
    public void done(List<AVIMConversation> convs,AVIMException e){
      if(e==null){
        if(convs!=null && !convs.isEmpty()){
          // convs.get(0) is the conversation being found
        }
      }
    }
});
```
```cs
var query = tom.GetQuery();
var conversation = await query.GetAsync("551260efe4b01608686c3e0f");
```

### Querying by Conditions

LeanMessage offers a variety of ways for you to look for conversations that satisfy certain conditions.

Let’s start with `equalTo` which is the simplest method for querying conversations. The code below looks for all the conversations that have `type` (a string field) to be `private`:

```js
var query = client.getQuery();
query.equalTo('attr.type','private');
query.find().then(function(conversations) {
  // conversations contains all the results
}).catch(console.error.bind(console));
```
```objc
AVIMConversationQuery *query = [tom conversationQuery];
[query whereKey:@"attr.type" equalTo:@"private"];
// Perform query
[query findConversationsWithCallback:^(NSArray *objects, NSError *error) {
    NSLog(@"%ld conversations found!", [objects count]);
}];
```
```java
AVIMConversationsQuery query = tom.getConversationsQuery();
query.whereEqualTo("attr.type","private");
// Perform query
query.findInBackground(new AVIMConversationQueryCallback(){
  @Override
  public void done(List<AVIMConversation> convs,AVIMException e){
    if(e == null){
      // convs contains all the results
    }
  }
});
```
```cs
// Since WhereXXX returns a new query instance each time, the code below will not work:
//   var query = tom.GetQuery();
//   query.WhereEqualTo("attr.type","private");
// You can use this way:
//   var query = tom.GetQuery();
//   query = query.WhereEqualTo("attr.type","private");
// This way is more recommended:
//   var query = tom.GetQuery().WhereEqualTo("attr.type","private");
var query = tom.GetQuery().WhereEqualTo("attr.type","private");
await query.FindAsync();
```

The interface for querying conversations is very similar to that for querying objects in LeanStorage. If you’re already familiar with LeanStorage, it shouldn’t be hard for you to learn how to query conversations:

- You can get query results with `find`
- You can get number of results with `count`
- You can get the first conversation satisfying conditions with `first`
- You can implement pagination with `skip` and `limit`

You can also apply conditions like “greater than”, “greater than or equal to”, “less than”, and “less than or equal to” to `Number` and `Date` fields:

{{ docs.langSpecStart('js') }}

| Logic | `ConversationQuery` Method |
| --- | --- |
| Equal to | `equalTo` |
| Not equal to | `notEqualTo` |
| Greater than | `greaterThan` |
| Greater than or equal to | `greaterThanOrEqualTo` |
| Less than | `lessThan` |
| Less than or equal to | `lessThanOrEqualTo` |

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('objc') }}

| Logic | `AVIMConversationQuery` Method |
| --- | --- |
| Equal to | `equalTo` |
| Not equal to | `notEqualTo` |
| Greater than | `greaterThan` |
| Greater than or equal to | `greaterThanOrEqualTo` |
| Less than | `lessThan` |
| Less than or equal to | `lessThanOrEqualTo` |

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

| Logic | `AVIMConversationQuery` Method |
| --- | --- |
| Equal to | `whereEqualTo` |
| Not equal to | `whereNotEqualsTo` |
| Greater than | `whereGreaterThan` |
| Greater than or equal to | `whereGreaterThanOrEqualsTo` |
| Less than | `whereLessThan` |
| Less than or equal to | `whereLessThanOrEqualsTo` |

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

| Logic | `AVIMConversationQuery` Method |
| --- | --- |
| Equal to | `WhereEqualTo` |
| Not equal to | `WhereNotEqualsTo` |
| Greater than | `WhereGreaterThan` |
| Greater than or equal to | `WhereGreaterThanOrEqualsTo` |
| Less than | `WhereLessThan` |
| Less than or equal to | `WhereLessThanOrEqualsTo` |

{{ docs.langSpecEnd('cs') }}

> Notes about default query conditions:
>
> When querying conversations, if there isn’t any `where` condition specified, `ConversationQuery` will look for conversations containing the current user by default. Such condition will be dismissed if any `where` condition is applied to the query. If you want to look for conversations containing certain `clientId`, you can follow the way introduced in [Queries on Array Values](#queries-on-array-values) to perform queries on `m` with the value of `clientId`. This won’t cause any conflict with the default condition.

### Using Regular Expressions

You can use regular expressions as conditions when querying with `ConversationsQuery`. For example, to look for all the conversations that have `language` to be Chinese:

```js
query.matches('language',/[\\u4e00-\\u9fa5]/);
```
```objc
[query whereKey:@"language" matchesRegex:@"[\u4e00-\u9fa5]"];
```
```java
query.whereMatches("language","[\\u4e00-\\u9fa5]"); // language is Chinese characters
```
```cs
query.WhereMatches("language","[\\u4e00-\\u9fa5]"); // language is Chinese characters
```

### Queries on String Values

You can look for conversations with string values that **start with** a particular string, which is similar to `LIKE 'keyword%'` in SQL. For example, to look for all conversations with names starting with `education`:

```js
query.startsWith('name','education');
```
```objc
[query whereKey:@"name" hasPrefix:@"education"];
```
```java
query.whereStartsWith("name","education"); 
```
```cs
query.WhereStartsWith("name","education"); 
```

You can also look for conversations with string values that **include** a particular string, which is similar to `LIKE '%keyword%'` in SQL. For example, to look for all conversations with names including `education`:

```js
query.contains('name','education');
```
```objc
[query whereKey:@"name" containsString:@"education"];
```
```java
query.whereContains("name","education"); 
```
```cs
query.WhereContains("name","education"); 
```

If you want to look for conversations with string values that **exclude** a particular string, you can use [regular expressions](#using-regular-expressions). For example, to look for all conversations with names excluding `education`:

```js
var regExp = new RegExp('^((?!education).)*$', 'i');
query.matches('name', regExp);
```
```objc
[query whereKey:@"name" matchesRegex:@"^((?!education).)* $ "]; 
```
```java
query.whereMatches("name","^((?!education).)* $ "); 
```
```cs
query.WhereMatches("name","^((?!education).)* $ ");
```

### Queries on Array Values

You can use `containsAll`, `containedIn`, and `notContainedIn` to perform queries no array values. For example, to look for all conversations containing `Tom`:

```js
query.containedIn('m', ['Tom']);
```
```objc
[query whereKey:@"m" containedIn:@[@"Tom"]];
```
```java
query.whereContainedIn("m", Arrays.asList("Tom")); 
```
```cs
List<string> members = new List<string>();
members.Add("Tom");
query.WhereContainedIn("m", members); 
```

### Queries on Existence

You can look for conversations with or without certain fields to be empty. For example, to look for all conversations with `lm` to be empty:

```js
query.doesNotExist('lm')
```
```objc
[query whereKeyDoesNotExist:@"lm"];
```
```java
query.whereDoesNotExist("lm");
```
```cs
query.WhereDoesNotExist("lm");
```

Or, to look for all conversations with `lm` not to be empty:

```js
query.exists('lm')
```
```objc
[query whereKeyExists:@"lm"];
```
```java
query.whereExists("lm");
```
```cs
query.WhereExists("lm");
```

### Compound Queries

To look for all conversations with `age` to be less than `18` and `keywords` containing `education`:

```js
// Look for all conversations with `age` to be less than `18` and `keywords` containing `education`
query.contains('keywords', 'education').lessThan('age', 18);
```
```objc
[query whereKey:@"keywords" containsString:@"'education'"];
[query whereKey:@"age" lessThan:@(18)];
```
```java
query.whereContains("keywords", "'education'");
query.whereLessThan("age", 18);
```
```cs
query.WhereContains("keywords", "'education'").WhereLessThan("age", 18);
```

You can also connect two queries with `and` or `or` to form a new query.

For example, to look for all conversations that either has `age` to be less than `18` or has `keywords` containing `education`:

```js
// Not supported yet
```
```objc
AVIMConversationQuery *ageQuery = [tom conversationQuery];
[ageQuery whereKey:@"age" greaterThan:@(18)];

AVIMConversationQuery *keywordsQuery = [tom conversationQuery];
[keywordsQuery whereKey:@"keywords" containsString:@"education"];

AVIMConversationQuery *query = [AVIMConversationQuery orQueryWithSubqueries:[NSArray arrayWithObjects:ageQuery,keywordsQuery,nil]];
```
```java
AVIMConversationsQuery ageQuery = tom.getConversationsQuery();
ageQuery.whereLessThan('age', 18);

AVIMConversationsQuery keywordsQuery = tom.getConversationsQuery();
keywordsQuery.whereContains('keywords', 'education');

AVIMConversationsQuery query = AVIMConversationsQuery.or(Arrays.asList(priorityQuery, statusQuery));
```
```cs
var ageQuery = tom.GetQuery().WhereLessThan('age', 18);

var keywordsQuery = tom.GetQuery().WhereContains('keywords', 'education').

var query = AVIMConversationQuery.or(new AVIMConversationQuery[] { ageQuery, keywordsQuery});
```

### Sorting

You can sort the results of a query by ascending or descending order on certain fields. For example:

```js
// Ascend by name and descend by creation time
query.addAscending('name').addDescending('createdAt');
```
```objc
AVIMClient *client = [[AVIMClient alloc] initWithClientId:@"Tom"];

[client openWithCallback:^(BOOL succeeded, NSError *error) {
    AVIMConversationQuery *query = [client conversationQuery];
    /* Descend by creation time */
    [query orderByDescending:@"createdAt"];
    [query findConversationsWithCallback:^(NSArray *conversations, NSError *error) {
        NSLog(@"%ld conversations found!", [conversations count]);
    }];
}];
```
```java
AVIMClient tom = AVIMClient.getInstance("Tom");

tom.open(new AVIMClientCallback() {
  @Override
  public void done(AVIMClient client, AVIMException e) {
    if (e == null) {
      // Logged in
      AVIMConversationsQuery query = client.getConversationsQuery();

      // Descend by creation time
      query.orderByDescending("createdAt");

      query.findInBackground(new AVIMConversationQueryCallback() {
        @Override
        public void done(List<AVIMConversation> convs, AVIMException e) {
          if (e == null) {
            if(convs != null && !convs.isEmpty()) {
              // Get results
            }
          }
        }
      });
    }
  }
});
```
```cs
// Not supported yet
```

### Excluding Member Lists from Results

When searching conversations, you can exclude the lists of members from query results if you don’t need them. By doing so, their `members` fields will become empty arrays. This helps you improve the speed of your app and reduces the bandwidth needed.

```js
query.compact(true);
```
```objc
AVIMClient *client = [[AVIMClient alloc] initWithClientId:@"Tom"];

[client openWithCallback:^(BOOL succeeded, NSError *error) {
    AVIMConversationQuery *query = [client conversationQuery];
    /* Exclude member lists from results */
    query.option = AVIMConversationQueryOptionCompact;
    [query findConversationsWithCallback:^(NSArray *conversations, NSError *error) {
        NSLog(@"%ld conversations found!", [conversations count]);
    }];
}];
```
```java
public void queryConversationCompact() {
  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
      if (e == null) {
        // Logged in
        AVIMConversationsQuery query = client.getConversationsQuery();
        query.setCompact(true);
        query.findInBackground(new AVIMConversationQueryCallback() {
          @Override
          public void done(List<AVIMConversation> convs, AVIMException e) {
            if (e == null) {
              // Get results
            }
          }
        });
      }
    }
  });
}
```
```cs
// Not supported yet
```

### Including Latest Messages in Results

Many chatting apps show the latest messages of conversations together in a list. If you want the similar function in your app, you can turn on the option when querying conversations:

```js
// withLastMessagesRefreshed includes the latest messages of conversations in results
query.withLastMessagesRefreshed(true);
```
```objc
AVIMClient *client = [[AVIMClient alloc] initWithClientId:@"Tom"];

[client openWithCallback:^(BOOL succeeded, NSError *error) {
    AVIMConversationQuery *query = [client conversationQuery];
    /* Include the latest messages of conversations in results */
    query.option = AVIMConversationQueryOptionWithMessage;
    [query findConversationsWithCallback:^(NSArray *conversations, NSError *error) {
        NSLog(@"%ld conversations found!", [conversations count]);
    }];
}];
```
```java
public void queryConversationWithLastMessage() {
  AVIMClient tom = AVIMClient.getInstance("Tom");
  tom.open(new AVIMClientCallback() {
    @Override
    public void done(AVIMClient client, AVIMException e) {
      if (e == null) {
        // Logged in
        AVIMConversationsQuery query = client.getConversationsQuery();
        /* Include the latest messages of conversations in results */
        query.setWithLastMessagesRefreshed(true);
        query.findInBackground(new AVIMConversationQueryCallback() {
          @Override
          public void done(List<AVIMConversation> convs, AVIMException e) {
            if (e == null) {
              // Get results
            }
          }
        });
      }
    }
  });
}
```
```cs
// Not supported yet
```

Keep in mind that what this option really does is to refresh the latest messages of conversations. Due to the existence of cache, it is still possible for you to retrieve the outdated “latest messages” even though you set the option to be `false`.

### Caching Results

{{ docs.langSpecStart('js') }}

Conversations will be cached in memory using dictionaries according to their IDs. Such cache will not be persisted.

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('objc') }}

By caching query results locally, if the device is offline, or if the app is just opened and the request for synchronizing with cloud is not completed yet, there could still be some data available. You can also reduce the data usage of the user by performing queries with cloud only when the app is first opened and having subsequent queries completed with local cache.

Keep in mind that query results will be fed with local cache first and will be synchronized with the cloud right after that. The expiration time for cache is 1 hour. You can configure cache with the following methods provided by `AVIMConversationQuery`:

```objc
// Set caching policy; defaults to kAVCachePolicyCacheElseNetwork
@property (nonatomic) AVCachePolicy cachePolicy;

// Set expiration time; defaults to 1 hour (1 * 60 * 60)
@property (nonatomic) NSTimeInterval cacheMaxAge;
```

If you want cache to be accessed only when there’s an error querying with the cloud, you can do this way:

```objc
AVIMConversationQuery *query = [[AVIMClient defaultClient] conversationQuery];
query.cachePolicy = kAVCachePolicyNetworkElseCache;
[query findConversationsWithCallback:^(NSArray *objects, NSError *error) {

}];
```

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

By caching query results locally, if the device is offline, or if the app is just opened and the request for synchronizing with cloud is not completed yet, there could still be some data available. You can also reduce the data usage of the user by performing queries with cloud only when the app is first opened and having subsequent queries completed with local cache.

Keep in mind that query results will be fed with local cache first and will be synchronized with the cloud right after that. The expiration time for cache is 1 hour. You can configure cache with the following method provided by `AVIMConversationQuery`:

```java
// Set caching policy for AVIMConversationsQuery
public void setQueryPolicy(AVQuery.CachePolicy policy);
```

If you want cache to be accessed only when there’s an error querying with the cloud, you can do this way:

```java
AVIMConversationsQuery query = client.getConversationsQuery();
query.setQueryPolicy(AVQuery.CachePolicy.NETWORK_ELSE_CACHE);
query.findInBackground(new AVIMConversationQueryCallback() {
  @Override
  public void done(List<AVIMConversation> conversations, AVIMException e) {

  }
});
```

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

Not supported yet.

{{ docs.langSpecEnd('cs') }}

### Optimizing Performance

Since `Conversation` objects are stored on LeanStorage, you can make use of indexes to improve the efficiency of querying, just like how you would do to other classes. Here are some suggestions for optimizing performance:

- By default, indexes are created for `objectId`, `updatedAt`, and `createdAt` of `Conversation`, so querying by these fields would be naturally fast.
- Although it’s possible to implement pagination with `skip` and `limit`, the speed would slow down when the dataset grows larger. It would be more efficient to make use of `updatedAt` or `lastMessageAt` instead.
- When searching for conversations containing a certain user by using `contains` on `m`, it’s recommended that you stick to the default `limit` (which is 10) and make use of `updatedAt` or `lastMessageAt` for pagination.
- If you app has too many conversations, consider creating a cloud function that periodically cleans up inactive conversations.

## Retrieving Messages

By default, message histories are stored on the cloud for **180** days. You may either pay to extend the period (contact us at support@leancloud.rocks) or synchronize them to your own server with REST API.

Our SDKs offer various of ways for you to retrieve message histories. iOS and Android SDKs also provide caching mechanism to help you reduce the number of queries you have to perform and display message histories to users even their devices are offline.

### Retrieving Messages Chronologically (New to Old)

The most common way to retrieve messages is to fetch them from new to old with the help of pagination:

```js
conversation.queryMessages({
  limit: 10, // limit could be any number from 1 to 1000 (defaults to 100)
}).then(function(messages) {
  // The last 10 messages ordered from old to new
}).catch(console.error.bind(console));
```
```objc
// Retrieve the last 10 messages; limit could be any number from 1 to 1000 (defaults to 100)
[conversation queryMessagesWithLimit:10 callback:^(NSArray *objects, NSError *error) {
    NSLog(@"Messages Retrieved!");
}];
```
```java
// limit could be any number from 1 to 1000 (defaults to 100)
conv.queryMessages(10, new AVIMMessagesQueryCallback() {
  @Override
  public void done(List<AVIMMessage> messages, AVIMException e) {
    if (e == null) {
      // The last 10 messages retrieved
    }
  }
});
```
```cs
// limit could be any number from 1 to 1000 (defaults to 100)
var messages = await conversation.QueryMessageAsync(limit: 10);
foreach (var message in messages)
{
  if (message is AVIMTextMessage)
  {
    var textMessage = (AVIMTextMessage)message;
  }
}
```

Here `queryMessage` supports pagination. Given the fact that you can locate a single message with its `messageId` and timestamp, this means that you can retrieve the next few messages after a given message by providing the `messageId` and timestamp of that message:

```js
// JS SDK encloses the feature into an iterator so you can keep retrieving new data by calling next
// Create an iterator and retrieve 10 messages each time
var messageIterator = conversation.createMessagesIterator({ limit: 10 });
// Call next for the first time and get the first 10 messages; done equals to false means that there are more messages
messageIterator.next().then(function(result) {
  // result: {
  //   value: [message1, ..., message10],
  //   done: false,
  // }
}).catch(console.error.bind(console));
// Call next for the second time and get the 11th to 20th messages; done equals to false means that there are more messages
// The iterator will keep track of the breaking point so you don’t have to specify it
messageIterator.next().then(function(result) {
  // result: {
  //   value: [message11, ..., message20],
  //   done: false,
  // }
}).catch(console.error.bind(console));
```
```objc
// Retrieve the last 10 messages
[conversation queryMessagesWithLimit:10 callback:^(NSArray *messages, NSError *error) {
    NSLog(@"First retrieval completed!");
    // Get the messages right before the first message in the first page
    AVIMMessage *oldestMessage = [messages firstObject];
    [conversation queryMessagesBeforeId:oldestMessage.messageId timestamp:oldestMessage.sendTimestamp limit:10 callback:^(NSArray *messagesInPage, NSError *error) {
        NSLog(@"Second retrieval completed!");
    }];
}];
```
```java
// limit could be any number from 1 to 1000 (defaults to 100)
conv.queryMessages(10, new AVIMMessagesQueryCallback() {
  @Override
  public void done(List<AVIMMessage> messages, AVIMException e) {
    if (e == null) {
      // The last 10 messages retrieved
      // The earliest message will be the first one
      AVIMMessage oldestMessage = messages.get(0);

      conv.queryMessages(oldestMessage.getMessageId(), oldestMessage.getTimestamp(),20,
          new AVIMMessageQueryCallback(){
            @Override
            public void done(List<AVIMMessage> messagesInPage,AVIMException e){
              if(e== null){
                // Query completed
                Log.d("Tom & Jerry", "got " + messagesInPage.size()+" messages ");
              }
          }
      });
    }
  }
});
```
```cs
// limit could be any number from 1 to 1000 (defaults to 100)
var messages = await conversation.QueryMessageAsync(limit: 10);
var oldestMessage = messages.ToList()[0];
var messagesInPage = await conversation.QueryMessageAsync(beforeMessageId: oldestMessage.Id, beforeTimeStamp: oldestMessage.ServerTimestamp); 
```

### Retrieving Messages by Types

Beside retrieving messages in time orders, you can also do that based on the types of messages. This could be helpful in scenarios like displaying all the images in a conversation.

`queryMessage` can take in the type of messages:

```js
conversation.queryMessages({ type: ImageMessage.TYPE }).then(messages => {
  console.log(messages);
}).catch(console.error);
```
```objc
[conversation queryMediaMessagesFromServerWithType:kAVIMMessageMediaTypeImage limit:10 fromMessageId:nil fromTimestamp:0 callback:^(NSArray *messages, NSError *error) {
    if (!error) {
        NSLog(@"Query completed!");
    }
}];
```
```java
int msgType = .AVIMMessageType.TEXT_MESSAGE_TYPE;
conversation.queryMessagesByType(msgType, limit, new AVIMMessagesQueryCallback() {
    @Override
    public void done(List<AVIMMessage> messages, AVIMException e){
    }
});
```
```cs
// Pass in a generic type parameter and the SDK will automatically read the type and send it to the server for searching messages
var imageMessages = await conversation.QueryMessageAsync<AVIMImageMessage>();
```

To retrieve more images, follow the way introduced in the previous section to go through different pages.

### Retrieving Messages Chronologically (Old to New)

Beside the two ways mentioned above, you can also retrieve messages from old to new. The code below shows how you can retrieve messages starting from the time the conversation is created:

```js
var { MessageQueryDirection } = require('leancloud-realtime');
conversation.queryMessages({
  direction: MessageQueryDirection.OLD_TO_NEW,
}).then(function(messages) {
  // handle result
}.catch(function(error) {
  // handle error
});
```
```objc
[conversation queryMessagesInInterval:nil direction:AVIMMessageQueryDirectionFromOldToNew limit:20 callback:^(NSArray<AVIMMessage *> * _Nullable messages, NSError * _Nullable error) {
    if (messages.count) {
        // handle result.
    }
}];
```
```java
AVIMMessageInterval internal = new AVIMMessageInterval(null, null);
conversation.queryMessages(internal, AVIMMessageQueryDirectionFromOldToNew, limit,
  new AVIMMessagesQueryCallback(){
    public void done(List<AVIMMessage> messages, AVIMException exception) {
      // handle result
    }
});
```
```cs
var earliestMessages = await conversation.QueryMessageAsync(direction: 0);
```

It is a bit more complicated to implement pagination with this method. See the next section for more explanations.

### Retrieving Messages Chronologically (From a Timestamp to a Direction)

You can retrieve messages starting from a given message (determined by ID and timestamp) toward a certain direction:

- New to old: Retrieve messages sent **before** a given message
- Old to new: Retrieve messages sent **after** a given message

Now we can implement pagination on different directions.

```js
var { MessageQueryDirection } = require('leancloud-realtime');
conversation.queryMessages({
  startTime: timestamp,
  startMessageId: messageId,
startClosed: false,
  direction: MessageQueryDirection.OLD_TO_NEW,
}).then(function(messages) {
  // handle result
}.catch(function(error) {
  // handle error
});
```
```objc
AVIMMessageIntervalBound *start = [[AVIMMessageIntervalBound alloc] initWithMessageId:nil timestamp:timestamp closed:false];
AVIMMessageInterval *interval = [[AVIMMessageInterval alloc] initWithStartIntervalBound:start endIntervalBound:nil];
[conversation queryMessagesInInterval:interval direction:direction limit:20 callback:^(NSArray<AVIMMessage *> * _Nullable messages, NSError * _Nullable error) {
    if (messages.count) {
        // handle result.
    }
}];
```
```java
AVIMMessageIntervalBound start = AVIMMessageInterval.createBound(messageId, timestamp, false);
AVIMMessageInterval internal = new AVIMMessageInterval(start, null);
AVIMMessageQueryDirection direction;
conversation.queryMessages(internal, direction, limit,
  new AVIMMessagesQueryCallback(){
    public void done(List<AVIMMessage> messages, AVIMException exception) {
      // handle result
    }
});
```
```cs
var earliestMessages = await conversation.QueryMessageAsync(direction: 0, limit: 1);
// get some messages after earliestMessages.Last()
var nextPageMessages = await conversation.QueryMessageAfterAsync(earliestMessages.Last());
```

### Retrieving Messages Within a Period of Time

Beside retrieving messages chronologically, you can also retrieve messages within a period of time. For example, if you already have two messages, you can have one of them to be the starting point and another one to be the ending point to retrieve all the messages between them:

Note: **The limit of 100 messages per query still applies here. To fetch more messages, keep changing the starting point or the ending point until all the messages are retrieved.**

```js
conversation.queryMessages({
  startTime: timestamp,
  startMessageId: messageId,
  endTime: endTimestamp,
  endMessageId: endMessageId,
}).then(function(messages) {
  // handle result
}.catch(function(error) {
  // handle error
});
```
```objc
AVIMMessageIntervalBound *start = [[AVIMMessageIntervalBound alloc] initWithMessageId:nil timestamp:startTimestamp closed:false];
    AVIMMessageIntervalBound *end = [[AVIMMessageIntervalBound alloc] initWithMessageId:nil timestamp:endTimestamp closed:false];
AVIMMessageInterval *interval = [[AVIMMessageInterval alloc] initWithStartIntervalBound:start endIntervalBound:end];
[conversation queryMessagesInInterval:interval direction:direction limit:100 callback:^(NSArray<AVIMMessage *> * _Nullable messages, NSError * _Nullable error) {
    if (messages.count) {
        // handle result.
    }
}];
```
```java
AVIMMessageIntervalBound start = AVIMMessageInterval.createBound(messageId, timestamp, false);
AVIMMessageIntervalBound end = AVIMMessageInterval.createBound(endMessageId, endTimestamp, false);
AVIMMessageInterval internal = new AVIMMessageInterval(start, end);
AVIMMessageQueryDirection direction;
conversation.queryMessages(internal, direction, limit,
  new AVIMMessagesQueryCallback(){
    public void done(List<AVIMMessage> messages, AVIMException exception) {
      // handle result
    }
});
```
```cs
var earliestMessage = await conversation.QueryMessageAsync(direction: 0, limit: 1);
var latestMessage = await conversation.QueryMessageAsync(limit: 1);
// mex count for messagesInInterval is 100
var messagesInInterval = await conversation.QueryMessageInIntervalAsync(earliestMessage.FirstOrDefault(), latestMessage.FirstOrDefault());
```

### Caching Messages

iOS and Android SDKs come with the mechanism that automatically caches all the messages received and retrieved on the local device (not supported by JavaScript or C# SDKs). It provides the following benefits:

1. Message histories can be viewed even devices are offline
2. The frequency of querying and the consumption of data can be minimized
3. The speed for viewing messages can be increased

Caching is enabled by default. You can turn it off with the following interface:

```js
// Not supported yet
```
```objc
// Need to be set before calling [avimClient openWithCallback:callback]
avimClient.messageQueryCacheEnabled = false;
```
```java
// Need to be set before calling AVIMClient.open(callback)AVIMClient.setMessageQueryCacheEnable(false);
```
```cs
// Not supported yet
```

## Logging out and Network Changes

### Logging out

If your app allows users to log out, you can use the `close` method provided by `AVIMClient` to properly close the connection to the cloud:

```js
tom.close().then(function() {
  console.log('Tom logged out.');
}).catch(console.error.bind(console));
```
```objc
[tom closeWithCallback:^(BOOL succeeded, NSError * _Nullable error) {
    if (succeeded) {
        NSLog(@"Logged out.");
    }
}];
```
```java
tom.close(new AVIMClientCallback(){
    @Override
    public void done(AVIMClient client,AVIMException e){
        if(e==null){
        // Logged out
        }
    }
});
```
```cs
await tom.CloseAsync();
```

After the function is called, the connection between the client and the server will be terminated. If you check the status of the corresponding `clientId` on the cloud, it would show as “offline”.

### Network Changes

The availability of the messaging service is highly dependent on the Internet connection. If the connection is lost, all the operations regarding messages and conversations will fail. At this time, there needs to be some indicators on the UI to tell users about the network status.

Our SDKs maintain a heartbeat mechanism with the cloud which detects the change of network status and have your app notified if certain events occur. To be specific, if the connection status changes (becomes lost or recovered), the following events will be populated:

{{ docs.langSpecStart('js') }}

* `DISCONNECT`: Connection to the server is lost. The messaging service is unavailable at this time.
* `OFFLINE`: Network is unavailable.
* `ONLINE`: Network is recovered.
* `SCHEDULE`: Scheduled to reconnect after a period of time. The messaging service is still unavailable at this time.
* `RETRY`: Reconnecting.
* `RECONNECT`: Connection to the server is recovered. The messaging service is available at this time.

```js
var { Event } = require('leancloud-realtime');

realtime.on(Event.DISCONNECT, function() {
  console.log('Connection to the server is lost.');
});
realtime.on(Event.OFFLINE, function() {
  console.log('Network is unavailable.');
});
realtime.on(Event.ONLINE, function() {
  console.log('Network is recovered.');
});
realtime.on(Event.SCHEDULE, function(attempt, delay) {
  console.log('Reconnecting in ' + delay + ' ms as attempt ' + (attempt + 1));
});
realtime.on(Event.RETRY, function(attempt) {
  console.log('Reconnecting as attempt ' + (attempt + 1));
});
realtime.on(Event.RECONNECT, function() {
  console.log('Connection to the server is recovered.');
});
```

{{ docs.langSpecEnd('js') }}

{{ docs.langSpecStart('objc') }}

The following events will be populated on `AVIMClientDelegate`:

- `imClientPaused:(AVIMClient *)imClient` occurs when the connection is lost. The messaging service is unavailable at this time.
- `imClientResuming:(AVIMClient *)imClient` occurs when trying to reconnect. The messaging service is still unavailable at this time.
- `imClientResumed:(AVIMClient *)imClient` occurs when the connection is recovered. The messaging service is available at this time.

{{ docs.langSpecEnd('objc') }}

{{ docs.langSpecStart('java') }}

The following events will be populated on `AVIMClientEventHandler`:

- `onConnectionPaused()` occurs when the connection is lost. The messaging service is unavailable at this time.
- `onConnectionResume()` occurs when the connection is recovered. The messaging service is available at this time.
- `onClientOffline()` occurs when single device sign-on is enabled and the current device is forced to go offline.

{{ docs.langSpecEnd('java') }}

{{ docs.langSpecStart('cs') }}

The following events will be populated on `AVRealtime`:

- `OnDisconnected` occurs when the connection is lost. The messaging service is unavailable at this time.
- `OnReconnecting` occurs when trying to reconnect. The messaging service is still unavailable at this time.
- `OnReconnected` occurs when the connection is recovered. The messaging service is available at this time.
- `OnReconnectFailed` occurs when it fails to reconnect. The messaging service is unavailable at this time.

{{ docs.langSpecEnd('cs') }}

## More Suggestions

### Sorting Conversations by Last Activities

In many scenarios you may need to sort conversations based on the time the last message in each of them is sent. Here we offer a property `lastMessageAt` for each `AVIMConversation` (`lm` in the `_Conversation` table) which dynamically changes to reflect the time of the last message. The time is server-based (accurate to a second) so you don’t have to worry about the time on the clients. `AVIMConversation` also offers a method for you to retrieve the last message of each conversation, which gives you more flexibility to design the UI of your app.

### Auto Reconnecting

If the connection between a client and the cloud is not properly closed, our iOS and Android SDKs will automatically reconnect when the network is recovered. You can listen to `IMClient` to get updated about the network status.

### More Conversation Types

Beside the [one-on-one chatting](#one-on-one-chatting) and [group chats](#group-chats) mentioned earlier, LeanMessage also supports these types of conversations:

- Chat room: This can be used to build conversations that serve scenarios like live streaming. It’s different than a basic group chat on the number of members supported and the deliverability promised. See [3. Chat Rooms](realtime-guide-senior.html#chat-rooms) for more details.
- Temporary conversation: This can be used to build conversations between users and customer service representatives. It’s different than a basic one-on-one chatting on the fact that it has a shorter TTL which brings higher flexibility and lower cost (on data storage). See [3. Temporary Conversations](realtime-guide-senior.html#temporary-conversations) for more details.
- System conversation: This can be used to build accounts that could broadcast messages to all their subscribers. It’s different than a basic group chat on the fact that users can subscribe to it and there isn’t a number limit of members. Subscribers can also send one-on-one messages to these accounts and these messages won’t be seen by other users. See [4. System Conversations](realtime-guide-systemconv.html#system-conversations) for more details.

## Continue Reading

[2. Advanced Messages, Push Notifications, Synchronization, and Multi Device Sign-on](realtime-guide-intermediate.html)

[3. Security, Permission Management, Chat Rooms, and Temporary Conversations](realtime-guide-senior.html)

[4. Hooks and System Conversations](realtime-guide-systemconv.html)