# Python3-cookbook-06

### 读写JSON数据

> `json`模块提供了一种很简单的方式来编码和解码JSON(JavaScript Object Notation)数据。其中两个主要的函数是`json.dumps()`和`json.loads()`，要比其它序列化函数库如`pickle`的接口少的多。

> 下面演示如何将一个Python数据结构转换为JSON：

```
import json

data = {
    'name' : 'ACME',
    'shares' : 100,
    'price' : 542.23
}

json_str = json.dumps(data)
```

> 下面演示如果将一个JSON编码的字符串转换回一个Python数据结构：

```
data = json.loads(json_str)
```

> 如果你要处理的是文件而不是字符串，你可以使用`json.dump()`和`json.load()`来编码和解码JSON数据。例如：

```
# Writing JSON data
with open('data.json', 'w') as f:
    json.dump(data, f)

# Reading data back
with open('data.json', 'r') as f:
    data = json.load(f)
```

> JSON编码支持的基本数据类型为`None`，`bool`，`int`，`float`和`str`，以及包含这些类型数据的`list`，`tuple`和`dictionaries`。对于`dictionaries`，`keys`需要是字符串类型(字典中任何非字符串类型的`key`在编码时会先转换为字符串)。为了遵循JSON规范，你应该只编码Python的`list`和`dictionaries`。而且，在web应用程序中，顶层对象被编码为一个字典时一个标准做法。

> JSON编码的格式对于Python语法而已几乎是完全一样的，除了一些小的差异之外。 比如，True会被映射为true，False被映射为false，而None会被映射为null。 下面是一个例子，演示了编码后的字符串效果：

```
>>> json.dumps(False)
'false'
>>> d = {'a': True,
...     'b': 'Hello',
...     'c': None}
>>> json.dumps(d)
'{"b": "Hello", "c": null, "a": true}'
>>>
```

> 注意，你可以使用`pprint`模块的`pprint()`函数来代替普通的`print()`函数。它会按照`key`的字母顺序并以一种更加美观的方式输出。

> 在编码JSON的时候，还有一些选项很有用。如果你像获得漂亮的格式化字符串后输出，可以使用`json.dumps()`的`indent`参数。它会使得输出和`pprint()`函数效果类似。比如：

```
>>> print(json.dumps(data))
{"price": 542.23, "name": "ACME", "shares": 100}
>>> print(json.dumps(data, indent=4))
{
    "price": 542.23,
    "name": "ACME",
    "shares": 100
}
>>>
```
