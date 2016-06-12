# Python3-cookbook-05

### 读写文本数据

> 使用带有`rt`模式的`open()`函数读取文本文件

> 使用带有`wt`模式的`open()`函数写入文本文件，会清除并覆盖掉之前文件的内容

> 使用带有`at`模式的`open()`函数在已存在的文件中添加内容

> 文件的读写操作默认使用的是系统编码；如果你已经知道你要读写的文本是其它编码方式，那么可以通过传递一个可选的`encoding`参数给`open()`函数。如下：

```
with open('somefile.txt', 'rt', encoding='latin-1') as f:
    ...
```

> 注意，读写文本文件一般来讲是比较简单的，但是也有几点是需要注意的；首先，在例子程序中的`with`语句给被使用到的文件创建了一个上下文环境，当`with`控制块结束时，文件会自动关闭。你也可以不使用`with`语句，但是这时候你就必须记得手动关闭文件：

```
f = open('somefile.txt', 'rt')
data = f.read()
f.close()
```

> 另外一个文件是关于换行符的识别问题，在Unix和Windows中是不一样的(分别是n和rn)。默认情况下，Python会以统一模式处理换行符。这种模式下，在读取文本的时候，Python可以识别所有的普通换行符并将其转换为但个`\n`字符。类似的，在输出时会将换行符`\n`转换为系统默认的换行符。如果你不希望这种默认的处理方式，可以给`open()`函数传入参数`newline=''`。

> 为了说明两者之间的差异，下面我在Unix机器上面读取一个Windows上面的文本文件，里面的内容时`hello world!\r\n`：


```
>>> # Newline translation enabled (the default)
>>> f = open('hello.txt', 'rt')
>>> f.read()
'hello world!\n'

>>> # Newline translation disabled
>>> g = open('hello.txt', 'rt', newline='')
>>> g.read()
'hello world!\r\n'
>>>
```

> 最后一个问题就是文本文件中可能出现的编码错误。但你读取或者写入一个文本文件时，你可能会遇到一个编码或者解码错误。比如：

```
>>> f = open('sample.txt', 'rt', encoding='ascii')
>>> f.read()
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/usr/local/lib/python3.3/encodings/ascii.py", line 26, in decode
        return codecs.ascii_decode(input, self.errors)[0]
UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position
12: ordinal not in range(128)
>>>
```

> 如果出现这个错误，通常表示你读取文本时指定的编码不正确。你最好仔细阅读说明并确认你的文件编码时正确的。如果编码错误还是存在，你可以给`open()`函数传递一个可选的`errors`参数来处理这些错误。如下：

```
>>> # Replace bad chars with Unicode U+fffd replacement char
>>> f = open('sample.txt', 'rt', encoding='ascii', errors='replace')
>>> f.read()
'Spicy Jalape?o!'
>>> # Ignore bad chars entirely
>>> g = open('sample.txt', 'rt', encoding='ascii', errors='ignore')
>>> g.read()
'Spicy Jalapeo!'
>>>
```

> 注意，如果你经常使用`errors`参数来处理编码错误，可能会让你的生活变的很糟糕。对于文本处理的首要原则时确保你总是使用的时正确编码。当模棱两可的时候，就使用默认的设置(通常都是UTF-8)。

### 打印输出至文件

> 在`print()`函数中指定`file`关键字参数，如下：

```
with open('d:/work/test.txt', 'wt') as f:
    print('Hello World!', file=f)
```

> 关于输出重定向到文件中就这些了。但是有一点要注意的就是文件必须是以文本模式打开。如果文件是二进制模式的话，打印就会出错。

### 使用其它分隔符或行终止符打印

> 你想使用`print()`函数输出数据，但是想改变默认的分隔符或者行尾符。
> 你可以在`print()`函数中使用`sep`和`end`关键字参数，以你想要的方式输出。比如：

```
>>> print('ACME', 50, 91.5)
ACME 50 91.5
>>> print('ACME', 50, 91.5, sep=',')
ACME,50,91.5
>>> print('ACME', 50, 91.5, sep=',', end='!!\n')
ACME,50,91.5!!
>>>
```

> 使用`end`参数也可以在输出中禁止换行。比如：

```
>>> for i in range(5):
...     print(i)
...
0
1
2
3
4
>>> for i in range(5):
...     print(i, end=' ')
...
0 1 2 3 4 >>>
```

> 当你想使用非空格分隔符来输出数据的时候，给`print()`函数传递一个`sep`参数是最简单的方案。

```
>>> row = ('ACME', 50, 91.5)
>>> print(*row, sep=',')
ACME,50,91.5
>>>
```

> 注意，有时候你会看到一些程序员使用`str.join()`函数来完成同样的事情，但`str.join()`仅仅适用于字符串。这一位这你通常需要执行另外一些转换才能让它正常工作。

### 读写字节数据

> 你想读写二进制文件，比如图片，声音文件等等。
> 使用模式为`rb`或`wb`的`open()`函数来读取或写入二进制数据。比如：

```
# Read the entire file as a single byte string
with open('somefile.bin', 'rb') as f:
    data = f.read()

# Write binary data to a file
with open('somefile.bin', 'wb') as f:
    f.write(b'Hello World')
```

> 在读取二进制数据时，需要指明的时所有返回的数据都是字节字符串格式的，而不是文本字符串。类似的，在写入的时候，必须保证参数是以字节形式对外暴露数据的对象(比如字节字符串，字节数组对象等)。

> 注意，在读取二进制数据的时候，字节字符串和文本字符串的语义差异可能会导致一个潜在的陷阱。特别需要注意的是，索引和迭代动作返回的是字节的值，而不是字节字符串。比如：

```
>>> # Text string
>>> t = 'Hello World'
>>> t[0]
'H'
>>> for c in t:
...     print(c)
...
H
e
l
l
o
...
>>> # Byte string
>>> b = b'Hello World'
>>> b[0]
72
>>> for c in b:
...     print(c)
...
72
101
108
108
111
...
>>>
```

> 如果你想从二进制模式的文件中读取或写入文本数据，必须确保要进行解码和编码操作。比如：

```
with open('somefile.bin', 'rb') as f:
    data = f.read(16)
    text = data.decode('utf-8')

with open('somefile.bin', 'wb') as f:
    text = 'Hello World'
    f.write(text.encode('utf-8'))
```

> 二进制I/O还有一个鲜为人知的特性就是数组和C结构体类型能直接被写入，而不需要中间转换为字节对象。比如：

```
import array
nums = array.array('i', [1, 2, 3, 4])
with open('data.bin','wb') as f:
    f.write(nums)
```

### 文件不存在才能写入

> 你想像一个文件中写入数据，但是前提必须是这个文件在文件系统上不存在。也就是不允许覆盖已存在的文件内容。

> 可以在`open()`函数中使用`x`模式来代替`w`模式的方法来解决这个问题。比如：

```
>>> with open('somefile', 'wt') as f:
...     f.write('Hello\n')
...
>>> with open('somefile', 'xt') as f:
...     f.write('Hello\n')
...
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
FileExistsError: [Errno 17] File exists: 'somefile'
>>>
```

> 注意，`x`模式是Python3对`open()`函数特有的扩展。在Python的旧版本或者是Python实现的底层C函数库中都是没有这个模式的。

### 读写压缩文件

> 你像读写一个`gizp`或`bz2`格式的压缩文件

> `gzip`和`bz2`模块可以很容易的处理这些文件。两个模块都为`open()`函数提供了另外的实现来解决这个问题。比如为了以文本形式读取压缩文件，可以这样做：

```
# gzip compression
import gzip
with gzip.open('somefile.gz', 'rt') as f:
    text = f.read()

# bz2 compression
import bz2
with bz2.open('somefile.bz2', 'rt') as f:
    text = f.read()
```

> 类似的，为了写入压缩数据，可以这样做：

```
# gzip compression
import gzip
with gzip.open('somefile.gz', 'wt') as f:
    f.write(text)

# bz2 compression
import bz2
with bz2.open('somefile.bz2', 'wt') as f:
    f.write(text)
```

> 如上，所有的I/O操作都使用文本模式并执行Unicode的编码/解码。类似的，如果你想操作二进制数据，使用`rb`或者`wb`文件模式即可。

> `gzip.open()`和`bz2.open()`接受跟内置的`open()`函数一样的参数，包括`encoding`，`errors`，`newline`等等。

> 当写入压缩数据时，可以使用`compresslevel`这个可选的的关键字参数来指定一个压缩级别。比如：

```
with gzip.open('somefile.gz', 'wt', compresslevel=5) as f:
    f.write(text)
```

> 默认的等级是9，也是最高的压缩等级。等级越低性能越好，但是数据压缩成都也越低。

### 固定大小记录的文件迭代

> 你想在一个固定长度记录或者数据块的集合上迭代，而不是在一个文件中一行一行的迭代。

> 通过下面这个小技巧`iter`和`functools.partial()`函数：

```
from functools import partial

RECORD_SIZE = 32

with open('somefile.data', 'rb') as f:
    records = iter(partial(f.read, RECORD_SIZE), b'')
    for r in records:
        ...
```

> 这个例子中的`records`对象是一个可迭代对象，它会不断的产生固定大小的数据块，直到文件末尾。要注意的是如果总记录大小不是块大小的整数倍的话，最后一个返回元素的字节数会比期望值少。

> `iter()`函数有一个鲜为人知的特性就是，如果你给它传递一个可调用对象和一个标记值，它会创建一个迭代器。这个迭代器会一直调用传入的可调用对象直到它返回标记值为止，这时候迭代终止。

> 在例子中，`functools.partial`用来创建一个每次被调用时从文件中读取固定数目字节的可调用对象。标记值`b''`就是当到达文件结尾时的返回值。

> 注意，上面的例子中的文件是以二进制模式打开的，如果是读取固定大小的记录，这通常是最普遍的情况。而对于文本文件，一行一行的读取(默认的迭代行为)更普遍点。

### 读取二进制数据到可变缓冲区中

> 为了读取数据到一个可变数组中，使用文件对象的`readinto()`方法。比如：

```
import os.path

def read_into_buffer(filename):
    buf = bytearray(os.path.getsize(filename))
    with open(filename, 'rb') as f:
        f.readinto(buf)
    return buf
```

> 下面是一个演示这个函数使用方法的例子：

```
>>> # Write a sample file
>>> with open('sample.bin', 'wb') as f:
...     f.write(b'Hello World')
...
>>> buf = read_into_buffer('sample.bin')
>>> buf
bytearray(b'Hello World')
>>> buf[0:5] = b'Hallo'
>>> buf
bytearray(b'Hallo World')
>>> with open('newsample.bin', 'wb') as f:
...     f.write(buf)
...
11
>>>
```

> 文件对象的`readinto()`方法能被用来预先分配内存的数组填充数据，甚至包括由`array`模块或`numpy`库创建的数组。和普通`read()`方法不同的是，`readinto()`填充已存在的缓冲区而不是为新对象重新分配内存再返回它们。因此，你可以使用它来避免大量的内存分配操作。比如，如果你读取一个由相同大小的记录组成的二进制文件时，你可以像下面这样写：

```
record_size = 32 # Size of each record (adjust value)

buf = bytearray(record_size)
with open('somefile', 'rb') as f:
    while True:
        n = f.readinto(buf)
        if n < record_size:
            break
        # Use the contents of buf
        ...
```

### 创建临时文件和目录

> 你需要在程序执行时创建一个临时文件或目录，并希望使用完后可以自动的销毁掉。

> `tempfile`模块中有很多的函数可以完成这任务。为了创建一个匿名的临时文件，可以使用`tempfile.TemporaryFile'：

```
from tempfile import TemporaryFile

with TemporaryFile('w+t') as f:
    # Read/write to the file
    f.write('Hello World\n')
    f.write('Testing\n')

    # Seek back to beginning and read the data
    f.seek(0)
    data = f.read()

# Temporary file is destroyed
```

> `TemporaryFile()`的第一个参数是文件模式，通常来讲文本模式使用`w+t`，二进制模式使用`w+b`。这个模式同时支持读和写操作，在这里是很有用的。因为当你关闭文件去改变模式的时候，文件实际上已经不存在了。`TemporaryFile()`另外还支持跟内置的`open()`函数一样的参数。比如：

```
with TemporaryFile('w+t', encoding='utf-8', errors='ignore') as f:
    ...
```

> 在大多数Unix雄上，通过`TemporaryFile()`创建的文件都是匿名的，甚至连目录都没有。如果你像打破这个限制，可以使用`NamedTemporaryFile()`来代替。比如：

```
from tempfile import NamedTemporaryFile

with NamedTemporaryFile('w+t') as f:
    print('filename is:', f.name)
    ...

# File automatically destroyed
```

> 这里，被打开文件的`f.name`属性包含了该临时我文件的文件名。当你需要将文件名传递给其它代码来打开这个文件的时候，这个就很有用了。和`TemporaryFile()`一样，结果文件文件关闭时会被自动删除掉。如果你不想这么做，可以传递一个关键字参数`delete=False`即可。比如：

```
with NamedTemporaryFile('w+t', delete=False) as f:
    print('filename is:', f.name)
    ...
```

> 为了创建一个临时目录，可以使用`tempfile.TemporaryDirectory()`。比如：

```
from tempfile import TemporaryDirectory

with TemporaryDirectory() as dirname:
    print('dirname is:', dirname)
    # Use the directory
    ...
# Directory and all contents destroyed
```

> 注意，所有和临时文件相关的函数都允许你通过使用关键字参数`prefix`、`suffix`和`dir`来自定义目录以及命名规则。比如：

```
>>> f = NamedTemporaryFile(prefix='mytemp', suffix='.txt', dir='/tmp')
>>> f.name
'/tmp/mytemp8ee899.txt'
>>>
```

### 序列化Python对象

> 对于序列化最普遍的做法就是使用`pickle`模块。为了将一个对象保存到一个文件中，可以这样做：

```
import pickle

data = ... # Some Python object
f = open('somefile', 'wb')
pickle.dump(data, f)
```

> 为了将一个对象转存为一个字符串，可以使用`pickle.dumps()`：

```
s = pickle.dumps(data)
```

> 为了从字节流中恢复一个对象，使用`pickle.load()`或`pickle.loads()`函数。比如：

```
# Restore from a file
f = open('somefile', 'rb')
data = pickle.load(f)

# Restore from a string
data = pickle.loads(s)
```

> 当数据反序列化回来的时候，会先假定所有的源数据是可用的。模块、类和函数会自动按需导入进来。对于Python数据被不同机器上的解析器所共享的应用程序而言，数据的保存可能会有问题，因为所有的机器都必须访问同一个源代码。

> 注意，千万不要对不信任的数据使用`pickle.load()`，`pickle`在家在时有一个副作用就是它会自动加载相应模块并构造实例对象。当某个坏人如果直到`pickle`的工作原理，他就可以创建一个恶意的数据导致Python执行随意指定的系统命令。因此，一定要保证`pickle`只在相互之间可以认证对方的解析器的内部使用。
