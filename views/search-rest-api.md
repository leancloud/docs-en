# Full-Text Search REST API Guide

Full-text search is a common feature used by a lot of modern applications.
You might already know that it is possible to implement full-text search with [`$regex` queries](rest_api.html#regex-queries),
but this approach will not work efficiently when there are massive objects or fields in a class.
To solve the problem, LeanCloud provides dedicated REST APIs for full-text search powered by [the Elasticsearch engine][elastic].

[elastic]: https://www.elastic.co/elasticsearch/

## Overview

URL | HTTP Method | Functionality
- | - | -
/search/select | GET | full-text search
/search/mlt | GET | "more like this" query; used to find similar documents

The current API version is `1.1`.
For request format and response format, please refer to the [Request Format section](rest_api.html#Request-Format) and [Response Format section](rest_api.html#Response-Format) of REST API Guide.

## Enable Search Indexes

Before using search APIs, you must first enable search indexes for classes.
To do so, go to your app's Dashboard > LeanStorage > In-app searching, and click on the `Add Class` button.

Be aware of the following limitations:

- An application with a Developer/Business Plan can enable search indexes for at most 5/10 classes.
- An application with a Developer/Business Plan can enable search indexes for at most 5/10 columns for each search-enabled class.
  However, `objectId`, `createdAt`, and `updatedAt` are always enabled for a search-enabled class, and they do not count into the column limit.
- If LeanCloud has not received any full-text search API requests within two weeks after you enabled search indexes for a class, the search indexes of this class will be disabled.

## Full-Text Search

To search for objects, you need to send a `GET` request to `/search/select`.
For example, to search for `dennis` in `GameScore` class:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  "https://{{host}}/1.1/search/select?q=dennis&clazz=GameScore"
```

The following query parameters are available:

Parameter|Required|Description
---|---|---
`q`|Required|[Elasticsearch query string]
`skip`|Optional|Skipped results. Used for pagination.
`limit`|Optional|The number of returned objects. The default value is 100 and the maximum value is 1000.
`sid`|Optional|Elasticsearch [scroll id]. Returned by a previous search. Used for pagination.
`fields`|Optional|Comma-seperated column list.
`highlights`|Optional|Highlighted keywords. It can be a comma-separated string or wildcard `*`.
`clazz`|Optional|Class name. If not specified, all search enabled classes will be searched.
`include`|Optional|Include objects referenced by Pointers. Example: `user,comment`.
`order`|Optional|Order by. Prefix `-` for descending. Example: `-score,createdAt`.
`sort`|Optional|Refer to [Elasticsearch sort] documentation and the [GeoPoints sorting](#geopoints-sorting) section below for details. 

[Elasticsearch query string]: https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-query-string-query.html#query-string-syntax
[scroll id]: https://www.elastic.co/guide/en/elasticsearch/reference/6.5/search-request-scroll.html
[Elasticsearch sort]: https://www.elastic.co/guide/en/elasticsearch/reference/6.5/search-request-sort.html

The response will be something like:

``` json
{
"hits": 1,
"results": [
  {
    "_app_url": "http://stg.pass.com//1/go/com.leancloud/classes/GameScore/51e3a334e4b0b3eb44adbe1a",
    "_deeplink": "com.leancloud.appSearchTest://leancloud/classes/GameScore/51e3a334e4b0b3eb44adbe1a",
    "_highlight": null,
    "updatedAt": "2011-08-20T02:06:57.931Z",
    "playerName": "Sean Plott",
    "objectId": "51e3a334e4b0b3eb44adbe1a",
    "createdAt": "2011-08-20T02:06:57.931Z",
    "cheatMode": false,
    "score": 1337
  }
],
"sid": "cXVlcnlUaGVuRmV0Y2g7Mzs0NDpWX0NFUmFjY1JtMnpaRDFrNUlBcTNnOzQzOlZfQ0VSYWNjUm0yelpEMWs1SUFxM2c7NDU6Vl9DRVJhY2NSbTJ6WkQxazVJQXEzZzswOw=="
}
```

In the above response body:

- `hits`: Total number of matched results.
- `_highlight`: Highlighted search results with keywords wrapped in `em` tags. If you did not pass the `highlights` query parameter, `null` will be returned.

## GeoPoints Sorting

GeoPoints can be sorted by their distances to a given GeoPoint.
For example, to sort players near a GeoPoint (`[39.9, 116.4]`):

```json
{
  "_geo_distance" : {
                "location" : [39.9, 116.4],
                "order" : "asc",
                "unit" : "km",
                "mode" : "min",
   }
}
```

## More Like This Query

To search for similar objects, you can send a `GET` request to `/search/mlt`.
This may be used in a recommendation system.

For example, to search for posts with tags similar to `clojure`:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  "https://{{host}}/1.1/search/mlt?like=clojure&clazz=Post&fields=tags"
```

The response body will be something like:

```json
{
"results": [
  {  
    "tags":[  
          "clojure",
           "data structure and algorithm"
     ],
     "updatedAt":"2016-07-07T08:54:50.268Z",
     "_deeplink":"cn.leancloud.qfo17qmvr8w2y6g5gtk5zitcqg7fyv4l612qiqxv8uqyo61n:\/\/leancloud\/classes\/Article\/577e18b50a2b580057469a5e",
     "_app_url":"https:\/\/leancloud.cn\/1\/go\/cn.leancloud.qfo17qmvr8w2y6g5gtk5zitcqg7fyv4l612qiqxv8uqyo61n\/classes\/Article\/577e18b50a2b580057469a5e",
     "objectId":"577e18b50a2b580057469a5e",
     "_highlight":null,
     "createdAt":"2016-07-07T08:54:13.250Z",
     "className":"Article",
     "title":"clojure persistent vector"
  },
  // …
],
"sid": null
}
```

You can use the `likeObjectIds` parameter instead of `like` to search for posts with similar tags:

```sh
curl -X GET \
  -H "X-LC-Id: {{appid}}" \
  -H "X-LC-Key: {{appkey}}" \
  "https://{{host}}/1.1/search/mlt?likeObjectIds=577e18b50a2b580057469a5e&clazz=Post&fields=tags"
```

Below are all the query parameters available:

Parameter|Required|Description
---|---|---
`clazz`|Required|Class name.
`like`|Optional|Keywords. **You need to specify either this parameter or the `likeObjectIds` parameter.**
`likeObjectIds`|Optional|Comma-seperated objectId list. **You need to specify either this parameter or the `like` parameter.**
`min_term_freq`|Optional|The minimum term frequency below which the terms will be ignored from the input document. Defaults to 2.
`min_doc_freq`|Optional|The minimum document frequency below which the terms will be ignored from the input document. Defaults to 5.
`max_doc_freq`|Optional|The maximum document frequency above which the terms will be ignored from the input document. This could be useful to ignore highly frequent words such as stop words. Defaults to 0.
`skip`|Optional|Skipped results. Used for pagination.
`limit`|Optional|The number of returned objects. The default value is 100 and the maximum value is 1000.
`fields`|Optional|Comma-seperated column list.
`highlights`|Optional|Highlighted keywords. It can be a comma-separated string or wildcard `*`.
`include`|Optional|Include objects referenced by Pointers. Example: `user,comment`.

You can also refer to the [Elasticsearch documentation][elastic-more-like-this] for more information.

[elastic-more-like-this]: https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-mlt-query.html
