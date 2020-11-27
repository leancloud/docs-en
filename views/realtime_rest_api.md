# LeanMessage REST API Guide

## Request Format

For POST and PUT requests, the body of the requests should be JSON. The Content-Type needs to be configured as `application/json`.

## Authentication

Authentication of the request is via the key-value pair included in the HTTP Header. The parameters are as follows:

Key|Value|Meaning|Source
---|----|---|---
`X-LC-Id`|{{appid}}| App ID of the current application| Check in Dashboard -> Settings -> App keys
`X-LC-Key`|{{appkey}}|App Key of the current application | Check in Dashboard -> Settings -> App keys

Some administrative interfaces require master key.

## Relevant Concept

The `_Conversation` table includes some built-in fields to define the attributes and participants of the conversations. For one-on-one conversations, group chats, chat rooms, official accounts, and bots, see [LeanMessage Overview - Conversation](./realtime_v2.html#conversation) for more information.

## Architecture

For the concept of "Conversation", we have categorized it into three subclasses:

- For one-on-one conversations and group chats, the related API is labeled with `rtm/conversations`.
- For chat rooms, the related API is labeled with `rtm/chatrooms`. In the `_Conversation` table, `tr` is set to true to indicate this type of conversation.
- For official accounts and bots, the related API is labeled with `rtm/service-conversations`. In the `_Conversation` table, `sys` is set to true to indicate this.

Besides these, there are also client-related requests which are labeled with `rtm/clients`.

Lastly, some [global APIs](#Global_API) are labeled with `rtm/{function}`, like `rtm/all-conversations` which will search for conversations of all types.

{% include "views/_rtm_rest_api_normal.njk" %}
{% include "views/_rtm_rest_api_chatroom.njk" %}
{% include "views/_rtm_rest_api_system.njk" %}
{% include "views/_rtm_rest_api_client.njk" %}

## Global API

### Count the Users

This interface will return the total number of the online users and the total number of the independent users logged in today. This interface requires master key.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.2/rtm/stats
```

It returns:

```
{"result":{"online_user_count":10212,"user_count_today":1002324}}
```

`online_user_count` indicates the currently online users on the app and `user_count_today` indicates the independent users logged in today.

### Query all the Conversations

This interface will return all the One-on-One Chatting/Group Chats/Chat Rooms/Official Accounts and bots. In `_Conversation`, the default ACL authority requires the master key to access.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.2/rtm/all-conversations
```

Parameters | Optionality | description
---|---|---
skip |optional |
limit | optional | together with 'skip' to implement paging
where | optional | see [Leanstorage - Query](rest_api.html#Queries).

It returns:
```
{"results"=>[{"name"=>"test conv1", "m"=>["tom", "jerry"], "createdAt"=>"2018-01-17T04:15:33.386Z", "updatedAt"=>"2018-01-17T04:15:33.386Z", "objectId"=>"5a5ecde6c3422b738c8779d7"}]}
```

### Global Broadcasts

This interface is able to broadcast messages (at most 30 per day) to clients in the app. This interface requires master key to access.

```
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"from_client": "1a", "message": "{\"_lctype\":-1,\"_lctext\":\"pure text message\",\"_lcattrs\":{\"a\":\"_lcattrs to hold some user-specified key-value pairs\"}}", "conv_id": "..."}' \
  https://{{host}}/1.2/rtm/broadcasts
```

Parameters | Optionality | Type | Description
---|---|---|---
from_client | required | string | sender ID
conv_id | required | string | conversation id to send (only in the official accounts)
message | required | string | message content (generally the content should be of string type but no rigorous limitations on the type. <br/> as long as the size is less than 5kb, the developer can send messages of any types.)
valid_till | Optional | number | expiry time, UTC timestamp(millisecond), at most and default on 1 month later.
push | Optional | string or JSON | attached push, **all** the iOS and Android users will receive the notification if sets.
transient | Optional | boolean | default as false. This field is marked as whether the broadcast is transient. This will broadcast to all the online users. The offline users will not receive the copies after later logging in.

Push formats are in alignment with the `data` part of  [Push Notification REST API message content](push_guide.html#message content_Data). If you need to specify the developer push certificate, you have to configure the `"_profile": "dev"` in the JSON for example:

```
{
   "alert": "message content",
   "category": "push category",
   "badge": "Increment",
   "_profile": "dev"
}
```

Frequency limitation:

This interface has a frequency limitation. Click [here](#interface requests frequency limitation) to view.

### Modify the Broadcast Messages

This interface requires the master key.

The modification will only occur on the devices that have not received the original message yet. The received broadcast cannot be modified. Please be cautious when broadcasting.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  -H "Content-Type: application/json" \
  -d '{"from_client": "", "message": "", "timestamp": 123}' \
  https://{{host}}/1.2/rtm/service-conversations/{conv_id}/messages/{message_id}
```

Parameters | Optionality | Description
---|---|---
from_client | required | sender ID
message | required | message content
timestamp | required | timestamp of the message

It will return status code `200 OK` if succeeded.

Frequency limitation:

This interface has a frequency limitation. Click [here](#interface requests frequency limitation) to view.

### Delete Broadcast Messages

The deletions will only occur on the devices do not receive the original message yet. The received broadcast cannot be deleted. This interface requires the master key.

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.2/rtm/broadcasts/{message_id}
```

Parameters | Optionality | Description
---|---|---
message_id | required | target id, String

It will return status code `200 OK` if succeeded.

### Query the Broadcast Messages

Invoke this API to query all the valid broadcast messages. This interface requires master key to query.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.2/rtm/broadcasts?conv_id={conv_id}
```

Parameters | Optionality | Description
---|---|---
conv_id | required | official account ID
limit | optional | quantity of the messages returned
skip | optional | number of the messages to skip, for paging.

### Query all the Messages History in the App

This interface requires the master key.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{masterkey}},master" \
  https://{{host}}/1.2/rtm/messages
```

Refer to [One-on-One Chatting/Group Chats query the history messages](realtime_rest_api.html#query history messages) interface.

## Interface requests Frequency Limitation

Operations related to realtime messages invoking REST API has request frequency and quantity limitations (**realtime message SDK API is not influenced**),  details as follows:

* [One-on-One Chatting/Group Chats-send messages](#One-on-One Chatting/Group Chats-send messages)
* [One-on-One Chatting/Group Chats-modify messages](#One-on-One Chatting/Group Chats-modify messages)
* [One-on-One Chatting/Group Chats-recall messages](#One-on-One Chatting/Group Chats-recall messages)
* [Chats Rooms-send messages](#Chats Rooms-send messages)
* [Chats Rooms-modify messages](#Chats Rooms-modify messages)
* [Chats Rooms-recall messages](#Chats Rooms-recalll messages)
* [Official accounts-send message to any user individually](#send message to any user individually)
* [Official accounts-modify message to any user individually](#modify message to any user individually)
* [Official accounts-recall message to any user individually](#recall message to any user individually)

#### Limitation
|Business (per App) |Developer (per App) |
|---------|---------------|
|maximum 9000 requests/min, 1800 requests/min by default|120 requests/min|

The daily usage is calculated based on all the interfaces. LeanCloud will respond with error code 429 if exceeding the limit. The REST API will continue to handle requests after one minute.

The Buisness call ceiling can get mutated in [Dashboard > Messaging > LeanMessage > Settings > Thresholds > Frequency limit for calling API for basic messages][dashboard-rtm-limit].
You will be billed for daily request frequency rate peak, as below:

[dashboard-rtm-limit]: /dashboard/messaging.html?appid={{appid}}#/message/realtime/conf

| calling / min | pricing |
| - | - |
| - | - |
| 0 ～ 1800 | Free |
| 1801 ~ 3600 | $6 USD / day |
| 3601 ~ 5400 | $9 USD / day |
| 5401 ~ 7200 | $12 USD / day |
| 7201 ~ 9000 | $15 USD / day 

Daily calling peak rate can be viewed in [Dashboard > Messaging > LeanMessage > API Peak connections][dashboard-rtm-stats].

[dashboard-rtm-stats]: /dashboard/messaging.html?appid={{appid}}#/message/realtime/stat

### Subscription Messages

* [Send messages to all the subscribers](#Send message to all the subscribers)
* [Modify all the sent messages](#Modify all the sent messages)
* [Recall all the sent messages](#Recall all the sent messages)

#### Limitations

|Limitation |Business |Developer |
|----------|----------|---------------|
|Frequency limit | 30 times per app /min | 10 times per app/min |
|Daily usage | 1000 at maximum | 100 at maximum |

The limitation is calculated based on all the interfaces. LeanCloud will respond with error code 429 if exceeding the limit. The REST API will continue to handle requests after one minute. The request will not be processed if the daily usage amount is exceeded. All the requests will be responded by error 429.

### Brodcast Messages
* [Global Broadcasts](#Global Broadcasts)
* [Modify the Broadcast Messages](#Modify the Broadcast Messages)

#### Limitations

|Limitation |Business |Developer |
|----------|----------|---------------|
|Frequency limit | 10 times per app /min | once per app/min |
|Daily usage | 30 at maximum | 10 at maximum |

The limitation is calculated based on all the interfaces. LeanCloud will respond with error code 429 if exceeding the limit. The REST API will continue to handle requests after one minute. The request will not be processed if the daily usage amount is exceeded. All the requests will be responded by error 429.

