{% import "views/_helper.njk" as docs %}
{% import "views/_data.njk" as data %}
# REST API Guide

You can access LeanCloud from any devices supporting HTTP requests with REST API, for example:

- You can manipulate data on LeanCloud with any programming language.
- If you want to migrate from LeanCloud to other services, you can export all your data.
- Your mobile site can fetch data from LeanCloud via JavaScript directly if you regard importing LeanCloud JavaScript SDK as overkill.
- You can add new data in batch, to be consumed by mobile applications later.
- You can export recent data for offline analysis or additional incremental backup.

## API Version

The current API version is `1.1`.

### Testing

This guide provides curl command line examples.
You may need to modify some syntax if using cmd.exe on Windows.
For example, `\` in curl examples means to be continued on the next line, but cmd.exe will consider it as path separator.
Therefore, we recommend you to use [Postman] for testing on Windows.

[Postman]: https://www.getpostman.com/

Postman can [import curl commands][import-curl] directly and automatically [generate code snippets][generate-snippets] in various languages and frameworks.

[import-curl]: https://learning.getpostman.com/docs/postman/collections/data-formats/#importing-curl
[generate-snippets]: https://learning.getpostman.com/docs/postman/sending-api-requests/generate-code-snippets/

### Objects

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP Method</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/classes/&lt;className&gt;</td>
      <td>POST</td>
      <td>create an object</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>retrieve an object</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>update an object</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;</td>
      <td>GET</td>
      <td>query objects</td>
    </tr>
    <tr>
      <td>/1.1/classes/&lt;className&gt;/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>delete an object</td>
    </tr>
    <tr>
      <td>/1.1/scan/classes/&lt;className&gt;</td>
      <td>GET</td>
      <td>iterate over objects</td>
    </tr>
  </tbody>
</table>

### Users

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP Method</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/users</td>
      <td>POST</td>
      <td>register</td>
    </tr>
    <tr>
      <td>/1.1/usersByMobilePhone</td>
      <td>POST</td>
      <td>register or login via mobile phone</td>
    </tr>
    <tr>
      <td>/1.1/login</td>
      <td>POST</td>
      <td>login</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>retrieve a user</td>
    </tr>
    <tr>
      <td>/1.1/users/me</td>
      <td>GET</td>
      <td>retrieve a user via session token</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;/refreshSessionToken</td>
      <td>PUT</td>
      <td>reset session token</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;/updatePassword</td>
      <td>PUT</td>
      <td>reset password via email</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>update user info</td>
    </tr>
    <tr>
      <td>/1.1/users</td>
      <td>GET</td>
      <td>query users</td>
    </tr>
    <tr>
      <td>/1.1/users/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>delete a user</td>
    </tr>
    <tr>
      <td>/1.1/requestPasswordReset</td>
      <td>POST</td>
      <td>request to reset password via email</td>
    </tr>
    <tr>
      <td>/1.1/requestEmailVerify</td>
      <td>POST</td>
      <td>request to verify email</td>
    </tr>
    <tr>
      <td>/1.1/requestMobilePhoneVerify</td>
      <td>POST</td>
      <td>request to verify mobile phone</td>
    </tr>
    <tr>
      <td>/1.1/verifyMobilePhone/&lt;code&gt;</td>
      <td>POST</td>
      <td>verify mobile phone number</td>
    </tr>
    <tr>
      <td>/1.1/requestChangePhoneNumber</td>
      <td>POST</td>
      <td>request to change mobile phone number</td>
    </tr>
    <tr>
      <td>/1.1/changePhoneNumber</td>
      <td>POST</td>
      <td>change mobile phone number</td>
    </tr>
    <tr>
      <td>/1.1/requestLoginSmsCode</td>
      <td>POST</td>
      <td>request to send an SMS for logging in</td>
    </tr>
    <tr>
      <td>/1.1/requestPasswordResetBySmsCode</td>
      <td>POST</td>
      <td>request to reset password via SMS</td>
    </tr>
    <tr>
      <td>/1.1/resetPasswordBySmsCode/&lt;code&gt;</td>
      <td>PUT</td>
      <td>reset password via SMS</td>
    </tr>
  </tbody>
</table>

### Roles

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP Method</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/roles</td>
      <td>POST</td>
      <td>create a role</td>
    </tr>
    <tr>
      <td>/1.1/roles/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>retrieve a role</td>
    </tr>
    <tr>
      <td>/1.1/roles/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>update a role</td>
    </tr>
    <tr>
      <td>/1.1/roles</td>
      <td>GET</td>
      <td>query roles</td>
    </tr>
    <tr>
      <td>/1.1/roles/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>delete a role</td>
    </tr>
  </tbody>
</table>

### Push Notifications

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>Functionality</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/push</td>
      <td>POST</td>
      <td>push notifications</td>
    </tr>
    <tr>
      <td>/1.1/notifications</td>
      <td>GET</td>
      <td>query push records</td>
    </tr>
    <tr>
      <td>/1.1/notifications/:notification_id</td>
      <td>GET</td>
      <td>query a push record via ID</td>
    </tr>
    <tr>
      <td>/1.1/notifications/:notification_id</td>
      <td>DELETE</td>
      <td>delete a push record</td>
    </tr>
    <tr>
      <td>/1.1/scheduledPushMessages</td>
      <td>GET</td>
      <td>query all scheduled push notifications</td>
    </tr>
    <tr>
      <td>/1.1/scheduledPushMessages/:id</td>
      <td>DELETE</td>
      <td>delete an scheduled push notification</td>
    </tr>
  </tbody>
</table>

### Installations

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/installations</td>
      <td>POST</td>
      <td>upload installation data</td>
    </tr>
    <tr>
      <td>/1.1/installations/&lt;objectId&gt;</td>
      <td>GET</td>
      <td>retrieve installation data</td>
    </tr>
    <tr>
      <td>/1.1/installations/&lt;objectId&gt;</td>
      <td>PUT</td>
      <td>update installation data</td>
    </tr>
    <tr>
      <td>/1.1/installations</td>
      <td>GET</td>
      <td>query installation data</td>
    </tr>
    <tr>
      <td>/1.1/installations/&lt;objectId&gt;</td>
      <td>DELETE</td>
      <td>delete installation data</td>
    </tr>
  </tbody>
</table>

### Data Schema

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/schemas</td>
      <td>GET</td>
      <td>retrieve all schemas</td>
    </tr>
    <tr>
      <td>/1.1/schemas/&lt;className&gt;</td>
      <td>POST</td>
      <td>retrieve specified class schema</td>
    </tr>
  </tbody>
</table>

### Cloud Functions

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/functions/&lt;functionName&gt;</td>
      <td>POST</td>
      <td>invoke a cloud function</td>
    </tr>
    <tr>
      <td>/1.1/call/&lt;functionName&gt;</td>
      <td>POST</td>
      <td>call a cloud function with AVObjects as parameters and return values</td>
    </tr>
  </tbody>
</table>

### LeanMessage

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/rtm/messages/logs</td>
      <td>GET</td>
      <td>retrieve chat history</td>
    </tr>
    <tr>
      <td>/1.1/rtm/messages</td>
      <td>POST</td>
      <td>send a message</td>
    </tr>
    <tr>
      <td>/1.1/rtm/transient_group/onlines</td>
      <td>GET</td>
      <td>count online members of a transient conversation</td>
    </tr>
  </tbody>
</table>

### Other

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>Functionality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/date</td>
      <td>GET</td>
      <td>retrieve date and time at server side</td>
    </tr>
    <tr>
      <td>/1.1/exportData</td>
      <td>POST</td>
      <td>request to export data</td>
    </tr>
    <tr>
      <td>/1.1/exportData/&lt;id&gt;</td>
      <td>GET</td>
      <td>retrieve status and result of export data job</td>
    </tr>
  </tbody>
</table>

### Request Format

The request body should be JSON, and the `Content-Type` should be `application/json` accordingly.

`X-LC-Id` and `X-LC-Key` HTTP headers are used for authentication.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"content": "update blog post"}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

`X-LC-Id` is App ID.
`X-LC-Key` is App Key or Master Key.
A `,master` postfix is used to distinguish Master Key and App Key, e.g.:

```
X-LC-Key: {{masterkey}},master
```

Cross-origin resource sharing is supported so that you can use these headers with XMLHttpRequest in JavaScript.

#### X-LC-Sign

You may also authenticate requests via `X-LC-Sign` instead of `X-LC-Key`:

```
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Sign: d5bcbb897e19b2f6633c716dfdfaf9be,1453014943466" \
  -H "Content-Type: application/json" \
  -d '{"content": "reduce risk of App Key exposure"}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

The value of `X-LC-Sign` is a string `sign,timestamp[,master]`:

| Name       | Optionality   | Description                                       |
| --------- | ---- | ---------------------------------------- |
| sign      | required   | Concat timestamp and App Key (or Master Key), then calculate its MD5 hash value. |
| timestamp | required   | Unix timestamp of the current request, accurate to **milliseconds**.  |
| master    | optional   | Use this postfix to indicate Master Key is used. |

For example, given the following application:

<table class="noheading">
  <tbody>
    <tr>
      <td scope="row">App Id</td>
      <td><code>FFnN2hso42Wego3pWq4X5qlu</code></td>
    </tr>
    <tr>
      <td scope="row">App Key</td>
      <td><code>UtOCzqb67d3sN12Kts4URwy8</code></td>
    </tr>
    <tr>
      <td scope="row">Master Key</td>
      <td><code>DyJegPlemooo4X1tg94gQkw1</code></td>
    </tr>
    <tr>
      <td scope="row">Request date</td>
      <td>2016-01-17 7:15:43.466 UTC</td>
    </tr>
    <tr>
      <td scope="row">timestamp</td>
      <td><code>1453014943466</code></td>
    </tr>
  </tbody>
</table>

**Calculate sign with App Key**

>md5( timestamp + App Key ) <br/>
>= md5( <code><u>1453014943466</u>UtOCzqb67d3sN12Kts4URwy8</code> )<br/>= d5bcbb897e19b2f6633c716dfdfaf9be

```sh
  -H "X-LC-Sign: d5bcbb897e19b2f6633c716dfdfaf9be,1453014943466" \
```

**Calculate sign with Master Key**

>md5( timestamp + Master Key )<br/>
>= md5( <code><u>1453014943466</u>DyJegPlemooo4X1tg94gQkw1</code> ) <br>
>= e074720658078c898aa0d4b1b82bdf4b

```sh
  -H "X-LC-Sign: e074720658078c898aa0d4b1b82bdf4b,1453014943466,master" \
```

<div class="callout callout-danger">Using master key will bypass all permission validations. Make sure you do not leak the master key and only use it in restrained environments.</div>

#### Specify hook invocation environment

Requests may trigger [hooks](leanengine_cloudfunction_guide-node.html#Hooking), and you can use `X-LC-Prod` HTTP header to specify the invocation environment:

| X-LC-Prod Value | Environment |
| - | - |
| 0 | stage |
| 1 | production |

If you do not specify the `X-LC-Prod` HTTP header, LeanCloud will invoke the hook in the production environment.

### Response Format

The response body is a JSON object.

HTTP status code is used to indicate whether a request succeeded or failed.
A 2xx status code indicates success, and a 4xx/5xx status code indicates an error is encountered.
When an error is encountered, the response body is a JSON object with two fields: `code` and `error`,
where `code` is the error code (integer) and `error` is a brief error message (string).
The `code` may be identical to the HTTP status code,
but usually it is a customized error code more specific than the HTTP status code.
For example, if you try to save an object with an invalid key name:

```json
{
  "code":105,
  "error":"Invalid key name. Keys are case-sensitive and 'a-zA-Z0-9_' are the only valid characters. The column is: 'invalid?'."}
```

{# TODO 错误代码请看 [错误代码详解](./error_code.html)。#}

## Object

### Object Format

LeanStorage is built around objects.
Each object is consist of several key-value pairs,
where values are in a JSON compatible format.
Objects are schemaless, so you do not need to allocate keys at the beginning.
You only need to set key-value pairs as you wish and when needed.

For example, if you are implementing a Twitter-like social App, and a tweet may contain the following attributes (key-value pairs):

```json
{
  "content": "Serverless cloud for lightning-fast development.",
  "pubUser": "LeanCloud",
  "pubTimestamp": 1435541999
}
```

Keys can only contain letters, numbers, and underscores.
Values can be anything encoded in JSON.

Each object belongs to a class (table in traditional database terms).
We recommend to use `CapitalizedWords` to name your classes, and `mixedCases` to name your attributes.
This naming style helps to improve code readability.

Each time when an object is saved to the cloud, a unique `objectId` will be assigned to it.
`createdAt` and `updatedAt` will also be filled in by the cloud which indicate the time the object is created and updated.
These attributes are preserved, and you cannot modify them yourself.
For example, the object above could look like this when retrieved:

```json
{
  "content": "Serverless cloud for lightning-fast development.",
  "pubUser": "LeanCloud",
  "pubTimestamp": 1435541999,
  "createdAt": "2015-06-29T01:39:35.931Z",
  "updatedAt": "2015-06-29T01:39:35.931Z",
  "objectId": "558e20cbe4b060308e3eb36c"
}
```

`createdAt` and `updatedAt` are strings whose content is a UTC timestamp in ISO 8601 format with millisecond precision `YYYY-MM-DDTHH:MM:SS.MMMZ`.
`objectId` is a string unique in the class,
like the primary key of a relational database.

In LeanCloud's REST API, class-level operations use the class name as its endpoint.
For example:

```
https://{{host}}/1.1/classes/Post
```

Users (the built-in `_User` class) have a special endpoint:

```
https://{{host}}/1.1/users
```

Object-specific operations use nested URLs under the class.
For example:

```
https://{{host}}/1.1/classes/Post/<objectId>
```

### Creating Objects

To create a new object:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"content": "Serverless cloud for lightning-fast development.","pubUser": "LeanCloud","pubTimestamp": 1435541999}' \
  https://{{host}}/1.1/classes/Post
```

If succeed, you will receive `201 Created` with a `Location` header point to the URL of the object just created:

```sh
Status: 201 Created
Location: https://{{host}}/1.1/classes/Post/558e20cbe4b060308e3eb36c
```

And the response body is a JSON object with `objectId` and `createdAt` key-value pairs:

```json
{
  "createdAt": "2015-06-29T01:39:35.931Z",
  "objectId": "558e20cbe4b060308e3eb36c"
}
```

To tell LeanCloud to return full data, set the `fetchWhenSave` parameter to `true`:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"content": "Serverless cloud for lightning-fast development.","pubUser": "LeanCloud","pubTimestamp": 1435541999}' \
  https://{{host}}/1.1/classes/Post?fetchWhenSave=true
```

Class names can only contain letters, numbers, and underscores.
Every application can contain up to 500 classes, and there is no limit on objects in each class.

### Retrieving Objects

To fetch the object we just created:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://{{host}}/1.1/classes/Post/558e20cbe4b060308e3eb36c
```

The response body is a JSON object containing all attributes you specified above, and three preserved attributes `objectId`, `createdAt`, and `updatedAt`:

```json
{
  "content": "Serverless cloud for lightning-fast development.",
  "pubUser": "LeanCloud",
  "pubTimestamp": 1435541999,
  "createdAt": "2015-06-29T01:39:35.931Z",
  "updatedAt": "2015-06-29T01:39:35.931Z",
  "objectId": "558e20cbe4b060308e3eb36c"
}
```

To fetch the other objects this object points to, specify them in `include` parameter:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'include=author' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

If the class does not exist, you will receive a `404 Not Found` error:

```json
{
  "code": 101,
  "error": "Class or object doesn't exists."
}
```

If LeanCloud cannot find the object according to the  `objectId` you specified, you will receive an empty object (`200 OK`）:

```json
{}
```

Some built-in classes (class names with a leading underscore) may return a different result when an object does not exist.
For example:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://{{host}}/1.1/classes/_User/<NonexistObjectId>
```

will return:

```json
{
  "code": 211,
  "error": "Could not find user."
}
```

BTW, we recommend using `GET /users/<objectId>` to fetch user information, instead of directly querying the `_User` class.
See also [Retrieving Users](#Retrieving_Users).

### Updating Objects

To update an object, you can send a PUT request to the object URL.
LeanCloud will only update attributes you explicitly specified in the request (except for `updatedAt`).
For example, just update the content of a post:


```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"content": "Serverless cloud for lightning-fast development. https://leancloud.app"}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

If the update succeeds, `updatedAt` will be returned:

```json
{
  "updatedAt": "2015-06-30T18:02:52.248Z"
}
```

You can also pass the `fetchWhenSave` parameter,
which will return all updated attributes.

#### Counter

LeanCloud provides an `Increment` atomic operator to increase a counter-like attribute.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"upvotes":{"__op":"Increment","amount":1}}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

There is also a `Decrement` operator.
`Decrement`ing a positive number is equivalent to `Increment`ing a negative number.

#### Bitwise Operators

LeanCloud provides three bitwise operators for integers:

* `BitAnd`: bitwise `and`
* `BitOr`: bitwise `or`
* `BitXor`: bitwise `xor`

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"flags":{"__op":"BitOr","value": 0x0000000000000004}}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

#### Arrays

LeanCloud provides three atomic operators for arrays:

* `Add` extends an array attribute by appending elements from the given array.
* `AddUnique` is similar to `Add`, but only appending elements not already contained in the array attribute.
* `Remove` removes all occurrences of elements specified in the given array.

The given array mentioned above is passed in as the value of the `objects` key.

For example, to add some tags to the post:

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"tags":{"__op":"AddUnique","objects":["Frontend","JavaScript"]}}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

#### Conditional Updates

Suppose we are going to deduct some money from an `Account`, and we want to make sure this deduction will not result in a negative balance.
Then we can use conditional updates.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"balance":{"__op":"Decrement","amount": 30}}' \
  "https://{{host}}/1.1/classes/Account/558e20cbe4b060308e3eb36c?where=%7B%22balance%22%3A%7B%22%24gte%22%3A%2030%7D%7D"
```

Here `%7B%22balance%22%3A%7B%22%24gte%22%3A%2030%7D%7D` is the URL-encoded condition `{"balance":{"$gte": 30}}`.
Refer to [Queries](#Queries) for more examples.

If the condition is not met, the update will not be performed, and you will receive a `305` error:

```json
{
  "code" : 305,
  "error": "No effect on updating/deleting a document."
}
```

**Note: `where` must be passed in as the query parameter of URL.**

### Deleting Objects

To delete an object, send a `DELETE` request:

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://{{host}}/1.1/classes/Post/<objectId>
```

To delete an attribute from an object, send a `PUT` request with the `Delete` operator:

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"downvotes":{"__op":"Delete"}}' \
  https://{{host}}/1.1/classes/Post/<objectId>
```

#### Conditional Delete

Similar to conditional updates, we pass a URL-encoded `where` parameter to the `DELETE` request.

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  "https://{{host}}/1.1/classes/Post/<objectId>?where=%7B%22clicks%22%3A%200%7D" # {"clicks": 0}
```

Again, if the condition is not met, the update will not be performed, and you will receive a `305` error: 

```json
{
  "code" : 305,
  "error": "No effect on updating/deleting a document."
}
```

### Iterate over Objects

For classes with a moderate number of objects, we can iterate over all objects in the class via queries (with `skip` and `limit`).
However, for classes with a large number of objects, `skip` is inefficient.
Therefore, LeanCloud provides a `scan` endpoint to iterate over objects of a class efficiently.
By default, `scan` returns 100 results in ascending order by `objectId`.
You can ask LeanCloud to return up to 1000 results via specifying the `limit` parameter.

```sh
curl -X GET \
   -H "X-LC-Id: {{appid}}" \
   -H "X-LC-Key: {{masterkey}},master" \ # requires master key
   -G \
   --data-urlencode 'limit=10' \
   https://{{host}}/1.1/scan/classes/Article
```

LeanCloud will return a `results` array and a `cursor`.

```json
{
  "results":
   [
      {
        "tags"     :  ["clojure","\u7b97\u6cd5"],
        "createdAt":  "2016-07-07T08:54:13.250Z",
        "updatedAt":  "2016-07-07T08:54:50.268Z",
        "title"    :  "clojure persistent vector",
        "objectId" :  "577e18b50a2b580057469a5e"
       },
       ...
    ],
    "cursor": "pQRhIrac3AEpLzCA"}
```

The `cursor` will be `null` if there are no more results.
When `cursor` is not `null`, you can pass the `cursor` value to continue the iteration:

```sh
curl -X GET \
   -H "X-LC-Id: {{appid}}" \
   -H "X-LC-Key: {{masterkey}},master" \
   -G \
   --data-urlencode 'limit=10' \
   --data-urlencode 'cursor=pQRhIrac3AEpLzCA' \
   https://{{host}}/1.1/scan/classes/Article
```

Each `cursor` must be consumed within 10 minutes.
It becomes invalid after 10 minutes.

You can also specify `where` conditions for filtering:

```sh
curl -X GET \
   -H "X-LC-Id: {{appid}}" \
   -H "X-LC-Key: {{masterkey}},master" \
   -G \
   --data-urlencode 'limit=10' \
   --data-urlencode 'where={"score": 100}' \
   https://{{host}}/1.1/scan/classes/Article
```

As mentioned above, by default the results are in ascending order by `objectId`.
To return results ordered by another attribute,
pass that attribute as the `scan_key` parameter.

```sh
curl -X GET \
   -H "X-LC-Id: {{appid}}" \
   -H "X-LC-Key: {{masterkey}},master" \
   -G \
   --data-urlencode 'limit=10' \
   --data-urlencode 'scan_key=score' \
   https://{{host}}/1.1/scan/classes/Article
```

To return results in descending order, prefix a minus sign (`-`) to the value of the `scan_key`, e.g., `-score`.

The value of the `scan_key` passed must be strictly monotonous, and it cannot be used in where conditions.

### Batch Operations

To reduce network interactions, you can wrap create, update, and delete operations on multiple objects in one request.

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "requests": [
          {
            "method": "POST",
            "path": "/1.1/classes/Post",
            "body": {
              "content": "Most Internet-based applications are data-driven and share a very similar architecture...",
              "pubUser": "LeanCloud"
            }
          },
          {
            "method": "POST",
            "path": "/1.1/classes/Post",
            "body": {
              "content": "LeanMessage is designed with the following goals:...",
              "pubUser": "LeanCloud"
            }
          }
        ]
      }' \
  https://{{host}}/1.1/batch
```

Currently there is no limit on wrapped requests number,
but LeanCloud has a 20 MB size limit on request body for all API requests.

The wrapped operations will be performed according to the order given in the `requests` array.
And the response body will also be an array,
with corresponding length and order.
Each member of the results array will be a JSON object with one and only one key, and that key will be either `success` or `error`.
The value of `success` or `error` will be the response to the corresponding single request on success or failure respectively.

```
[
  {
    "error": {
      "code": 1,
      "error": "Could not find object by id '558e20cbe4b060308e3eb36c' for class 'Post'."
    }
  },
  {
    "success": {
      "updatedAt": "2017-02-22T06:35:29.419Z",
      "objectId": "58ad2e850ce463006b217888"
    }
  }
]
```

Be aware that the HTTP status `200` returned by a batch request only means LeanCloud had received and performed the operations.
It does not mean all operations within the batch request succeeded.

Besides `POST` requests in the above example,
you can also wrap `PUT` and `DELETE` requests in a batch request:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "requests": [
          {
            "method": "PUT",
            "path": "/1.1/classes/Post/55a39634e4b0ed48f0c1845b",
            "body": {
              "upvotes": 2
            }
          },
          {
            "method": "DELETE",
            "path": "/1.1/classes/Post/55a39634e4b0ed48f0c1845c"
          }
        ]
      }' \
  https://{{host}}/1.1/batch
```

Batch requests can also be used to replace requests with very long URLs (usually constructed with very complex queries or conditions) to bypass the limit on URL length enforced by the server side or the client side.

### Advanced Data Types

Besides standard JSON values, LeanCloud also supports advanced data types like Date and Files.
These advanced data types are encoded as a JSON object with a `__type` key.

**Date** contains an `iso` key, whose value is a UTC timestamp string in ISO 8601 format with millisecond precision `YYYY-MM-DDTHH:MM:SS.MMMZ`.


```json
{
  "__type": "Date",
  "iso": "2015-06-21T18:02:52.249Z"
}
```

As mentioned above, built-in date attributes `createdAt` and `updatedAt` are UTC timestamp strings. They are not enclosed in a JSON object.

**Byte** contains a `base64` key, whose value is a MIME base64 string (no whitespace characters).

```json
{
  "__type": "Bytes",
  "base64": "5b6I5aSa55So5oi36KGo56S65b6I5Zac5qyi5oiR5Lus55qE5paH5qGj6aOO5qC877yM5oiR5Lus5bey5bCGIExlYW5DbG91ZCDmiYDmnInmlofmoaPnmoQgTWFya2Rvd24g5qC85byP55qE5rqQ56CB5byA5pS+5Ye65p2l44CC"
}
```

**Pointer** contains a `className` key and an `objectId` key, whose values are the corresponding class name and objectId of the pointed value.

```json
{
  "__type": "Pointer",
  "className": "Post",
  "objectId": "55a39634e4b0ed48f0c1845c"
}
```

A pointer to a user contains a `className` of `_User`.
The leading underscore indicates that the `_User` class is built-in.
Similarly, pointers to roles and installations contain a `className` of `_Role` or `_Installation` respectively.
However, a pointer to a file is a special case:

```json
{
  "id": "543cbaede4b07db196f50f3c",
  "__type": "File"
}
```

**GeoPoint** contains `latitude` and `longitude` of the location:

```json
{
  "__type": "GeoPoint",
  "latitude": 39.9,
  "longitude": 116.4
}
```

We may add more advanced data types in the future, so you should not use `__type` on your own JSON objects.

## Queries

### Basic Queries

To list objects in a class, just send a GET request to the class URL:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  https://{{host}}/1.1/classes/Post
```

The response body is a JSON object containing a `results` key,
whose value is an array of objects:

```json
{
  "results": [
    {
      "content": "Most Internet-based applications are data-driven and share a very similar architecture...",
      "pubUser": "LeanCloud",
      "upvotes": 2,
      "createdAt": "2015-06-29T03:43:35.931Z",
      "objectId": "55a39634e4b0ed48f0c1845b"
    },
    {
      "content": "LeanMessage is designed with the following goals:...",
      "pubUser": "LeanCloud",
      "pubTimestamp": 1435541999,
      "createdAt": "2015-06-29T01:39:35.931Z",
      "updatedAt": "2015-06-29T01:39:35.931Z",
      "objectId": "558e20cbe4b060308e3eb36c"
    }
  ]
}
```

### Query Constraints

The `where` parameter is used to apply query constraints.
It should be encoded as JSON first, then URL encoded.

The simplest form of `where` parameter is a key-value pair (exact match).
For example, to query posts published by LeanCloud:


```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"pubUser":"LeanCloud"}' \
  https://{{host}}/1.1/classes/Post
```

Other operators available in the `where` parameter:

| Operator      | Description                           |
| ------------- | ------------------------------------- |
| `$ne`         | not equal to                          |
| `$lt`         | less than                             |
| `$lte`        | less than or equal to                 |
| `$gt`         | greater than                          |
| `$gte`        | greater than or equal to              |
| `$regex`      | match a regular expression            |
| `$in`         | contain                               |
| `$nin`        | not contain                           |
| `$all`        | contain all (for array type)          |
| `$exists`     | the given key exists                  |
| `$select`     | match the result of another query     |
| `$dontSelect` | not match the result of another query |

To query posts published on 2015-06-29 ：

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"createdAt":{"$gte":{"__type":"Date","iso":"2015-06-29T00:00:00.000Z"},"$lt":{"__type":"Date","iso":"2015-06-30T00:00:00.000Z"}}}' \
  https://{{host}}/1.1/classes/Post
```

Posts whose votes is an odd number less than 10:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"upvotes":{"$in":[1,3,5,7,9]}}' \
  https://{{host}}/1.1/classes/Post
```

Posts *not* published by LeanCloud:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"pubUser":{"$nin":["LeanCloud"]}}' \
  https://{{host}}/1.1/classes/Post
```

Posts have been voted by someone:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"upvotes":{"$exists":true}}' \
  https://{{host}}/1.1/classes/Post
```

Posts have not been voted:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"upvotes":{"$exists":false}}' \
  https://{{host}}/1.1/classes/Post
```

Suppose we use `_Followee` and `_Follower` classes for the following relationship, then we can query posts published by someone followed by the current user like this:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={
    "author": {
      "$select": {
        "query": {
          "className":"_Followee",
           "where": {
             "user":{
               "__type": "Pointer",
               "className": "_User",
               "objectId": "55a39634e4b0ed48f0c1845c"
             }
           }
        },
        "key":"followee"
      }
    }
  }' \
  https://{{host}}/1.1/classes/Post
```

Other parameters:

Parameter | Description
- | -
order | Order by. Prefix `-` for descending.
limit | The number of returned objects. Defaults to 100 and the max is 1000.
skip | Used for pagination.
keys | Only return specified keys. Prefix `-` for not including.

To query posts ordered by creation time:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=createdAt' \ # ascending
  https://{{host}}/1.1/classes/Post

curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=-createdAt' \ # descending 
  https://{{host}}/1.1/classes/Post
```

To sort results by multiple keys, use a comma to separate them:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=createdAt,-pubUser' \
  https://{{host}}/1.1/classes/Post
```

An example of pagination:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'limit=200' \
  --data-urlencode 'skip=400' \
  https://{{host}}/1.1/classes/Post
```

If you only care about the content and author of posts:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'keys=pubUser,content' \
  https://{{host}}/1.1/classes/Post
```

Note that returned objects will always contain preserved attributes such as `objectId`, `createdAt`, and `updatedAt`.

If you do not care about the post author:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'keys=-author' \
  https://{{host}}/1.1/classes/Post
```

The parameters mentioned above can be combined.

### Regex Queries

To query posts whose title begins with `WTO`:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -G \
  --data-urlencode 'where={"title":{"$regex":"^WTO.*","$options":"i"}}' \
  https://{{host}}/1.1/classes/Post
```

The above example specifies `i` in `options`.
Thus it is a case-insensitive search.

Available options (flags):

Options | Description | Notes
- | - | -
`i` | case-insensitive search | same as in JavaScript 
`m` | multi-line search | same as in JavaScript
`s` | allow `.` to match newline characters | added in ES2018
`x` | free-spacing and line comments | same as in Perl

Options can be combined, for example `"$options":"sixm"`.

### Array Queries

Queries on keys with an array type overloads some operators mentioned above:

Operators | Example | k is not an array | k is an array
- | - | - | -
n/a | `where={"k":2}` | `k == 2` | `k` contains 2
`in` | `where={"k":{"$in":[2,3]}}` | `k` is 2 or 3 | `k` contains 2 or 3
`all` | `where={"arrayKey":{"$all":[2,3]}}` | n/a | `k` contains 2 and 3

### Pointer Queries

To query a Pointer type attribute:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"post":{"__type":"Pointer","className":"Post","objectId":"558e20cbe4b060308e3eb36c"}}' \
  https://{{host}}/1.1/classes/Comment
```

To query objects with pointers according to another query on the pointed object, you can use the `$inQuery` operator.

For example, to query comments on posts with attached images:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"post":{"$inQuery":{"where":{"image":{"$exists":true}},"className":"Post"}}}' \
  https://{{host}}/1.1/classes/Comment
```

Be aware that the `limit` parameter (default 100, max 1000) also applies to inner queries.
Thus you may need to construct queries to get expected result deliberately.
Refer to [Caveats about Inner Queries](leanstorage_guide-js.html#Caveats_about_Inner_Queries) for more details.

To include pointed objects in one query, use `include` parameter.
For example, to query most recent 10 comments with the posts commented on:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=-createdAt' \
  --data-urlencode 'limit=10' \
  --data-urlencode 'include=post' \
  https://{{host}}/1.1/classes/Comment
```

Without the `include` parameter, the post attribute of returned comments will look like this:

```json
{
  "__type": "Pointer",
  "className": "Post",
  "objectId": "51e3a359e4b015ead4d95ddc"
}
```

With the `include=post` parameter, the post attribute will be dereferenced:

```json
{
  "__type": "Object",
  "className": "Post",
  "objectId": "51e3a359e4b015ead4d95ddc",
  "createdAt": "2015-06-29T09:31:20.371Z",
  "updatedAt": "2015-06-29T09:31:20.371Z",
  "content": "this is a post",
  "author": "LeanCloud"
}
```

You can use dots (`.`) for multi-level dereference:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'order=-createdAt' \
  --data-urlencode 'limit=10' \
  --data-urlencode 'include=post.author' \
  https://{{host}}/1.1/classes/Comment
```

And you can use comma (`,`) to separate multiple pointers to `include`.

### GeoPoint Queries

We have briefly described GeoPoint in the [Advanced Data Types](#Advanced_Data_Types) section above.

There is currently one limit on GeoPoints: every class can only contain one GeoPoint attribute.
Also, be aware that the range of `latitude` is `[-90.0, 90.0]`, and the range of `longitude` is `[-180.0, 180.0]`.

To query near objects, you can use the `$nearSphere` operator.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'limit=10' \
  --data-urlencode 'where={
        "location": {
          "$nearSphere": {
            "__type": "GeoPoint",
            "latitude": 39.9,
            "longitude": 116.4
          }
        }
      }' \
  https://{{host}}/1.1/classes/Post
```

Returned results will be ordered by distance.
The first result is the post published at the nearest location.
This order can be overwritten by the `order` parameter.  

To limit the maximum distance, you can use `$maxDistanceInMiles`, `$maxDistanceInKilometers`, or `$maxDistanceInRadians`.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={
        "location": {
          "$nearSphere": {
            "__type": "GeoPoint",
            "latitude": 39.9,
            "longitude": 116.4
          },
          "$maxDistanceInMiles": 10.0
        }
      }' \
  https://{{host}}/1.1/classes/Post
```

To query for objects within a rectangular area:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={
        "location": {
          "$within": {
            "$box": [
              { // southwestGeoPoint
                "__type": "GeoPoint",
                "latitude": 39.97,
                "longitude": 116.33
              },
              { // northeastGeoPoint
                "__type": "GeoPoint",
                "latitude": 39.99,
                "longitude": 116.37
              }
            ]
          }
        }
      }' \
  https://{{host}}/1.1/classes/Post
```

### Counting Results

You can pass `count=1` parameter to retrieve the count of matched results.
For example, if you just need to know a specific user have posted how many posts:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"pubUser":"LeanCloud"}' \
  --data-urlencode 'count=1' \
  --data-urlencode 'limit=0' \
  https://{{host}}/1.1/classes/Post
```

Since `limit=0`, LeanCloud will only return the count, and the results array will be empty.

```json
{
  "results": [],
  "count": 7
}
```

Given a nonzero `limit` parameter, LeanCloud will also return results together with the count.

### Compound Queries

You can use `$or` operator to query objects matching any one of several queries.

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -G \
  --data-urlencode 'where={"$or":[{"priority":{"$gt":2}},{"isComplete":true}]}' \
  https://{{host}}/1.1/classes/Todo
```

Similarly, you can use `$and` operator to query objects matching all subqueries.

```
--data-urlencode 'where={"$and":[{"price": {"$ne":0}},{"price":{"$exists":true}}]}' \
```

Be aware that non-filtering constraints such as `limit`, `skip`, `order`, and `include` are not allowed in subqueries of a compound query.

## Users

With users API, you can build an account system for your application quickly and conveniently.

Users (the `_User` class) share many traits with other classes. For example, `_User` is schema-free as well.
However, all user objects must have `username` and `password` attributes. `password` will be encrypted automatically.
`username` and `email` (if available) attributes must be unique (case sensitive).

### Signing Up

To create a new user:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"username":"hjiang","password":"f32@ds*@&dsa","region": "China"}' \
  https://{{host}}/1.1/users
```

As mentioned above, `username` and `password` are required. `password` will be stored in encrypted form, and LeanCloud will never return its value to the client side.

You may add any extra custom attributes.
`region` in the above is just an example.

If the registration succeed, LeanCloud will return `201 Created` and the `Location` will contain the URL for that user:

```sh
Status: 201 Created
Location: https://{{host}}/1.1/users/55a47496e4b05001a7732c5f
```

The response body is a JSON object containing three attributes:

```json
{
  "sessionToken":"qmdj8pdidnmyzp0c7yqil91oc",
  "createdAt":"2015-07-14T02:31:50.100Z",
  "objectId":"55a47496e4b05001a7732c5f"
}
```

`sessionToken` can be used to authenticate subsequent requests as that user.
It never changes, unless:

- the user forgets the password and resets it via email;
- the user modifies the password, and the developer enabled "Log out the user when password is updated" in dashboard;
- the `refreshSessionToken` API is invoked.

Logged in users will encounter a `403 Forbidden` permission error when invoking related APIs after the `sessionToken` refreshed.

LeanCloud can verify a user's email address automatically.
To enable this feature, access "Dashboard > LeanStorage > User > Setting", and select "Send verification emails when users register or change email addresses from clients".

The `emailVerified` of `_User` will be set to `true` once the new user clicked the verification link in the email.

You can also enable "Do not allow users with unverified phone numbers to log in" in the dashboard.

### Logging In

To log in a user with username and password:

```sh
curl -X POST \
-H "Content-Type: application/json" \
-H "X-LC-Id: {{appid}}" \
-H "X-LC-Key: {{appkey}}" \
-d '{"username":"hjiang","password":"f32@ds*@&dsa"}' \
https://{{host}}/1.1/login
```

The response body is a JSON object containing all the attributes of that user, except `password`:

```json
{
  "sessionToken":"qmdj8pdidnmyzp0c7yqil91oc",
  "updatedAt":"2015-07-14T02:31:50.100Z",
  "phone":"18612340000",
  "objectId":"55a47496e4b05001a7732c5f",
  "username":"hjiang",
  "createdAt":"2015-07-14T02:31:50.100Z",
  "emailVerified":false,
  "mobilePhoneVerified":false
}
```

To log in a user with email and password, just replace `username` with `email`:

```json
{"email":"hjiang@example.com","password":"f32@ds*@&dsa"}
```

### Refresh sessionToken

To refresh a user's `sessionToken`:

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  https://{{host}}/1.1/users/57e3bcca67f35600577c3063/refreshSessionToken
```

`X-LC-Session` can be omitted when using Master Key.

If succeed, new `sessionToken` will be returned, with user information:

```json
{
 "sessionToken":"5frlikqlwzx1nh3wzsdtfr4q7",
 "updatedAt":"2016-10-20T03:10:57.926Z",
 "objectId":"57e3bcca67f35600577c3063",
 "username":"leancloud",
 "createdAt":"2016-09-22T11:13:14.842Z",
 "emailVerified":false,
 "mobilePhoneVerified":false
}
```

#### Locking Users

Seven consecutive failed login attempts for a user within 15 minutes will trigger a lock.
After that, LeanCloud will return the following error:

```json
{
  "code":219,
  "error":"Tried too many times to signin."
}
```

LeanCloud will release this lock automatically in 15 minutes after the last login failure.
Developers cannot adjust this behavior via SDK or REST API.
During the locking period, the user is not allowed to log in,
even if they provide the correct password.
This restriction also applies to SDK and LeanEngine.

{# TODO
### 使用手机号码注册或登录

请参考 [短信服务 REST API 详解 &middot; 使用手机号码注册或登录](rest_sms_api.html#使用手机号码注册或登录)。
#}

### Verifying Email Address

As mentioned above, once a user clicked the verification link in the email, their `emailVerified` will be set to `true`.

`emailVerified` is a Boolean:

1. `true`: the user has verified their email address via clicking the link in the verification mail.
2. `false`: when a user's `email` attribute is set or modified, LeanCloud will set their `emailVerified` to `false` and send a verification email to the user. After the user clicks the verification link in the email, LeanCloud will set `emailVerified` to `true`. 
3. `null`: The user does not have an `email`, or the user object is created when the verifying new user's email address option is disabled. 

{# TODO 关于自定义邮件模板和验证链接 https://github.com/leancloud/docs/pull/3408 #}

The verification link expires in one week.
To resend the verification email:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"email":"hang@leancloud.rocks"}' \
  https://{{host}}/1.1/requestEmailVerify
```

### Resetting a Password

A user can reset their password via email:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"email":"hang@leancloud.rocks"}' \
  https://{{host}}/1.1/requestPasswordReset
```

If succeed, the response body will be an empty JSON object:

```json
{}
```

{# TODO 关于自定义邮件模板和验证链接 https://github.com/leancloud/docs/pull/3408 #}

{# TODO
### 手机号码验证

请参考 [短信服务 REST API 详解 - 用户账户与手机号码验证](rest_sms_api.html#用户账户与手机号码验证)。
#}

### Retrieving Users

To retrieve a user, you can send a GET request to the user URL (as in the `Location` header returned on [successful signing up](#Signing_Up)).

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://{{host}}/1.1/users/55a47496e4b05001a7732c5f
```

Alternatively, you can retrieve a user via their `sessionToken`:

```
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  https://{{host}}/1.1/users/me
```

The returned JSON object is the same as in [`/login`](#Logging_In).

If the user does not exist, a `400 Bad Request` will be returned:

```json
{
  "code": 211,
  "error": "Could not find user."
}
```

### Updating Users

Similar to [Updating Objects](#Updating_Objects), you can send a `PUT` request to the user URL to update a user's data.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  -H "Content-Type: application/json" \
  -d '{"phone":"18600001234"}' \
  https://{{host}}/1.1/users/55a47496e4b05001a7732c5f
```

The `X-LC-Session` HTTP header is to authenticate the modification,
whose value is the user's `sessionToken`.

If succeed, `updatedAt` will be returned.
This is the same as [Updating Objects](#Updating_Objects).

If you want to update `username`, then you have to ensure that the new value of `username` must not conflict with other existing users.

If you want to update `password` after verifying the old password,
then you can use `PUT /1.1/users/:objectId/updatePassword` instead.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  -H "Content-Type: application/json" \
  -d '{"old_password":"the_old_password", "new_password":"the_new_password"}' \
  https://{{host}}/1.1/users/55a47496e4b05001a7732c5f/updatePassword
```

Note this API still requires the `X-LC-Session` header.

### Querying Users

You can query users like [how you query regular objects](#Queries) by sending `GET` requests to `/1.1/users`.

However, for security concerns, all queries on users will be rejected by LeanCloud unless you use master key or have properly configured the `_User` class's ACL settings (in Dashboard > LeanStorage > _User > Permission).

### Deleting Users

Just like delete an object, send a `DELETE` request to delete a user.

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  https://{{host}}/1.1/users/55a47496e4b05001a7732c5f
```

The `X-LC-Session` HTTP header is used for authenticating this request.

### Linking Users

To allow users to use third-party accounts to log in to your application, you can use the `authData` attribute of users.

`authData` is a JSON object whose schema may be different for different services.

The simplest form of `authData` is as follows:

```json
{
  "anonymous": {
    "id": "random UUID with lowercase hexadecimal digits"
    // other optional keys 
  }
}
```

This is used for anonymous users,
for example, to provide a "try it before signing up" or "guest login" feature for your application.

The `authData` for an arbitrary platform:

```json
{
    "platform_name":
    {
      "uid": "unique user id on that platform (string)",
      "access_token": "access token for the user"
      // other optional keys 
    }
}
```

`authData` can have other additional keys, but it must contain both `uid` and `access_token`.
Often you need to verify `authData` yourself (except for certain platforms, see below).
Also, to avoid binding a third-party account to multiple users, you need to create a unique index for `authData.platform_name.uid` in Dashboard (LeanStorage > `_User`).

LeanCloud has built-in support for some popular social networks in China, such as [Weibo](http://weibo.com/), [WeChat](https://www.wechat.com/en) (*weixin* in Chinese pinyin), and [QQ](https://imqq.com/English1033.html):

```json
{
  "authData": {
    "weibo": {
      "uid": "123456789",
      "access_token": "2.00vs3XtCI5FevCff4981adb5jj1lXE",
      "expiration_in": "36000"
    }
  }
}

{
  "authData": {
    "weixin": {
      "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
      "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
      "expires_in": 1382686496
    }
  }
}

{
  "authData": {
    "qq": {
      "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
      "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
      "expires_in": 1382686496
    }
  }
}
```

LeanCloud will automatically verify access token for these platforms and will also create a unique index for each of these platforms.

In Weibo, WeChat, and QQ, a user's `openid` will be different for different applications.
If you have multiple applications, you may want to link them across all your applications.
To fulfill this requirement, you can utilize the UnionID function offered by Weibo, WeChat, and QQ.
For details, see the [UnionID](#UnionID) section below.

#### Third-Party Signing Up and Login

To sign up or log in via a third party account, you also send a POST request with the `authData`.
For example, to use a QQ account to login:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
     "authData": {
       "qq": {
         "openid": "0395BA18A5CD6255E5BA185E7BEBA242",
         "access_token": "12345678-SaMpLeTuo3m2avZxh5cjJmIrAfx4ZYyamdofM7IjU",
         "expires_in": 1382686496
         }
    }
    }' \
  https://{{host}}/1.1/users
```

As mentioned above, LeanCloud will verify `access_token` for QQ accounts.
If the verification succeeds, LeanCloud will return `200 OK` or `201 Created` depending on if there is an existing user linking this QQ account.
In both cases, the user URL will also be returned in the `Location` HTTP header.

The response body will be a JSON object whose content is similar to the one returned when creating or logging in a regular user.
For new users, LeanCloud will automatically assign a random username, e.g., `ec9m07bo32cko6soqtvn6bko5`.

#### UnionID

As mentioned above, we can utilize the UnionID mechanism offered by Weibo, WeChat, and QQ to link users across multiple applications.

To do so, you need to use a specific schema of `authData`, with the following keys:

- `unionid`: the user's UnionID;
- `main_account`: `true`, indicating we will use UnionID;
- `plat_form`: `weibo`, `weixn`, or `qq`.

Let's look at an example of WeChat UnionID.

Suppose you have a WeChat mini program named `foo`.
Then to sign up or log in a user with UnionID:

```sh
curl -X POST \
  -H "X-LC-Id: zbOycL3cTIr3gOTgFAyQYiop-gzGzoHsz" \
  -H "X-LC-Key: pp4T5TmSR8mscQONix0xFXpT" \
  -H "Content-Type: application/json" \
  -d '{
     "authData": {
       "foo": {
         "openid": "oTY851cqL0gk3DqW3xINqG1Q4PTc",
         "access_token": "12_b6mz7ujXbTY4vpbqCRaKVa_y0Ij3N9grCeVtM8VJT8KFd4qnQ9lXtBsZVxG6x9c9Nay_oNgvbKK7KYKbn8R2P7uEgA0EhsXMHmxkx-xU-Tk",
         "expires_in": 7200,
         "refresh_token": "12_71UYUnqHDuIfekimsJsYjBDfY67ilo30fDqrYkqlwZtxNgcBhMmQgDVhT6mJWkRg0mngvX9kXeCGP8kmBWdvUtc5ngRiN5LDTWAau4du838",
         "scope": "snsapi_userinfo",
         "unionid": "ox7NLs-e-32ZyHg2URi_F2iPEI2U",
         "platform": "weixin",
         "main_account":"true"
        }
      }
    }' \
   https://{{host}}/1.1/users
```

And to sign up or log in a user for your another WeChat mini program `bar`:

```sh
curl -X POST \
  -H "X-LC-Id: zbOycL3cTIr3gOTgFAyQYiop-gzGzoHsz" \
  -H "X-LC-Key: pp4T5TmSR8mscQONix0xFXpT" \
  -H "Content-Type: application/json" \
  -d '{
     "authData": {
       "bar": {
         "openid": "ohxoK3ldpsGDGGSaniEEexxx",
         "access_token": "10_QfDeXVp8fUKMBYC_d4PKujpuLo3sBV_pxxxxIZivS77JojQPLrZ7OgP9PC9ZvFCXxIa9G6BcBn45wSBebsv9Pih7Xdr4-hzr5hYpUoSA",
    "unionid": "ox7NLs06ZGfdxxxxxe0F1po78qE",
         "expires_in": 7200,
         "refresh_token": "10_RZXedP8Ia9G6B_Xxoxjxpu1o4DBV_hzr5hYpUoSAd87JojQPLrZ7OgP10e9ZvFCXxcon54wSfaero_CBebsd20h5ddr3-4PKuIZivS",
         "scope": "snsapi_userinfo",
         "unionid": "ox7NLs-e-32ZyHg2URi_F2iPEI2U",
         "platform": "weixin",
         "main_account":"true"
        }
      }
    }' \
   https://{{host}}/1.1/users
```

Note that `openid` is different but `unionid` is the same.

And the `authData` stored on LeanCloud will be something like:

```json
{
  "foo": {
    // ...
  },
  "_weixin_unionid": {
    "uid": "ox7NLs-e-32ZyHg2URi_F2iPEI2U"
  },
  "bar": {
    // ...
  }
}
```

The `_weixin_unionid` key is automatically created by LeanCloud.

#### Linking a Third-Party Account

To link a third-party account to an existing user,
just update this user's `authData` attribute.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  -H "Content-Type: application/json" \
  -d '{
        "authData": {
          "weixin": {
            "uid": "123456789",
            "access_token": "2.00vs3XtCI5FevCff4981adb5jj1lXE",
            "expiration_in": "36000"
            // ...
          }
        }
      }' \
  https://{{host}}/1.1/users/55a47496e4b05001a7732c5f
```

This user can be authenticated via matching `authData` afterward.

#### Unlinking a Third-Party Account

Similarly, to unlink a user from a third party account,
just delete the platform in their `authData` attribute.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: 6fehqhr2t2na5mv1aq2om7jgz" \
  -H "Content-Type: application/json" \
  -d '{"authData.weixin":{"__op":"Delete"}}' \
  https://{{host}}/1.1/users/5b7e53a767f356005fb374f6
```

## Roles

Role is an abstraction to control data access based on ACL.
Roles are objects containing users and other roles.
For example, moderators and administrators are common roles in an application.

LeanCloud has a preserved class `_Role` for roles.
Roles have some specific attributes:

- `name`: Required. The name of the role, which can only be set on creation. The name can only contain alphanumeric characters, spaces, dashes, and underscores.
- `users`: A relation to a set of users.
- `roles`: A relation to a set of child roles.

For security concerns, roles are typically created and managed manually or via a separate management interface, not directly in your applications.

### Creating Roles

Creating a role is similar to creating an object, except that you must specify the `name` and `ACL` attributes. To prevent allowing wrong users to modify a role accidentally, you should set a restrictive and rigid `ACL`.

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "Manager",
        "ACL": {
          "*": {
            "read": true
          }
        }
      }' \
  https://{{host}}/1.1/roles
```

The response is the same as creating an object.

To create a role with existing child roles and users:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "CLevel",
        "ACL": {
          "*": {
            "read": true
          }
        },
        "roles": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_Role",
              "objectId": "55a48351e4b05001a774a89f"
            }
          ]
        },
        "users": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_User",
              "objectId": "55a47496e4b05001a7732c5f"
            }
          ]
        }
      }' \
  https://{{host}}/1.1/roles
```

You may have noticed that there is a new operator `AddRelation` we have not seen before.
This operator adds a relation to an object.
The actual implementation of relation is quite complicated for performance issues,
but conceptually you can consider them as arrays of pointers, and they are only used in roles.

### Retrieving Roles

Retrieving a role is similar to retrieving an object:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://{{host}}/1.1/roles/55a483f0e4b05001a774b837
```

The response body will be a JSON object:

```json
{
  "name":"CLevel",
  "createdAt":"2015-07-14T03:37:20.992Z",
  "updatedAt":"2015-07-14T03:37:20.994Z",
  "objectId":"55a483f0e4b05001a774b837",
  "users":{
    "__type":"Relation",
    "className":"_User"
  },
  "roles":{
    "__type":"Relation",
    "className":"_Role"
  }
}
```

To query for the specific users and roles in a role, you need to send another query using the `$relatedTo` operator:

```sh
where={
  "$relatedTo":{
    "object":{
      "__type":"Pointer",
      "className":"_Role",
      "objectId":"objectId of a role"
  },
  "key":"users"}
}
```

### Updating Roles

Updating roles are similar to update objects, except that `name` cannot be modified, as mentioned above.
To add or remove users and child roles, you can use `AddRelation` and `RemoveRelation` operators.

Suppose we have a `Manager` role with objectId `55a48351e4b05001a774a89f`, we can add a user to it as below:

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "users": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_User",
              "objectId": "55a4800fe4b05001a7745c41"
            }
          ]
        }
      }' \
  https://{{host}}/1.1/roles/55a48351e4b05001a774a89f
```

Similarly, to remove a child role:

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "roles": {
          "__op": "RemoveRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_Role",
              "objectId": "55a483f0e4b05001a774b837"
            }
          ]
        }
      }' \
  https://{{host}}/1.1/roles/55a48351e4b05001a774a89f
```

### Deleting Roles

Deleting roles are similar to delete objects.
It is authenticated with the `X-LC-Session` HTTP header.
The session token passed in must belong to a user who has the permission to delete the specified role.

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  https://{{host}}/1.1/roles/55a483f0e4b05001a774b837
```

### Roles and ACL

As demonstrated above, accessing LeanCloud data via REST API is also restricted by ACL, just as SDKs.

Roles make maintaining ACL easier.
For example, to set an ACL of an object with the following permissions:

- It can be read by `Staff`s.
- It can only be written by `Manager`s and its creator.

```json
{
  "55a4800fe4b05001a7745c41": {
    "write": true
  },
  "role:Staff": {
    "read": true
  },
  "role:Manager": {
    "write": true
  }
}
```

The creator belongs to the `Staff` role, and the `Manager` role is a child role of the `Staff` role.
Therefore, since they will inherit read permissions, we did not grant them the read permission manually.

Let's look at another example of permission inherence among roles.
In UGC applications such as forums, `Administrators` typically have all the permissions of `Moderators`.
Thus `Administrators` should be a child role of `Moderators`.

```sh
curl -X PUT \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{
        "roles": {
          "__op": "AddRelation",
          "objects": [
            {
              "__type": "Pointer",
              "className": "_Role",
              "objectId": "<AdministratorsRoleObjectId>"
            }
          ]
        }
      }' \
  https://{{host}}/1.1/roles/<ModeratorsRoleObjectId>
```

## Files

### Associating with Objects

As mentioned above, files can be considered as a special form of pointers.
To associate a file object with an object, we just pass the file object `{"id": "objectId of the file", "__type": "File"}` to an attribute of that file.
For example, to create a `Staff` object with a photo:

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
   -H "Content-Type: application/json" \
  -d '{
        "name": "hjiang",
        "picture": {
          "id": "543cbaede4b07db196f50f3c",
          "__type": "File"
        }
      }' \
  https://{{host}}/1.1/classes/Staff
```

### Deleting Files

Deleting files is similar to deleting objects:

```sh
curl -X DELETE \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  https://{{host}}/1.1/files/543cbaede4b07db196f50f3c
```

## Schema

You can use REST API to fetch the data schema of your application.
For security concerns, master key is required to fetch data schema.

To fetch the schema of all classes:

```sh
curl -X GET \
   -H "X-LC-Id: {{appid}}" \
   -H "X-LC-Key: {{masterkey}},master" \
   https://{{host}}/1.1/schemas
```

Result:

```json
{
  "_User":{
    "username"     : {"type":"String"},
    "password"     : {"type":"String"},
    "objectId"     : {"type":"String"},
    "emailVerified": {"type":"Boolean"},
    "email"        : {"type":"String"},
    "createdAt"    : {"type":"Date"},
    "updatedAt"    : {"type":"Date"},
    "authData"     : {"type":"Object"}
  }
  // other classes
}
```

You can also fetch a single class's schema:

```sh
curl -X GET \
   -H "X-LC-Id: {{appid}}" \
   -H "X-LC-Key: {{masterkey}},master" \
   https://{{host}}/1.1/schemas/_User
```

Data schema can be used with tools such as code generators and internal management interfaces.
