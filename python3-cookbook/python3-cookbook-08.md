# Python3-cookbook-08

### 改变对象的字符串显示

你想改变对象实例的打印或现实输出，让他们更具可读性；你可以重新定义它的`__str__()`和`__repr__()`方法。例如：

```python
class Pair:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return 'Pair({0.x!r}, {0.y!r})'.format(self)

    def __str__(self):
        return '({0.x!s}, {0.y!s})'.format(self)
```

`__repr__()`方法返回一个实例的代码表示形式，通常需要重新构造这个实例。内值的`repr()`函数返回这个字符串，跟我们使用交互式解释器显示的值是一样的。`__str__()`方法将实例转换为一个字符串，使用`str()`或`print()`函数会输出这个字符串。比如：

```python
>>> p = Pair(3, 4)
>>> p
Pair(3, 4) # __repr__() output
>>> print(p)
(3, 4) # __str__() output
>>>
```

我们在这里还演示了在格式化的时候怎样使用不同的字符串表现形式。特别来讲，`!r`格式化代码指明输出使用`__repr__()`来代替默认的`__str__()`。你可以用前面的类来试着测试下：

```python
>>> p = Pair(3, 4)
>>> print('p is {0!r}'.format(p))
p is Pair(3, 4)
>>> print('p is {0}'.format(p))
p is (3, 4)
>>>
```

自定义`__repr__()`和`__str__()`通常是很好的习惯，因为它能简化调试是实例输出。例如，如果仅仅只是打印输出或日志输出某个实例，那么程序员会看到实例更加详细与有用的信息。

`__repr__()`生成的文本字符串标准做法是需要让`eval(repr(x)) == x`为真。如果实在不能这样子做，应该创建一个有用的文本表示，并使用`<`和`>`括起来。比如：

```python
>>> f = open('file.dat')
>>> f
<_io.TextIOWrapper name='file.dat' mode='r' encoding='UTF-8'>
>>>
```

如果`__str__()`没有被定义，那么就会使用`__repr__()`来代替输出。

上面的`format()`方法的使用看上去很有趣，格式化代码`{0.x}`对应的是第一个参数的`x`属性。因此，在下面的函数中，`0`实际上指的就是`self`本身：

```python
def __repr__(self):
    return 'Pair({0.x!r}, {0.y!r})'.format(self)
```

作为这种实现的一个替代，你也可以使用`%`操作符，就像下面这样：

```python
def __repr__(self):
    return 'Pair(%r, %r)' % (self.x, self.y)
```

### 自定义字符串的格式化

为了自定义字符串的格式化，你需要在类上面定义`__format__()`方法。例如：

```python
_formats = {
    'ymd' : '{d.year}-{d.month}-{d.day}',
    'mdy' : '{d.month}/{d.day}/{d.year}',
    'dmy' : '{d.day}/{d.month}/{d.year}'
    }

class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    def __format__(self, code):
        if code == '':
            code = 'ymd'
        fmt = _formats[code]
        return fmt.format(d=self)
```

现在`Date`类的实例可以支持格式化操作了，如同下面这样：

```python
>>> d = Date(2012, 12, 21)
>>> format(d)
'2012-12-21'
>>> format(d, 'mdy')
'12/21/2012'
>>> 'The date is {:ymd}'.format(d)
'The date is 2012-12-21'
>>> 'The date is {:mdy}'.format(d)
'The date is 12/21/2012'
>>>
```

`__format__()`方法给Python的字符串格式化功能提供了一个钩子。这里需要着重强调的是格式化代码的解析工作完全由类自己决定。因此，格式化代码可以是任何值。例如，参考下面来自`datetime`模块中的代码：

```python
>>> from datetime import date
>>> d = date(2012, 12, 21)
>>> format(d)
'2012-12-21'
>>> format(d,'%A, %B %d, %Y')
'Friday, December 21, 2012'
>>> 'The end is {:%d %b %Y}. Goodbye'.format(d)
'The end is 21 Dec 2012. Goodbye'
>>>
```

### 让对象支持上下文管理协议

为了让一个对象兼容`with`语句，你需要实现`__enter__()`和`__exit__()`方法。例如，考虑如下的一个类，它能为我们创建一个网络连接：

```python
from socket import socket, AF_INET, SOCK_STREAM

class LazyConnection:
    def __init__(self, address, family=AF_INET, type=SOCK_STREAM):
        self.address = address
        self.family = family
        self.type = type
        self.sock = None

    def __enter__(self):
        if self.sock is not None:
            raise RuntimeError('Already connected')
        self.sock = socket(self.family, self.type)
        self.sock.connect(self.address)
        return self.sock

    def __exit__(self, exc_ty, exc_val, tb):
        self.sock.close()
        self.sock = None
```

这个类的关键特点在于它表示了一个网络连接，但是初始化的时候并不会做任何事情(比如它没有建立一个连接)。连接的建立和关闭是使用`with`语句自动完成的，例如：

```python
from functools import partial

conn = LazyConnection(('www.python.org', 80))
# Connection closed
with conn as s:
    # conn.__enter__() executes: connection open
    s.send(b'GET /index.html HTTP/1.0\r\n')
    s.send(b'Host: www.python.org\r\n')
    s.send(b'\r\n')
    resp = b''.join(iter(partial(s.recv, 8192), b''))
    # conn.__exit__() executes: connection closed
```

编写上下文管理器的主要原理是你的代码会放到`with`语句块中执行。当出现`with`语句的时候，对象的`__enter__()`方法被触发，它返回的值(如果有的话)会被赋值给`as`声明的变量。然后，`with`语句块里面的代码开始执行。最后，当`with`语句结束时`__exit__()`方法被触发进行清理工作。

不管`with`代码块中发生什么，上面的控制流都会执行完，就算代码块中发生了异常也是一样的。事实上，`__exit__()`方法的三个参数包含了异常类型、异常值和追溯信息(如果有的话)。`__exit__()`方法能自己决定怎样利用这个异常信息，或者忽略它并返回一个None值。如果`__exit__()`返回`True`，那么异常会被清空，就好像什么都没有发生一样，`with`语句后面的程序继续在正常执行。

### 在类中封装属性名

Python程序员不要依赖语言特性去封装数据，而是通过遵循一定的属性和方法命名约定来达到这个效果。第一个约定时任何以单下划线`_`开头的名字都应该是内部实现。比如：

```python
class A:
    def __init__(self):
        self._internal = 0 # An internal attribute
        self.public = 1 # A public attribute

    def public_method(self):
        '''
        A public method
        '''
        pass

    def _internal_method(self):
        pass
```

> 注意，Python不会真的阻止别人访问内部名称。但是如果你这么做肯定是不好的，可能会导致脆弱的代码。同时还要注意到，使用下划线开头的约定同样适用于模块名和模块级别函数。例如，如果你看到某个模块名以单下划线开头(比如`_socket`)，那它就是内部实现。类似的，模块级别函数比如`sys._getframe()`在使用的时候就得加倍小心了。

你可能还遇到在类定义中使用两个下划线(`__`)开头的命名。比如：

```python
class B:
    def __init__(self):
        self.__private = 0

    def __private_method(self):
        pass

    def public_method(self):
        pass
        self.__private_method()
```

使用双下划线开始会导致访问名称变成其它形式。比如，在前面的类B中，私有属性会被分别重命名为`_B__private`和`_B__private_method`。这时候你可能会问这样重命名的目的是什么，答案就是继承；这种属性通过继承是无法被覆盖的。比如：

```python
class C(B):
    def __init__(self):
        super().__init__()
        self.__private = 1 # Does not override B.__private

    # Does not override B.__private_method()
    def __private_method(self):
        pass
```

这里，私有名称`__private`和`__private_method`被重命名为`_C__private`和`_C__private_method`，这个跟父类B中的名称是完全不同的。

> 注意，上面提到由两种不同的编码约定(单下划线和双下划线)来命名私有属性，那么问题就来了，到底哪种方式好呢？大多数而言，你应该让你的非公共名称以单下划线开头。但是，如果你清除你的代码会涉及到子类，并且有些内部属性应该在子类中隐藏起来，那么才考虑使用双下划线方案。

### 创建可管理的属性

自定义某个属性的一种简单方法是将它定义为一个`property`。例如，下面的代码定义了一个`property`，增加对一个属性简单的类型检查：

```python
class Person:
    def __init__(self, first_name):
        self.first_name = first_name

    # Getter function
    @property
    def first_name(self):
        return self._first_name

    # Setter function
    @first_name.setter
    def first_name(self, value):
        if not isinstance(value, str):
            raise TypeError('Expected a string')
        self._first_name = value

    # Deleter function (optional)
    @first_name.deleter
    def first_name(self):
        raise AttributeError("Can't delete attribute")
```

上述代码中由三个相关联的方法，这三个方法的名字都必须一样。第一个方法是一个`getter`函数，它使得`first_name`成为一个属性。其它两个方法给`first_name`属性添加了`setter`和`deleter`函数。需要强调的是，只有在`first_name`属性被创建后，后面的两个装饰器`@first_name.setter`和`@first_name.deleter`才能被定义。

> 注意，`property`的一个关键特征是它看上去跟普通的attribute没什么两样，但是访问它的时候会自动触发`getter`、`setter`和`deleter`方法。例如：

```python
>>> a = Person('Guido')
>>> a.first_name # Calls the getter
'Guido'
>>> a.first_name = 42 # Calls the setter
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "prop.py", line 14, in first_name
        raise TypeError('Expected a string')
TypeError: Expected a string
>>> del a.first_name
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
AttributeError: can`t delete attribute
>>>
```

当然，你也可以直接使用`setattr`和`getattr`以及`deleter`方法。如下：

```python
>>> a = Person('Guido')
>>> print(a.first_name)
Guido
>>> setattr(a, 'first_name', 'jason')
>>> print(getattr(a, 'first_name'))
jason
```

在实现一个property的时候，底层数据(如果有的话)仍然需要存储在某个地方。因此，在`get`和`set`方法中，你会看到对`_first_name`属性的操作，这也是实际数据保存的地方。另外，你可能还会问为什么`__init__()`方法中设置了`self.first_name`而不是`self._first_name`。在这个例子中，我们创建一个property的目的就是在设置attribute的时候进行检查。因此，你可能想在初始化的时候也进行这种类型检查，通过设置`self.first_name`，自动调用`setter`方法，这个方法里面会进行参数的检查，否则就是直接访问`self._first_name`了。

一个property属性其实就是一系列相关绑定方法的集合。如果你去查看拥有property的类，就会发现property本身的get、gset和fdel属性就是类里面的普通方法。比如：

```python
>>> Person.first_name.fget
<function Person.first_name at 0x1006a60e0>
>>> Person.first_name.fset
<function Person.first_name at 0x1006a6170>
>>> Person.first_name.fdel
<function Person.first_name at 0x1006a62e0>
>>>
```

### 调用父类的方法

为了调用父类(超类)的一个方法，可以使用`super()`函数，比如：

```python
class A:
    def spam(self):
        print('A.spam')

class B(A):
    def spam(self):
        print('B.spam')
        super().spam()  # Call parent spam()
```

`super()`函数的一个常见用法是在`__init__()`方法中确保父类被正确的初始化了：

```python
class A:
    def __init__(self):
        self.x = 0

class B(A):
    def __init__(self):
        super().__init__()
        self.y = 1
```

`super()`的另外一个常见用法是出现在覆盖Python特殊方法的代码中，比如：

```python
class Proxy:
    def __init__(self, obj):
        self._obj = obj

    # Delegate attribute lookup to internal obj
    def __getattr__(self, name):
        return getattr(self._obj, name)

    # Delegate attribute assignment
    def __setattr__(self, name, value):
        if name.startswith('_'):
            super().__setattr__(name, value) # Call original __setattr__
        else:
            setattr(self._obj, name, value)
```

在上面代码中，`__setattr__()`的实现包含一个名字检查。如果某个属性名以下划线`_`开头，就通过`super()`调用原始的`__setattr__()`，否则的话就委派给内部的代理对象`self._obj`去处理。这看上去有点意思，因为就算没有显示的指明某个类的父类，`super()`仍然可以有效的工作。

### 创建新的类或实例属性

如果你想创建一个全新的实例属性，可以通过一个描述器类的形式来定义它的功能。下面是一个例子：

```python
# Descriptor attribute for an integer type-checked attribute
class Integer:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, int):
            raise TypeError('Expected an int')
        instance.__dict__[self.name] = value

    def __delete__(self, instance):
        del instance.__dict__[self.name]
```

> 注意，一个描述器就是一个实现了三个核心的属性访问操作(get, set, delete)的类，分别为`__get__()`、`__set__()`和`__delete__()`这三个特殊的方法。这些方法接收一个实例作为输入，之后相应的操作实例底层的字典。

为了使用一个描述器，需将这个描述器的实例作为类属性放到一个类的定义中。例如：

```python
class Point:
    x = Integer('x')
    y = Integer('y')

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

当你这样做后，所有对描述器属性(比如`x`或`y`)的访问会被`__get__()`、`__set__()`和`__delete__()`方法捕获到。例如：

```python
>>> p = Point(2, 3)
>>> p.x # Calls Point.x.__get__(p,Point)
2
>>> p.y = 5 # Calls Point.y.__set__(p, 5)
>>> p.x = 2.3 # Calls Point.x.__set__(p, 2.3)
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "descrip.py", line 12, in __set__
        raise TypeError('Expected an int')
TypeError: Expected an int
>>>
```

作为输入，描述器的每一个方法会接受一个操作实例。为了实现请求操作，会相应的操作实例底层的字典(__dict__属性)。描述器的`self.name`属性存储在实例字典中被实际使用到的`key`。

描述器可实现大部分Python类特性中的底层魔法，包括`@classmethod`、`@staticmethod`、`@property`，甚至是`__slots__`特性。

描述器的一个比较困惑的地方是它只能在类级别被定义，而不能为每个实例单独定义。因此，下面的代码是无法工作的：

```python
# Does NOT work
class Point:
    def __init__(self, x, y):
        self.x = Integer('x') # No! Must be a class variable
        self.y = Integer('y')
        self.x = x
        self.y = y
```

同时，`__get__()`方法实现起来比看上去要复杂的多：

```python
# Descriptor attribute for an integer type-checked attribute
class Integer:

    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            return instance.__dict__[self.name]
```

`__get__()`看上去有点复杂的原因归结于实例变量和类变量的不同。如果一个描述器被当作一个类变量来访问，那么`instance`参数被设置成`None`。这种情况下，标准做法就是简单的返回这个描述器本身即可(尽管你还可以添加其它的自定义操作)。例如：

```python
>>> p = Point(2,3)
>>> p.x # Calls Point.x.__get__(p, Point)
2
>>> Point.x # Calls Point.x.__get__(None, Point)
<__main__.Integer object at 0x100671890>
>>>
```

描述器通常是那些使用到装饰器或元类的大型框架中的一个组件。同时他们的使用也被隐藏在后面。举个例子，下面是一些更高级的基于描述器的代码，并涉及到一个类装饰器：

```python
# Descriptor for a type-checked attribute
class Typed:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type
    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError('Expected ' + str(self.expected_type))
        instance.__dict__[self.name] = value
    def __delete__(self, instance):
        del instance.__dict__[self.name]

# Class decorator that applies it to selected attributes
def typeassert(**kwargs):
    def decorate(cls):
        for name, expected_type in kwargs.items():
            # Attach a Typed descriptor to the class
            setattr(cls, name, Typed(name, expected_type))
        return cls
    return decorate

# Example use
@typeassert(name=str, shares=int, price=float)
class Stock:
    def __init__(self, name, shares, price):
        self.name = name
        self.shares = shares
        self.price = price
```

### 实现自定义容器

`collections`定义了很多抽象基类，当你想自定义容器类的时候他们会非常有用。比如你想让你的类支持迭代，那就让你的类继承`collections.Iterable`即可：

```python
import collections
class A(collections.Iterable):
    pass
```

> 注意，你需要实现`collections.Iterable`所有的抽象方法，否则会报错：

```python
>>> a = A()
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class A with abstract methods __iter__
>>>
```

你只要实现`__iter__()`方法就不会报错了。

你可以先试着去实例化一个对象，在错误提示中可以找到需要实现哪些方法：

```python
>>> import collections
>>> collections.Sequence()
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class Sequence with abstract methods \
__getitem__, __len__
>>>
```

下面是一个简单的示例，继承自上面的`Sequence`抽象类，并且实现元素按照顺序存储：

```python
import collections
import bisect

class SortedItems(collections.Sequence):
    def __init__(self, initial=None):
        self._items = sorted(initial) if initial is not None else []

    # Required sequence methods
    def __getitem__(self, index):
        return self._items[index]

    def __len__(self):
        return len(self._items)

    # Method for adding an item in the right location
    def add(self, item):
        bisect.insort(self._items, item)


items = SortedItems([5, 1, 3])
print(list(items))
print(items[0], items[-1])
items.add(2)
print(list(items))
print(items.__getitem__(2))
```

上面执行后结果为：

```python
[1, 3, 5]
1 5
[1, 2, 3, 5]
3
```

可以看到，`Sortedltems`跟普通的序列没什么两样，支持所有常用操作，包括索引、迭代、包含判断，甚至是切片操作。

这里使用到了`bisect`模块，它是一个在排序列表中插入元素的高效方式。可以保证元素插入后还保持顺序。

使用`collections`中的抽象基类可以确保你自定义的容器实现了所有必要的方法。并且还能简化类型检查。你的自定义容器会满足大部分类型检查需要，如下所示：

```python
>>> items = SortedItems()
>>> import collections
>>> isinstance(items, collections.Iterable)
True
>>> isinstance(items, collections.Sequence)
True
>>> isinstance(items, collections.Container)
True
>>> isinstance(items, collections.Sized)
True
>>> isinstance(items, collections.Mapping)
False
>>>
```

`collections`中很多抽象类会为一些常见容器操作提供默认的实现,这样一来你只需要实现那些你最感兴趣的方法即可.假设你的类继承自`collections.MutableSequence`,如下:

```python
class Items(collections.MutableSequence):
    def __init__(self, initial=None):
        self._items = list(initial) if initial is not None else []

    # Required sequence methods
    def __getitem__(self, index):
        print('Getting:', index)
        return self._items[index]

    def __setitem__(self, index, value):
        print('Setting:', index, value)
        self._items[index] = value

    def __delitem__(self, index):
        print('Deleting:', index)
        del self._items[index]

    def insert(self, index, value):
        print('Inserting:', index, value)
        self._items.insert(index, value)

    def __len__(self):
        print('Len')
        return len(self._items)
```

如果你創建`Items`的实例,你会发现它支持几乎所有的核心列表方法(如append()、remove()、count()等)。下面是使用演示：

```python
>>> a = Items([1, 2, 3])
>>> len(a)
Len
3
>>> a.append(4)
Len
Inserting: 3 4
>>> a.append(2)
Len
Inserting: 4 2
>>> a.count(2)
Getting: 0
Getting: 1
Getting: 2
Getting: 3
Getting: 4
Getting: 5
2
>>> a.remove(3)
Getting: 0
Getting: 1
Getting: 2
Deleting: 2
>>>
```

### 属性的代理访问

你想将某个实例的属性访问代理到内部另一个实例中去，目的可能是作为继承的一个替代方法或者实现代理模式。简单来说，代理是一种编程模式，它将某个操作转移给另外一个对象来实现。最简单的形式可能是像下面这样：

```python
class A:
    def spam(self, x):
        pass

    def foo(self):
        pass


class B1:
    """简单的代理"""

    def __init__(self):
        self._a = A()

    def spam(self, x):
        # Delegate to the internal self._a instance
        return self._a.spam(x)

    def foo(self):
        # Delegate to the internal self._a instance
        return self._a.foo()

    def bar(self):
        pass
```

如果仅仅就两个方法需要代理，那么像这样写就足够了。但是，如果有大量的方法需要代理，那么使用`__getattr__()`方法或许更好些：

```python
class B2:
    """使用__getattr__的代理，代理方法比较多时候"""

    def __init__(self):
        self._a = A()

    def bar(self):
        pass

    # Expose all of the methods defined on class A
    def __getattr__(self, name):
        """这个方法在访问的attribute不存在的时候被调用
        the __getattr__() method is actually a fallback method
        that only gets called when an attribute is not found"""
        return getattr(self._a, name)
```

`__getattr__`方法是在访问attribute不存在的时候被调用，使用演示：

```python
b = B1()
b.bar() # Calls B1.bar() (exists on B1)
b.spam(42) # Calls B1.__getattr__('spam') and delegates to A.spam
```

另外一个代理例子是实现代理模式，例如：

```python
# A proxy class that wraps around another object, but
# exposes its public attributes
class Proxy:
    def __init__(self, obj):
        self._obj = obj

    # Delegate attribute lookup to internal obj
    def __getattr__(self, name):
        print('getattr:', name)
        return getattr(self._obj, name)

    # Delegate attribute assignment
    def __setattr__(self, name, value):
        print('name => {0}'.format(name))
        print('value => {0}'.format(value))
        if name.startswith('_'):
            super().__setattr__(name, value)
        else:
            print('setattr:', name, value)
            setattr(self._obj, name, value)

    # Delegate attribute deletion
    def __delattr__(self, name):
        if name.startswith('_'):
            super().__delattr__(name)
        else:
            print('delattr:', name)
            delattr(self._obj, name)
```

使用这个代理类的时候，你只需要用它来包装下其它类即可：

```python
class Spam:
    def __init__(self, x):
        self.x = x

    def bar(self, y):
        print('Spam.bar:', self.x, y)

# Create an instance
s = Spam(2)
# Create a proxy around it
p = Proxy(s)
print('*' * 10)
# Access the proxy
print(p.x)  # Outputs 2
print('*' * 10)
p.bar(3)  # Outputs "Spam.bar: 2 3"
print('*' * 10)
p.x = 37  # Changes s.x to 37
print('*' * 10)
p.z = 30
print('*' * 10)
print(p.z)
```

依次输出：

```python
name => _obj
value => <test.Spam object at 0x7f4be17e2f60>
**********
getattr: x
2
**********
getattr: bar
Spam.bar: 2 3
**********
name => x
value => 37
setattr: x 37
**********
name => z
value => 30
setattr: z 30
**********
getattr: z
30
```

再看一个简单点的例子：

```python
class A:
    def spam(self, x):
        print('A.spam', x)
    def foo(self):
        print('A.foo')

class B:
    def __init__(self):
        self._a = A()
    def spam(self, x):
        print('B.spam', x)
        self._a.spam(x)
    def bar(self):
        print('B.bar')
    def __getattr__(self, name):
        return getattr(self._a, name)

a = B()
a.foo()	# Output 'A.foo'
```

通过自定义属性访问方法，你可以用不同方式自定义代理类行为(比如加入日志功能、只读访问等)。

> 注意，代理类有时候可以作为继承的替代方案。

当实现代理模式时，还有写细节需要注意。首先，`__getattr__()`实际是一个后备方法，只有在属性不存在时才会调用。因此，如果代理类实例本身有这个属性的话，那么不会触发这个方法的。另外，`__setattr__()`和`__delattr__()`需要额外的魔法来区分代理实例和被代理实例`_obj`的属性。一个通常的约定是只代理那些不以下划线`_`开头的属性(代理类只暴露被代理类的公共属性)。

> 还有一点需要注意，`__getattr__()`对于大部分以双下划线`__`开始和结尾的属性并不适用。

### 在类中定义多个构造器

你想实现一个类，除了使用`__init__()`方法外，还有其它方法可以初始化它。为了实现多个构造器。你需要使用到类方法。例如：

```python
import time

class Date:
    """方法一：使用类方法"""
    # Primary constructor
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    # Alternate constructor
    @classmethod
    def today(cls):
        t = time.localtime()
        return cls(t.tm_year, t.tm_mon, t.tm_mday)
```

直接调用类方法即可，下面是使用示例：

```python
a = Date(2012, 12, 21) # Primary
b = Date.today() # Alternate
```

类方法的一个主要用途就是定义多个构造器。它接受一个`class`作为第一个参数(cls)。你应该注意到了这个类被用来创建并返回最终的实例。

```python
print(b.day, b.month, b.year)	# Output 24 6 2016; It's today.
```
