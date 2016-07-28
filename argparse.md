# argparse

`argparse`模块使得编写用户友好的命令行接口非常容易。程序只需定义好它要求的参数，然后`argparse`将负责如何从`sys.argv`中解析出这些参数。`argparse`模块还会自动生成帮助和使用信息并且当用户赋给程序非法的参数时产生错误信息。

### 示例

```python
import argparse

parser = argparse.ArgumentParser(description='Process some integers.')
parser.add_argument('integers', metavar='N', type=int, nargs='+',
                   help='an integer for the accumulator')
parser.add_argument('--sum', dest='accumulate', action='store_const',
                   const=sum, default=max,
                   help='sum the integers (default: find the max)')

args = parser.parse_args()
print(args.accumulate(args.integers))
```

假设上面的Python代码保存为一个叫做`prog.py`的文件，它可以在命令行上执行并提供有用的帮助信息：

```shell
[root@localhost test_scripts]# python prog.py -h
usage: prog.py [-h] [--sum] N [N ...]

Process some integers.

positional arguments:
  N           an integer for the accumulator

optional arguments:
  -h, --help  show this help message and exit
  --sum       sum the integers (default: find the max)
```

当你以适当的参数运行时，它会打印出命令行整数的和或者最大值：

```shell
[root@localhost test_scripts]# python prog.py 1 2 3 4 5
5
[root@localhost test_scripts]# python prog.py 1 2 3 4 5 --sum
15
[root@localhost test_scripts]#
```

如果传递非法参数，它将产生错误信息：

```shell
[root@localhost test_scripts]# python prog.py a b c
usage: prog.py [-h] [--sum] N [N ...]
prog.py: error: argument N: invalid int value: 'a'
```

下面将会带你一步一步的深入这个示例。

### 创建一个解析器

使用`argparse`的第一个步骤就是创建一个`ArgumentParser`对象：

```python
>>> parser = argparse.ArgumentParser(description='Process some integers.')
```

`ArgumentParser`对象会把命令行参数解析成Python数据类型所需要的所有信息。

### 添加参数

通过调用`add_argument()`方法向`ArgumentParser`添加程序的参数信息。通常情况下，这些信息告诉`ArgumentParser`如何接收命令行上的字符串并将它们转换成对象。这些信息被保存下来并在调用`parse_args()`时会用到。例如：

```python
>>> parser.add_argument('integers', metavar='N', type=int, nargs='+',
...                     help='an integer for the accumulator')
>>> parser.add_argument('--sum', dest='accumulate', action='store_const',
...                     const=sum, default=max,
...                     help='sum the integers (default: find the max)')
```

接下来，调用`parsee_args()`返回的对象将带有两个属性，`integers`和`accumulate`。属性`integers`将是一个包含一个或多个整数的列表，如果命令行上指定`--sum`，那么属性`accumulate`将是**sum()**函数，如果没有指定，则是**max()**函数。

### 参数解析

`ArgumentnParser`通过`parse_args()`方法解析参数。它将检查命令行，把每个参数转换成恰当的类型并采取恰当的动作。在大部分情况下，这意味这将从命令行中解析出来的属性会建立一个简单的`Namespace`对象。

```python
>>> parser.parse_args(['--sum', '7', '-1', '42'])
Namespace(accumulate=<built-in function sum>, integers=[7, -1, 42])
```

在脚本中，`parse_args()`调用一般不带参数，`ArgumentParser`将根据`sys.argv`自动确定命令行参数。

# ArgumentParser对象

```python
class argparse.ArgumentParser(prog=None, 
                              usage=None, 
                              description=None, 
                              epilog=None, 
                              parents=[], 
                              formatter_class=argparse.HelpFormatter,
                              prefix_chars='-', 
                              fromfile_prefix_chars=None, 
                              argument_default=None,
                              conflict_handler='error', 
                              add_help=True, 
                              allow_abbrev=True)
```

`ArgumentParser`对象所有的参数都应该以关键字参数传递。下面是每个参数简短的描述：

|         参数        |描述                                                             |
|:-------------------:|:----------------------------------------------------------------|
|prog                 |程序名，默认为`sys.argv[0]`。                                    |
|usage                |程序的简短描述，`str`类型；默认从解析器的参数生成。              |
|description          |参数帮助信息之前的文本；默认为空。                               |
|epilog               |参数帮助信息之后的文本；默认为空。                               |
|parents              |`ArgumentParser`对象的一个列表，注意，对象的参数应该包括进去。   |
|formatter_class      |定制化帮助信息的类。                                             |
|prefix_chars         |可选参数的前缀字符集；默认为 **-**。                             |
|formfile_prefix_chars|额外的参数应该读取的文件前缀字符集；默认为**None**。             |
|argument_default     |参数的全局默认值；默认为**None**。                               |
|conflict_handler     |解决冲突的可选参数的策略；通常没有必要。                         |
|add_help             |给解析器添加`-h/--help`选项；默认为**True**。                    |
|allow_abbrev         |允许简写的长选项；默认为**True**。在Python 3.5被添加。           |

下面将会描述这些参数该如何使用。

### prog参数

默认情况下，`ArgumentParser`对象使用`sys.argv[0]`决定在帮助信息中如何显示程序名。这个默认值几乎总总能满足需求，因为帮助信息（中的程序名称）会自动匹配命令行中调用的程序名称。例如，参考下面这段`myprogram.py`文件中的代码：

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo', help='foo help')
args = parser.parse_args()
```

该程序的帮助信息将显示`myprogram.py`作为程序的名字（无论程序在哪被调用）：

```shell
[root@localhost test_scripts]# python myprogram.py --help
usage: myprogram.py [-h] [--foo FOO]

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   foo help
[root@localhost test_scripts]#
```

如果要改变这个默认行为，可以使用`prog=`参数给`ArgumentParser`提供另外一个值：

```python
>>> parser = argparse.ArgumentParser(prog='myprogram')
>>> parser.print_help()
usage: myprogram [-h]

optional arguments:
 -h, --help  show this help message and exit
```

> 注意，无论是来自`sys.argv[0]`还是来自`prog=argument`，在帮助信息中都可以使用`%(prog)s`格式符得到程序的名字。

```python
>>> parser = argparse.ArgumentParser(prog='myprogram')
>>> parser.add_argument('--foo', help='foo of the %(prog)s program')
>>> parser.print_help()
usage: myprogram [-h] [--foo FOO]

optional arguments:
 -h, --help  show this help message and exit
 --foo FOO   foo of the myprogram program
```

### usage参数

默认情况下，`ArgumentParser`依据它包含的参数计算出帮助信息：

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('--foo', nargs='?', help='foo help')
>>> parser.add_argument('bar', nargs='+', help='bar help')
>>> parser.print_help()
usage: PROG [-h] [--foo [FOO]] bar [bar ...]

positional arguments:
 bar          bar help

optional arguments:
 -h, --help   show this help message and exit
 --foo [FOO]  foo help
```

可以通过关键字参数`usage=`覆盖默认的信息：

```python
In [1]: import argparse
In [2]: parser = argparse.ArgumentParser(prog='PROG', usage='%(prog)s [options]')

In [3]: parser.add_argument('--foo', nargs='?', help='foo help')
Out[3]: _StoreAction(option_strings=['--foo'], dest='foo', nargs='?', const=None, default=None, type=None, choices=None, help='foo help', metavar=None)

In [4]: parser.add_argument('bar', nargs='+', help='bar help')
Out[4]: _StoreAction(option_strings=[], dest='bar', nargs='+', const=None, default=None, type=None, choices=None, help='bar help', metavar=None)

In [5]: parser.print_help()
usage: PROG [options]

positional arguments:
  bar          bar help

optional arguments:
  -h, --help   show this help message and exit
  --foo [FOO]  foo help
```

在你的帮助信息中，可以使用`%(prog)s`格式指示符替代程序的名字。

### description参数

`ArgumentParser`构造器的大部分调用都将使用`description=`关键字参数。这个参数给出程序有什么用以及如何工作的简短描述。在帮助信息中，该描述在命令行用法字符串和各个参数的帮助信息之间显示：

```python
In [1]: import argparse
In [2]: parser = argparse.ArgumentParser(description='A foo that bars')
In [3]: parser.print_help()
usage: ipython [-h]

A foo that bars

optional arguments:
  -h, --help  show this help message and exit
```

默认情况下，该描述会自动换行以适应给定的空间。如果要改变这个行为，可以参考`formatter_class`参数。

### epilog参数

有些程序喜欢在参数的描述之后显示额外的关于程序的描述。这些文本可以使用`ArgumentParser`的`epilog=`参数指定：

```python
In [2]: import argparse
In [3]: parser = argparse.ArgumentParser(
   ...: description='A foo that bars',
   ...: epilog="And that's how you'd foo a bar")

In [4]: parser.print_help()
usage: ipython [-h]

A foo that bars

optional arguments:
  -h, --help  show this help message and exit

And that's how you'd foo a bar
```

和`description`一样，`epilog=`文本默认会换行，但是可以通过`ArgumentParser`的`formatter_class`参数调整这个行为。

### parents参数

有时候，几个解析器会共享一个共同的参数集。可以使用一个带有所有共享参数的解析器传递给`ArgumentParser`的`parents=`参数，而不用重复定义这些参数。`parents=`参数接收一个`ArgumentParser`对象的列表，然后收集他们当中所有的位置参数和可选参数，并将这些参数添加到正在构建的`ArgumentParser`对象：

```python
>>> parent_parser = argparse.ArgumentParser(add_help=False)
>>> parent_parser.add_argument('--parent', type=int)

>>> foo_parser = argparse.ArgumentParser(parents=[parent_parser])
>>> foo_parser.add_argument('foo')
>>> foo_parser.parse_args(['--parent', '2', 'XXX'])
Namespace(foo='XXX', parent=2)

>>> bar_parser = argparse.ArgumentParser(parents=[parent_parser])
>>> bar_parser.add_argument('--bar')
>>> bar_parser.parse_args(['--bar', 'YYY'])
Namespace(bar='YYY', parent=None)
```

注意大部分父解析器将指定`add_help=False`；否则，`ArgumentParser`将看到两个`-h/--help`选项（一个在父解析器中，一个在子解析器中）并引发一个错误。

> 注意；在通过`parents=`传递父解析器之前，你必须完全初始化他们。如果在子解析器之后你改变了父解析器，这些改变不会反映在子解析器中。

### formatter_class参数

`ArgumentParser`对象允许通过指定一个格式化类来定制帮助信息的格式。当前，有三中这样的类：
> - **class** argparse.**RawDescriptionHelpFormatter**
> - **class** argparse.**RawTextHelpFormatter**
> - **class** argparse.**ArgumentDefaultsHelpFormatter**

前两个在文本信息如何显示上允许更多控制，最后一个会自动添加关于参数默认值的信息。

默认情况下，`ArgumentParser`对象会对命令行帮助信息中的`description`和`epilog`文本进行换行：

```python
>>> import argparse
>>> parser = argparse.ArgumentParser(
...     prog='PROG',
...     description='''this description
...         was indented weird
...             but that is okay''',
...     epilog='''
...             likewise for this epilog whose whitespace will
...         be cleaned up and whose words will be wrapped
...         across a couple lines''')
>>> parser.print_help()
usage: PROG [-h]

this description was indented weird but that is okay

optional arguments:
  -h, --help  show this help message and exit

likewise for this epilog whose whitespace will be cleaned up and whose words
will be wrapped across a couple lines
>>>
```

把`RawDescriptionHelpFormatter`传递给`formatter_class=`表示`description`和`epilog`已经是正确的格式而不应该再拆行：

```python
In [1]: import argparse
In [2]: import textwrap

In [3]: parser = argparse.ArgumentParser(
   ...:     prog='PROG',
   ...:     formatter_class=argparse.RawDescriptionHelpFormatter,
   ...:     description=textwrap.dedent('''
   ...:         Please do not mess up this text!
   ...:         --------------------------------
   ...:             I have indented it
   ...:             exactly the way
   ...:             I want it
   ...:         '''))
   
In [4]: parser.print_help()
usage: PROG [-h]

Please do not mess up this text!
--------------------------------
    I have indented it
    exactly the way
    I want it

optional arguments:
  -h, --help  show this help message and exit
```

`RawTextHelpFormatter`将保留所有帮助文本的空白，包括参数的描述。

另外一个格式化类`ArgumentDefaultsHelpFormatter`，将为每个参数添加默认值信息。

```python
In [1]: import argparse

In [2]: parser = argparse.ArgumentParser(
   ...:     prog = 'PROG',
   ...:     formatter_class = argparse.ArgumentDefaultsHelpFormatter)

In [3]: parser.add_argument('--foo', type=int, default=42, help='FOO!')
Out[3]: _StoreAction(option_strings=['--foo'], dest='foo', nargs=None, const=None, default=42, type=<class 'int'>, choices=None, help='FOO!', metavar=None)

In [4]: parser.add_argument('bar', nargs='*', default=[1, 2, 3], help='BAR!')
Out[4]: _StoreAction(option_strings=[], dest='bar', nargs='*', const=None, default=[1, 2, 3], type=None, choices=None, help='BAR!', metavar=None)

In [5]: parser.print_help()
usage: PROG [-h] [--foo FOO] [bar [bar ...]]

positional arguments:
  bar         BAR! (default: [1, 2, 3])

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   FOO! (default: 42)
```

### prefix_chars参数

大部分命令行选项使用`-`作为前缀，例如`-f/--foo`。需要指出不同的或者额外前缀的字符解析器，例如类似`+f`或`/foo`这样的选项，可以使用`ArgumentParser`构造器的`prefix_chars=`参数指定他们：

```python
In [1]: import argparse

In [2]: parser = argparse.ArgumentParser(prog='PROG', prefix_chars='-+')

In [3]: parser.add_argument('+f')
Out[3]: _StoreAction(option_strings=['+f'], dest='f', nargs=None, const=None, default=None, type=None, choices=None, help=None, metavar=None)

In [4]: parser.add_argument('++bar')
Out[4]: _StoreAction(option_strings=['++bar'], dest='bar', nargs=None, const=None, default=None, type=None, choices=None, help=None, metavar=None)

In [5]: parser.parse_args('+f X ++bar Y'.split())
Out[5]: Namespace(bar='Y', f='X')
```

`prefix_chars=`参数默认为`-`；提供不包含`-`的字符集将导致异常。

### formfile_prefix_chars参数

当你想要处理一个特别长的参数列表的时候，把参数列表保存在文件中而不是在命令行中敲出来可能会比较合理。如果给出`ArgumentParser`构造器的`fromfile_prefix_chars=`参数，那么以任意一个给定字符开始的参数将被当作文件，并且将这些文件包含的参数替换。例如：

```python
>>> with open('args.txt', 'w') as fp:
...    fp.write('-f\nbar')
>>> parser = argparse.ArgumentParser(fromfile_prefix_chars='@')
>>> parser.add_argument('-f')
>>> parser.parse_args(['-f', 'foo', '@args.txt'])
Namespace(f='bar')
```

从文件中读入的参数必须默认是每行一个参数，并且将被当做在命令行上原始文件所在的位置。所有在上面的例子中，表达式['-f', 'foo', '@args.txt'] 被认为等同于表达式['-f', 'foo', '-f', 'bar']。

`fromfile_prefix_chars=`参数默认为**None**，意味着参数永远不会被当作文件。

### argument_default参数

通常情况下，参数默认值的指定通过传递一个默认值给`add_argument()`或者以一个键值对的集合方式调用`set_default()`方法。然而有时候，指定一个解析器参数的默认值会比较有用。这可以通过传递`argument_default=`关键字参数给`ArgumentParser`来完成。例如，为了全局的阻止`parse_args()`调用时不必要的属性创建，我们可以提供`argument_default=SUPPRESS`：

```python
>>> parser = argparse.ArgumentParser(argument_default=argparse.SUPPRESS)
>>> parser.add_argument('--foo')
>>> parser.add_argument('bar', nargs='?')
>>> parser.parse_args(['--foo', '1', 'BAR'])
Namespace(bar='BAR', foo='1')
>>> parser.parse_args([])
Namespace()
```

### conflict_handler参数

`ArgumentParser`对象不允许同一个选项具有两个动作。默认情况下，如果试图创建一个已经使用的选项，`ArgumentParser`对象将抛出异常。

```python
>>> import argparse
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('-f', '--foo', help='old foo help')
_StoreAction(option_strings=['-f', '--foo'], dest='foo', nargs=None, const=None, default=None, type=None, choices=None, help='old foo help', metavar=None)
>>> parser.add_argument('--foo', help='new foo help')
Traceback (most recent call last):
...
argparse.ArgumentError: argument --foo: conflicting option string: --foo
```

有时候（例如使用`parents`的时候）简单的用相同的选项覆盖旧的参数是有用的。为了得到这样的行为，可以提供`resolve`值给`ArgumentParser`的`conflict_handler=`参数：

```python
>>> parser = argparse.ArgumentParser(prog='PROG', conflict_handler='resolve')
>>> parser.add_argument('-f', '--foo', help='old foo help')
>>> parser.add_argument('--foo', help='new foo help')
>>> parser.print_help()
usage: PROG [-h] [-f FOO] [--foo FOO]

optional arguments:
 -h, --help  show this help message and exit
 -f FOO      old foo help
 --foo FOO   new foo help
```

> 注意，`ArgumentParser`对象只有在所有的选项字符串被覆盖时才删除某个动作。所以，在上面的例子中，旧的`-f/--foo`动作依然保留为`-f`的动作，因为只有`--foo`选项字符串被覆盖。

### add_help参数

默认情况下，`ArgumentParser`对象会添加一个选项简单的显示解析器的帮助信息。例如，考虑一个包含如下代码的`myprogram.py`文件：

```python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('--foo', help='foo help')
args = parser.parse_args()
```

如果在命令行中提供`-h`或者`--help`，`ArgumentParser`的帮助信息将打印出来：

```shell
$ python myprogram.py --help
usage: myprogram.py [-h] [--foo FOO]

optional arguments:
 -h, --help  show this help message and exit
 --foo FOO   foo help
```

偶尔禁止这个帮助选项也可能是有用的。这可以通过传递`False`给`ArgumentParser`的`Add_help=`参数来实现：

```python
>>> parser = argparse.ArgumentParser(prog='PROG', add_help=False)
>>> parser.add_argument('--foo', help='foo help')
>>> parser.print_help()
usage: PROG [--foo FOO]

optional arguments:
 --foo FOO  foo help
```

该帮助选项通常是`-h/--help`。例外情况是指定了`prefix_chars=`而且不包括-，在这种情况下`-h`和`--help`不是合法的选项。在这种情况下，`prefix_chars`中的第一个字符将用于该帮助选项的前缀：

```python
>>> parser = argparse.ArgumentParser(prog='PROG', prefix_chars='+/')
>>> parser.print_help()
usage: PROG [+h]

optional arguments:
  +h, ++help  show this help message and exit
```

# add_argument()方法

```python
ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])
```

该方法定义应该如何解析一个命令行参数。下面是每个参数的简短描述：

|     参数    |描述                                                 |
|:-----------:|:----------------------------------------------------|
|name or flags|选项字符串的名字或者列表，例如`foo`或者`-f, --foo`。 |
|action       |在命令行遇到该参数时采取的基本动作类型。             |
|nargs        |应该读取的命令行参数数目。                           |
|const        |某些`action`和`nargs`选项要求的常数值。              |
|default      |如果命令行中没有出现该参数时的默认值。               |
|type         |命令行参数应该被转换成的类型。                       |
|choices      |包含该参数的值的容器。                               |
|required     |该命令行选项是否可以省略（只针对可选参数）。         |
|help         |参数的简短描述。                                     |
|metavar      |参数在帮助信息中的名字。                             |
|dest         |给`parse_args()`返回的对象要添加的属性名称。         |

下面将对每个参数作详细描述。

### name或flags参数

`add_argument()`方法必须知道期望的是可选参数（比如`-f`或者`--foo`）还是位置参数（比如一个文件列表）。传递给`add_argument()`的第一个参数必须是一个标记序列或者一个简单的参数名字。

例如，一个可选的参数可以像这样创建：

```python
>>> parser.add_argument('-f', '--foo')
```

而一个位置参数可以像这样创建：

```python
>>> parser.add_argument('bar')
```

当调用`parse_args()`时，可选的参数将以`-`前缀标识，剩余的参数将被假定为位置参数：

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('-f', '--foo')
>>> parser.add_argument('bar')
>>> parser.parse_args(['BAR'])
Namespace(bar='BAR', foo=None)
>>> parser.parse_args(['BAR', '--foo', 'FOO'])
Namespace(bar='BAR', foo='FOO')
>>> parser.parse_args(['--foo', 'FOO'])
usage: PROG [-h] [-f FOO] bar
PROG: error: too few arguments
```

### action参数

`ArgumentParse`对象将命令行参数和动作关联起来。这些动作可以完成与命令行参数关联的任何事情，尽管大部分动作只是简单的给`parse_args()`返回的对象添加一个属性。`action`关键字参数指出应该如何处理命令行参数。支持的动作有：

|action args  |description                                                                  |
|:-----------:|:----------------------------------------------------------------------------|
|store        |只保存参数的值；这是默认动作。                                               |
|store_const  |保存由**const**关键字指出的值；**const**默认为**None**。                           |
|store_true   |指定后值为**True**。它是**store_const**的特殊情形。                              |
|store_false  |指定后值为**False**。它是**store_const**的特殊情形。                             |
|append       |保存为一个列表，将每个参数的值附加到列表后面。                               |
|append_const |保存为一个列表，将**const**参数指定的值附加到列表后面。                        |
|count        |计算关键字参数出现的次数；可用于增加详细的级别。                             |
|help         |打印当前解析器中所有选项的完整帮助信息并退出。默认会自动添加到解析器中。     |
|version      |它期待**version=**参数出现在**add_argument()**中，在调用时打印出版本信息并退出。 |
|parse        |包含该动作的**ArgumentParser**对象。                                           |
|namespace    |**parse_args()**返回的**Namespace**对象。大部分动作都会给该对象添加一个属性。    |
|values       |相关联的命令行参数于类型转换之后的值。可通过**add_argument()**的**type**转换。   |
|option_string|调用该动作的选项字符串。**option_string**是可选的。                            |

>注意，`action`中的`append`和`nargs`中的`N`这两者的区别；`append`的指定方式是` --foo 1 --foo 2 --foo 3...`，程序接收到的是一个列表`['1','2','3'...]`；而当指定`nargs=2`时，其命令行接受参数的方式为`--bar 1 2`，程序接收到的是一个列表`['1', '2']`；另外，`nargs`值为`2`，那么在命令行中就必须得指定两个`bar`的参数值，多与少都报错。

> - `store`：只保存参数的值。这是默认动作。例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo')
>>> parser.parse_args('--foo 1'.split())
Namespace(foo='1')
```

> - `store_const`：保存由`const`关键字参数指出的值（注意`const`关键字参数默认为`None`）。`store_const`动作最常用于指定某种标记的可选参数。例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action='store_const', const=42)
>>> parser.parse_args('--foo'.split())
Namespace(foo=42)
```

> - `store_ture`和`store_false`：他们是`store_const`的特殊情形，分别用于保存值**True**和**False**。另外，他们分别会创建默认值**False**和**True**。例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action='store_true')
>>> parser.add_argument('--bar', action='store_false')
>>> parser.add_argument('--baz', action='store_false')
>>> parser.parse_args('--foo --bar'.split())
Namespace(bar=False, baz=True, foo=True)
```

> - `append`：保存为一个列表，并将每个参数值附加在列表的后面。这对于允许指定多次的选项很有帮助。示例用法：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action='append')
>>> parser.parse_args('--foo 1 --foo 2'.split())
Namespace(foo=['1', '2'])
```

> - `append_const`：保存为一个列表，并将`const`关键字参数指定的值附加在列表的后面。`append_const`动作在多个参数需要保存常量到相同的列表时特别有用。例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--str', dest='types', action='append_const', const=str)
>>> parser.add_argument('--int', dest='types', action='append_const', const=int)
>>> parser.parse_args('--str --int'.split())
Namespace(types=[<type 'str'>, <type 'int'>])
```

> - `count`：计算关键字参数出现的次数。这可用于增加详细的级别，例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--verbose', '-v', action='count')
>>> parser.parse_args('-vvv'.split())
Namespace(verbose=3)
```

> - `help`：打印当前解析器中所有选项的完整帮助信息然后退出。默认情况下，help动作会自动添加到解析器中。

> - `version`：他期待`version=`参数出现在`add_argument()`调用中，在调用时打印出版本信息并推出：

```python
>>> import argparse
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('--version', action='version', version='%(prog)s 2.0')
>>> parser.parse_args(['--version'])
PROG 2.0
```

你还可以通过传递一个实现了**Action API**的对象指定任意一个动作。实现该功能的最简单方法是扩展**argparse.Action**，并提供一个合适的`__call__`方法。`__call__`方法应该接受四个参数：

> - `parser`：包含该动作的`ArgumentParser`对象
> - `namespace`：`parse_args()`返回的`Namespace`对象。大部分动作会给该对象添加一个属性。
> - `values`：相关联的命令行参数于类型转换之后的值。（类型转换方式通过`add_argument()`的`type`关键字参数指定。）
> - `option_string`：调用该动作的选项字符串。`option_string`参数是可选的，如果动作是关联的位置参数将不会出现。

自定义动作的例子：

```python
>>> class FooAction(argparse.Action):
...     def __call__(self, parser, namespace, values, option_string=None):
...         print '%r %r %r' % (namespace, values, option_string)
...         setattr(namespace, self.dest, values)
...
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', action=FooAction)
>>> parser.add_argument('bar', action=FooAction)
>>> args = parser.parse_args('1 --foo 2'.split())
Namespace(bar=None, foo=None) '1' None
Namespace(bar='1', foo=None) '2' '--foo'
>>> args
Namespace(bar='1', foo='2')
```

### nargs参数

`ArgumentParser`对象通常将一个动作与一个命令行参数关联。`nargs`关键字参数将一个动作与不同数目的命令行参数关联在一起。它支持的值有：

> - `N`：（一个整数）。命令行中的`N`个参数将被一起收集在一个列表中。例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', nargs=2)
>>> parser.add_argument('bar', nargs=1)
>>> parser.parse_args('c --foo a b'.split())
Namespace(bar=['c'], foo=['a', 'b'])
```

注意`nargs=1`生成一个只有一个元素的列表。这和默认的行为是不一样的，默认情况下生成的是元素自己。

> - `?`：如果有的话就从命令行读取一个参数并生成一个元素。如果没有对应的命令行参数，则产生一个来自`default`的值。注意，对于可选参数有另外一种情况，有选项字符串但是后面没有跟命令行参数。在这种情况下，将产生一个来自`const`的值。用一些例子加以解释：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', nargs='?', const='c', default='d')
>>> parser.add_argument('bar', nargs='?', default='d')
>>> parser.parse_args('XX --foo YY'.split())
Namespace(bar='XX', foo='YY')
>>> parser.parse_args('XX --foo'.split())
Namespace(bar='XX', foo='c')
>>> parser.parse_args(''.split())
Namespace(bar='d', foo='d')
```

`nargs='?'`的一种更常见的用法是允许可选的输入和输出文件：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('infile', nargs='?', type=argparse.FileType('r'),
...                     default=sys.stdin)
>>> parser.add_argument('outfile', nargs='?', type=argparse.FileType('w'),
...                     default=sys.stdout)
>>> parser.parse_args(['input.txt', 'output.txt'])
Namespace(infile=<open file 'input.txt', mode 'r' at 0x...>,
          outfile=<open file 'output.txt', mode 'w' at 0x...>)
>>> parser.parse_args([])
Namespace(infile=<open file '<stdin>', mode 'r' at 0x...>,
          outfile=<open file '<stdout>', mode 'w' at 0x...>)
```

> - `*`：出现的所有命令行参数都被收集到一个列表中。注意，一般情况下具有多个带有`nargs='*'`的位置参数是不合理的，但是多个带有`nargs='*'`的可选参数是有可能的。例如：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', nargs='*')
>>> parser.add_argument('--bar', nargs='*')
>>> parser.add_argument('baz', nargs='*')
>>> parser.parse_args('a b --foo x y --bar 1 2'.split())
Namespace(bar=['1', '2'], baz=['a', 'b'], foo=['x', 'y'])
```

> - `+`：和`*`一样，出现的所有命令行参数都被收集到一个列表中。除此之外，如果没有一个命令行参数将会产生一个错误信息。例如：

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('foo', nargs='+')
>>> parser.parse_args('a b'.split())
Namespace(foo=['a', 'b'])
>>> parser.parse_args(''.split())
usage: PROG [-h] foo [foo ...]
PROG: error: too few arguments
```

> - `argparse.REMAINDER`：所有剩余的命令行参数都被收集到一个列表中去。这通常用于命令行工具分发到其它命令行工具：

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('--foo')
>>> parser.add_argument('command')
>>> parser.add_argument('args', nargs=argparse.REMAINDER)
>>> print parser.parse_args('--foo B cmd --arg1 XX ZZ'.split())
Namespace(args=['--arg1', 'XX', 'ZZ'], command='cmd', foo='B')
```

如果没有提供`nargs`关键字参数，那么读取的参数个数取决于`action`。通常这意味这将读取一个命令行参数并产生一个元素（不是一个列表）。

### const参数

`add_argument()`的`const`参数用于保存常量值，它不会从命令行读入但却是`ArgumentParser`的动作所必须的。他的两个最常见的用法是：

> 【1】 当以`action='store_const'`或者`action='append_const'`调用`add_argument()`时，这些动作向`parse_args()`返回对象的一个属性并添加`const`值。
> 【2】 当以选项字符串（例如`-f`或者`--foo`和`nargs='?'`）调用`add_argument()`时。它创建一个后面可以跟随零个或者一个命令行字符串的可选参数。当解析命令行参数时，如果选项字符串后面跟随命令行参数，则假定其为`const`的值。

`const`关键字的默认值是**None**。

### default参数

所有可选的参数以及某些位置参数可以在命令行中省略。`add_argument()`的`default`关键字参数，其默认值为**None**，指出如果命令行参数没有出现时他们应该是什么值。对于可选参数，`default`的值用于选项字符串没有出现在命令行中的时候：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', default=42)
>>> parser.parse_args('--foo 2'.split())
Namespace(foo='2')
>>> parser.parse_args(''.split())
Namespace(foo=42)
```

如果`default`的值是一个字符串，解析器将像命令行参数一样解析这个值。特别的，在设置`Namespace`返回值的属性之前，解析器会调用`type`转换参数。否则，解析器就是用其原始值：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--length', default='10', type=int)
>>> parser.add_argument('--width', default=10.5, type=int)
>>> parser.parse_args()
Namespace(length=10, width=10.5)
```

对于`nargs`等于`?`或者`*`的位置参数，`default`在没有命令行参数时使用：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('foo', nargs='?', default=42)
>>> parser.parse_args('a'.split())
Namespace(foo='a')
>>> parser.parse_args(''.split())
Namespace(foo=42)
```

`default=argparse.SUPPRESS`将导致如果没有命令行参数时不会添加属性：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', default=argparse.SUPPRESS)
>>> parser.parse_args([])
Namespace()
>>> parser.parse_args(['--foo', '1'])
Namespace(foo='1')
```

### type参数

默认情况下，`ArgumentParser`对象以简单字符串方式读入命令行参数。然而，很多时候命令行字符串应该被解释成另外一种类型，例如**浮点数**或者**整数**。`add_argument()`的`type`关键字参数允许任意必要的类型检查并作类型转换。常见的内建类型和函数可以直接用作`type`参数的值：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('foo', type=int)
>>> parser.add_argument('bar', type=file)
>>> parser.parse_args('2 temp.txt'.split())
Namespace(bar=<open file 'temp.txt', mode 'r' at 0x...>, foo=2)
```

为了简化各种文件你类型的使用，`argparse`提供了工厂类型`FileType`，它以`file`对象的`mode=`和`bufsize=`为参数。例如，`FileType('w')`可以用于创建一个可写的文件：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('bar', type=argparse.FileType('w'))
>>> parser.parse_args(['out.txt'])
Namespace(bar=<open file 'out.txt', mode 'w' at 0x...>)
```

`type=`可以接受任何可调用类型，只要该类型以一个字符串为参数并且返回转换后的类型：

```python
>>> def perfect_square(string):
...     value = int(string)
...     sqrt = math.sqrt(value)
...     if sqrt != int(sqrt):
...         msg = "%r is not a perfect square" % string
...         raise argparse.ArgumentTypeError(msg)
...     return value
...
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('foo', type=perfect_square)
>>> parser.parse_args('9'.split())
Namespace(foo=9)
>>> parser.parse_args('7'.split())
usage: PROG [-h] foo
PROG: error: argument foo: '7' is not a perfect square
```

`choices`关键字参数对于简单的某个范围内的类型检查可能更方便：

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('foo', type=int, choices=range(5, 10))
>>> parser.parse_args('7'.split())
Namespace(foo=7)
>>> parser.parse_args('11'.split())
usage: PROG [-h] {5,6,7,8,9}
PROG: error: argument foo: invalid choice: 11 (choose from 5, 6, 7, 8, 9)
```

### choices参数

某些命令行参数应该从一个受限的集合中选择。这种情况的处理可以通过传递一个容器对象作为`choices`关键字参数给`add_argument()`。当解析命令行时，将检查参数的值，如果参数不是一个可接受的值则显示一个错误信息：

```python
>>> parser = argparse.ArgumentParser(prog='game.py')
>>> parser.add_argument('move', choices=['rock', 'paper', 'scissors'])
>>> parser.parse_args(['rock'])
Namespace(move='rock')
>>> parser.parse_args(['fire'])
usage: game.py [-h] {rock,paper,scissors}
game.py: error: argument move: invalid choice: 'fire' (choose from 'rock',
'paper', 'scissors')
```

注意`choices`容器包含的对象在`type`转换之后检查，所以`choices`容器中对象的类型应该与`type`指出的类型相匹配：

```python
>>> parser = argparse.ArgumentParser(prog='doors.py')
>>> parser.add_argument('door', type=int, choices=range(1, 4))
>>> print(parser.parse_args(['3']))
Namespace(door=3)
>>> parser.parse_args(['4'])
usage: doors.py [-h] {1,2,3}
doors.py: error: argument door: invalid choice: 4 (choose from 1, 2, 3)
```

支持`in`操作符的任何对象都可以传递给**choices**作为它的值，所以**dict**对象、**set**对象以及自定义的容器等等都支持。

### required参数

一般情况下，`argparse`模块假定`-f`和`--bar`标记表示**可选**参数，他们在命令行中可以省略。如果要使得选项是**必需**的，可以指定**True**作为`required=`关键字参数的值给`add_argument()`：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', required=True)
>>> parser.parse_args(['--foo', 'BAR'])
Namespace(foo='BAR')
>>> parser.parse_args([])
usage: argparse.py [-h] [--foo FOO]
argparse.py: error: option --foo is required
```

正如例子所演示的，如果一个命令被标记为`required`，那么如果命令行中没有出现该参数`parse_args()` 将报告一个错误。

### help参数

`help`的值是一个包含参数简短描述的字符串。当用户要求帮助时（通常是通过使用`-h`或者`--help`在命令被指定），这些`help`的描述将随每个参数一起显示出来：

```python
>>> parser = argparse.ArgumentParser(prog='frobble')
>>> parser.add_argument('--foo', action='store_true',
...         help='foo the bars before frobbling')
>>> parser.add_argument('bar', nargs='+',
...         help='one of the bars to be frobbled')
>>> parser.parse_args('-h'.split())
usage: frobble [-h] [--foo] bar [bar ...]

positional arguments:
 bar     one of the bars to be frobbled

optional arguments:
 -h, --help  show this help message and exit
 --foo   foo the bars before frobbling
```

`help`字符串可以包含各种格式指示符以避免如程序名字和参数`default`的重复。可以的指示符包括程序的名称`%(prog)s`以及大部分的`add_argument()`的关键字参数，例如`%(default)s`、`%(type)s`等：

```python
>>> parser = argparse.ArgumentParser(prog='frobble')
>>> parser.add_argument('bar', nargs='?', type=int, default=42,
...         help='the bar to %(prog)s (default: %(default)s)')
>>> parser.print_help()
usage: frobble [-h] [bar]

positional arguments:
 bar     the bar to frobble (default: 42)

optional arguments:
 -h, --help  show this help message and exit
```

通过设置`help`值为`argparse.SUPPRESS`，`argparse`支持隐藏特定选项的帮助：

```python
>>> parser = argparse.ArgumentParser(prog='frobble')
>>> parser.add_argument('--foo', help=argparse.SUPPRESS)
>>> parser.print_help()
usage: frobble [-h]

optional arguments:
  -h, --help  show this help message and exit
```

### metavar参数

当`ArgumentParser`生成帮助信息时，它需要以某种方式引用每一个参数。默认情况下，`ArgumentParser`对象使用`dest`的值作为每个对象的**名字**。默认情况下，对于位置参数直接使用`dest`的值，对于可选参数则将`dest`的值变为大写。所以，位置参数`dest='bar'`将引用成`bar`。后面带有一个命令行参数的可选参数`--foo`将引用成`FOO`。如下：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo')
>>> parser.add_argument('bar')
>>> parser.parse_args('X --foo Y'.split())
Namespace(bar='X', foo='Y')
>>> parser.print_help()
usage:  [-h] [--foo FOO] bar

positional arguments:
 bar

optional arguments:
 -h, --help  show this help message and exit
 --foo FOO
```

可以用`metavar`指定另一个名字：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', metavar='YYY')
>>> parser.add_argument('bar', metavar='XXX')
>>> parser.parse_args('X --foo Y'.split())
Namespace(bar='X', foo='Y')
>>> parser.print_help()
usage:  [-h] [--foo YYY] XXX

positional arguments:
 XXX

optional arguments:
 -h, --help  show this help message and exit
 --foo YYY
```

> 注意`metavar`只会改变**显示出来**的名字；`parse_args()`对象中属性的名字仍然由`dest`的值决定。

`nargs`的不同值可能导致`metavar`使用多次。传递一个列表给`metavar`将给每个参数指定一个不同的显示名字：

```python
>>> parser = argparse.ArgumentParser(prog='PROG')
>>> parser.add_argument('-x', nargs=2)
>>> parser.add_argument('--foo', nargs=2, metavar=('bar', 'baz'))
>>> parser.print_help()
usage: PROG [-h] [-x X X] [--foo bar baz]

optional arguments:
 -h, --help     show this help message and exit
 -x X X
 --foo bar baz
```

### dest参数

大部分`ArgumentParser`动作给`parse_args()`返回对象的某个属性添加某些值。该属性的名字由`add_argument()`的`dest`关键字参数决定。对于位置参数的动作，`dest`通常作为第一个参数提供给`add_argument()`:

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('bar')
>>> parser.parse_args('XXX'.split())
Namespace(bar='XXX')
```

对于可选参数的动作，`dest`的动作通常从选项字符串中推导出来。`ArgumentParser`生成的`dest`的值将是第一个长选项字符串前面的`--`字符串去掉。如果没有提供长选项字符串，`dest`值的获得则是将第一个短选项字符串前面的`-`去掉。任何内部的`-`将被转换为`_`字符，以确保字符串是合法的属性名字。下面的实例将会解释这个行为：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('-f', '--foo-bar', '--foo')
>>> parser.add_argument('-x', '-y')
>>> parser.parse_args('-f 1 -x 2'.split())
Namespace(foo_bar='1', x='2')
>>> parser.parse_args('--foo 1 -y 2'.split())
Namespace(foo_bar='1', x='2')
```

`dest`允许提供自定义的属性名：

```python
>>> parser = argparse.ArgumentParser()
>>> parser.add_argument('--foo', dest='bar')
>>> parser.parse_args('--foo XXX'.split())
Namespace(bar='XXX')
```
