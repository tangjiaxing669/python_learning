# Requests

> 本文档讨论如何配置Requests使用多种身份认证方式

> 参考 http://docs.python-requests.org/zh_CN/latest/user/authentication.html

### 基本身份认证

许多要求身份认证的web服务都接受`HTTP Basic Auth`。这是最简单的一种身份认证，并且Requests对这种认证方式的支持是直接开箱即用的。下面使用`HTTP Basic Auth`来发送请求：

```python
>>> from requests.auth import HTTPBasicAuth
>>> requests.get('https://api.github.com/user', auth=HTTPBasicAuth('tangjiaxing669', 'pass'))
<Response [200]>
```

事实上，Requests提供了一种简写的使用方式：

```python
>>> requests.get('https://api.github.com/user', auth=('tangjiaxing669', 'pass'))
<Response [200]>
```

像这样在一个元组中提供认证信息与前一个`HTTPBasicAuth`例子是完全相同的。

### 摘要式身份认证

另一种非常流行的HTTP身份认证形式是摘要式身份认证，Requests对它的支持也是开箱即用：

```python
>>> from requests.auth import HTTPDigestAuth
>>> url = 'http://httpbin.org/digest-auth/auth/user/pass'
>>> requests.get(url, auth=HTTPDigestAuth('user', 'pass'))
<Response [200]>
```

### 其他身份认证形式

Requests被设计成允许其他形式的身份认证。开源社区的成员时常为更复杂或不那么常用的身份认证形式编写认证处理插件。其中一些最优秀的已被收集在[Requests organization](https://github.com/requests)页面中，包括：

> * [Kerberos](https://github.com/requests/requests-kerberos)
> * [NTLM](https://github.com/requests/requests-ntlm)

如果你想使用其中任何一种身份认证形式，直接去他们的Github页面，依照说明进行。

### 新的身份认证形式

如果你找不到所需要的身份认证形式，那么你也可以自己实现它。Requests非常易于添加你自己的身份认证形式。

要想自己实现，就从`requests.auth.AuthBase`继承一个子类，并实现`__call__()`方法：

```python
>>> import requests
>>> class MyAuth(requests.auth.AuthBase):
...     def __call__(self, r):
...         # Implement my authentication
...         return r
...
>>> url = 'http://httpbin.org/get'
>>> requests.get(url, auth=MyAuth())
<Response [200]>
```

当一个身份认证模块被附加到一个请求上，在设置request期间就会调用该模块。因此`__call__` 方法必须完成使得身份认证生效的所有事情。一些身份认证形式会额外地添加钩子来提供进一步的功能。
