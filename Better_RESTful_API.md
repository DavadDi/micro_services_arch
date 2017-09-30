# Better RESTful API实践

## 1. 资源采用名词，采用复数形式

| Resource  | GET read               | POST create                  | PUT update                               | PATCH  Partially update          | DELETE                 |
| --------- | ---------------------- | ---------------------------- | ---------------------------------------- | -------------------------------- | ---------------------- |
| /cars     | Returns a list of cars | Create a new car             | Bulk update of cars                      | Bulk Partially update of cars    | Delete all cars        |
| /cars/711 | Returns a specific car | Method not allowed (405)     | Updates a specific car [200/204, or 404] | Partially Updates a specific car | Deletes a specific car |
|           | 200 or 404             | 201[ Location' header or 404 | 200/204 or 404                           |                                  | 200 or 404             |

尽量采用复数格式，不是绝对，如果对于全局唯一的配置，则可以采用单数格式。



* Twitter: https://developer.twitter.com/en/docs/api-reference-index
* Facebook: https://developers.facebook.com/docs/graph-api
* LinkedIn: https://developer.linkedin.com/
* GitHub API: https://developer.github.com/v3/

## 2. 采用子资源关联

```
GET /cars/711/drivers/ Returns a list of drivers for car 711
GET /cars/711/drivers/4 Returns driver #4 for car 711
```

| Action | URI                    | DESC                                     |
| :----- | :--------------------- | :--------------------------------------- |
| GET    | /tickets/12/messages   | Retrieves list of messages for ticket #12 |
| GET    | /tickets/12/messages/5 | Retrieves message #5 for ticket #12      |
| POST   | /tickets/12/messages   | Creates a new message in ticket #12      |
| PUT    | /tickets/12/messages/5 | Updates message #5 for ticket #12        |
| PATCH  | /tickets/12/messages/5 | Partially updates message #5 for ticket #12 |
| DELETE | /tickets/12/messages/5 | Deletes message #5 for ticket #12        |

是否需要采用多层次的关联，则需要根据子资源能否脱离父类资源独立存在。



如果对于某些资源的 **Action** 不能够对应到 **CRUD** 操作，

1. 比如对于资源的 activated，操作则可以映射成 PATCH 操作
2. 比如对于GitHub上的gist点赞，则可以映射成资源的子属性

```
PUT /gists/:id/star
DELETE /gists/:id/star
```

3. 如果对于多个资源的 search，不能够映射到各个资源上

```
GET /search
```



## 3. 采用 HATEOAS

HATEOAS = ypermedia **a**s **t**he **E**ngine **o**f **A**pplication **S**tate

```json
{
  "id": 711,
  "manufacturer": "bmw",
  "model": "X5",
  "seats": 5,
  "drivers": [
   {
    # "id": "23", no need
    "name": "Stefan Jauker",
    "links": [
     {
     "rel": "self",
     "href": "/api/v1/drivers/23"
    }
   ]
  }
 ]
}
```



## 4. 提供过滤、排序、字段选择和分页

**过滤**

```
GET /cars?color=red Returns a list of red cars
GET /cars?seats<=2 Returns a list of cars with a maximum of 2 seats

gt – Greater than
lt – Less than
eq – Equal to
ge – Greater than or equal to
le – Less than or equal to
```

**排序**

```
GET /cars?sort=-manufactorer,+model
```

**字段选择**

```
GET /cars?fields=manufacturer,model,id,color
```

**分页**

```shell
GET /cars?offset=10&limit=5

# github存放在 http HEADer中，参见：https://developer.github.com/v3/#pagination
Link: <https://xxx/api/v1/cars?offset=15&limit=5>; rel="next",
<https://xxx/api/v1/cars?offset=50&limit=3>; rel="last",
<https://xxx/api/v1/cars?offset=0&limit=5>; rel="first",
<https://xxx/api/v1/cars?offset=5&limit=5>; rel="prev",
```

## 5. API 版本号

版本号放在url中，方便与日志收集后的过滤，同时版本号用整数， v1、v2， 不采用 v2.5的方式

```
/blog/api/v1
```



## 6. 创建和更新操作返回资源主题信息

如果是采用Post Create参数应该返回创建后的资源信息，HTTP Status Code == 201， 并在 Header 中填写 “ Location       = "Location" ":" absoluteURI” 表明创建资源后的URL地址；



如果是采用PUT操作资源，则应该返回更新后的资源字段，或者附件 created_at 或 updated_at等字段信息；

## 7. 采用HTTP Status Code处理错误

| code | Status                | Message                                  |
| ---- | --------------------- | ---------------------------------------- |
| 200  | OK                    | Eyerything is working                    |
| 201  | OK                    | New resource has been created            |
| 204  | OK                    | The resource was successfully deleted    |
| 304  | Not Modified          | The client can use cached data           |
| 400  | Bad Request           | The request was invalid or cannot be served. The exact error should be explained in the error payload. E.g. „The JSON is not valid“ |
| 401  | Unauthorized          | The request requires an user authentication |
| 403  | Forbidden             | The server understood the request, but is refusing it or the access is not allowed. |
| 404  | Not found             | There is no resource behind the URI.     |
| 409  | CONFLICT              | Whenever a resource conflict would be caused by fulfilling the request. Duplicate entries, deleting root objects when cascade-delete not supported are a couple of examples. |
| 422  | Unprocessable Entity  | Should be used if the server cannot process the enitity, e.g. if an image cannot be formatted or mandatory fields are missing in the payload. |
| 429  | Too Many Requests     | Too many requests hit the API too quickly. We recommend an exponential backoff of your requests. |
| 500  | Internal Server Error | API developers should avoid this error. If an error occurs in the global catch blog, the stracktrace should be logged and not returned as response. |
|      |                       |                                          |
|      |                       |                                          |
|      |                       |                                          |

使用Error Payload

```json
{
  "errors": [
   {
    "userMessage": "Sorry, the requested resource does not exist",
    "internalMessage": "No car found in the database",
    "code": 34,
    "more info": "http://dev.mwaysolutions.com/blog/api/v1/errors/12345"
   }
  ]
} 
```



```json
{
  "code" : 1234,
  "message" : "Something bad happened :(",
  "description" : "More details about the error here"
}
```



```json
{
  "code" : 1024,
  "message" : "Validation Failed",
  "errors" : [
    {
      "code" : 5432,
      "field" : "first_name",
      "message" : "First name cannot have fancy characters"
    },
    {
       "code" : 5622,
       "field" : "password",
       "message" : "Password cannot be blank"
    }
  ]
}
```



```json
{
  "code":200,
  "status":"success",
  "data": 
  {
    "lacksTOS":false,
    "invalidCredentials":false,
    "authToken":"4ee683baa2a3332c3c86026d"
  }
}

{
  "code":401,
  "status":"error",
  "message":"token is invalid",
  "data":"UnauthorizedException"
}
```



## 9. 输入和输出尽可能JSON格式

[XML API vs JSON API](http://www.google.com/trends/explore?q=xml+api#q=xml%20api%2C%20json%20api&cmpt=q)，目前JSON API的调用已经大幅度领先于XML；尽可能返回JSON格式的数据，并保证返回的JSON格式数据具备良好的阅读性，例如默认设置参数（?pretty=true），方便与用户展示和调试；返回的JSON格式内容启用压缩功能；JSON格式推荐采用下划线分割page_size，而不是大小写格式pageSize；



## 10. API 访问限速

RFC 6585 中 [429 Too Many Requests](http://tools.ietf.org/html/rfc6585#section-4) 定义了该种情况。

- X-Rate-Limit-Limit - The number of allowed requests in the current period
- X-Rate-Limit-Remaining - The number of remaining requests in the current period
- X-Rate-Limit-Reset - The number of seconds left in the current period

##参考资料

* [10 Best Practices for Better RESTful API](https://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/)
* [ReST APIs | Best Practices & Security](https://blog.wishtack.com/rest-apis-best-practices-and-security/)
* [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
* [ Node.js Restful API tutorial](http://howtocodejs.com/how-to/create-restful-api-node-js/)
* [Google Cloud Plateform](https://cloud.google.com/apis/design/resources)
* [Google Seller Rest Filter](https://developers.google.com/ad-exchange/seller-rest/reporting/filtering)
* [API version should be included in the URL or in a header](http://stackoverflow.com/questions/389169/best-practices-for-api-versioning)
* [GitHub API](https://developer.github.com/v3/)
* [Striper](https://stripe.com/docs/api)
* [XML API vs JSON API](http://www.google.com/trends/explore?q=xml+api#q=xml%20api%2C%20json%20api&cmpt=q)
* [Architectural Styles andthe Design of Network-based Software Architectures](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [RESTful Service Best Practices](www.restapitutorial.com/media/RESTful_Best_Practices-v1_0.pdf)