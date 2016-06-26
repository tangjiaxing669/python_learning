# Requests

> 本篇文档涵盖了Requests一些更加高级的特性。

> 参考 http://docs.python-requests.org/zh_CN/latest/user/advanced.html

### 会话对象

会话对象让你能够跨请求保持某些参数。它也会在同一个`Session`实例发出的所有请求之间保持`cookies`。

会话对象具有`Requests API`所有的方法。我们来跨请求保持一些`cookies`：

```python
s = requests.Session()

s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
r = s.get("http://httpbin.org/cookies")

print(r.text)
# '{"cookies": {"sessioncookie": "123456789"}}'
```

会话也可用来为请求方法提供缺省数据。这是通过为会话对象的属性提供数据来实现的：

```python
s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})
```

> 任何你传递给请求方法的字典都会与已设置的会话层数据合并；注意，方法层的参数会覆盖会话层的参数。例如：

```python
>>> s = requests.Session()
>>> s.headers.update({'x-test': 'tangjiaxing'})
>>> 
# both 'x-test' and 'x-test2' are sent
>>> s.get('http://httpbin.org/headers', headers={'x-test2': 'jason'})
>>> s.headers
{'Accept': '*/*', 'Connection': 'keep-alive', 'Accept-Encoding': 'gzip, deflate', 'x-texst': 'tangjiaxing', 'User-Agent': 'python-requests/2.9.1'}
```

> 注意，有时你想省略字典参数中一些会话层的键(从字典参数移出一个值)；要做到这一点，你只需要简单的在方法层参数中将那个键的值设置为`None`，那个键就会被自动省略掉。

> 包含在一个会话中所有的数据你都可以直接使用。学习更多细节请阅读[会话API文档](http://docs.python-requests.org/zh_CN/latest/api.html#sessionapi)。

### 请求与响应对象

任何时候调用`requests.*()`你都在做两件主要的事情。其一，你在构建一个`requests`对象，该对象将被发送到某个请求服务器或查询一些资源。其二，一旦`requests`得到一个从服务器返回的相应就会产生一个`Response`对象。该响应对象包含服务器返回的所有信息，也包含你原来创建的`Requests`对象。如下是一个简单的请求，从`Wikipedia`的服务器得到一些非常重要的信息：

```python
>>> r = requests.get('http://en.wikipedia.org/wiki/Monty_Python')
```

如果想访问服务器返回给我们的响应头部信息，可以这样做：

```python
>>> r.headers
{'Content-Length': '70577', 'Content-language': 'en', 'X-Powered-By': 'HHVM/3.12.1', 'Last-Modified': 'Sat, 25 Jun 2016 06:18:53 GMT', 'X-Client-IP': '124.109.3.34', 'Date': 'Sun, 26 Jun 2016 10:24:21 GMT', 'Age': '29509', 'X-Varnish': '488492262, 2295470517, 2586676894 2521474001, 1098364027 1031018072', 'X-Cache': 'cp1053 miss, cp2023 miss, cp4016 hit/4, cp4016 hit/9', 'Accept-Ranges': 'bytes','Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload', 'Server': 'mw1251.eqiad.wmnet', 'Connection': 'keep-alive', 'P3P': 'CP="This is not a P3P policy! See https://en.wikipedia.org/wiki/Special:CentralAutoLogin/P3P for more info."', 'Via': '1.1 varnish, 1.1 varnish, 1.1 varnish, 1.1 varnish', 'X-Analytics': 'page_id=18942;ns=0;WMF-Last-Access=26-Jun-2016;https=1', 'X-Content-Type-Options': 'nosniff', 'Content-Encoding': 'gzip', 'Vary': 'Accept-Encoding,Cookie,Authorization', 'X-UA-Compatible': 'IE=Edge', 'Cache-Control': 'private, s-maxage=0, max-age=0, must-revalidate', 'Content-Type': 'text/html; charset=UTF-8', 'Backend-Timing': 'D=100664 t=1466907153138499'}
```

然而，如果想得到发送到服务器的请求头部信息，我们可以简单的访问该请求，然后是该请求的头部：

```python
>>> r.request.headers
{'Connection': 'keep-alive', 'Cookie': 'GeoIP=TH:::13.75:100.47:v4; WMF-Last-Access=26-Jun-2016', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'User-Agent': 'python-requests/2.9.1'}
```

### SSL证书验证

`Requests`可以为`HTTPS`请求验证`SSL`证书，就像web浏览器一样。要想检查某个主机的SSL证书，你可以使用`verify`参数：

```python
>>> requests.get('https://kennethreitz.com', verify=True)
requests.exceptions.SSLError: hostname 'kennethreitz.com' doesn't match either of '*.herokuapp.com', 'herokuapp.com'
...
SSLError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:765)
```

在该域名上我没有设置SSL，所以失败了。但Github设置了SSL：

```python
>>> requests.get('https://github.com', verify=True)
<Response [200]>
```

对于私有证书，你可以传递一个`CA_BUNDLE`文件的路径给`verify`。也可以设置`REQUEST_CA_BUNDLE`环境变量。
如果你将`verify`设置为`False`，Requests也能忽略对SSL证书的验证。

```python
>>> requests.get('https://kennethreitz.com', verify=False)
<Response [200]>
```

默认情况下，`verify`是设置为`True`的，选项`verify`仅应用于主机证书。你可以指定一个本地证书用作客户端证书，可以是单个文件(包含密钥和证书)或一个包含两个文件路径的元组：

```python
>>> requests.get('https://kennethreitz.com', cert=('/path/server.crt', '/path/key'))
<Response [200]>
```

如果你指定了一个错误的路径或一个无效的证书：

```python
>>> requests.get('https://kennethreitz.com', cert='/wrong_path/server.pem')
SSLError: [Errno 336265225] _ssl.c:347: error:140B0009:SSL routines:SSL_CTX_use_PrivateKey_file:PEM lib
```

### 响应体内容工作流

默认情况下，当你进行网络请求后，响应体会立即被下载。你可以通过`stream`参数覆盖这个行为，推迟下载响应体直到访问`Response.content`属性：

```python
tarball_url = 'https://github.com/kennethreitz/requests/tarball/master'
r = requests.get(tarball_url, stream=True)
```

此时仅有响应头被下载下来了，连接保持打开状态，因此允许我们根据条件来获取内容。

你可以进一步使用[Response.iter_content](http://docs.python-requests.org/zh_CN/latest/api.html#requests.Response.iter_content)和[Response.iter_lines](http://docs.python-requests.org/zh_CN/latest/api.html#requests.Response.iter_lines)方法来控制工作流，或者以[Response.raw]()从底层`urllib3`的`urllib3.HTTPResponse`和`urllib3.response.HTTPResponse`读取

### 保持活动状态（持久连接）

这归功于`urllib3`，同一会话内的持久连接是完全自动处理的，同一会话内你发出的任何请求都会自动复用恰当的连接；

> 注意，只有所有的响应体数据被读取完毕后，连接才会被释放为连接池；所以确保`stream`设置为`False`或者读取`Response`对象的`content`属性。

### 流式上传

Requests支持流式上传，这允许你发送大的数据流或文件而无需先把他们读入内存。要使用流式上传，仅需为你的请求体提供一个类文件对象即可：

```python
with open('massive-body') as f:
    requests.post('http://some.url/streamed', data=f)
```

### 块编码请求

对于出去和进来的请求，Requests也支持分块传输编码。要发送一个块编码的请求，仅需为你的请求体提供一个生成器(或任意没有具体长的迭代器)：

```python
def gen():
    yield 'hi'
    yield 'there'

requests.post('http://some.url/chunked', data=gen())
```

### POST Multiple Multipart-Encoded Files

你可以发送多个文件到一个`request`请求；例如，你想上传多个图片文件到HTML表单所对应的多个场景**images**中：

```html
<input type="file" name="images" multiple="true" required="true">
```

你只需要创建一个由元组(`form_field_name`,`file_info`)组成的列表传递给`requests`对象即可：

```python
>>> url = 'http://httpbin.org/post'
>>> multiple_files = [('images', ('foo.png', open('foo.png', 'rb'), 'image/png')),
                      ('images', ('bar.png', open('bar.png', 'rb'), 'image/png'))]
>>> r = requests.post(url, files=multiple_files)
>>> r.text
{
  ...
  'files': {'images': 'data:image/png;base64,iVBORw ....'}
  'Content-Type': 'multipart/form-data; boundary=3131623adb2043caaeb5538cc7aa0b3a',
  ...
}
```

### 事件挂钩

Requests有一个钩子系统，你可以用来操控部分请求过程，或信号事件处理。
可用的钩子：**response**；从一个请求中产生响应。

你可以通过传递一个`{hook_name:callback_function}`字典给`hooks`请求参数为每个请求分配一个钩子函数：

```python
>>> hooks=dict(response=print_url)
```

`callback_function`会接受一个数据块作为它的第一个参数。

```python
def print_url(r):
    print(r.url)
```

若执行你的回调函数期间发生错误，系统会给出一个警告。
若回调函数返回一个值，默认以该值替换传来的数据。若函数未返回任何东西，也没有什么其他的影响。我们来在运行期间打印一些请求方法的参数：

```python
>>> requests.get('http://httpbin.org', hooks=dict(response=print_url(r)))
http://httpbin.org
<Response [200]>
```

### 自定义身份验证

Requests允许你使用自己指定的身份验证机制。任何传递给请求方法的`auth`参数的可调用对象，在请求发出之前都有机会修改请求。自定义的身份验证机制是作为`requests.auth.AuthBase`的子类来实现的，也非常容易定义。Requests在`requests.auth`中提供了两种常见的身份验证方案：`HTTPBasicAuth`和`HTTPDigestAuth`。

假设我们有一个web服务，仅在`X-Pizza`头被设置为一个密码值的情况下才会有响应。虽然这不太可能，但这只是一个例子：

```python
from requests.auth import AuthBase

class PizzaAuth(AuthBase):
    """Attaches HTTP Pizza Authentication to the given Request object."""
    def __init__(self, username):
        # setup any auth-related data here
        self.username = username

    def __call__(self, r):
        # modify and return the request
        r.headers['X-Pizza'] = self.username
        return r
```

然后就可以使用我们的`PizzaAuth`来进行网络请求：

```python
>>> requests.get('http://pizzabin.org/admin', auth=PizzaAuth('kenneth'))
<Response [200]>
```

### 流式请求

使用`requests.Response.iter_lines()`你可以很方便的对流式API(例如[Twitter的流式API](https://dev.twittercom/docs/streaming-api))进行迭代。简单的设置`stream`为`True`便可以使用`iter_lines()`对响应进行迭代：

```python
import json
import requests

r = requests.get('http://httpbin.org/stream/20', stream=True)

for line in r.iter_lines():

    # filter out keep-alive new lines
    if line:
        print(json.loads(line))
```

### 代理

如果需要使用代理，你可以通过为任意请求方法提供`proxies`参数来配置单个请求：

```python
import requests

proxies = {
  "http": "http://10.10.1.10:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("http://example.org", proxies=proxies)
```

你也可以通过环境变量`HTTP_PROXY`和`HTTPS_PROXY`来配置代理

```python
$ export HTTP_PROXY="http://10.10.1.10:3128"
$ export HTTPS_PROXY="http://10.10.1.10:1080"
$ python
>>> import requests
>>> requests.get("http://example.org")
```

如果你的代理需要使用**HTTP Basic Auth**，可以使用`http://user:pass@host/`语法：

```python
proxies = {
    "http": "http://user:pass@10.10.1.10:3128/",
}
```

### 合规性

Requests符合所有相关的规范和RFC，这样不会为用户造成不必要的困难。但这种对规范的考虑 导致一些行为对于不熟悉相关规范的人来说看似有点奇怪。

### 编码方式

当你收到一个响应时，Requests会猜测响应的编码方式，用于在你调用`Response.text`方法时对响应进行解码。Requests首先在HTTP头部检测是否存在指定的编码方式，如果不存在，则会使用`charade`来尝试猜测编码方式。

只有当HTTP头部存在明确指定的字符集，并且`Content-Type`头部字段包含`text`值时，Requests才不会去猜测编码方式。

在这种情况下，[RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.7.1)指定默认字符集必须是`ISO-888599-1`。Requests遵从这一规范。如果你需要一种不同的编码方式，你可以手动设置`Response.encoding`属性，或使用原始的`Response.content`。

### HTTP方法

Requests提供了几乎所有HTTP方法的功能：`GET`、`OPTIONS`、`HEAD`、`POST`、`PUT`、`PATCH`和`DELETE`。以下为使用Requests中这些方法以及Github API提供了详细示例。

我们将从最常使用的`GET`方法开始。`HTTP GET`是一个幂等的方法，从给定的URL返回一个资源。因而，当你试图从一个web位置获取数据时，你应该使用这个方法。一个使用示例是尝试从Github上获取关于一个特定的commit的信息。假设我们想获取Requests的commit a050faf的信息。我们可以这样去做：

```python
>>> import requests
>>> r = requests.get('https://api.github.com/repos/kennethreitz/requests/git/commits/a050faf084662f3a352dd1a941f2c7c9f886d4ad')
```

我们应该确认Github是否正确响应；如果正确响应，我们想弄清楚响应内容是什么类型的。像这样去做：

```python
>>> if r.status_code == requests.codes.ok:
...    print(r.headers['content-type'])
...
application/json; charset=utf-8
```

可见，GitHub返回了JSON数据，非常好，这样就可以使用`r.json`方法把这个返回的数据解析成Python对象：

```python
>>> commit_data = r.json()
>>> print commit_data.keys()
[u'committer', u'author', u'url', u'tree', u'sha', u'parents', u'message']
>>> print commit_data[u'committer']
{u'date': u'2012-05-10T11:10:50-07:00', u'email': u'me@kennethreitz.com', u'name': u'Kenneth Reitz'}
>>> print commit_data[u'message']
makin' history
```

到目前为止，一切都非常简单；现在我们来研究下Github的API。我们可以去看看文档，但如果使用Requests来研究也许会更有意思一点。我们可以借助Requests的`OPTIONS`方法来看看我们刚使用过的URL支持哪些HTTP方法。

```python
>>> verbs = requests.options(r.url)
>>> verbs.status_code
204
```
WTF，这是怎么回事？原来Github与许多API提供方一样，实际上并未实现`OPTIONS`方法。这是一个恼人的疏忽，但是没关系，我们可是使用枯燥的文档。然而，如果Github正确的实现了`OPTIONS`方法，那么服务器应该在响应头中返回允许用户使用的HTTP方法，例如：

```python
>>> verbs = requests.options('http://a-good-website.com/api/cats')
>>> print verbs.headers['allow']
GET,HEAD,POST,OPTIONS
```

转而去看文档，看到对于提交信息，另一个允许的方法是`POST`方法，它会创建一个新的提交。由于我们正在使用Requests代码库，我们应尽可能避免对它发送笨拙的`POST`。作为代替，我们来玩玩Github的ISSUE特性。

```python
>>> r = requests.get('https://api.github.com/repos/kennethreitz/requests/issues/482')
>>> r.status_code
200
>>> issue = json.loads(r.text)
>>> print issue[u'title']
Feature any http verb in docs
>>> print issue[u'comments']
10
```

看到有10个评论，我们来看看最后一个评论；

```python
>>> r = requests.get(r.url + u'/comments')
>>> r.status_code
200
>>> comments = r.json()
>>> print comments[0].keys()
[u'body', u'url', u'created_at', u'updated_at', u'user', u'id']
>>> print comments[9][u'body']
u'github great'
```

看起来似乎是ige愚蠢之处。我们发表个评论来告诉这个评论者它自己的愚蠢。那么，这个评论者是谁呢？

```python
>>> print comments[2][u'user'][u'login']
u'MarkThink
```

根据Github API文档，其方法是`POST`到该话题。我们来试试看：

```python
>>> body = json.dumps({u"body": u"Sounds great! I'll get right on it!"})
>>> url = u"https://api.github.com/repos/kennethreitz/requests/issues/482/comments"
>>> r = requests.post(url=url, data=body)
>>> r.status_code
404
```

WTF；也许，可能是我们需要验证身份。那就有点纠结；其实Requests简化了多种身份验证形式的使用，包括非常常见的`Basic Auth`。(此处URL的ISSUE地址已被锁了)

```python
>>> from requests.auth import HTTPBasicAuth
>>> auth = HTTPBasicAuth('fake@example.com', 'not_a_real_password')
>>> r = requests.post(url=url, data=body, auth=auth)
>>> r.status_code
201
>>> content = r.json()
>>> print(content[u'body'])
Sounds great! I'll get right on it.
```

GOD...如果能够编辑这条评论那就好了！幸运的是，Github允许我们使用另一个HTTP方法`PATCH`来编辑评论。我们试试：

```python
>>> print(content[u"id"])
5804413
>>> body = json.dumps({u"body": u"Sounds great! I'll get right on it once I feed my cat."})
>>> url = u"https://api.github.com/repos/kennethreitz/requests/issues/comments/5804413"
>>> r = requests.patch(url=url, data=body, auth=auth)
>>> r.status_code
200
```

GOOD...现在我们再来试试`DELETE`方法；Github允许我们使用完全名副其实的`DELETE`方法来删除评论：

```python
>>> r = requests.delete(url=url, auth=auth)
>>> r.status_code
204
>>> r.headers['status']
'204 No Content'
```

很好；最后一件事是我想知道我已经使用了多少限额(ratelimit)。查查看，Github在响应头部发送这个信息，因此不必下载整个网页，我将使用一个`HEAD`请求来获取响应头。

```python
>>> r = requests.head(url=url, auth=auth)
>>> print r.headers
...
'x-ratelimit-remaining': '4995'
'x-ratelimit-limit': '5000'
...
```

^_^...

### 响应头链接字段

许多HTTP API都有相依你个头链接字段的特性，它们使得API能够更好的自我描述和自我显露。Github在API中为[分页](http://developer.github.com/v3/#pagination)使用了这些特性，例如：

```python
>>> url = 'https://api.github.com/users/kennethreitz/repos?page=1&per_page=10'
>>> r = requests.head(url=url)
>>> r.headers['link']
'<https://api.github.com/users/kennethreitz/repos?page=2&per_page=10>; rel="next", <https://api.github.com/users/kennethreitz/repos?page=6&per_page=10>; rel="last"'
```

Requests会自动解析这些响应头链接字段，并使得他们非常易于使用：

```python
>>> r.links["next"]
{'url': 'https://api.github.com/users/kennethreitz/repos?page=2&per_page=10', 'rel': 'next'}

>>> r.links["last"]
{'url': 'https://api.github.com/users/kennethreitz/repos?page=7&per_page=10', 'rel': 'last'}
```
