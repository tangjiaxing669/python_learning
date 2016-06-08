# Python3-cookbook-02

### 使用多个界定符分割字符串

> `string`对象的`split()`方法只适应于非常简单的字符串分割情形，它并不允许有多个分隔符或这是分隔符周围不确定的空格。当你需要更加灵活的切割字符串的时候，最好使用`re.split()`方法：

```
>>> line = 'asdf fjdk; afed, fjek,asdf, foo'
>>> import re
>>> re.split(r'[;,\s]\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
```

> 函数`re.split()`是非常实用的，因为它允许你为分割符指定多个正则模式。比如，在上面的例子中，分隔符可以是逗号，分号或者是空格，并且后面紧跟着任意个空格；只要这个模式被找到，那么匹配的分隔符两边的实体都会被当成是结果中的元素返回。返回结果为一个子段列表，这个跟`str.split()`返回值类型是一样的。

> 当使用`re.split()`函数时候，需要特别注意的是正则表达是中是否包含一个括号捕获分组。如果使用捕获分组，那么被匹配的文本也将出现在结果列表中。比如，观察以下这段代码运行后的结果：

```
>>> fields = re.split(r'(;|,|\s)\s*', line)
>>> fields
['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
>>>
```

> 如果你不想保留分割字符串到结果列表中去，但仍然需要使用到括号来分组正则表达式的话，确保你的分组是非捕获分组，形如`(?:...)`。比如：

```
>>> re.split(r'(?:,|;|\s)\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>>
```

> *注意捕获分组与非捕获分组； `(...)`和`(?:...)`*

### 字符串开头或结尾匹配

> 检查字符串开头或结尾的一个简单方法是使用`str.startswith()`或者`str.endswith()`方法。比如：

```
>>> filename = 'spam.txt'
>>> filename.endswith('.txt')
True
>>> filename.startswith('file:')
False
>>> url = 'http://www.python.org'
>>> url.startswith('http:')
True
>>>
```

> 如果你想检查多种匹配可能，只需要将所有的匹配想放入到一个元组中去，然后传给`startswith()`或者`endswith()`方法：

```
>>> import os
>>> filenames = os.listdir('.')
>>> filenames
[ 'Makefile', 'foo.c', 'bar.py', 'spam.c', 'spam.h' ]
>>> [name for name in filenames if name.endswith(('.c', '.h')) ]
['foo.c', 'spam.c', 'spam.h'
>>> any(name.endswith('.py') for name in filenames)
True
>>>
```

> 注意；这个方法必须要输入一个元组作为参数。如果你恰巧有一个`list`或者`set`类型的选择项，要确保传递参数前先调用`tuple()`将其转换为元组类型。比如：

```
>>> choices = ['http:', 'ftp:']
>>> url = 'http://www.python.org'
>>> url.startswith(choices)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: startswith first arg must be str or a tuple of str, not list
>>> url.startswith(tuple(choices))
True
>>>
```

### 用shell通配符匹配字符串

> 你想使用*Unix Shell*中常用的通配符(比如`*.py`，`Dat[0-9]*.csv`等)去匹配文本字符串

> `fnmatch`模块提供了两个函数，`fnmatch()`和`fnmatchcase()`，可以用来实现这样的匹配。如下：

```
>>> from fnmatch import fnmatch, fnmatchcase
>>> fnmatch('foo.txt', '*.txt')
True
>>> fnmatch('foo.txt', '?oo.txt')
True
>>> fnmatch('Dat45.csv', 'Dat[0-9]*')
True
>>> names = ['Dat1.csv', 'Dat2.csv', 'config.ini', 'foo.py']
>>> [name for name in names if fnmatch(name, 'Dat*.csv')]
['Dat1.csv', 'Dat2.csv']
>>>
```

> `fnmatch()`函数使用底层操作系统的大小写明干规则(不同的系统是不一样的)来匹配模式；比如：

```
>>> # On OS X (Mac)
>>> fnmatch('foo.txt', '*.TXT')
False
>>> # On Windows
>>> fnmatch('foo.txt', '*.TXT')
True
>>>
```

> 如果你很对这个区别很再以，可以使用`fnmatchcase()`来代替。它完全使用你的模式大小写匹配。比如：

```
>>> fnmatchcase('foo.txt', '*.TXT')
False
>>>
```

> 这两个函数通常会被忽略的一个特性是在处理非文件名的字符串时候他们也是很有用的。比如，假设你有一个街道地址的列表数据：

```
addresses = [
    '5412 N CLARK ST',
    '1060 W ADDISON ST',
    '1039 W GRANVILLE AVE',
    '2122 N CLARK ST',
    '4802 N BROADWAY',
]
```

> 你可以像这样写列表推导：

```
>>> from fnmatch import fnmatchcase
>>> [addr for addr in addresses if fnmatchcase(addr, '* ST')]
['5412 N CLARK ST', '1060 W ADDISON ST', '2122 N CLARK ST']
>>> [addr for addr in addresses if fnmatchcase(addr, '54[0-9][0-9] *CLARK*')]
['5412 N CLARK ST']
>>>
```

> 注意，`fnmatch()`函数匹配能力介于简单的字符串方法和强大的正则表达式之间。如果在数据处理操作中只需要简单的通配符就能完成的时候，这通常是一个比较合理的方案。

> 使用`fnmatch.filter()`对一个`list`进行过滤；如下：

```
import fnmatch
import os

for dirpath, dirnames, filenames in os.walk('/root/python3/logscan'):
    for i in fnmatch.filter(filenames, '*.py'):
        print(os.path.join(dirpath, i))
```

执行结果如下：

```
/root/python3/logscan/app.py
/root/python3/logscan/logscan/__init__.py
/root/python3/logscan/logscan/match.py
/root/python3/logscan/logscan/watch.py
```

### 字符串匹配和搜索

> 对于复杂的匹配需要使用正则表达式和`re`模块。为了解释正则表达式的基本原理，假设你想匹配数字个是的日期字符串，比如`11/27/2012`，你可以这样做：

```
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

> 如果你想使用同一个模式去作多次匹配，你应该先将模式字符串预编译为模式对象。比如：

```
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

> 注意，`match()`总是从字符串开始去匹配，如果你想查找字符串任意部分的模式出现位置，使用`findall()`方法去代替。比如：

```
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> datepat.findall(text)
['11/27/2012', '3/13/2013']
>>>
```

> 在定义正则式的时候，通常会利用括号去捕获分组。比如：

```
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>>
```

> 捕获分组可以是的后面的处理更加简单，因为可以分别将每个组的内容提取出来。比如：

```
>>> m = datepat.match('11/27/2012')
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

> `findall()`方法会搜索文本并以列表形式返回所有的匹配。如果你想以迭代方式返回匹配，可以使用`finditer()`方法来代替，比如：

```
>>> for m in datepat.finditer(text):
... print(m.groups())
...
('11', '27', '2012')
('3', '13', '2013')
>>>
```

> 注意，关于正则表达式最基本的方法，核心步骤就是使用`re.compile()`编译正则表达式字符串，然后使用`match()`，`findall()`或者`finditer()`等方法。

```
>>> for m in datepat.finditer(text):
... print(m.groups())
...
('11', '27', '2012')
('3', '13', '2013')
>>>
```

> 注意，`match()`方法仅仅检查字符串的开始部分。

### 字符串搜索和替换

> 对于简单的字面模式，直接使用`str.replace()`方法即可；对于复杂的模式，请使用`re`模块中的`sub()`函数。为了说明这个，假设你想将形式为`11/27/2012`的日期字符串改成`2012-11-27`。如下：

```
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> import re
>>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>>
```

> `sub()`函数中的第一个参数被匹配的模式，第二个参数是替换模式。反斜杠数字比如`\3`指向前面模式的捕获组号。

> 如果你打算用相同的模式做多次替换，考虑先编译它来提升性能。比如：

```
>>> import re
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>> datepat.sub(r'\3-\1-\2', text)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>>
```

> 对于更加复杂的替换，可以传递一个替换回调函数来代替，比如：

```
>>> from calendar import month_abbr
>>> def change_date(m):
... mon_name = month_abbr[int(m.group(1))]
... return '{} {} {}'.format(m.group(2), mon_name, m.group(3))
...
>>> datepat.sub(change_date, text)
'Today is 27 Nov 2012. PyCon starts 13 Mar 2013.'
>>>
```

> 如果除了替换后的结果外，你还想知道有多少替换发生了，可是使用`re.subn()`来代替。比如：

```
>>> newtext, n = datepat.subn(r'\3-\1-\2', text)
>>> newtext
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>> n
2
>>>
```

### 字符串忽略大小写的搜索替换

> 为了在文本操作时忽略大小写，你需要在使用`re`模块的时候给这些操作提供`re.IGNORECASE`标志参数；比如：

```
>>> text = 'UPPER PYTHON, lower python, Mixed Python'
>>> re.findall('python', text, flags=re.IGNORECASE)
['PYTHON', 'python', 'Python']
>>> re.sub('python', 'snake', text, flags=re.IGNORECASE)
'UPPER snake, lower snake, Mixed snake'
>>>
```

### 最短匹配模式

> 这个问题一般出现在需要匹配一对分隔符之间的文本的时候（比如引号包含的字符串）。如下：

```
>>> str_pat = re.compile(r'\"(.*)\"')
>>> text1 = 'Computer says "no."'
>>> str_pat.findall(text1)
['no.']
>>> text2 = 'Computer says "no." Phone says "yes."'
>>> str_pat.findall(text2)
['no." Phone says "yes.']
>>>
```

> 在这里列子中，模式`r'\"(.*)\"'`的意图时匹配被双引号包含的文本。但是在正则表达式中`*`操作符是贪婪的，因此匹配操作会查找最长的可能匹配。于是在第二个例子中搜索`text2`的时候返回结果并不是我们想要的。

> 为了修正这个问题，可以在模式中的`*`操作符后面加上`?`修饰符，如下：

```
>>> str_pat = re.compile(r'\"(.*?)\"')
>>> str_pat.findall(text2)
['no.', 'yes.']
>>>
```

> 这样就使得匹配变成非贪婪模式，从而得到最短的匹配，也就是我们想要的结果。

> 注意，这一节在写包含点(.)字符的正则表达式的时候遇到的一些常见问题。在一个模式字符串中，点(.)匹配除了换行外的任何字符。然而，如果你将点(.)号放在开始与结束符(比如引号)之间的时候，那么匹配操作会查找符合模式的最长可能匹配。这样通常会导致很多中间的被开始与结束符包含的文本被忽略掉，并最终被包含在匹配结果字符串中返回。通过在`*`或者`+`这样的操作符后面添加一个`?`可以强制匹配算法改成寻找最短的可能匹配。

### 多行匹配模式

> 这个问题很典型的出现在当你用点(.)去匹配任意字符的时候，忘记了点(.)不能匹配换行符的事实。

> `re.compile()`函数接受一个标志参数叫`re.DOTALL`，在这里非常有用；它可以让正则表达式中的点(.)匹配包括换行符在内的任意字符。比如：

```
>>> text2 = '''/* this is a
... multiline comment */
... '''
>>> comment = re.compile(r'/\*(.*?)\*/', re.DOTALL)
>>> comment.findall(text2)
[' this is a\n multiline comment ']
>>>
```

> 注意，在正则表达式中，可以使用`?P<NAME>`来给模式命名，以供后面使用。

```
import re
NAME = r'(?P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'
NUM = r'(?P<NUM>\d+)'
PLUS = r'(?P<PLUS>\+)'
TIMES = r'(?P<TIMES>\*)'
EQ = r'(?P<EQ>=)'
WS = r'(?P<WS>\s+)'

master_pat = re.compile('|'.join([NAME, NUM, PLUS, TIMES, EQ, WS]))
```

### 字符串的对齐

> 对于基本的字符串对齐操作，可以使用字符串的`ljust()`，'rjust()`和`center()`方法。但是函数`format()`同样可以用来很容易的对齐字符串。你要做的就是使用`<,>`或者`^`字符后面紧跟一个指定的宽度；比如：

```
>>> format(text, '>20')
'         Hello World'
>>> format(text, '<20')
'Hello World         '
>>> format(text, '^20')
'    Hello World     '
>>>
```

> 如果你想指定一个非空格的填充字符，将它写到对齐字符的前面即可；

```
>>> format(text, '=>20s')
'=========Hello World'
>>> format(text, '*^20s')
'****Hello World*****'
>>>
```

> 当格式化多个值的时候，这个格式代码也可以被用在`format()`方法中；比如：

```
>>> '{:>10s} {:>10s}'.format('Hello', 'World')
'     Hello      World'
>>>
```

> 注意，不要忘记大括号中的`:`号

> `format()`函数的一个好处是它不仅适用于字符串。它可以用来格式化任何值，使得它非常的通用。

```
>>> x = 1.2345
>>> format(x, '>10')
'    1.2345'
>>> format(x, '^10.2f')
'   1.23   '
>>>
```

### 以指定列宽格式化字符串

> 使用`textwrap`模块来格式化字符串的输出，比如：

```
s = "Look into my eyes, look into my eyes, the eyes, the eyes, \
the eyes, not around the eyes, don't look around the eyes, \
look into my eyes, you're under."
```

> 下面演示使用`textwrap`格式化字符串的多种方式：

```
>>> import textwrap
>>> print(textwrap.fill(s, 70))
Look into my eyes, look into my eyes, the eyes, the eyes, the eyes,
not around the eyes, don't look around the eyes, look into my eyes,
you're under.

>>> print(textwrap.fill(s, 40))
Look into my eyes, look into my eyes,
the eyes, the eyes, the eyes, not around
the eyes, don't look around the eyes,
look into my eyes, you're under.

>>> print(textwrap.fill(s, 40, initial_indent='    '))
    Look into my eyes, look into my
eyes, the eyes, the eyes, the eyes, not
around the eyes, don't look around the
eyes, look into my eyes, you're under.

>>> print(textwrap.fill(s, 40, subsequent_indent='    '))
Look into my eyes, look into my eyes,
    the eyes, the eyes, the eyes, not
    around the eyes, don't look around
    the eyes, look into my eyes, you're
    under.
```
