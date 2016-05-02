# Two Day

***

### for和while循环的子句else
在`for`循环和`while`循环中，如果循环体遇到了`break`语句，则`for`循环和`while`循环退出后不会执行`else`子句；
如果`for`循环和`while`循环是正常退出的（没有遇到`break`语句），则`else`子句会被执行；

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

for i in range(5):
    print('i => {0}'.format(i))
    if i == 15:
      break
else:
    print('break')
```

执行结果如下：

```
i => 0
i => 1
i => 2
i => 3
i => 4
break
```

现在我们将上面的代码再改成如下：

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

for i in range(5):
    print('i => {0}'.format(i))
    if i == 2:
       break
else:
    print('break')
```

执行结果如下：

```
i => 0
i => 1
i => 2
```

> 注意：`while`循环也是一样的...

> 此前一直不明白循环中的`else`字句是怎么执行的，现在终于董了...

### Python3中的unpack
> 先看个例子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

x, *y = (1,2,3,4,5)
print('x => {0}'.format(x))
print('y => {0}'.format(y))
print()

*a, b = (1,2,3,4,5)
print('a => {0}'.format(a))
print('b => {0}'.format(b))
print()

c, *_, d = (1,2,3,4,5)
print('c => {0}'.format(c))
print('d => {0}'.format(d))
print()

e, *_, f, g = (1,2,3,4,5)
print('e => {0}'.format(e))
print('f => {0}'.format(f))
print('g => {0}'.format(g))
print()

h, (i, j) = (1, (2, 3))
print('h => {0}'.format(h))
print('i => {0}'.format(i))
print('j => {0}'.format(j))
print()

k = 1
l = 2
k, l = l, k
print('k => {0}'.format(k))
print('l => {0}'.format(l))
```

以上代码输出如下：

```
x => 1
y => [2, 3, 4, 5]

a => [1, 2, 3, 4]
b => 5

c => 1
d => 5

e => 1
f => 4
g => 5

h => 1
i => 2
j => 3

k => 2
l => 1
```

> `python3`中的unpack，是不是觉得非常的神奇...

### Python3中的字符串与文本操作

- `Python2`和`Python3`最大的区别就在于字符串
- `Python2`中字符串是`byte`的有序序列
- `Python3`中字符串是`unicode`的有序序列
- 字符串是不可变的
- 字符串支持下标与切片

### 字符串格式化

- print style format

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

print('i love %s' % ('python'))

print('i love %(name)s' % {'name':'python'})

print('i love %(name)s, %(name)s is my first lang.' % {'name':'python'})

print('i love %s, %s is my first lang.' % ('python', 'python'))
```

执行结果如下：

```
i love python
i love python
i love python, python is my first lang.
i love python, python is my first lang.
```

### 字符串常用操作

- `join`字符串连接
- 字符串分割`split`，`rsplit`，`splitlines`，`partition`，`rpartition`
- 字符串修改-大小写 `capitalize`，`title`，`lower`，`upper`，`swapcase`
- 字符串修改-填充清除`center`，`ljust`，`ljust`，`rjust`，`zfill`，`strip`，`rstrip`，`lstrip`
- 字符串判断`startswith`，`endswith`，`is*`
- 字符串查找替换`count`，`find`，`rfind`，`index`，`rindex`，`replace`

### STR与BYTES

- `Python3`中严格区分了文本和二进制数据
- `Python2`并没有严格区分
- 文本数据使用`str`类型，底层实现是`unicode`
- 二进制数据使用`bytes`类型，底层是`byte`
- `str`使用`encode`方法转化为`bytes`
- `bytes`方法使用`decode`方法转化为`str`
- 由于清晰的区分文本和二进制，`Python3`解决了大多数`Python2`的编码问题 

### 将函数作为另一个函数的参数

> 函数的参数可以是另一个函数；如下

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

def sort(cmp, *args):   # 此处的cmp接受的是一个函数名，分别为cmp1和cmp2
    ret = []
    for item in args:
        for i, v in enumerate(ret):
            if cmp(item, v):    # 调用cmp1和cmp2函数进行判断
                ret.insert(i, item)
                break
        else:
            ret.append(item)
    return ret

def cmp1(x, y):
    return x >= y

def cmp2(x, y):
    return x <= y

print('cmp1 => ', sort(cmp1, 3, 1, 2, 5))   # 将cmp1函数名传递过去
print('cmp2 => ', sort(cmp2, 3, 1, 2, 5))   # 将cmp2函数名传递过去
```

执行结果如下：

```
cmp1 =>  [5, 3, 2, 1]
cmp2 =>  [1, 2, 3, 5]
```

### 函数作为返回值

> 如下：

```
__author__ = 'bbk'
# -*- coding:utf-8 -*-

def make_inc(f=1):
    def inc(x):
        return x + f
    return inc

inc1 = make_inc(1)
print('inc1_type => {0}'.format(type(inc1)))
print('inc1(5) => {0}'.format(inc1(5)))
inc2 = make_inc(2)
print('inc2_type => {0}'.format(type(inc2)))
print('inc2(5) => {0}'.format(inc2(5)))
```

执行结果如下：

```
inc1_type => <class 'function'>
inc1(5) => 6
inc2_type => <class 'function'>
inc2(5) => 7
```

> 注意：上面的函数调用可以使用如下简单的方式来调用；

```
make_inc(1)(5)
```

结果依然返回`6`；这被称为`柯里化`；类似于`f(x, y) -> g(x)(y)`
再看一个例子：

```
def bigger(x):
    def inner_bigger(y):
        return y > x
    return inner_bigger

print(list(filter(bigger(5), range(10))))
print(list(filter(bigger(3), range(10))))
```

执行结果如下：

```
[6, 7, 8, 9]
[4, 5, 6, 7, 8, 9]
```

> 说明：第一个`print()`打印出`range(10)`中比`5`大的值；第二个`print()`打印出`range(10)`中比`3`大的值；

> 解析：`bigger(5)`返回的是一个函数`inner_bigger`，而此时`bigger`函数中的`x`参数已经被固定了为`5`；`filter`是`python`中内置过滤函数，接收两个参数，一个是`function or None`，第二个是`iterable`；返回一个迭代对象；此时；`filter`内部应该是`filter(inner_bigger, range(10))`；函数`inner_bigger`接收一个参数，这个参数来自于`range(10)`中的每一个值，而这个函数返回`y > x`的值，存进`filter`内部，而`filter`又返回的是一个可迭代的对象，所以此处使用`list`来获取这个可以迭代对象中的值，获取出来的结果也就是上面执行后的结果；

我们再详细的解剖下上面的函数...

```
def bigger(x):
    def inner_bigger(y):
        return y > x
    return inner_bigger

print(list(filter(bigger(5), range(10))))
print(list(filter(bigger(3), range(10))))

# bigger(5) 返回的是一个函数 inner_bigger，此时，x 参数的值已经被固定为 5，知道下次调用 bigger 函数时才会改变其值
bigger_inner = bigger(5)   

# bigger_inner 也是一个函数，就是 inner_bigger
print('bigger_inner => {0}'.format(bigger_inner))   

# 这里的 bigger_inner 函数就是上面的 inner_bigger 函数；传入参数为 4，所以 y 的值就是 4
print('bigger_inner(4) => {0}'.format(bigger_inner(4)))
print('bigger_inner(6) => {0}'.format(bigger_inner(6)))

# 注意：filter返回的是一个可迭代的对象
print('filter(bigger_inner, range(10)) => {0}'.format(filter(bigger_inner, range(10))))

# 所以这里使用list来获取迭代对象中的值
print('list(filter(bigger_inner, range(10))) => {0}'.format(list(filter(bigger_inner, range(10)))))
```

执行结果如下：

```
[6, 7, 8, 9]
[4, 5, 6, 7, 8, 9]
bigger_inner => <function bigger.<locals>.inner_bigger at 0x00000000021FD9D8>
bigger_inner(4) => False
bigger_inner(6) => True
filter(bigger_inner, range(10)) => <filter object at 0x00000000021FCAC8>
list(filter(bigger_inner, range(10))) => [6, 7, 8, 9]
```

### 减少调用函数时输入的参数个数

> 先看例子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

import pymysql
from functools import partial

# partial 接收一个函数对象，和若干个参数；注意，这些参数将作为形参传递给partial所接收的这个函数
# 并且，partial返回的也是一个函数
connect = partial(pymysql.connect, port=3306, user='root', password='redhat')

# 调用partial返回的这个函数，由于mysql连接没有指定host参数
conn = connect(host='127.0.0.1')

cur = conn.cursor()
conn.select_db('mysql')
cur.execute('select user,host,password from user;')
for i in cur.fetchall():
    print(i)

cur.close()
conn.close()
```

执行结果如下：

```
('root', 'localhost', '*84BB5DF4823DA319BBF86C99624479A198E6EEE9')
('root', '127.0.0.1', '*84BB5DF4823DA319BBF86C99624479A198E6EEE9')
```

> 减少调用函数时输入的参数个数；
我们可以将那些固定不需要变化的参数写入到 partial 中；然后再调用 partial 函数作引用...
以此来减少重复代码.

> 我们再来看一个简单的例子.

```
__author__ = 'Jason tom'
# -*- coding:utf-8 -*-

from functools import partial

def many_args(a, b, c, d, e, f, g):
    print('a is {0}'.format(a))
    print('b is {0}'.format(b))
    print('c is {0}'.format(c))
    print('d is {0}'.format(d))
    print('e is {0}'.format(e))
    print('f is {0}'.format(f))
    print('g is {0}'.format(g))

# 这里传入的参数 e, f, g 都是以默认参数进行传递的，在调用此函数的时候，这些默认参数的值都是可以修改的
args_1 = partial(many_args, e=1, f=2, g=3)

args_1(4, 5, 6, 7)
```

执行结果如下：

```
a is 4
b is 5
c is 6
d is 7
e is 1
f is 2
g is 3
```

> 这种方式在很多场合都是可以用到的....比如上面演示的 mysql 连接....

### 装饰器
