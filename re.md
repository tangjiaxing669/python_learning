# 正则表达式

正则表达式是一种用来匹配字符串的强有力的武器。它的设计思想是用一种描述性的语言来给字符串定义一个规则，凡是符合规则的字符串，我们就认为它“匹配”了，否则，该字符串就是不合法的。

对于正则表达式的写法请参考[正则表达式30分钟入门教程](http://deerchao.net/tutorials/regex/regex.htm#mission)或者[Regular简易教程](https://github.com/tangjiaxing669/python_learning/blob/master/picture/regular.png)。

# re模块

### re.split方法

`re.split()`方法主要用于更加灵活的来切割字符串；例如下面一个简单的例子：

```python
>>> line = 'asdf fjdk; afed, fjek,asdf, foo'
>>> import re
>>> re.split(r'[;,\s]\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
```

上面的例子中，**分隔符可以是逗号，分号或者是空格，并且后面紧跟着零个或多个空格。**只要这个模式被找到，那么匹配的分隔符两边的实体都会被当成是结果中的元素返回。返回结果为一个字段列表。

> 注意，上面的例子中，被匹配到的文本是没有出现在返回的列表元素中的。

在某些场景，当你需要把匹配到的文本也放入到返回的列表元素中时，你可以使用括号捕获分组（**保留分割字符**）。例如：

```python
>>> line = 'asdf fjdk; afed, fjek,asdf, foo'
>>> import re
>>> re.split(r'(;|,|\s)\s*', line)
['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
```

如果你不想保留分割字符串到结果列表中去，但仍然需要使用到括号来分组正则表达式的话，请确保你的分组是非捕获分组，形如`(?:...)`。比如：

```python
>>> line = 'asdf fjdk; afed, fjek,asdf, foo'
>>> import re
>>> re.split(r'(?:;|,|\s)\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
```

> 总结
> `()`：表示捕获分组，会保留分割字符串到结果列表中；
> `(?:...)`：表示非捕获分组，不会保留分割字符串到结果列表中；

### re.match()方法

假设你x数字格式的日期字符串，比如`11/27/2012`，你可以这样做：

```python
>>> text1 = '11/27/2012'
>>> text2 = 'Nov 27, 2012'
>>>
>>> import re
>>> # Simple matching: \d+ means match one or more digits
>>> if re.match(r'\d+/\d+/\d+', text1):
... print('yes')
... else:
... print('no')
...
yes
>>> if re.match(r'\d+/\d+/\d+', text2):
... print('yes')
... else:
... print('no')
...
no
>>>
```

> 注意，在你的程序中如果一个模式匹配了多次，那么你应该先将模式字符串预编译为模式对象。

例如：

```python
>>> datepat = re.compile(r'\d+/\d+/\d+')
>>> if datepat.match(text1):
... print('yes')
... else:
... print('no')
...
yes
>>> if datepat.match(text2):
... print('yes')
... else:
... print('no')
...
no
>>>
```

> 注意，`re.match()`方法是从第一个字符开始匹配的，如果第一个字符没有匹配上，就返回`None`，否则返回`match`对象。

### re.search()方法

```python
>>> import re
>>> text2 = 'Nov 27, 2012'
>>> text1 = '11/27/2012'
>>> re.search(r'(\d+)/(\d+)/(\d+)', text1)
<_sre.SRE_Match object; span=(0, 10), match='11/27/2012'>
>>> re.search(r'(\d+)/(\d+)/(\d+)', text2)
>>> re.search(r'(\d+)', text2)
<_sre.SRE_Match object; span=(4, 6), match='27'>
>>> re.search(r'(\d+)+', text2)
<_sre.SRE_Match object; span=(4, 6), match='27'>
```

> 注意，`re.search()`方法只会匹配最开始找到的项。

### re.findall()方法

如果你想查找字符串任意部分的模式出现为止，则应该使用`findall()`方法去代替。比如：

```python
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> import re
>>> datepat = re.compile(r'\d+/\d+/\d+')
>>> datepat.findall(text)
['11/27/2012', '3/13/2013']
>>>
```

在定义正则式的时候，通常会利用括号去捕获分组。比如：

```python
>>> text1 = '11/27/2012'
>>> import re
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>> m = datepat.match(text1)
>>> m
<_sre.SRE_Match object at 0x1005d2750>
>>> # Extract the contents of each group
>>> m.group(0)
'11/27/2012'
>>> m.group(1)
'11'
>>> m.group(2)
'27'
>>> m.group(3)
'2012'
>>> m.groups()
('11', '27', '2012')
>>> month, day, year = m.groups()
>>>
>>> # Find all matches (notice splitting into tuples)
>>> text
'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> datepat.findall(text)
[('11', '27', '2012'), ('3', '13', '2013')]
>>> for month, day, year in datepat.findall(text):
... print('{}-{}-{}'.format(year, month, day))
...
2012-11-27
2013-3-13
>>>
```

### re.finditer()方法

`findall()`方法会搜索文本并以列表形式返回所有的匹配。如果你想以迭代方式返回匹配，可以使用`finditer()`方法来代替。比如：

```python
>>> for m in datepat.finditer(text):
... print(m.groups())
...
('11', '27', '2012')
('3', '13', '2013')
>>>
```

> 注意，如果你打算做大量的匹配和搜索操作的话，最好先编译正则表达式，然后再重复使用它。 模块级别的函数会将最近编译过的模式缓存起来，因此并不会消耗太多的性能， 但是如果使用预编译模式的话，你将会减少查找和一些额外的处理损耗。

### re.sub()和re.subn()方法

对于简单的字符串的搜索和替换可以使用`str.replace()`方法即可；而对于复杂的模式就得使用`re.sub()`方法了。

假设你想将形式为`11/27/2012`的日期字符串改成`2012-11-27`。示例如下：

```python
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> import re
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>> datepat.sub(r'\3-\1-\2', text)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>>
>>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
```

`sub()`函数中的第一个参数是被匹配的模式，第二个参数是替换模式。反斜杠数字比如`\3`指向前面模式的捕获组号。

> 注意，还是那句话；如果你打算用相同的模式做多次替换，请考虑编译(`re.compile()`)它来提升性能。

如果除了替换后的结果外，你还想知道有多少替换发生了，可以使用`re.subn()`来代替。比如：

```python
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> import re
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>> newtext, n = datepat.subn(r'\3-\1-\2', text)
>>> newtext
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>> n
2
>>>
```

### re.IGNORECASE标志

为了在文本操作时忽略大小写，你需要在使用`re`模块的时候给这些操作提供`re.IGNORECASE`标志参数。比如：

```python
>>> text = 'UPPER PYTHON, lower python, Mixed Python'
>>> re.findall('python', text, flags=re.IGNORECASE)
['PYTHON', 'python', 'Python']
>>> re.sub('python', 'snake', text, flags=re.IGNORECASE)
'UPPER snake, lower snake, Mixed snake'
>>>
```

### 贪婪/非贪婪模式

当你试着用正则表达式匹配某个文本模式时，它找到的是模式的最长可能匹配。而你想修改它变成查找最短的可能匹配。

这个问题一般出现在需要匹配一对分隔符之间的文本的时候(比如引号包含的字符串)。比如：

```python
>>> str_pat = re.compile(r'\"(.*)\"')
>>> text1 = 'Computer says "no."'
>>> str_pat.findall(text1)
['no.']
>>>
>>> text2 = 'Computer says "no." Phone says "yes."'
>>> str_pat.findall(text2)
['no." Phone says "yes.']
>>>
```

在这个例子中，模式`r'\"(.*)\"'`的意图是匹配被双引号包含的文本。但是在正则表达式中`*`操作符是贪婪的，因此匹配操作会查找最长的可能匹配。于是在第二个例子中搜索`text2`的时候返回结果并不是我们想要的。

为了修正这个问题，可以在模式中的`*`操作符后面加上`?`修饰符，就像这样：

```python
>>> str_pat = re.compile(r'\"(.*?)\"')
>>> str_pat.findall(text2)
['no.', 'yes.']
>>>
```

这样就使得匹配变成非贪婪模式，从而得到最短的匹配，也就是我们想要的结果。

这一节展示了在写包含点(.)字符的正则表达式的时候遇到的一些常见问题。 在一个模式字符串中，点(.)匹配除了换行外的任何字符。 然而，如果你将点(.)号放在开始与结束符(比如引号)之间的时候，那么匹配操作会查找符合模式的最长可能匹配。 这样通常会导致很多中间的被开始与结束符包含的文本被忽略掉，并最终被包含在匹配结果字符串中返回。 通过在`*`或者`+`这样的操作符后面添加一个`?`可以强制匹配算法改成寻找最短的可能匹配。

### re.DOTALL多行匹配

在Python中，正则表达式中的点`.`是不能匹配换行符的。例如：

```python
>>> comment = re.compile(r'/\*(.*?)\*/')
>>> text1 = '/* this is a comment */'
>>> text2 = '''/* this is a
... multiline comment */
... '''
>>> comment.findall(text1)
[' this is a comment ']
>>> comment.findall(text2)
[]
>>>
```

`re.compile()`函数接受一个标志参数叫`re.DOTALL`，在这里非常有用。它可以让正则表达式中的点`.`匹配包括换行符在内的任意字符。比如：

```python
>>> comment = re.compile(r'/\*(.*?)\*/', flags=re.DOTALL)
>>> comment.findall(text2)
[' this is a\n multiline comment ']
```

对于简单的情况使用`re.DOTALL`标记参数工作的很好， 但是如果模式非常复杂或者是为了构造字符串令牌而将多个模式合并起来， 这时候使用这个标记参数就可能出现一些问题。 如果让你选择的话，最好还是定义自己的正则表达式模式，这样它可以在不需要额外的标记参数下也能工作的很好。例如：

```python
>>> comment = re.compile(r'/\*((?:.|\n)*?)\*/')
>>> comment.findall(text2)
[' this is a\n multiline comment ']
>>>
```
