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
