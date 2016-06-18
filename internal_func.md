# Python 内置函数

### filter()

filter()函数包括两个参数，分别是`function`和`list`。该函数根据`function`参数返回的结果是否为真来过滤`list`参数中的项，最后返回一个`filter object`，如下：

```python
>>> a = [0,1,2,3,4,5,6,7]
>>> filter(lambda x: x>4, a)
<filter object at 0x7fd63f684f28>
>>> list(filter(lambda x: x>4, a))
[5, 6, 7]
```

如果`filter`参数值为`None`，就使用`identity()`函数，`list`参数中所有为假的元素都将被删除。如下：

```python
>>> list(filter(None, a))
[1, 2, 3, 4, 5, 6, 7]
```

### any()

> Return True if bool(x) is True for any x in the iterable. If the iterable is empty, return False.
> 只要迭代器中有一个元素为真就为真。

```python
>>> a = [True, False]
>>> any(a)
True
>>> b = ['good','bad','good','good']  
>>> any(i == 'bad' for i in b)
True
>>> b.pop(1)
'bad'
>>> b
['good', 'good', 'good']
>>> any(i == 'bad' for i in b)
False
```

### all()

> Return True if bool(x) is True for all values x in the iterable. If the iterable is empty, return True.
> 迭代器中所有的判断项返回都是真，结果才为真

```python
>>> a = [True, False]
>>> all(a)
False
>>> b = ['good','bad','good','good']
>>> all(i == 'good' for i in b)
False
>>> b.pop(1)
'bad'
>>> all(i == 'good' for i in b)
True
```

### chr()

> `chr()`函数返回所对应的ASCII字符；值是0-255

```python
>>> chr(65)
'A'
>>> chr(66)
'B'
>>> chr(67)
'C'
>>> chr(68)
'D'
....
```

### ord()

> `ord()`函数返回所对应的ASCII码值

```python
>>> ord('A')
65
>>> ord('B')
66
>>> ord('C')
67
>>> ord('D')
68
....
```

> 利用`chr()`和`ord()`以及`random()`可以实现一个简单的验证码的功能。

### calendar模块

```python
>>> import calendar

>>> calendar.day_abbr[:]		# 返回每周的英文缩写
['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
>>> calendar.day_abbr[0]		# 返回星期一的英文缩写
'Mon'
>>> calendar.day_abbr[1]		# 返回星期二的英文缩写
'Tue'
....

>>> calendar.day_name[:]		# 返回每周的英文全称
['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
>>> calendar.day_name[0]		# 返回星期一的英文全称
'Monday'
>>> calendar.day_name[1]		# 返回星期二的英文全称
'Tuesday'
....

>>> calendar.mdays[3]	# 返回三月份有多少天
31
>>> calendar.mdays[2]	# 返回而月份有多少天
28
....

>>> print(calendar.month(2016, 6))	# 以日历的形式返回2016年6月
     June 2016
Mo Tu We Th Fr Sa Su
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30
>>> print(calendar.month(2016, 9))	# 以日历的形式返回2016年9月
   September 2016
Mo Tu We Th Fr Sa Su
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30
....

>>> calendar.month_abbr[:]	# 返回英文缩写的月份
['', 'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
>>> calendar.month_abbr[1]	# 返回英文缩写的一月
'Jan'
>>> calendar.month_abbr[2]	# 返回英文缩写的二月
'Feb'
....

>>> calendar.month_name[:]	# 返回英文全称的月份
['', 'January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December']
>>> calendar.month_name[1]	# 返回英文全称的一月
'January'
>>> calendar.month_name[2]	# 返回英文全称的二月
'February'
....

>>> calendar.weekday(2016, 6, 5)	# 返回2016年6月5号是星期日
6						# 注意，星期是从0开始的，0代表星期一，6代表星期日
>>> calendar.weekday(2016, 6, 6)	# 返回2016年6月5号是星期一
0
```

### os.walk()

> 以指定的目录为根目录，对旗下的每一个子目录进行遍历；返回一个包含三个元素的元组`dirpath`，`dirnames`和`filenames`；

> `dirpath`：以字符串形式返回当前目录的绝对路径

> `dirnames`：以列表的形式返回当前目录下所有的子目录名，非绝对路径，可用`os.path.join()`

> `filenames`：以列表的形式返回当前目录下所有的文件名，非绝对路径，可用`os.path.join()`

假设我有如下目录结构：

```shell
[root@localhost python3]# tree -a /root/python3/logscan/
/root/python3/logscan/
|-- app.py
|-- .idea
|   |-- logscan.iml
|   |-- misc.xml
|   |-- modules.xml
|   `-- workspace.xml
`-- logscan
    |-- __init__.py
    |-- match.py
    `-- watch.py

2 directories, 8 files
[root@localhost python3]# 
```

示例代码如下：

```python
import os
for dirpath, dirnames, filenames in os.walk('/root/python3/logscan/'):
    print('dirpath => {0}'.format(dirpath))
    print('dirnames => {0}'.format(dirnames))
    print('filenames => {0}'.format(filenames))
    print('=' * 20)
```

执行结果如下：

```python
dirpath => /root/python3/logscan
dirnames => ['.idea', 'logscan']
filenames => ['app.py']
====================
dirpath => /root/python3/logscan/.idea
dirnames => []
filenames => ['misc.xml', 'logscan.iml', 'modules.xml', 'workspace.xml']
====================
dirpath => /root/python3/logscan/logscan
dirnames => []
filenames => ['__init__.py', 'match.py', 'watch.py']
====================
```

### iter()

> `iter`函数一个鲜为人知的特性是它接受一个可选的`callable`对象和一个标记(结尾)值作为输入参数。当以这种方式使用的时候，它会创建一个迭代器，这个迭代器会不断调用`callable`对象知道返回值和标记值像等为止。

> 这种特殊的方法对于一些特定的会被重复调用的函数很有效果，比如涉及到I/O调用的函数。举例来讲，如果你想从套接字或文件中以数据块的方式读取数据，通常你得要不断重复的执行`read()`或`recv()`，并再后面紧跟一个文件结尾测试来决定是否终止。这节中使用一个简单的`iter()`调用就可以将两者结合起来了。其中`lambda`函数参数是为了创建一个无参的`callable`对象，并为`recv`或`read()`方法提供了`size`参数。

> 一个常见的IO操作程序可能会像下面这样：

```python
CHUNKSIZE = 8192

def reader(s):
    while True:
        data = s.recv(CHUNKSIZE)
        if data == b'':
            break
        process_data(data)
```

> 这种代码通常可以使用`iter()`来代替，如下：

```python
def reader2(s):
    for chunk in iter(lambda: s.recv(CHUNKSIZE), b''):
        pass
        # process_data(data)
```

> 如果你怀疑它到底能不能正常工作，可试验下一个简单的列子。比如：

```python
>>> import sys
>>> f = open('/etc/passwd')
>>> for chunk in iter(lambda: f.read(10), ''):
...     n = sys.stdout.write(chunk)
...
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
_uucp:*:4:4:Unix to Unix Copy Protocol:/var/spool/uucp:/usr/sbin/uucico
...
>>>
```

### yield

> yield有以下几个方法：

- `__next__`，每次for循环还有next都会调用这个方法。
- `send(value)`，用`value`对`yield`语句赋值，再执行接下来的代码直到下个`yield`。
- `throw(typ[,val[,tb]])，抛出错误，类似于`raise`。
- `close()`，告诉生成器你已经死了，再调用会抛出`StopIteration`异常。

> 我们可以使用类似列表生成式的方法来创建一个生成器，如下：

```python
>>> g = (x * x for x in range(5))
>>> g
<generator object <genexpr> at 0x7f78065335e8>
```

> 我们可以通过上面所说的`next()`方法来获得Generator的值，如下：

```python
>>> next(g)       
0
>>> next(g) 
1
>>> next(g) 
4
>>> next(g) 
9
>>> next(g) 
16
>>> next(g) 
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

> 注意，Generator保存的是算法，每次调用`next(g)`，就计算出`g`的下一个元素的值，直到计算到最后一个元素；当没有更多元素时，就抛出`StopIteration`异常。

> 使用`yield`来构建一个Generator：

```python
def G():
    i = 1
    while True:
        m = yield i
        print('m => {!r}'.format(m))
        i+=1
        if i > 100:break

a = G()
print(next(a))
print(next(a))
print(next(a))
print(next(a))
```

> 上面的代码执行后结果将会变成如下：

```python
1
m => None
2
m => None
3
m => None
4
```

> 1、2、3、4我知道是`yield i`返回的，但是`m`的值为啥会是`None`？此时，就需要用到`send()`方法了，我们将上面的代码改成如下：

```python
def G():
    i = 1
    while True:
        print('to here 1')
        m = yield i
        print('to here 2')
        print('m => {!r}'.format(m))
        i += 1
        if i > 100: break

a = G()
next(a)
print(a.send('0'))
```

> 执行结果如下：

```python
to here 1
to here 2
m => '0'
to here 1
2
```

> 现在我们一步步来分析代码的走向；当执行`next(a)`时，程序会输出`to here 1`并执行到`m = yield i`暂停运行，回到`print(a.send('0'))`，我们上面说了`send()`方法的作用，它会用send括号中的值来覆盖`yield`语句(注意，是语句，不是`yield`的返回值)，然后输出`to here 2`和`m`的值，程序依次向下执行，直到下个`yield`暂停执行(在程序到达下个`yield`之前会输出`to here 1`)；那么最后的那个`2`是怎么来的？这个我也比较纠结，猜想是`send()`之后的第一个`yield`会被执行并返回。

> 我们再做如下一个测试：

```python
def G():
    i = 1
    while True:
        print('to here 1')
        m = yield i
        print('to here 2')
        print('m => {!r}'.format(m))
        i += 1
        if i > 100: break


a = G()
next(a)
print(a.send('0'))
print(a.send('hello'))
```

> 上面执行后的结果如下：

```python
to here 1
to here 2
m => '0'
to here 1
2
to here 2
m => 'hello'
to here 1
3
```

> 我们接着上面的代码解析再来分析下；当程序`yield`返回`2`之后，遇到`a.send('hello')`，那么`m`的值就改成了`hello`，并输出`to here 2`和`m`的值(`hello`)，在程序到达下一个`yield`之前回输出`to here 1`并执行`yield i`(此时的**i**为3)。

> 注意，`yield`还有一个比较微妙的地方需要提及以下，就是在调用`send`方法之前，必须先调用一次`next()`方法，让生成器执行到`yield`语句处才能进行赋值。外面加上`while`循环是为了避免出现`send`之后，生成器没有`yield`语句了而抛出`TypError`的情况。

> 我们演示下：

```python
def G():
    i = 1
    while True:
        print('to here 1')
        m = yield i
        print('to here 2')
        print('m => {!r}'.format(m))
        i += 1
        if i > 100: break


a = G()
# next(a)
print(a.send('0'))
```

> 当我们执行上面的代码后会抛出如下的异常：

```python
Traceback (most recent call last):
  File "/root/python3/terminal_size.py", line 18, in <module>
    print(a.send('0'))
TypeError: can't send non-None value to a just-started generator
```

> OK，下面再来看看`yield from`语法，它允许你从另一个可迭代的对象中`yield`数据；比如：

```python
>>> def g(x):
...     yield from range(x) 
...
>>> list(g(5))
[0, 1, 2, 3, 4]
>>> 
```

> 另一个例子：

```python
>>> def a(x): 
...     yield from range(x)
... 
>>> def b(x):
...     yield from a(x)
...
>>> list(b(5))
[0, 1, 2, 3, 4]
>>>
```

### 类中的self

> 我们知道类中的`self`代表了这个类对象的本身(也就是实例对象本身)；我们先看如下一段代码：

```python
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def method(self):
        return self

p = Pair(3, 4)

print('p is {0!r}'.format(p))
print('p is {0}'.format(p))
print('self is {0}'.format(p.method()))
```

> 上面代码执行结果如下：

```python
p is <__main__.Pair object at 0x7f85b3380ac8>
p is <__main__.Pair object at 0x7f85b3380ac8>
self is <__main__.Pair object at 0x7f85b3380ac8>
```

> 上面代码中的`!r`表示指定使用`__repr__()`方法输出；第二个`print()`语句默认会调用类的`__str__()`方法来输出；第三个`print()`指定输出类方法`p.method()`，注意，类方法`method()`中定义的是`return self`语句；上面的输出结果都是一样(输出实例对象的本身)。
> 现在我们把代码稍稍改动一下：

```python
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Pair({0.x!r}, {0.y!r})'.format(self)

    def __str__(self):
        return '({0.x!s}, {0.y!s})'.format(self)

    def method(self):
        return self

p = Pair(3, 4)

print('p is {0!r}'.format(p))
print('p is {0}'.format(p))
print('self is {0}'.format(p.method()))
```

> 上面代码执行结果如下：

```python
p is Pair(3, 4)
p is (3, 4)
self is (3, 4)
```

> 我们看到`return self`默认会调用`__str__()`方法，而当类中没有定义`__str__()`方法的时候，那么它会去调用`__repr__()`方法；如果这两个方法都没有，那么就会输出实例对象的本身，也就是我们上面看到的那样。我们演示以下：

```python
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Pair({0.x!r}, {0.y!r})'.format(self)

    def method(self):
        return self

p = Pair(3, 4)

print('p is {0!r}'.format(p))
print('p is {0!s}'.format(p))
print('self is {0}'.format(p.method()))
```

> 上面代码执行结果如下：

```python
p is Pair(3, 4)
p is Pair(3, 4)
self is Pair(3, 4)
```

> 注意，你可以使用`!s`来指定输出；如`print('p is {0!s}'.format(p))`；
> 当你使用`!s`选项而类中却没有定义`__str__()`方法，那么它会自动去调用`__repr__()`方法；
> 当你使用`!r`选项而类中却没有定义`__repr__()`方法时，它会输出实例对象本身。如下：

```python
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
    def __str__(self):
        return '({0.x!s}, {0.y!s})'.format(self)

    def method(self):
        return self

p = Pair(3, 4)

print('p is {0!r}'.format(p))
print('p is {0!s}'.format(p))
print('self is {0}'.format(p.method()))
```

> 执行结果如下：

```python
p is <__main__.Pair object at 0x7f2e72434b00>
p is (3, 4)
self is (3, 4)
```

> 总结(基于对对象实例的打印)
- `return self`默认会调用类中定义的`__str__()`方法，如果没有此方法，则会调用`__repr__()`方法，如果也没有此方法，那么会输出实例对象本身。
- `format()`方法中的`!r`会指定使用类中的`__repr__()`方法来输出，如果类中没有定义此方法，则会输出实例对象本身。
- `format()`方法中的`!s`会指定使用类中的`__str__()`方法来输出，如果类中没有定义此方法，则会使用`__repr__()`方法来输出，如果也没有此方法，那么会输出实例对象本身。
- 
