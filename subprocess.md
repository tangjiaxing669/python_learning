# subprocess - 管理子进程

### 1. subprocess模块方法

> 启动一个子进程建议的方式是使用下面的便捷函数。当它们不能满足你的需求时可以使用底层的Popen接口。

**subprocess.call(args, \*, stdin=None, stdout=None, stderr=None, shell=False)**

> 执行**args**描述的命令。**等待命令完成**，返回状态码(**returncode**)属性。

> 注意；**shell**默认为**False**，表示使用Python的执行器来执行**args***描述的命令；如果**shell**为**True**，则会使用系统默认环境来执行**args**描述的命令；默认为`/bin/sh`。

例子：

```shell
>>> subprocess.call(["ls", "-l"])
total 0
drwxrwx--- 2 root root 6 Jun 12 21:37 gateone
0

>>> subprocess.call("exit 1", shell=True)
1
```

<hr>

**subprocess.check_call(args, \*, stdin=None, stdout=None, stderr=None, shell=False)**

> 执行**args**描述的命令。**等待命令完成**，如果返回码是**0**则立即返回命令执行的信息，否则抛出**CalledProcessError**。**CalledProcessError**对象会把返回码保存在**returncode**属性中。

例子：

```shell
>>> subprocess.check_call(["ls", "-l"])
total 0
drwxrwx--- 2 root root 6 Jun 12 21:37 gateone
0

>>> subprocess.check_call('exit 1', shell=True)
---------------------------------------------------------------------------
CalledProcessError                        Traceback (most recent call last)
<ipython-input-13-e9d26287d6f6> in <module>()
----> 1 subprocess.check_call('exit 1', shell=True)

/usr/lib64/python2.7/subprocess.pyc in check_call(*popenargs, **kwargs)
    540         if cmd is None:
    541             cmd = popenargs[0]
--> 542         raise CalledProcessError(retcode, cmd)
    543     return 0
    544 

CalledProcessError: Command 'exit 1' returned non-zero exit status 1
```

<hr>

**subprocess.check_output(args, *, stdin=None, stderr=None, shell=False, universal_newlines=False)**

> 执行**args**描述的命令，并将它的输出作为字节字符串返回。

> 注意，正常情况它不会返回返回码(**returncode**)，但是方法内部会自动做返回码(**returncode**)的判断；如果返回码(**returncode**)非零，将会引发**CalledProcessError**异常，而返回码(**returncode**)将会包含在**CalledProcessError**异常中。

例子：

```shell
>>> subprocess.check_output(['ls', '-l'])
'total 0\ndrwxrwx--- 2 root root 6 Jun 12 21:37 gateone\n'

>>> subprocess.check_output('exit 1', shell=True)
---------------------------------------------------------------------------
CalledProcessError                        Traceback (most recent call last)
<ipython-input-23-acd69db21aa3> in <module>()
----> 1 subprocess.check_output('exit 1', shell=True)

/usr/lib64/python2.7/subprocess.pyc in check_output(*popenargs, **kwargs)
    573         if cmd is None:
    574             cmd = popenargs[0]
--> 575         raise CalledProcessError(retcode, cmd, output=output)
    576     return output
    577 

CalledProcessError: Command 'exit 1' returned non-zero exit status 1
```

> **exception subprocess.CalledProcessError**

> `check_call()`或者`check_output()`运行的进程在返回非零的退出状态时所抛出的异常。

如下：

```python
>>> import subprocess
>>> try:
...     out_bytes = subprocess.check_output("echo 'jason tom'; exit 1", shell=True)
... except subprocess.CalledProcessError as e:
...     out_bytes = e.output
...     code = e.returncode
...
>>> out_bytes
b'jason tom\n'
>>> code
1
>>>
```

<hr>

### 2. 经常用到的参数

> 为了支持广泛的使用场景，Popen构造函数(以及便捷函数)接受大量的可选参数；而这些参数在大部分的使用场景中都可以保持为默认值。下面是最常用到的参数：

【1】**args**：在所有的调用中都需要且应该是一个字符串或者是一个程序参数的序列。一般倾向于提供一个参数序列，因为这可以让该模块来处理参数的转义和引用(例如允许文件名中包含空格等)。如果传递一个单一的字符串，那么`shell`必须为`True`或者是该字符串必须只是简单的指明要执行的程序的名字且不带任何参数。

【2】**stdin、stdout、stderr**：这三个参数分别指向程序的标准输入、标准输出和标准错误文件的句柄。合法的值有`PIPE`、一个已经存在的文件描述符(一个正整数)、一个已经存在的文件对象和`None`。`PIPE`表示为子进程创建一个新的管道。如果默认设置为`None`，则不会发生重定向；子进程的文件句柄将从父进程中继承。另外，`stderr`可以为`STDOUT`，它表示来自子进程中标准错误的数据应该被捕获到和标准输出相同的文件中。
> 注意，当**stdout**或者**stderr**为管道且**universal_newlines**为**True**时，那么所有的行结束符将被转换为`\n`，正如**open()**的**universal newlines**的`U`模式参数所描述的一样。

【3】**shell**：如果`shell`为`True`，那么指定的命令将通过`shell`来执行。如果你使用`Python`作为主要的控制流并仍然像方便的访问`shell`功能，例如`shell`管道、文件通配符、环境变量的扩展以及**`~`**访问某个用户的`home`目录，那么这将非常有用。然而，请注意`Python`自身提供许多类似`shell`功能的实现(特别地，如`glob`，`fnmatch`，`os.walk()`，`os.path.expandvars()`，`os.path.expanduser()`和`shutil`)。

> 注意，为什么使用**shell=True**会导致安全问题？
> 在执行**shell**命令时，如果来自不可信任的输入源将会使得程序容易受到[**shell注入**](http://en.wikipedia.org/wiki/Shell_injection#Shell_injection)攻击，一个严重的安全缺陷可能导致执行任意的命令。因为这个原因，在命令字符串是从外部输入的情况下使用**shell=True**是强烈不建议的。如下：

```python
>>> from subprocess import call
>>> filename = input("What file would you like to display?\n")
What file would you like to display?
non_existent; rm -rf / #
>>> call("cat " + filename, shell=True) # Uh-oh. This will end badly...
```

> **shell=True**将禁用所有基于**shell**的功能，所以不会受此漏洞影响；当使用**shell=True**时，**pipes.quote()**可以用来正确的转义字符串中将用来构造**shell**命令的空白和**shell**元字符。

<hr>

### 3. Popen构造函数

> 该模块中进程的创建和管理底层是通过`Popen`类来处理的。

**class subprocess.Popen(args, bufsize=0, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=False, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0)**

> 在新的进程中执行子程序。在Unix上，该类使用**os.execvp()**类执行子进程的行为；在Windows上，该类使用Windows的**CreateProcess()**函数执行子进程的行为。

【1】**args**：`args`应该是一个程序参数的序列或者是一个单一的字符串。默认情况下，如果`args`是一个序列，那么它的执行程序将是`args`序列的第一个元素。如果`args`是一个字符串，那么其解释将与平台无关；建议传递一个序列给`args`。在Unix上，如果`args`是一个字符串，那么该字符串将解释为要执行的程序名称或程序路径；然而，这只能在不传递参数给程序时才可行。如下：

```python
>>> import shlex, subprocess
>>> command_line = raw_input()
/bin/vikings -input eggs.txt -output "spam spam.txt" -cmd "echo '$MONEY'"
>>> args = shlex.split(command_line)
>>> print args
['/bin/vikings', '-input', 'eggs.txt', '-output', 'spam spam.txt', '-cmd', "echo '$MONEY'"]
>>> p = subprocess.Popen(args) # Success!
```

> 注意，`shlex.split()`在决定`args`正确的粉刺时非常有用，特别是复杂的情形；另外，`shell`中由空格分隔的选项(例如**-input**)和参数(例如**eggs.txt**)在不同的列表元素中，而`shell`中需要引号和反斜线转义的参数(例如包含空格的文件名和上面展示的**echo**命令)是单一的列表元素。

> 在Windows上，如果`args`是一个序列，它会将序列转换为一个字符串；这是因为底层的`CreateProcess()`需要在字符串上运作。

【2】**shell**：默认为`False`，指定是否使用shell来执行程序。如果`shell`为`True`，那么你需要传递一个字符串给`args`。在Unix上，如果`shell=True`，那么默认`shell`为`/bin/sh`。如果`args`是一个字符串，改字符串指令通过shell来执行。这意味着该字符串必须和在shell提示符中输入的格式完全一致，包括引号和反斜线的转移以及文件名中的空格。如果`args`是一个序列，那么序列中的第一个元素将指向命令执行程序，其他额外的任何元素都将作为shell本身额外的参数。等同于下：

```python
Popen(['/bin/sh', '-c', args[0], args[1], ...])
```

> 在Windows上，如果`shell=True`，那么**COMSPEC**环境变量将指向默认的shell。在Windows上，你唯一需要指定`shell=True`是当你想要执行的命令是内建在shell中的时候(例如，**dir**和**copy**)。你不需要`shell=True`来运行一个批处理文件和基于控制台的可执行程序。

【3】**bufsize**：默认值为**0**(不缓存)；含义和内建的`open()`函数对应参数相同；**0**表示不缓存，**1**表示按行缓存，其他正数表示使用(近似)该大小的缓存，负数表示使用系统默认的缓存值，通常表示完全缓存。

> 注意，如果你遇到性能问题，建议你尝试启用缓存并设置**bufsize**为**-1**，或者是一个足够大的值(例如4096)。

【4】**executable**：该参数指定一个要执行的替换程序；它很少被用到。当`shell=False`时，`executable`将替换`args`中要执行的程序；但是，原始的`args`仍然会传递给该程序。大部分程序将`args`当作命令的名字，它可以不同于真实执行的程序。在Unix上，`args`名字变成`utilities`中可执行程序的显示名字，例如`ps`。如果`shell=True`，在Unix上`executable`参数表示指定一个替换默认的`/bin/sh`的shell。例如：

```shell
>>> import subprocess
>>> subprocess.Popen(['ls', '-lh'], executable='df')
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   50G   14G   37G  27% /
devtmpfs                 7.9G     0  7.9G   0% /dev
tmpfs                    7.9G   24K  7.9G   1% /dev/shm
tmpfs                    7.9G   17M  7.9G   1% /run
tmpfs                    7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/centos-home   42G  9.4G   33G  23% /home
/dev/sda1                497M  102M  395M  21% /boot
tmpfs
```

> 当`shell=False`时，实际就是将`executable`替换到`args`中要执行的程序，而参数不变，如果`executable`替换的程序没有该参数的话，那么程序将会抛出原执行程序的异常，如下：

```python
>>> import subprocess
>>> subprocess.Popen(['ls', '-lhTL'], executable='df')
ls: invalid option -- 'L'
Try 'ls --help' for more information.
```

> 当`shell=True`时，那么`executable`参数应该指向一个默认的`/bin/sh`的shell；如下：

```shell
>>> import subprocess
>>> subprocess.Popen('ls -l', executable='/bin/bash', shell=True)
-rw-r--r-- 1 root root  99391 Feb 25 17:25 content.txt
-rw-r--r-- 1 root root    773 Feb  3 17:34 create_domain_timeout.py
-rw-r--r-- 1 root root    287 Feb  6 15:35 deploy_fabric.py
```

【5】**stdin、stdout、stderr**：分别指向要执行的程序的标准输入、标准输出和标准错误文件句柄。合法的值有`PIPE`、一个已经存在的文件描述符(一个正整数)、一个已经存在的文件对象和`None`。`PIPE`表示为子进程创建一个新的管道。如果默认设置为`None`，则不会发生重定向；子进程的文件句柄将从父进程中继承。另外，`stderr`可以为`STDOUT`，它表示来自子进程中标准错误的数据应该被捕获到和标准输出相同的文件中。例如：

```python
>>> import subprocess
>>> out_file = open('../outfile.log', 'wt')
>>> err_file = open('../errfile.log', 'wt')
>>> subprocess.Popen('ls -l', shell=True, stdout=out_file, stderr=err_file)
>>> out_file.close()
>>> err_file.close()
>>>
>>> out_file = open('../outfile.log', 'wt')
>>> err_file = open('../errfile.log', 'wt')
>>> subprocess.Popen('ldfs -l', shell=True, stdout=out_file, stderr=err_file)
>>> subprocess.Popen('ldfs -l', shell=True, stdout=out_file, stderr=err_file)
>>> out_file.close()
>>> err_file.close()
```

现在我们看下文件中的内容：

```shell
[root@localhost python]# cat outfile.log 
total 360
-rw-r--r-- 1 root root  99391 Feb 25 17:25 content.txt
-rw-r--r-- 1 root root    773 Feb  3 17:34 create_domain_timeout.py
-rw-r--r-- 1 root root    287 Feb  6 15:35 deploy_fabric.py
-rw-r--r-- 1 root root    687 Feb  6 15:35 deploy_fabric.pyc
[root@localhost python]# 
[root@localhost python]# cat errfile.log 
/bin/sh: ldfs: command not found
/bin/sh: ldfs: command not found
```

【6】**preexec_fn**：可设定为一个可调用对象，该对象将在子进程执行之前调用；只在`Unix`平台有用。

【7】**close_fds**：此参数如果为真(True)，所有的描述符除了`0`、`1`和`2`之外将会在子进程执行之前关闭(只在Unix系统上)。而在Windows上，`close_fds`如果为真，那么子进程不会继承人和句柄。注意，在Windows上，你可以不设置`close_fds`为真并同时通过设置`stdin`，`stdout`或者`stderr`重定向到标准句柄。

【8】**cwd**：如果不为`None`，那么子进程在执行之前将子进程执行的目录改为`cwd`。注意在搜索可执行程序时，该目录不会考虑在内。例如：

```python
>>> import subprocess
>>> subprocess.Popen('pwd', cwd='/home/')
/home
>>> subprocess.Popen('pwd', cwd='/etc/')
/etc
```

【9】**env**：默认为`None`，表示子进程继承父进程的环境变量；如果不为`None`，那么它必须是一个定义了新进程环境变量的映射；注意，如果该选项被指定，那么`env`必须提供要执行程序的任何变量。

【10】**universal_newlines**：如果为`True`，那么文件对象`stdout`和`stderr`作为文本文件以[universal newlines](http://python.usyiyi.cn/python_278/glossary.html#term-universal-newlines)模式打开(通用换行符)。每一行的终止符可能是Unix风格的`\n`，旧式Macintosh风格的`\r`或者Windows风格的`\r\n`。所有这些外部的表示在Python程序看来都是`\n`。注意，该功能只在Python构建过程中选择支持统一换行符(默认行为)时才可用。同时文件对象`stdout`、`stdin`和`stderr`的换行符属性不会被`communicate()`方法更新。

【11】**startupinfo、creatiionflags**：只在Windows上有效。

<hr>

### 4. 异常

> 在子进程中引发的异常通过会在父进程中被重新引发。另外该异常对象将包含一个额外的属性**child_traceback**，它是一个包含子进程回溯信息的字符串。

引发最常见的异常是`OSError`。例如，当试图执行一个不存在的文件时会发生该异常。应用程序应该提前准备好捕获`OSError`异常。

如果`Popen`的调用带有非法的参数将引发`ValueError`。

如果调用的进程返回非零的返回码，`check_call()`和`check_output()`将引发`CalledProcessError`。

<hr>

### 5. Popen对象

> `Popen`类的实例具有以下方法：

【1】**Popen.poll()**
检查子进程是否已经终止。设置并返回`returncode`属性。

【2】**Popen.wait()**
等待子进程终止。设置并返回`returncode`属性。

【3】**Popen.communicate(input=None)**
与子进程交互，将数据发送到标准输入，然后从标准输出和标准错误读取数据，直到文件末尾或等待进程终止。可选的`input`参数应该是一个要发送给子进程的字符串，如果没有数据要发送给子进程则应该为`None`。`communicate()`会返回一个元组(`stdoutdata`,`stderrdata`)。注意，如果你需要发送数据到进程的标准输入，那么你在创建`Popen`对象的时候应该指定`stdin=PIPE`；类似的，在返回的结果元组中若要得到非`None`的数据，你就必须给出`stdout=PIPE`或者`stderr=PIPE`或者同时给出。例如：

```python
>>> import subprocess
>>> s = subprocess.Popen("read -p '>>> ' name; echo $name", stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
>>> s.communicate(input='jason tom')
('jason tom\n', '')
>>>
>>> s = subprocess.Popen("whoami", stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True)
>>> s.communicate(input=None)
('root\n', '')
```

> 注意，读取的数据是缓存在内存中的，所以如果数据大小很大或者无限制的话，请不要使用这个方法。

【4】**Popen.send_signal(signal)**
发送信号`signal`给子进程。

【5】**Popen.terminate()**
终止子进程。在Posix操作系统上，该方法发送SIGTERM给子进程。

【6】**Popen.kill()**
杀死子进程。在Posix操作系统上，改函数发送SIGKILL给子进程。

> **注意，请使用`communicate()`而不要用`.stdin.write`，`.stdout.read`或者`.stderr.read`，以避免由于其他操作系统管道缓存填满并阻塞子进程引起的死锁。**

【7】**Popen.stdin**
如果`stdin`参数为`PIPE`，则该属性为一个文件对象，它提供给子进程输入。否则为`None`。

【8】**Popen.stdout**
如果`stdout`参数为`PIPE`，则该属性为一个文件对象，它提供给子进程输出。否则为`None`。

【9】**Popen.stderr**
如果`stderr`参数为`PIPE`，则该属性为一个文件对象，它提供给子进程的标准错误输出。否则为`None`。

【10】**Popen.pid**
子进程的进程ID；如果你设置了`shell=True`，那么它是产生的shell进程ID。

【11】**Popen.returncode**
子进程的返回码，由`poll()`和`wait()`设置(`communicate()`是间接设置)。`None`表示子进程还没有终止。一个负值`-N`表示子进程被信号`N`终止了(仅在Unix上)。

### 6. 实例，替换shell的管道

```python
>>> import subprocess
>>> commands = 'dmesg | grep sda'
>>> p1 = subprocess.Popen(["dmesg"], stdout=subprocess.PIPE)
>>> p2 = subprocess.Popen(["grep", "sda"], stdin=p1.stdout, stdout=subprocess.PIPE)
>>> p1.stdout.close()   # Allow p1 to receive a SIGPIPE if p2 exits
>>> result = p2.communicate()[0]    # [0]为stdout，[1]为stderr
```

> 注意，在`p2`之后调用`p1.stdout.close()`是非常重要的，如果`p2`在`p1`之前退出，那么`p1`会收到一个`SIGPIPE`的信号。

> 另外一种方法，就是对于可信任的输入可以直接使用`shell`：

```python
>>> subprocess.check_output("dmesg | grep sda", shell=True)
```
