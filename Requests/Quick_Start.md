# Requests

Request是使用[Apache2 Licensed](http://docs.python-requests.org/zh_CN/latest/user/intro.html#apache2)许可证的HTTP库。用Python编写，真正的为人类着想。

Python标准库中的`urllib2`模块提供了你所需要的大多数HTTP功能，但是它的API太渣了。它是为另一个时代、另一个互联网所创建的。它需要巨量的工作，甚至包括各种方法覆盖，来完成最简单的任务。

在Python的世界里，事情不应该这么麻烦。

```python
>>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
u'{"type":"User"...'
>>> r.json()
{u'private_gists': 419, u'total_private_repos': 77, ...}
```

所拥有的实例方法包括：

```python
>>> r.
r.apparent_encoding      r.iter_lines
r.close                  r.json
r.connection             r.links
r.content                r.ok
r.cookies                r.raise_for_status
r.elapsed                r.raw
r.encoding               r.reason
r.headers                r.request
r.history                r.status_code
r.is_permanent_redirect  r.text
r.is_redirect            r.url
r.iter_content   
```

Request使用的是`urllib3`，因此继承了它所有的特性。Requests支持HTTP连接保持和连接池，支持使用cookie保持会话，支持文件上传，支持自动确定响应内容的编码，支持国际化的URL和POST数据自动编码。现代、国际化、人性化。

Requests功能特性包括：
> * 国际化域名和URLs
> * Keep-Alive & 连接池
> * 持久的Cookie会话
> * 类浏览器式的SSL加密认证
> * 基本/摘要式的身份认证
> * 优雅的键/值Cookies
> * 自动解压
> * Unicode编码的响应体
> * 多段文件上传
> * 连接超时
> * 支持.netrc
> * 适用于Python 2.6-3.4
> * 线程安全


> **注意，HTTP的`GET`方法是明文提交，且长度有限；`POST`方法是隐藏的方式提交(不一定是加密)，且只能提交文字解码或字符串。**

### 1. 安装

使用`pip`安装Requests非常简单

```shell
$ pip install requests
```

或者使用`easy_install`安装

```shell
$ easy_install requests
```

Requests一直在Github上被积极的开发着，你可以在此获得源代码。比如克隆公共版本库：

```shell
git clone git://github.com/kennethreitz/requests.git
```

下载源码

```shell
$ curl -OL https://github.com/kennethreitz/requests/tarball/master
```

或者下载zipball

```shell
$ curl -OL https://github.com/kennethreitz/requests/zipball/master
```

一旦你获得了复本，你就可以轻松的将它嵌入到你的Python包里或者安装到你的`site-packages`：

```shell
$ python setup.py install
```

### 2. 发送请求

> 使用Requests发送网络请求非常简单

一开始你需要导入Requests模块，例如：

```python
>>> import requests
```

然后尝试获取某个网页。例如获取Github的公共时间线

```python
>>> r = requests.get('https://github.com/timeline.json')
```

现在我们有一个名为`r`的`Response`对象。可以从这个对象中获取我们想要的信息。

Request简便的API意味着所有HTTP请求类型都是显而易见的。例如，你可以这样发送一个`HTTP POST`请求：

```python
>>> r = requests.post("http://httpbin.org/post")
```

而对于其他的HTTP请求类型`PUT`、`DELETE`、`HEAD`以及`OPTIONS`也同样适用：

```python
>>> r = requests.put("http://httpbin.org/put")
>>> r = requests.delete("http://httpbin.org/delete")
>>> r = requests.head("http://httpbin.org/get")
>>> r = requests.options("http://httpbin.org/get")
```

### 3. 为URL传递参数

你也许经常想为URL的查询字符串(query string)传递某种数据。如果你是手工构建URL，那么数值会以键/值对的形式置于URL中，跟在一个问号的后面。例如`httpbin.org/get?key=val`。`Requests`允许你使用`params`关键字参数，以一个字典来提供这些参数。举例来说，如果你想传递`key1=value1`和`key2=value2`到`httpbin.org/get`，那么你可以使用如下代码：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get("http://httpbin.org/get", params=payload)
```

通过打印输出该URL，你能看到URL已被正确编码：

```python
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
```

> 注意，字典里面值为`None`的键都不会被添加到URL的查询字符串里。如下：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2', 'key3':None}
>>> r = requests.get("http://httpbin.org/get", params=payload)
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
```

### 4. 响应内容

我们能读取服务器响应的内容。再次以Github时间线为例：

```python
>>> import requests
>>> r = requests.get('https://github.com/timeline.json')
>>> r.text
u'[{"repository":{"open_issues":0,"url":"https://github.com/...
```

`Requests`会自动解码来自服务器的内容。大多数unicode字符集都能被无缝的解码。

请求发出后，`Requests`会基于HTTP头部对响应的编码做出有根据的推测。当你访问`r.text`之时，`Requests`会使用其推测的文本编码。你可以找出`Requests`使用了什么编码，并且能够使用`r.eencoding`属性来改变它：

```python
>>> r.encoding
'utf-8'
>>> r.encoding = 'ISO-8859-1'
>>> r.encoding
'ISO-8859-1'
```

如果你改变了编码，每当你访问`r.text`，Requests都将会使用`r.encoding`的新值。你可能希望在使用特殊逻辑计算出文本的编码的情况下来修改编码。比如HTTP和XML自身都可以指定编码。这样的话，你应该使用`r.content`来找到编码，然后设置`r.encoding`为相应的编码。这样就能使用正确的编码解析`r.text`了。

在你需要的情况下，Requests也可以使用定制的编码。如果你创建了自己的编码，并使用`codecs`模块进行注册额，你就可以轻松的使用这个解码器名称作为`r.encoding`的值，然后由Requests来为你处理编码。

### 5. 二进制响应内容

你也能以字节的方式访问请求响应体，对于非文本请求：

```python
>>> r.content
b'[{"repository":{"open_issues":0,"url":"https://github.com/...
```

Requests会自动为你解码`gzip`和`deflate`传输编码的响应数据。
例如，以请求返回的二进制数据创建一张图片，你可以使用如下代码：

```python
>>> from PIL import Image
>>> from StringIO import StringIO
>>> i = Image.open(StringIO(r.content))
```

### 6. JSON响应内容

Requests中也有一个内置的`JSON`解码器，助你处理`JSON`数据：

```python
>>> import requests
>>> r = requests.get('https://github.com/timeline.json')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
```

如果`JSON`解码失败，`r.json`就会抛出一个异常。例如，响应内容是`401`(Unauthorized)，尝试访问`r.json`就会抛出`ValueError: No JSON object could be decoded`异常，不过你应该事先捕获此异常。

### 7. 原始响应内容

在罕见的情况下你可能想获取来自服务器的原始套接字响应，那么你可以访问`r.raw()`；如果你确实想这么干，那请你确保在初始化请求中设置了`stream=True`。如下：

```python
>>> r = requests.get('https://github.com/timeline.json', stream=True)
>>> r.raw
<requests.packages.urllib3.response.HTTPResponse object at 0x101194810>
>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03
```

但一般情况下，你应该以下面的模式将文本流保存到文件：

```python
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size):
        fd.write(chunk)
```

使用`Response.iter_content`将会处理大量你直接使用`Response.raw`。当流下载时，上面是有限推荐的获取内容方式。

### 8. 定制请求头

如果你想为请求添加HTTP头部，只要简单的传递一个`dict`给`headers`参数就可以了。例如：

```python
>>> import json
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> headers = {'content-type': 'application/json'}

>>> r = requests.post(url, data=json.dumps(payload), headers=headers)
```

### 9. 更加复杂的POST请求

通常，你想要发送一些编码为表单形式的数据，非常像一个HTML表单。要实现这个，只需要简单的传递一个字典给`data`参数。你的数据字典在发出请求时会自动编码为表单形式：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print r.text
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

很多时候你想要发送的数据并非编码为表单形式；如果你传递一个`string`而不是一个`dict`，那么数据会被直接发布出去。例如：

```python
>>> import json
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```

### 10. POST一个多部分编码(Multipart-Encoded)的文件

`Requests`使得上传多部分编码文件变得很简单：

```python
>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('report.xls', 'rb')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```

你可以设置文件名、文件类型和请求头：

```python
>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```

如果你想，你也可以发送作为文件来接受字符串：

```python
>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "some,data,to,send\\nanother,row,to,send\\n"
  },
  ...
}
```

> 注意，如果你发送一个非常大的文件来作为`multipart/form-data`请求，你可能希望使用流请求来实现。但默认情况下`requests`不支持流，有个第三方包支持`requests-toolbelt`。你可以阅读[toolbelt文档](https://toolbelt.rtfd.org/)来了解使用方法。

### 11. 响应状态码

我们可以检测响应状态码：

```python
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
```

为了方便引用，`Requests`还附带了一个内置的状态码查询对象：

```python
>>> r.status_code == requests.codes.ok
True
```

如果发送了一个失败请求(非200响应)，我们可以通过`Response.raise_for_status()`来抛出异常：

```python
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404

>>> bad_r.raise_for_status()
Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
```

但是，由于我们的例子中，`r`的`status_code`是200，当我们调用`raise_for_status()`时，得到的是：

```python
>>> r.raise_for_status()
None
```

一切都是那么的和谐！

### 12. 响应头

我们可以查看以一个Python字典形式展示的服务器响应头：

```python
>>> r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
```

这个字典比较特殊，它是仅为HTTP头部而生的。根据[RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)，HTTP头部是大小写不敏感的。因此我们可以使用任意大写形式来访问这些响应头字段：

```python
>>> r.headers['Content-Type']
'application/json'

>>> r.headers.get('content-type')
'application/json'
```

### 13. Cookies

如果某个响应中包含一些`Cookie`，你可以快速访问他们：

```python
>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'

>>> URL = 'http://www.baidu.com'
>>> r = requests.get(URL)
>>> for k, v in r.cookies.items():
...    print k, v
...
BAIDUID D7B8B7625F4F7BFA609568DC54E0868B:FG=1
BIDUPSID D7B8B7625F4F7BFA609568DC54E0868B
H_PS_PSSID 20408_1455_20318_20369_20388_19689_20417_20456_19861_15014_12334
PSTM 1466786236
BDSVRTM 0
BD_HOME 0
```

要想发送你的cookies到服务器，可以使用`cookies`参数：

```python
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

### 14. 重定向与请求历史

默认情况下，除了`HEAD`外，`Requests`会自动处理所有重定向。可以使用响应对象的`history`方法来追踪重定向。`Response.history`是一个`:class:Response<requests.Response>`对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从最老到最近的请求进行排序。例如，Github将所有的HTTP请求重定向到HTTPS：

```python
>>> r = requests.get('http://github.com')
>>> r.url
'https://github.com/'
>>> r.status_code
200
>>> r.history
[<Response [301]>]
```

如果你使用的是`GET`，`OPTIONS`，`POST`，`PUT`，`PATCH`或者`DELETE`，那么你可以通过`allow_redirects`参数禁用重定向处理：

```python
>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]
```

如果你使用的是`HEAD`，你也可以启用重定向：

```python
>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[<Response [301]>]
```

### 15. 超时

你可以告诉`requests`在经过以`timeout`参数设定的秒数时间之后停止等待响应：

```python
>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```

> 注意，`timeout`仅对间接过程有效，与响应体的下载无关。`timeout`并不是整个下载响应的时间限制，而是如果服务器在‘`timeout`’秒内没有应答，将会引发一个异常(更精确的说是在'`timeout`'秒内没有从基础套接字上接收到任何字节的数据时)。

### 16. 错误与异常

> * 遇到网络问题(如DNS查询失败、拒绝连接2等)时，`Requests`会抛出一个`ConnectionError`异常。
> * 遇到罕见的无效的HTTP响应时，`Requests`则会抛出一个`HTTPError`异常。
> * 若请求超时，则抛出一个`Timeout`异常。
> * 若请求超过了设定的最大重定向次数，则会抛出一个`TooManyRedirects`异常。
> * 所有`Requests`显示抛出的异常都继承自`requests.exceptions.RequestException`。
