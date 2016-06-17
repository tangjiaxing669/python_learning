# Python3-cookbook-07

### 可接受任意数量参数的函数

> 为了能让一个函数接收任意数量的位置参数，可以使用一个`*参数`；

> 为了接收人物数量的关键字参数，使用一个以`**`开头的参数；

```python
def anyargs(*args, **kwargs):
    print(args) # A tuple
    print(kwargs) # A dict
```

> 使用这个函数时，所有位置参数会被放到args元组中，所有关键字参数会被放到字典kwargs中。

> 一个`*`参数只能出现在函数定义中最后一个位置参数后面，而`**`参数只能出现在最后一个参数。有一点需要注意的是，在`*`参数后面仍然可以定义其它参数。

```python
def a(x, *args, y):
    pass

def b(x, *args, y, **kwargs):
    pass
```

> 这中参数就是我们所说的强制关键字参数。

### 只接受关键字参数的函数

> 你希望函数的某些参数强制使用关键字参数传递

> 将强制关键字参数放到某个`*`参数后者单个`*`后面就能达到这中效果。比如：

```python
def recv(maxsize, *, block):
    'Receives a message'
    pass

recv(1024, True) # TypeError
recv(1024, block=True) # Ok
```

> 利用这中技术，我们还能在接收任意多个位置参数的函数中指定关键字参数。比如：

```python
def mininum(*values, clip=None):
    m = min(values)
    if clip is not None:
        m = clip if clip > m else m
    return m

minimum(1, 5, 2, -5, 10) # Returns -5
minimum(1, 5, 2, -5, 10, clip=0) # Returns 0
```

### 给函数参数增加元信息

> 你写好了一个函数，然后想为这个函数的参数增加一些额外的信息，这样的话其它使用者就能清楚的直到这个函数应该则么使用。

> 使用函数参数注解是一个很好的办法，它能提示程序员应该怎样正确使用这个函数。例如：下面有一个被注解了的函数：

```python
def add(x:int, y:int) -> int:
    return x + y
```

> Python解释器不会对这些注解添加任何的语义。它们不会被类型检查，运行时跟没有加注解之前的效果也没有任何差距。 然而，对于那些阅读源码的人来讲就很有帮助啦。第三方工具和框架可能会对这些注解添加语义。同时它们也会出现在文档中。

### 返回多个值的函数

> 为了能返回多个值，函数直接return一个元组就行了。例如：

```python
>>> def myfun():
... return 1, 2, 3
...
>>> a, b, c = myfun()
>>> a
1
>>> b
2
>>> c
3
```

> 尽管`myfun()`看上去返回了多个值，实际上是先创建了一个元组然后返回的。这个语法看上去比较奇怪，实际上我们使用的是逗号来生成一个元组，而不是用括号。比如下面的：

```python
>>> a = (1, 2) # With parentheses
>>> a
(1, 2)
>>> b = 1, 2 # Without parentheses
>>> b
(1, 2)
>>>
```

### 定义有默认参数的函数

> 定义一个有可选参数的函数是非常简单的，直接在函数定义中给参数指定一个默认值，并放到参数列表最后就行了。例如：

```python
def spam(a, b=42):
    print(a, b)

spam(1) # Ok. a=1, b=42
spam(1, 2) # Ok. a=1, b=2
```

> 如果默认参数是一个可修改的容器，比如一个列表、集合或者字典，可以使用None作为默认值，就像下面这样：

```python
# Using a list as a default value
def spam(a, b=None):
    if b is None:
        b = []
    ...
```

> 如果你不想提供一个默认值，而是想仅仅测试下某个参数是不是有传递进来，可以像下面这样写：

```python
_no_value = object()

def spam(a, b=_no_value):
    if b is _no_value:
        print('No b value supplied')
    ...
```

> 我们测试下这个函数：

```python
>>> spam(1)
No b value supplied
>>> spam(1, 2) # b = 2
>>> spam(1, None) # b = None
>>>
```

> 仔细观察可以发现到传递一个None值和不传值两种情况是有差别的。

> 注意，默认参数的值仅仅在函数定义的时候赋值一次。试着运行下面这个列子：

```python
>>> x = 42
>>> def spam(a, b=x):
...     print(a, b)
...
>>> spam(1)
1 42
>>> x = 23 # Has no effect
>>> spam(1)
1 42
>>>
```

> 注意到当我们改变x的值的时候对默认参数值并没有影响，这是因为在函数定义的时候就已经确定了它的默认值了。
> 其次，默认参数的值应该是不可变的对象，比如None、True、False、数字或字符串。特别的，千万不要像下面这样写代码：

```python
def spam(a, b=[]): # NO!
    ...
```

> 如果你这么做了，当默认值在其它地方被修改后你将会遇到各种麻烦。这些修改会影响到下次调用这个函数时的默认值。比如：

```python
>>> def spam(a, b=[]):
...     print(b)
...     return b
...
>>> x = spam(1)
>>> x
[]
>>> x.append(99)
>>> x.append('Yow!')
>>> x
[99, 'Yow!']
>>> spam(1) # Modified list gets returned!
[99, 'Yow!']
>>>
```

> 最后一个问题比较微妙，那就是一个函数需要测试某个可选参数是否被使用者传递进来。 这时候需要小心的是你不能用某个默认值比如None、 0或者False值来测试用户提供的值(因为这些值都是合法的值，是可能被用户传递进来的)。 因此，你需要其他的解决方案了。

> 为了解决这个问题，你可以创建一个独一无二的私有对象实例，就像上面的_no_value变量那样。 在函数里面，你可以通过检查被传递参数值跟这个实例是否一样来判断。 这里的思路是用户不可能去传递这个_no_value实例作为输入。 因此，这里通过检查这个值就能确定某个参数是否被传递进来了。

### 定义匿名或内联函数

> 当一些函数很简单，仅仅值时计算一个表达式的值的时候，就可以使用`lambda`表达式来代替了。比如：

```python
>>> add = lambda x, y: x + y
>>> add(2,3)
5
>>> add('hello', 'world')
'helloworld'
>>>
```

> 这里使用的`lambda`表达式跟下面的效果是一样的：

```python
>>> def add(x, y):
...     return x + y
...
>>> add(2,3)
5
>>>
```

> `lambda`表达式典型的使用场景是排序或数据`reduce`等。

```python
>>> names = ['David Beazley', 'Brian Jones',
...         'Raymond Hettinger', 'Ned Batchelder']
>>> sorted(names, key=lambda name: name.split()[-1].lower())
['Ned Batchelder', 'David Beazley', 'Raymond Hettinger', 'Brian Jones']
>>>
```

### 匿名函数捕获变量值

> 你用`lambda`定义了一个匿名函数，并想在定义时捕获到某些变量的值。
> 先看看下面的代码：

```python
>>> x = 10
>>> a = lambda y: x + y
>>> x = 20
>>> b = lambda y: x + y
>>>
```

> 现在问你，`a(10)`和`b(10)`返回的结果是什么？如果你认为结果是`20`和`30`，那么你就错了：

```python
>>> a(10)
30
>>> b(10)
30
>>>
```

> 这其中的奥妙再与`lambda`表达式中的`x`是一个自由变量，在运行时绑定值，而不是定义时就绑定，这个函数的默认值参数定义是不同的(函数的默认值参数是函数定义后，其默认值参数的值就绑定好了)，因此，在调用这个`lambda`表达式的时候，`x`的值是执行时的值。例如：

```python
>>> x = 15
>>> a(10)
25
>>> x = 3
>>> a(10)
13
>>>
```

> 注意，如果你想让某个匿名函数在定义时就捕获到值，可以将那个参数值定义成默认参数即可，就像下面这样：

```python
>>> x = 10
>>> a = lambda y, x=x: x + y
>>> x = 20
>>> b = lambda y, x=x: x + y
>>> a(10)
20
>>> b(10)
30
>>>
```

### 将单方法的类转换为函数

> 你有一个除`__init__()`方法外只定义了一个方法的类。为了简化代码，你想将他转换为一个函数。

> 大多数情况下，可以使用闭包来将单个方法的类转换成函数。举个列子，下面示例中的类允许使用者根据某个模板方案来获取到URL连接地址。

```python
from urllib.request import urlopen

class UrlTemplate:
    def __init__(self, template):
        self.template = template

    def open(self, **kwargs):
        return urlopen(self.template.format_map(kwargs))

# Example use. Download stock data from yahoo
yahoo = UrlTemplate('http://finance.yahoo.com/d/quotes.csv?s={names}&f={fields}')
for line in yahoo.open(names='IBM,AAPL,FB', fields='sl1c1v'):
    print(line.decode('utf-8'))
```

> 这个类可以被一个更简单的函数来代替：

```python
def urltemplate(template):
    def opener(**kwargs):
        return urlopen(template.format_map(kwargs))
    return opener

# Example use
yahoo = urltemplate('http://finance.yahoo.com/d/quotes.csv?s={names}&f={fields}')
for line in yahoo(names='IBM,AAPL,FB', fields='sl1c1v'):
    print(line.decode('utf-8'))
```

> 大部分情况下，你拥有一个单方法类的原因是需要存储某些额外的状态来给方法使用。比如，定义Urlemplate类的唯一目的就是先在某个地方存储模板值，以便将来可以在`open()`方法中使用。

> 使用一个内部函数或者闭包的方案通常会更优雅一些。简单来讲，一个闭包就是一个函数，只不过在函数内部带上了一个额外的变量环境。闭包关键特点就是它会记住自己被定义时的环境。因此，在我们的解决方案中，`opener()`函数记住了`template`参数的值，并在接下来的调用中使用它。

> 任何时候只要你碰到需要给某个函数增加额外的状态信息的问题，都可以考虑使用闭包。相比将你的函数转换成一个类而言，闭包通常是一种更加简洁和优雅的方案。

### 带额外状态信息的回调函数

> 这节主要讨论的是那些出现在很多函数库和框架中的回调函数的使用，特别是跟异步处理有关的。为了演示与测试，我们先定义如下一个需要调用回调函数的函数：

```python
def apply_async(func, args, *, callback):
    # Compute the result
    result = func(*args)

    # Invoke the callback with the result
    callback(result)
```

> 实际上，这段代码可以做任何更高级的处理，包括线程、进程和定时器，但是这些都不是我们要关心的。我们仅仅只需要关注回调函数的调用。下面是一个演示怎样使用上述代码的例子：

```python
>>> def print_result(result):
...     print('Got:', result)
...
>>> def add(x, y):
...     return x + y
...
>>> apply_async(add, (2, 3), callback=print_result)
Got: 5
>>> apply_async(add, ('hello', 'world'), callback=print_result)
Got: helloworld
>>>
```

> 注意到`print_result()`函数仅仅只接受一个参数`result`。不能再传入其它信息。而当你想让回调函数访问其它变量或者特定环境的变量值的时后就会遇到麻烦。

> 为了让回调函数访问外部信息，一种方法是使用一个绑定方法来代替一个简单函数。比如，下面这个类会保存一个内部序列号，每次接收到一个`result`的时后序列号加1：

```python
class ResultHandler:

    def __init__(self):
        self.sequence = 0

    def handler(self, result):
        self.sequence += 1
        print('[{}] Got: {}'.format(self.sequence, result))
```

> 使用这个类的时候，你先创建一个类的实例，然后用它的`handler()`绑定方法来作为回调函数：

```python
>>> r = ResultHandler()
>>> apply_async(add, (2, 3), callback=r.handler)
[1] Got: 5
>>> apply_async(add, ('hello', 'world'), callback=r.handler)
[2] Got: helloworld
>>>
```

> 第二种方式，作为类的代替，可以使用一个闭包捕获状态值，例如：

```python
def make_handler():
    sequence = 0
    def handler(result):
        nonlocal sequence
        sequence += 1
        print('[{}] Got: {}'.format(sequence, result))
    return handler
```

> 下面是使用闭包方式的一个例子：

```python
>>> handler = make_handler()
>>> apply_async(add, (2, 3), callback=handler)
[1] Got: 5
>>> apply_async(add, ('hello', 'world'), callback=handler)
[2] Got: helloworld
>>>
```

> 还有另外一个更高级的方法，可以使用协程来完成同样的事情：

```python
def make_handler():
    sequence = 0
    while True:
        result = yield
        sequence += 1
        print('[{}] Got: {}'.format(sequence, result))
```

> 对于协程，你需要使用它的`send()`方法作为回调函数，如下所示：

```python
>>> handler = make_handler()
>>> next(handler) # Advance to the yield
>>> apply_async(add, (2, 3), callback=handler.send)
[1] Got: 5
>>> apply_async(add, ('hello', 'world'), callback=handler.send)
[2] Got: helloworld
>>>
```

> 基于回调函数通常都有可能变得非常复杂。一部分原因是回调函数通常会跟请求执行代码断开。因此，请求执行和处理结果之间的执行环境实际上已经丢失了。如果你想让回调连续执行多步操作，那你就必须去解决如果保存和恢复相关的状态信息了。

> 注意，如果你使用闭包，你需要注意对那些可修改变量的操作。在上面的反感中，`nonlocal`声明语句用来指示接下来的变量会在回调函数中被修改。如果没有这个声明，代码会报错。

> 如果你仅仅只需要给回调函数传递额外的值的话，还有一种使用`partial()`的方式也很有用。在没有使用`partial()`的时候，你可能经常看到下面这种使用`lambda`表达式复杂代码：

```python
>>> apply_async(add, (2, 3), callback=lambda r: handler(r, seq))
[1] Got: 5
>>>
```
