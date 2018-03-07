# readPlainTextDotNetCoreWepApi

总有些时候我们希望获得webApi Request的纯文本
那么怎么做呢？很简单。如下所示
```c
        public string GetJsonString([FromBody]string content)
        {
            return "content: " + content ;
        }
```
测试结果如下
```
request：
POST http://localhost:5000/api/values/GetJsonString HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: application/json
Content-Length: 6
Host: localhost:5000
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
"test"

response：
HTTP/1.1 200 OK
Date: Wed, 07 Mar 2018 14:36:37 GMT
Content-Type: text/plain; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked

content: test 
```
可以看到content被赋值test。 但有个问题request body的内容必须是合法的json而且request 的media type也得是json

举个例子，

```
request：
POST http://localhost:5000/api/values/GetJsonString HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: application/xml
Content-Length: 4
Host: localhost:5000
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
test

response：
HTTP/1.1 200 OK
Date: Wed, 07 Mar 2018 14:31:07 GMT
Content-Type: text/plain; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked

content:  
```
可以看到由于request body的内容 test 并不是个合法的xml，所以我们返回的content是空。



有个更好的方法 如下所示，这种方法不论是media type都可以获得request body 的纯文本
```c
        public string GetJsonString3(string content)
        {
            var reader = new StreamReader(Request.Body);
            var contentFromBody = reader.ReadToEnd();
            return "content: " + content 
                   + " contentFromBody: " + contentFromBody;
        }
```
测试结果
```
request：
POST http://localhost:5000/api/values/GetJsonString3 HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: application/xml
Content-Length: 4
Host: localhost:5000
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
test

response：
HTTP/1.1 200 OK
Date: Wed, 07 Mar 2018 14:43:59 GMT
Content-Type: text/plain; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked

content:  contentFromBody: test 
```
可以看到contentFromBody中我们得到了request body的内容。
注意参数没有[FromBody]这个属性
如果加了这个属性，那么如果request body内容匹配request的media type那么Request.body的position会被置于结尾的位置。
举个例子
```c
        public string GetJsonString2([FromBody]string content)
        {

            var reader = new StreamReader(Request.Body);
            var contentFromBody = reader.ReadToEnd();
            
            Request.Body.Position = 0;
            
            var reader2 = new StreamReader(Request.Body);
            var contentFromBody2 = reader2.ReadToEnd();

            return "content: " + content 
                   + " contentFromBody: " + contentFromBody
                   + " contentFromBody2: " + contentFromBody2;
        }
```
测试结果

```
request:
POST http://localhost:5000/api/values/GetJsonString2 HTTP/1.1
Accept-Encoding: gzip,deflate
Content-Type: application/json
Content-Length: 4
Host: localhost:5000
Connection: Keep-Alive
User-Agent: Apache-HttpClient/4.1.1 (java 1.5)
test

response:
HTTP/1.1 200 OK
Date: Wed, 07 Mar 2018 14:53:03 GMT
Content-Type: text/plain; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked

content:  contentFromBody:  contentFromBody2: test
```

