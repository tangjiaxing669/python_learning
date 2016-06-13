# Python3-cookbook-01

```seq
Title: Here is a title
A->B: Normal line
B-->C: Dashed line
C->>D: Open arrow
D-->>A: Dashed open arrow
```

### 解压序列赋值给多个变量

代码示例：

```python
>>> p = (4, 5)
>>> x, y = p
>>> x
4
>>> y
5
>>>
>>> data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
>>> name, shares, price, date = data
>>> name
'ACME'
>>> date
(2012, 12, 21)
>>> name, shares, price, (year, mon, day) = data
>>> name
'ACME'
>>> year
2012
>>> mon
12
>>> day
21
>>>
```

> 如果变量个数和序列元素的个数不匹配，会产生一个异常。

示例代码：

```python
>>> p = (4, 5)
>>> x, y, z = p
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
ValueError: need more than 2 values to unpack
>>>
```

> 实际上，这种解压赋值可以用在任何可迭代对象上面，而不仅仅是列表或者元组。包括字符串，文件对象，迭代器和生成器。

代码示例：

```python
>>> s = 'Hello'
>>> a, b, c, d, e = s
>>> a
'H'
>>> b
'e'
>>> e
'o'
>>>
```

> 有时候，你可能只想解压一部分，丢弃其他的值；对于这中情况Python并没有提供特殊的语法。但是你可以使用任意变量名去占位，到时候丢掉这些变量就行了。

代码示例：

```python
>>> data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
>>> _, shares, price, _ = data
>>> shares
50
>>> price
91.1
>>>
```

> 你必须保证你选用的那些占位变量名在其他地方没被使用到。

> 还有一种情况，假设你现在有一些用户的记录列表，每条记录包含一个名字、邮件，接着就是不确定数量的电话号码。可以像下面这样分解这些记录：

```python
>>> record = ('Dave', 'dave@example.com', '773-555-1212', '847-555-1212')
>>> name, email, *phone_numbers = record
>>> name
'Dave'
>>> email
'dave@example.com'
>>> phone_numbers
['773-555-1212', '847-555-1212']
>>>
```

> 注意，上面解压出的`phone_numbers`变量永远都是列表类型，不管解压的电话号码数量是多少(包括0个)。所以，任何使用到`phone_numbers`变量的代码就不需要作多余的类型检查去确认它是否是列表类型了。

```python
>>> *trailing, current = [10, 8, 7, 1, 9, 5, 10, 3]
>>> trailing
[10, 8, 7, 1, 9, 5, 10]
>>> current
3
```

> 值的注意的是，星号表达是在迭代元素为可变长元组的序列时时很有用的。比如，下面是一个带有标签的元组序列：

```python
records = [
    ('foo', 1, 2),
    ('bar', 'hello'),
    ('foo', 3, 4),
]

def do_foo(x, y):
    print('foo', x, y)

def do_bar(s):
    print('bar', s)

for tag, *args in records:
    if tag == 'foo':
        do_foo(*args)
    elif tag == 'bar':
        do_bar(*args)
```

> 星号同样可以用在字符串的分割中，如下：

```python
>>> line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'
>>> uname, *fields, homedir, sh = line.split(':')
>>> uname
'nobody'
>>> homedir
'/var/empty'
>>> sh
'/usr/bin/false'
>>>
```

> 有时候，你向解压一些元素后丢弃他们，你不能简单就使用`*`，但是你可以使用一个普通的废弃名称，比如`_`或者`ign`。

代码示例：
```python
>>> record = ('ACME', 50, 123.45, (12, 18, 2012))
>>> name, *_, (*_, year) = record
>>> name
'ACME'
>>> year
2012
>>>
```

### 保留最后N个元素`collections.deque`

> 使用`deque(maxlen=N)`构造函数会新建一个固定大小的队列。当新的元素加入并且这个队列已满的时候，最老的元素会自动被移除掉。

代码示例：

```python
>>> from collections import deque
>>> q = deque(maxlen=3)
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3], maxlen=3)
>>> q.append(4)
>>> q
deque([2, 3, 4], maxlen=3)
>>> q.append(5)
>>> q
deque([3, 4, 5], maxlen=3)
```

> 注意，`deque`类可以被用在任何你只需要一个简单队列数据结构的场合。如果你不设置最大队列大小，那么就会得到一个无限大小队列，你可以在队列的两端执行添加和弹出元素的操作。

代码示例：

```python
>>> q = deque()
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3])
>>> q.appendleft(4)
>>> q
deque([4, 1, 2, 3])
>>> q.pop()
3
>>> q
deque([4, 1, 2])
>>> q.popleft()
4
```

> 注意，在队列两端插入或删除元素时间复杂度都是`0(1)`，而在列表的开头插入或删除元素的时间复杂度为`0(N)`。

### 字典中键映射多个值 collections.defaultdict

> 怎么实现一个键对应多个值的字典(也叫`multidict`)？

> 你可以很方便的使用`collections`模块中的`defaultdict`来构造这样的字典。`defaultdict`的一个特征时它会自动初始化每个`key`刚开始对应的值，所以你只需要关注添加元素操作就可以了。比如：

```python
from collections import defaultdict

d = defaultdict(list)
d['a'].append(1)
d['a'].append(2)
d['b'].append(4)

d = defaultdict(set)
d['a'].add(1)
d['a'].add(2)
d['b'].add(4)
```

> 注意，选择使用列表还是集和取决于你的实际需求。如果你向保持元素的插入顺序就应该使用列表，如果你想去掉重复元素就使用集和(并且不需要关心元素的顺序问题)。

### 字典排序

> 你想创建一个字典，并且在迭代或序列化这个字典的时候能够控制元素的顺序。

> 为了能控制一个字典中元素的顺序，你可以使用`collections`模块中的`OrderedDict`类。在迭代操作的时候它会保持元素被插入时的顺序，示例如下：

```python
from collections import OrderedDict
def ordered_dict():
    d = OrderedDict()
    d['foo'] = 1
    d['bar'] = 2
    d['spam'] = 3
    d['grok'] = 4
    # Outputs "foo 1", "bar 2", "spam 3", "grok 4"
    for key in d:
        print(key, d[key])
```

> 当你想要构建一个将来需要序列化或编码成其它个时的映射的时候，`OrderedDict`是非常有用的。比如，你想精确控制以JSON编码后子段的顺序，你可以先使用`OrderedDict`来构建这样的数据：

```python
>>> import json
>>> json.dumps(d)
'{"foo": 1, "bar": 2, "spam": 3, "grok": 4}'
>>>
```

### 字典的运算

> 怎样在数据字典红执行一些计算操作（比如求最小值、最大值、排序等等）？

> 为了对字典值执行计算操作，通常需要使用`zip()`函数先将键和值反转过来。比如，下面是查找最小值和最大值：

```python
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}

min_price = min(zip(prices.values(), prices.keys()))
# min_price is (10.75, 'FB')
max_price = max(zip(prices.values(), prices.keys()))
# max_price is (612.78, 'AAPL')
```

类似的，可以使用`sorted()`函数来沛旭字典数据：

```python
prices_sorted = sorted(zip(prices.values(), prices.keys()))
# prices_sorted is [(10.75, 'FB'), (37.2, 'HPQ'),
#                   (45.23, 'ACME'), (205.55, 'IBM'),
#                   (612.78, 'AAPL')]
```

> 需要注意的是，在`Python3`中，`zip()`函数创建的是一个只能访问一次的迭代器。比如，下面的代码就会产生错误：

```python
prices_and_names = zip(prices.values(), prices.keys())
print(min(prices_and_names)) # OK
print(max(prices_and_names)) # ValueError: max() arg is an empty sequence
```

> 另一个方法，你可以在`min()`和`max()`函数中提供`key`函数参数来获取最小值或最大值对应的键的信息。比如：

```python
min(prices, key=lambda k: prices[k]) # Returns 'FB'
max(prices, key=lambda k: prices[k]) # Returns 'AAPL'
```

但是，如果还想要得到最小值，你又得执行一次查找操作，比如：

```python
min_value = prices[min(prices, key=lambda k: prices[k])]
```

> 注意，当多个实体拥有相同的值的时候，键会决定返回结果。比如，在执行`min()`和`max()`操作的时候，如果恰巧最小或最大值有重复，那么拥有最小或最大键的实体会返回：

```python
>>> prices = { 'AAA' : 45.23, 'ZZZ': 45.23 }
>>> min(zip(prices.values(), prices.keys()))
(45.23, 'AAA')
>>> max(zip(prices.values(), prices.keys()))
(45.23, 'ZZZ')
>>>
```
