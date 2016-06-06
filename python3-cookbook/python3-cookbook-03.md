# Python3-cookbook-03

### 二八十六进制整数

> 注意，使用八进制的程序员有一点需要注意下。Python指定八进制数的语法跟其他语言稍有不同。比如，如果你像下面这样指定八进制，会出现语法错误：

```
>>> import os
>>> os.chmod('script.py', 0755)
    File "<stdin>", line 1
        os.chmod('script.py', 0755)
                            ^
SyntaxError: invalid token
>>>
```

> 需确保八进制数的前缀时`0o`，就像下面这样：

```
>>> os.chmod('script.py', 0o755)
>>>
```

> 注意，在`Python2`中无需关心此问题。

### 随即选择

> 你想从一个序列中随即抽取若干元素，或者想生成几个随机数。

> `random`模块有大量的函数用来产生随机数和随机选择元素。比如，你想从一个序列中随机的抽取一个元素，可以使用`random.choice()`：

```
>>> import random
>>> values = [1, 2, 3, 4, 5, 6]
>>> random.choice(values)
2
>>> random.choice(values)
3
>>> random.choice(values)
1
>>> random.choice(values)
4
>>> random.choice(values)
6
>>>
```

> 为了提取N个不同元素的样本用来作进一步的操作，可以使用`random.sample()`:

```
>>> random.sample(values, 2)
[6, 2]
>>> random.sample(values, 2)
[4, 3]
>>> random.sample(values, 3)
[4, 3, 1]
>>> random.sample(values, 3)
[5, 4, 1]
>>>
```

> 如果你仅仅只是想打乱序列中元素的顺序，可以使用`random.shuffle()`：

```
>>> random.shuffle(values)
>>> values
[2, 4, 6, 5, 3, 1]
>>> random.shuffle(values)
>>> values
[3, 5, 2, 1, 6, 4]
>>>
```

> 生成随机整数，请使用`random.randint()`：

```
>>> random.randint(0,10)
2
>>> random.randint(0,10)
5
>>> random.randint(0,10)
0
>>> random.randint(0,10)
7
>>> random.randint(0,10)
10
>>> random.randint(0,10)
3
>>>
```

> 为了生成`0`到`1`范围内均匀分布的浮点数，使用`random.random()`：

```
>>> random.random()
0.9406677561675867
>>> random.random()
0.133129581343897
>>> random.random()
0.4144991136919316
>>>
```

> 如果要获取N位随机位(二进制)的整数，使用`random.getrandbits()`：

```
>>> random.getrandbits(200)
335837000776573622800628485064121869519521710558559406913275
>>>
```

### 计算当前月份的日期范围

> 使用生成器来为日期迭代创建一个同内置的`range()`函数：

```
def date_range(start, stop, step):
    while start < stop:
        yield start
        start += step
```

> 下面是使用这个生成器的例子：

```
>>> for d in date_range(datetime(2012, 9, 1), datetime(2012,10,1),
                        timedelta(hours=6)):
...     print(d)
...
2012-09-01 00:00:00
2012-09-01 06:00:00
2012-09-01 12:00:00
2012-09-01 18:00:00
2012-09-02 00:00:00
2012-09-02 06:00:00
...
>>>
```
