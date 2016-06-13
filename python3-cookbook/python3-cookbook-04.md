# Python3-cookbook-04

### 代理迭代

> 你构建了一个自定义容器对象，里面包含有列表、元组或其它可迭代对象。你想直接在你的这个新容器对象上执行迭代操作。
> 实际上你只需要定义一个`__iter__()`方法，将迭代操作代理到容器内部的对象上去。比如：

```python
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

# Example
if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    # Outputs Node(1), Node(2)
    for ch in root:
        print(ch)
```

> 在上面代码中，`__iter__()`方法只是简单的将迭代请求传递给内部的`_children`属性。
> 而`__repr__()`方法是获取被实例化后对象的内容。
> 上面的代码中，如果你不加`__repr__()`方法，那么执行代码后的结果将会变成如下：

```python
<__main__.Node object at 0x7f0c26eb1f98>
<__main__.Node object at 0x7f0c26eb1fd0>
<__main__.Node object at 0x7f0c26eb1f60>
```

### 实现迭代器协议

> 目前为止，在一个对象上实现迭代最简单的方式是使用一个生成器函数。
> 在上一节中，使用`Node`类来表示树形数据结构。你可能想实现一个以深度优先方式遍历树形节点的生成器。如下：

```python
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

    def depth_first(self):
        yield self
        for c in self:
            yield from c.depth_first()

# Example
if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    child1.add_child(Node(3))
    child1.add_child(Node(4))
    child2.add_child(Node(5))

    for ch in root.depth_first():
        print(ch)
    # Outputs Node(0), Node(1), Node(3), Node(4), Node(2), Node(5)
```

> 在这段代码中，`depth_first()`方法简单直观。它首先返回自己本身并迭代每一个子节点并通过调用子节点的`depth_first()`方法(使用`yield from`语句)返回对应元素。

> `yield self`表示返回对象本身。它的值依赖`__repr__()`方法；如果对象中有定义`__repr__()`方法，那么`yield self`将返回`__repr__()`方法中所定义的值。

> 这里的`yield from`很特别，它允许你从另一个可迭代的对象中`yield`数据，如下：

```python
>>> def g(x):
...     yield from range(x) 
...
>>> list(g(5))
[0, 1, 2, 3, 4]
>>> 
```

另一个例子：

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

### 反向迭代

> 很多程序员并不知道可以通过在自定义类上实现`__reversed__()`方法来实现反向迭代。比如：

```python
class Countdown:
    def __init__(self, start):
        self.start = start

    # Forward iterator
    def __iter__(self):
        n = self.start
        while n > 0:
            yield n
            n -= 1

    # Reverse iterator
    def __reversed__(self):
        n = 1
        while n <= self.start:
            yield n
            n += 1

for rr in reversed(Countdown(30)):
    print(rr)
for rr in Countdown(30):
    print(rr)
```

> 注意，使用内置的`reversed()`函数同样可以实现这种需求，但是，如果可迭代对象元素很多的话，将其预先转换为一个列表要消耗大量的内存。

> 定义一个反向迭代器可以使得代码非常的高效，因为它不再需要将数据填充到一个列表中然后再去反向迭代这个列表。

### 同时迭代多个序列

> 为了同时迭代多个序列，最简单直接的方法就是使用`zip()`函数，比如：

```python
>>> xpts = [1, 5, 4, 2, 10, 7]
>>> ypts = [101, 78, 37, 15, 62, 99]
>>> for x, y in zip(xpts, ypts):
...     print(x,y)
...
1 101
5 78
4 37
2 15
10 62
7 99
>>>
```

> 注意，`zip(a, b)`会生成一个可返回元组`(x, y)`的迭代器，其中 x 来子 a，y 来值 b。一旦其中某个序列到底结尾，迭代宣告结束。因此迭代长度跟参数中最短序列长度一致。

```python
>>> a = [1, 2, 3]
>>> b = ['w', 'x', 'y', 'z']
>>> for i in zip(a,b):
...     print(i)
...
(1, 'w')
(2, 'x')
(3, 'y')
>>>
```

> 如果这不是你想要的效果，那么还可以使用`itertools.zip_longest()`函数来代替。比如：

```python
>>> from itertools import zip_longest
>>> for i in zip_longest(a,b):
...     print(i)
...
(1, 'w')
(2, 'x')
(3, 'y')
(None, 'z')

>>> for i in zip_longest(a, b, fillvalue=0):
...     print(i)
...
(1, 'w')
(2, 'x')
(3, 'y')
(0, 'z')
>>>
```

> 最后强调一点就是，`zip()`会创建一个迭代器来作为结果返回。如果你需要将结对的值存储在列表中，要使用`list()`函数，比如：

```python
>>> zip(a, b)
<zip object at 0x1007001b8>
>>> list(zip(a, b))
[(1, 10), (2, 11), (3, 12)]
>>>
```

> 注意，`zip()`函数生成是一个一次性的迭代器。

```python
>>> a = [1,2,3]
>>> b = ['w', 'x', 'y', 'z']
>>> c = zip(a, b)
>>> print(list(c))
[(1, 'w'), (2, 'x'), (3, 'y')]
>>> print(list(c))
[ ]
```

### 不同集合上元素的迭代

> `itertools.chain()`方法可以用来简化这个任务。它接受一个可迭代对象列表作为输入，并返回一个迭代器，有效的屏蔽掉爱多个容器中迭代细节。

```python
>>> from itertools import chain
>>> a = [1, 2, 3, 4]
>>> b = ['x', 'y', 'z']
>>> for x in chain(a, b):
... print(x)
...
1
2
3
4
x
y
z
>>>
```

> `itertools.chain()`接受一个或多个可迭代对象作为输入参数。然后创建一个迭代器，依次连续的返回每个可迭代对象中的元素。这种方式要比先将序列合并再迭代要高效的多。

### 迭代器代器while无限循环

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

> `iter`函数一个鲜为人知的特性是它接受一个可选的`callable`对象和一个标记(结尾)值作为输入参数。当以这种方式使用的时候，它会创建一个迭代器，这个迭代器会不断调用`callable`对象知道返回值和标记值像等为止。

> 这种特殊的方法对于一些特定的会被重复调用的函数很有效果，比如涉及到I/O调用的函数。举例来讲，如果你想从套接字或文件中以数据块的方式读取数据，通常你得要不断重复的执行`read()`或`recv()`，并再后面紧跟一个文件结尾测试来决定是否终止。这节中使用一个简单的`iter()`调用就可以将两者结合起来了。其中`lambda`函数参数是为了创建一个无参的`callable`对象，并为`recv`或`read()`方法提供了`size`参数。
