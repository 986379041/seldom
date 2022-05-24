# HTTP接口测试

## 前言

HTTP接口测试很简单，不管工具、框架、还是平台，只要很的好的几个点就是好工具。

1. 测试数据问题：比如删除接口，重复执行还能保持结果一致，必定要做数据初始化。
2. 接口依赖问题：B接口依赖A的返回值，C接口依赖B接口的返回值。
3. 加密问题：不同的接口加密规则不一样。有些用到时间戳、md5、base64、AES，如何提供种能力。
4. 断言问题：有些接口返回的结构体很复杂，如何灵活的做到断言。

对于以上问题，工具和平台要么不支持，要么很麻烦，然而框架是最灵活的。 

> unittest/pytest + requests/https 直接上手写代码就好了，既简单又灵活。

那么同样是写代码，A框架需要10行，B框架只需要5行，然而又不失灵活性，那我当然是选择更少的了，毕竟，人生苦短嘛。

seldom适合个人接口自动化项目，它有以下优势。

* 可以写更少的代码
* 自动生成HTML/XML测试报告
* 支持参数化，减少重复的代码
* 支持生成随机数据
* 支持har文件转case
* 支持数据库操作

这些是seldom支持的功能，我们只需要集成HTTP接口库，并提供强大的断言即可。`seldom 2.0` 加入了HTTP接口自动化测试支持。

Seldom 兼容 [Requests](https://docs.python-requests.org/en/master/) API 如下:

|  seldom   | requests  |
|  ----  | ----  |
| self.get()  | requests.get() |
| self.post()  | requests.post() |
| self.put()  | requests.put() |
| self.delete()  | requests.delete() |

### Seldom VS Request+unittest

先来看看unittest + requests是如何来做接口自动化的：

```python
import unittest
import requests


class TestAPI(unittest.TestCase):

    def test_get_method(self):
        payload = {'key1': 'value1', 'key2': 'value2'}
        r = requests.get("http://httpbin.org/get", params=payload)
        self.assertEqual(r.status_code, 200)


if __name__ == '__main__':
    unittest.main()
```

这其实已经非常简洁了。同样的用例，用seldom实现。

```python
# test_req.py
import seldom


class TestAPI(seldom.TestCase):

    def test_get_method(self):
        payload = {'key1': 'value1', 'key2': 'value2'}
        self.get("http://httpbin.org/get", params=payload)
        self.assertStatusCode(200)


if __name__ == '__main__':
    seldom.main()
```

主要简化点在，接口的返回数据的处理。当然，seldom真正的优势在断言、日志和报告。

## har to case

对于不熟悉 Requests 库的人来说，通过Seldom来写接口测试用例还是会有一点难度。于是，seldom提供了`har` 文件转 `case` 的命令。

首先，打开fiddler 工具进行抓包，选中某一个请求。

然后，选择菜单栏：`file` -> `Export Sessions` -> `Selected Sessions...`

![](/image/fiddler.png)

选择导出的文件格式。

![](/image/fiddler2.png)

点击`next` 保存为`demo.har` 文件。

最后，通过`seldom -h2c` 转为`demo.py` 脚本文件。

```shell
> seldom -h2c .\demo.har
.\demo.py
2021-06-14 18:05:50 [INFO] Start to generate testcase.
2021-06-14 18:05:50 [INFO] created file: D:\.\demo.py
```

`demo.py` 文件。

```python
import seldom


class TestRequest(seldom.TestCase):

    def start(self):
        self.url = "http://httpbin.org/post"

    def test_case(self):
        headers = {"User-Agent": "python-requests/2.25.0", "Accept-Encoding": "gzip, deflate", "Accept": "application/json", "Connection": "keep-alive", "Host": "httpbin.org", "Content-Length": "36", "Origin": "http://httpbin.org", "Content-Type": "application/json", "Cookie": "lang=zh"}
        cookies = {"lang": "zh"}
        self.post(self.url, json={"key1": "value1", "key2": "value2"}, headers=headers, cookies=cookies)
        self.assertStatusCode(200)


if __name__ == '__main__':
    seldom.main()

```

### 运行测试

打开debug模式`seldom.run(debug=True)` 运行上面的用例。

```shell
> python .\test_req.py

              __    __
   ________  / /___/ /___  ____ ____
  / ___/ _ \/ / __  / __ \/ __ ` ___/
 (__  )  __/ / /_/ / /_/ / / / / / /
/____/\___/_/\__,_/\____/_/ /_/ /_/  v2.x.x
-----------------------------------------
                             @itest.info

.\test_req.py
test_case (test_req.TestRequest) ... 
2022-04-30 18:20:47 log.py | INFO | -------------- Request -----------------[🚀]
2022-04-30 18:20:47 log.py | INFO | [method]: POST      [url]: http://httpbin.org/post

2022-04-30 18:20:47 log.py | DEBUG | [headers]:
 {'User-Agent': 'python-requests/2.25.0', 'Accept-Encoding': 'gzip, deflate', 'Accept': 'application/json', 'Connection': 'keep-alive', 'Host': 'httpbin.org', 'Content-Length': '36', 'Origin': 'http://httpbin.org', 'Content-Type': 'application/json', 'Cookie': 'lang=zh'}

2022-04-30 18:20:47 log.py | DEBUG | [cookies]:
 {'lang': 'zh'}

2022-04-30 18:20:47 log.py | DEBUG | [json]:
 {'key1': 'value1', 'key2': 'value2'}

2022-04-30 18:20:47 log.py | INFO | -------------- Response ----------------[🛬️]
2022-04-30 18:20:47 log.py | INFO | successful with status 200

2022-04-30 18:20:47 log.py | DEBUG | [type]: json      [time]: 0.582786

2022-04-30 18:20:47 log.py | DEBUG | [response]:
 {'args': {}, 'data': '{"key1": "value1", "key2": "value2"}', 'files': {}, 'form': {}, 'headers': {'Accept': 'application/json', 'Accept-Encoding': 'gzip, deflate', 'Content-Length': '36', 'Content-Type': 'application/json', 'Cookie': 'lang=zh', 'Host': 'httpbin.org', 'Origin': 'http://httpbin.org', 'User-Agent': 'python-requests/2.25.0', 'X-Amzn-Trace-Id': 'Root=1-626d0d7e-69a616b20139cd6869cc5e90'}, 'json': {'key1': 'value1', 'key2': 'value2'}, 'origin': '173.248.248.88', 'url': 'http://httpbin.org/post'}

ok

----------------------------------------------------------------------
Ran 1 test in 0.594s

OK
2022-04-30 18:20:47 log.py | SUCCESS | A run the test in debug mode without generating HTML report!
```

通过日志/报告都可以清楚的看到。

* 请求的方法
* 请求url
* 响应的类型
* 响应的数据

## 更强大的断言

断言接口返回的数据是我们在做接口自动化很重要的工作。

### assertJSON

`assertJSON()` 断言接口返回的某部分数据。

* 请求参数

```json
{
  "name": "tom",
  "hobby": [
    "basketball",
    "swim"
  ]
}
```

* 返回结果

```json
{
  "args": {
    "hobby": [
      "basketball",
      "swim"
    ],
    "name": "tom"
  },
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Host": "httpbin.org",
    "User-Agent": "python-requests/2.25.0",
    "X-Amzn-Trace-Id": "Root=1-62851614-1ca9fdb276238c60406c118f"
  },
  "origin": "113.87.15.99",
  "url": "http://httpbin.org/get?name=tom&hobby=basketball&hobby=swim"
}
```

我的目标是断言`name` 和 `hobby` 部分的内容。

```python
import seldom


class TestAPI(seldom.TestCase):

    def test_assert_json(self):
        # 接口参数
        payload = {"name": "tom", "hobby": ["basketball", "swim"]}
        # 接口调用
        self.get("http://httpbin.org/get", params=payload)

        # 断言数据
        assert_data = {
            "hobby": ["swim", "basketball"],
            "name": "tom"
        }
        self.assertJSON(assert_data, self.response["args"])
```


### assertPath

`assertPath` 是基于 `jmespath` 实现的断言，功能非常强大。

jmespath: https://jmespath.org/specification.html

接口返回数据如下：

```json
{
  "args": {
    "hobby": 
      ["basketball", "swim"], 
    "name": "tom"
  }
}
```

seldom中可以通过path进行断言：

```python
import seldom


class TestAPI(seldom.TestCase):

    def test_assert_path(self):
        payload = {'name': 'tom', 'hobby': ['basketball', 'swim']}
        self.get("http://httpbin.org/get", params=payload)
        self.assertPath("args.name", "tom")
        self.assertPath("args.hobby[0]", "basketball")
        self.assertInPath("args.hobby[0]", "ball")

```

* `args.hobby[0]` 提取接口返回的数据。
* `assertPath()` 判断提取的数据是否等于`basketball`; 
* `assertInPath()` 判断提取的数据是否包含`ball`。

### assertSchema

有时并不关心数据本身是什么，而是需要断言数据的结构和类型。 `assertSchema` 是基于 `jsonschema` 实现的断言方法。

jsonschema: https://json-schema.org/learn/

```python
import seldom
from seldom.utils import genson


class TestAPI(seldom.TestCase):

    def test_assert_schema(self):
        # 接口参数
        payload = {"hobby": ["basketball", "swim"], "name": "tom", "age": "18"}
        # 调用接口
        self.get("/get", params=payload)

        # 生成数据结构和类型
        schema = genson(self.response["args"])
        print("json Schema: \n", schema)

        # 断言数据结构和类型
        assert_data = {
            "$schema": "http://json-schema.org/schema#",
            "type": "object",
            "properties": {
                "age": {
                    "type": "string"
                },
                "hobby": {
                    "type": "array", "items": {"type": "string"}
                },
                "name": {
                    "type": "string"
                }
            },
            "required":
                ["age", "hobby", "name"]
        }
        self.assertSchema(assert_data, self.response["args"])

```

* `genson`: 可以生成`jsonschema`数据结构和类型（`seldom 2.9` 新增）。


### 接口数据依赖

在场景测试中，我们需要利用上一个接口的数据，调用下一个接口。

* 简单的接口依赖

```python
import seldom

class TestRespData(seldom.TestCase):

    def test_data_dependency(self):
        """
        Test for interface data dependencies
        """
        headers = {"X-Account-Fullname": "bugmaster"}
        self.get("/get", headers=headers)
        self.assertStatusCode(200)

        username = self.response["headers"]["X-Account-Fullname"]
        self.post("/post", data={'username': username})
        self.assertStatusCode(200)
```

seldom提供了`self.response`用于记录上个接口返回的结果，直接拿来用即可。

* 封装接口依赖

1. 创建公共模块

```python
# common.py
from seldom.request import check_response 
from seldom.request import HttpRequest


class Common(HttpRequest):
    
    @check_response("获取登录用户名", 200, "headers.Account", {"headers.Host": "httpbin.org"}, debug=True)
    def get_login_user(self):
        """
        调用接口获得用户名
        """
        headers = {"Account": "bugmaster"}
        r = self.get("http://httpbin.org/get", headers=headers)
        return r


if __name__ == '__main__':
    c = Common()
    c.get_login_user()
```

* 运行日志

```shell
2022-04-24 22:21:38 [DEBUG] Execute get_login_user - args: (<__main__.Common object at 0x000001A6B028F970>,)
2022-04-24 22:21:38 [DEBUG] Execute get_login_user - kwargs: {}
2022-04-24 22:21:38.831 | DEBUG    | seldom.logging.log:debug:34 - Execute get_login_user - args: (<__main__.Common object at 0x000001A6B028F970>,)
2022-04-24 22:21:38.832 | DEBUG    | seldom.logging.log:debug:34 - Execute get_login_user - kwargs: {}
2022-04-24 22:21:39 [DEBUG] Execute get_login_user - response:
 {'args': {}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Account': 'bugmaster', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.25.0', 'X-Amzn-Trace-Id': 'Root=1-62655cf3-18c082b413a51b840fa9a449'}, 'origin': '173.248.248.88', 'url': 'http://httpbin.org/get'}
2022-04-24 22:21:39 [INFO] Execute get_login_user - 用户登录 success!
2022-04-24 22:21:39.402 | DEBUG    | seldom.logging.log:debug:34 - Execute get_login_user - response:
 {'args': {}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Account': 'bugmaster', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.25.0', 'X-Amzn-Trace-Id': 'Root=1-62655cf3-18c082b413a51b840fa9a449'}, 'origin': '173.248.248.88', 'url': 'http://httpbin.org/get'}
2022-04-24 22:21:39.402 | INFO     | seldom.logging.log:info:45 - Execute get_login_user - 用户登录 success!
```

* check_response

`@check_response` 专门用于处理封装的方法。__参数说明：__

* `describe` : 封装方法描述。
* `status_code`: 判断接口返回的 HTTP 状态码，默认`200`。
* `ret`: 提取接口返回的字段，参考`jmespath` 提取规则。
* `check`: 检查接口返回的字段。参考`jmespath` 提取规则。
* `debug`: 开启`debug`，打印更多信息。


2. 引用公共模块

```python
import seldom
from common import Common


class TestRequest(seldom.TestCase):

    def start(self):
        self.c = Common()

    def test_case(self):
        # 调用 get_login_user() 获取
        user = self.c.get_login_user()
        self.post("http://httpbin.org/post", data={'username': user})
        self.assertStatusCode(200)


if __name__ == '__main__':
    seldom.main(debug=True)

```


### 数据驱动

seldom本来就提供的有强大的数据驱动，拿来做接口测试非常方便。

__@data__

```python
import seldom
from seldom import data


class TestDDT(seldom.TestCase):

    @data([
        ("key1", 'value1'),
        ("key2", 'value2'),
        ("key3", 'value3')
    ])
    def test_data(self, key, value):
        """
        Data-Driver Tests
        """
        payload = {key: value}
        self.post("/post", data=payload)
        self.assertStatusCode(200)
        self.assertEqual(self.response["form"][key], value)

```

__@file_data__

创建`data.json`数据文件

```json
{
 "login":  [
    ["admin", "admin123"],
    ["guest", "guest123"]
 ]
}
```

通过`file_data`实现数据驱动。

```python
import seldom
from seldom import file_data


class TestDDT(seldom.TestCase):

    @file_data("data.json", key="login")
    def test_data(self, username, password):
        """
        Data-Driver Tests
        """
        payload = {username: password}
        self.post("http://httpbin.org/post", data=payload)
        self.assertStatusCode(200)
        self.assertEqual(self.response["form"][username], password)

```

更多数据文件(csv/excel/yaml)，[参考](https://github.com/SeldomQA/seldom/blob/master/docs/advanced.md)

### Session使用

在实际测试过程中，大部分接口需要登录，`Session` 是一种非常简单记录登录状态的方式。

```python
import seldom


class TestCase(seldom.TestCase):

    def start(self):
        self.s = self.Session()
        self.s.get('/cookies/set/sessioncookie/123456789')

    def test_get_cookie1(self):
        self.s.get('/cookies')

    def test_get_cookie2(self):
        self.s.get('/cookies')


if __name__ == '__main__':
    seldom.main(debug=True, base_url="https://httpbin.org")
```

用法非常简单，你只需要在每个接口之前调用一次`登录`， `self.s`对象就记录下了登录状态，通过`self.s` 再去调用其他接口就不需要登录。

### 随机生成测试数据

seldom提供随机生成测试数据方法，可以生成一些常用的数据。

```python
import seldom
from seldom import testdata


class TestAPI(seldom.TestCase):

    def test_data(self):
        phone = testdata.get_phone()
        payload = {'phone': phone}
        self.get("http://httpbin.org/get", params=payload)
        self.assertPath("args.phone", phone)

```

更过类型的测试数据，[参考](https://github.com/SeldomQA/seldom/blob/master/docs/advanced.md)


### 提取接口返回数据

当接口返回的数据比较复杂时，我们需要有更方便方式去提取数据，seldom提供 `jmespath`、`jsonpath` 来简化数据提取。

* 接口返回数据

```json
{
  "args": {
    "hobby": [
      "basketball",
      "swim"
    ],
    "name": "tom"
  },
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Host": "httpbin.org",
    "User-Agent": "python-requests/2.25.0",
    "X-Amzn-Trace-Id": "Root=1-62851614-1ca9fdb276238c60406c118f"
  },
  "origin": "113.87.15.99",
  "url": "http://httpbin.org/get?name=tom&hobby=basketball&hobby=swim"
}
```

* 常规提取

```python
import seldom


class TestAPI(seldom.TestCase):

    def test_extract_responses(self):
        """
        提取 response 数据
        """
        payload = {"hobby": ["basketball", "swim"], "name": "tom", "age": "18"}
        self.get("http://httpbin.org/get", params=payload)

        # response
        response1 = self.response["args"]["name"]
        response2 = self.response["args"]["hobby"]
        response3 = self.response["args"]["hobby"][0]
        print(f"response1 --> {response1}")
        print(f"response2 --> {response2}")
        print(f"response3 --> {response3}")

        # jmespath
        jmespath1 = self.jmespath("args.name")
        jmespath2 = self.jmespath("args.hobby")
        jmespath3 = self.jmespath("args.hobby[0]")
        jmespath4 = self.jmespath("hobby[0]", response=self.response["args"])
        print(f"\njmespath1 --> {jmespath1}")
        print(f"jmespath2 --> {jmespath2}")
        print(f"jmespath3 --> {jmespath3}")
        print(f"jmespath4 --> {jmespath4}")

        # jsonpath
        jsonpath1 = self.jsonpath("$..name")
        jsonpath2 = self.jsonpath("$..hobby")
        jsonpath3 = self.jsonpath("$..hobby[0]")
        jsonpath4 = self.jsonpath("$..hobby[0]", index=0)
        jsonpath5 = self.jsonpath("$..hobby[0]", index=0, response=self.response["args"])
        print(f"\njsonpath1 --> {jsonpath1}")
        print(f"jsonpath2 --> {jsonpath2}")
        print(f"jsonpath3 --> {jsonpath3}")
        print(f"jsonpath4 --> {jsonpath4}")
        print(f"jsonpath5 --> {jsonpath5}")
...
```

说明：
* `response`: 保存接口返回的数据，可以直接以，字典列表的方式提取。
* `jmespath()`: 根据 JMESPath 语法规则，默认提取接口返回的数据，也可指定`resposne`数据提取。
* `jsonpath()`: 根据 JsonPath 语法规则，默认提取接口返回的数据, `index`指定下标，也可指定`resposne`数据提取。

运行结果：

```shell
2022-05-19 00:57:08 log.py | DEBUG | [response]:
 {'args': {'age': '18', 'hobby': ['basketball', 'swim'], 'name': 'tom'}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.25.0', 'X-Amzn-Trace-Id': 'Root=1-62852563-2fe77d4b1ce544696af60f10'}, 'origin': '113.87.15.99', 'url': 'http://httpbin.org/get?hobby=basketball&hobby=swim&name=tom&age=18'}

response1 --> tom
response2 --> ['basketball', 'swim']
response3 --> basketball

jmespath1 --> tom
jmespath2 --> ['basketball', 'swim']
jmespath3 --> basketball
jmespath4 --> basketball

jsonpath1 --> ['tom']
jsonpath2 --> [['basketball', 'swim']]
jsonpath3 --> ['basketball']
jsonpath4 --> basketball
jsonpath5 --> basketball
```

* ~~jresponse()~~ 用法

在接口测试中通过`jresponse()` 方法可以直接提取数据。

> 注：该方法从命名、参数都不规范，不推荐使用，后续版本将会移除。

```python
import seldom


class TestAPI(seldom.TestCase):

    def test_jresponse(self):
        payload = {"hobby": ["basketball", "swim"], "name": "tom", "age": "18"}
        self.get("http://httpbin.org/get", params=payload)
        self.jresponse("$..hobby[0]")  # 提取hobby (默认 jsonpath)
        self.jresponse("$..age")   # 提取age (默认 jsonpath)
        self.jresponse("hobby[0]", j="jmes")  # 提取hobby (jmespath)
        self.jresponse("age", j="jmes")  # 提取age (jmespath)


if __name__ == '__main__':
    seldom.main(base_url="http://httpbin.org", debug=True)
```

运行结果

```shell
...
2022-04-10 21:05:17.683 | DEBUG    | seldom.logging.log:debug:34 - [response]:
 {'args': {'age': '18', 'hobby': ['basketball', 'swim'], 'name': 'tom'}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.25.0', 'X-Amzn-Trace-Id': 'Root=1-6252d60c-551433d744b6869e5d1944d7'}, 'origin': '113.87.12.14', 'url': 'http://httpbin.org/get?hobby=basketball&hobby=swim&name=tom&age=18'}

2022-04-10 21:05:17.686 | DEBUG    | seldom.logging.log:debug:34 - [jresponse]:
 ['basketball']
2022-04-10 21:05:17.689 | DEBUG    | seldom.logging.log:debug:34 - [jresponse]:
 ['18']
```

### genson

通过 `assertSchema()` 断言时需要写JSON Schema，但是这个写起来需要学习成本，seldom集成了[GenSON](https://github.com/wolverdude/GenSON) ,可以帮你自动生成。

* 例子

```python
import seldom
from seldom.utils import genson


class TestAPI(seldom.TestCase):

    def test_assert_schema(self):
        payload = {"hobby": ["basketball", "swim"], "name": "tom", "age": "18"}
        self.get("/get", params=payload)
        print("response \n", self.response)
        
        schema = genson(self.response)
        print("json Schema \n", schema)
        
        self.assertSchema(schema)
```

* 运行日志

```shell
...
response
 {'args': {'age': '18', 'hobby': ['basketball', 'swim'], 'name': 'tom'}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Host': 'httpbin.org', 'User-Agent': 'python-requests/2.25.0', 'X-Amzn-Trace-Id': 'Root=1-626574d0-4c04bb7e76a53e8042c9d856'}, 'origin': '173.248.248.88', 'url': 'http://httpbin.org/get?hobby=basketball&hobby=swim&name=tom&age=18'}

json Schema
 {'$schema': 'http://json-schema.org/schema#', 'type': 'object', 'properties': {'args': {'type': 'object', 'properties': {'age': {'type': 'string'}, 'hobby': {'type': 'array', 'items': {'type': 'string'}}, 'name': {'type': 'string'}}, 'required': ['age', 'hobby', 'name']}, 'headers': {'type': 'object', 'properties': {'Accept': {'type': 'string'}, 'Accept-Encoding': {'type': 'string'}, 'Host': {'type': 'string'}, 'User-Agent': {'type': 'string'}, 'X-Amzn-Trace-Id': {'type': 'string'}}, 'required': ['Accept', 'Accept-Encoding', 'Host', 'User-Agent', 'X-Amzn-Trace-Id']}, 'origin': {'type': 'string'}, 'url': {'type': 'string'}}, 'required': ['args', 'headers', 'origin', 'url']}
```

