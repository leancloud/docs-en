
{% import "views/_parts.html" as include %}

{{ include.setService('engine') }}

# LeanEngine REST API Guide

## Switching Environments

You can use HTTP header `X-LC-Prod` to specify LeanEngine environment:

* `X-LC-Prod: 0`: the staging environment;
* `X-LC-Prod: 1`: the production environment.

## Cloud Functions

To invoke a cloud function, send a POST request to `/functions/:name`.
Its parameters and result are both in JSON format:

```sh
curl -X POST -H "Content-Type: application/json; charset=utf-8" \
       -H "X-LC-Id: {{appid}}" \
       -H "X-LC-Key: {{appkey}}" \
       -d '{"movie":"Despicable Me"}' \
https://{{host}}/1.1/functions/averageStars
```

Response:

```json
{
  "result": {
    "movie": "Despicable Me",
    "stars": "2.5"
  }
}
```

If the cloud function needs to associate with a user,
you can pass the corresponding `sessionToken` in the `X-LC-Session` HTTP header:


```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "X-LC-Session: qmdj8pdidnmyzp0c7yqil91oc" \
  -H "Content-Type: application/json" \
  -d '{}' \
  https://{{host}}/1.1/functions/hello
```

Alternatively, you can send a POST request to `/call/:name`,
whose parameters and result are in AVObject format (JSON with metadata):

```sh
curl -X POST \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"__type": "Object", "className": "Post", "pubUser": "LeanCloud"}' \
  https://{{host}}/1.1/call/addPost
```

Response:

```json
{
  "result": {
    "__type": "Object",
    "className": "Post",
    "pubUser": "LeanCloud"
  }
}
```

Besides a simple AVObject, the result can be data structures consist of AVObjects.
For example, an array with a number and a Todo object:

```json
{
  "result": [
    1,
    {
      "title": "engineer meeting",
      "createdAt": {
        "__type": "Date",
        "iso": "2019-04-28T08:34:12.932Z"
      },
      "updatedAt": {
        "__type": "Date",
        "iso": "2019-04-28T08:34:12.932Z"
      },
      "objectId": "5cc5658443e78cb53fe7b731",
      "__type": "Object",
      "className": "Todo"
    }
  ]
}
```

If a timeout is encountered, you will receive a response with HTTP status code 503, 524, or 141.

You can refer to the following documents for more information:

* [Node.js](leanengine_cloudfunction_guide-node.html)
* [Python](leanengine_cloudfunction_guide-python.html)
* [PHP](leanengine_cloudfunction_guide-php.html)
* [Java](leanengine_cloudfunction_guide-java.html)
* [.Net](leanengine_cloudfunction_guide-dotnet.html)
