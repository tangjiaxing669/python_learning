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
> 先看几个列子...

```
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
__author__ = "Jason Tom"

x, *y = (1,2,3,4,5)

print('x => {0}'.format(x))
print('y => {0}'.format(y))
```

执行结果如下：

```
x => 1
y => [2, 3, 4, 5]
```

***

