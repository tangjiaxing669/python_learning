# logging 模块

### 简单实用

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import logging

logging.debug('debug message')
logging.info('info message')
logging.warning('warn message')
logging.error('error message')
logging.critical('critical message')
```

输出：

```
WARNING:root:warn message
ERROR:root:error message
CRITICAL:root:critical message
```

默认情况下，`logging`模块将日志打印到屏幕上(stdout)，日志级别为`WARNING`(即只有日志级别高于`WARNING`的日志信息才会输出)，日志格式如下：
![此处输入图片的描述][1]


  [1]: http://upload-images.jianshu.io/upload_images/477558-da69f58ffd67989c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240

### 日志级别

|  Level  |Numeric value|
|:-------:|:-----------:|
|CRITICAL |50           |
|ERROR    |40           |
|WARNING  |30           |
|INFO     |20           |
|DEBUG    |10           |
|NOTSET   |0            |

### 简单配置

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import logging

# 通过下面的方式进行简单配置输出方式与日志级别
logging.basicConfig(filename='logger.log', level=logging.INFO)

logging.debug('debug message')
logging.info('info message')
logging.warning('warn message')
logging.error('error message')
logging.critical('critical message')
```

输出：
> 标准输出(屏幕)未显示任何信息，发现当前工作目录下生成了**logger.log**文件，如下：

```shell
[root@localhost ~]# cat logger.log
INFO:root:info message
WARNING:root:warn message
ERROR:root:error message
CRITICAL:root:critical message
```

因为通过`level=logging.INFO`设置日志级别为**INFO**，所以日志级别高于**INFO**的都输出来了。

### logging.basicConfig(**kwargs)说明

创建一个带默认**Formatter**的**StreamHandler**，将其加到**root logger**，从而配置基本的日志系统。函数`debug()`，`info()`，`warning()`，`error()`和`critiical()`在**root logger**没有**handler**的情况下会自动调用`basicConfig()`。

> 如果`root logger`已经配置了`handler`，则该函数什么都不做。

支持以下关键字：

|  格式  |描述|
|:-------:|:-----------|
|filename |创建一个`FileHandler`，使用指定的文件名，而不是使用`StreamHandler`(将日志输出到文件)。|
|filemode |如果指明了文件名，指明打开文件的模式（如果没有指明`filemode`，默认为`a`）。|
|format   |指定`handler`使用的字符串格式(指定日志输出格式)。|
|datefmt  |使用指定的`date/time`格式。可参照`time.strftime()`格式。|
|style    |如果指定了`format`选项，则使用`style`指定的格式字符串来输出。其中`%`，`{`或`$`来替换`format`的`%`。|
|level    |指定`root logger`的级别(指定日志级别)。|
|stream   |使用指定的`stream`来初始化`StreamHandler`。这个参数与`filename`不兼容，如果两个参数都被指定，则抛出`ValueError`异常。|
|handlers |如果该值被指定，则该值应该是一个创建的`handler`程序的迭代，并已添加到`root logger`。拥有`formatter`格式的的任何`handler`程序将被分配到创建这个的函数中去，并且是默认格式。注意，这个参数与`filename`和`stream`不兼容，如果两个参数都被指定，则抛出`ValueError`异常。|

现在我们看下`format`的参数：

|       Format      | Description |
|:-----------------:|:-----------|
|%(asctime)s        |打印日志的时间。如果`format`指定了`datefmt`则会覆盖此选项。|
|%(created)f        |打印日志创建的时间(`LogRecord`函数创建的时间，返回的是`time.time()`)。|
|%(filename)s       |打印当前执行程序名称。|
|%(funcName)s       |打印调用日志记录的函数名。|
|%(levelname)s      |打印日志级别名称(`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`)。|
|%(levelno)s        |打印日志级别的数值。|
|%(lineno)d         |打印日志的当前行号。|
|%(module)s         |打印模块名(文件名的部分)。|
|%(msecs)d          |打印`LogRecord`函数被创建时的时间，返回四舍五入后的毫秒。|
|%(message)s        |打印日志信息。|
|%(name)s           |打印调用的`logger`名称。|
|%(pathname)s       |打印当前执行程序的路径。|
|%(process)d        |打印进程ID。|
|%(processName)s    |打印进程名称。|
|%(relativeCreated)d|打印以毫秒为单位，当`LogRecord`被创建时，相对于日志模块被加载的时间。|
|%(thread)d         |打印线程id。|
|%(threadName)s     |打印线程名称。|

下面我们演示下：

```python
#!/usr/bin/env python

import logging

logging.basicConfig(format='%(asctime)s-[%(levelname)s]-%(message)s', level=logging.INFO)

logging.critical('critical log. ')
logging.error('error log. ')
logging.warning('warning log. ')
logging.info('info log. ')
logging.debug('debug log. ')
```

输出如下：

```shell
2016-07-11 22:27:47,330-[CRITICAL]-critical log.
2016-07-11 22:27:47,330-[ERROR]-error log.
2016-07-11 22:27:47,330-[WARNING]-warning log.
2016-07-11 22:27:47,331-[INFO]-info log.
```

另一个例子：

```python
#!/usr/bin/env python

import logging

logging.basicConfig(format='%(asctime)s-[%(levelname)s]-%(message)s',
                    level=logging.INFO,
                    filename='logger.log',
                    datefmt='%Y-%m-%d %H:%M:%S',
                    filemode='a',
                    style='%')

logging.critical('critical log. ')
logging.error('error log. ')
logging.warning('warning log. ')
logging.info('info log. ')
logging.debug('debug log. ')
```

输出如下：

```shell
[root@localhost ~]# cat logger.log
2016-07-11 23:03:24-[CRITICAL]-critical log.
2016-07-11 23:03:24-[ERROR]-error log.
2016-07-11 23:03:24-[WARNING]-warning log.
2016-07-11 23:03:24-[INFO]-info log.
```

> 注意，我们设置的日志级别`level`都是`logging.INFO`，所以只有日志级别高于`logging.INFO`的日志条目才会被显示出来。
