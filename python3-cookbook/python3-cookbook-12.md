# Python3-cookbook-12

### 启动于停止线程

你要为需要并发执行的代码创建或销毁线程。`threading`库可以在单独的线程中执行任何在python中可以调用的对象。你可以创建一个`Thread`对象，并将你要执行的对象以`target`参数的形式提供给该对象。下面是一个简单的例子：

```python
# Code to execute in an independent thread
import time
def countdown(n):
    while n > 0:
        print('T-minus', n)
        n -= 1
        time.sleep(5)

# Create and launch a thread
from threading import Thread
t = Thread(target=countdown, args=(10,))
t.start()
```

当你创建好一个线程对象后，该对象并不会立即执行，除非你调用它的`start()`方法(当你调用`start()`方法时，它会调用你传递进来的函数，并把你传递进来的参数传递给该函数)。Python中的线程会在一个单独的系统级线程中执行(比如说一个POSIX线程或者一个Windows线程)，这些线程将由操作系统来全权管理。线程一旦启动，将独立执行到目标函数返回。你可以查询一个线程对象的状态，看它是否还在执行：

```python
if t.is_alive():
    print('Still running')
else:
    print('Completed')
```

你可以将一个线程加入到当前线程，并等待它终止：

```python
t.join()
```

Python解释器直到所有线程都终止前仍保持运行。对于需要长时间运行的线程或者需要一直运行的后台任务，你应当考虑使用后台线程。例如：

```python
t = Thread(target=countdown, args=(10,), daemon=True)
t.start()
```

> 总结下：

```python
t = threading.Thread(target='', daemon=True/False...)
```

> * `t.join()`表示主线程会等待所有的子线程执行完成后再执行
> * `daemon = True`表示主线程不会等待子线程执行完成；当主线程退出后，所有子线程都结束
> * `daemon`默认为`False`；一般设置为`True`

后台线程无法等待，不过，这些线程会在主线程终止时自动销毁。除了如上所示的两个操作，并没有太多可以对线程做的时请。你无法结束一个线程，无法给它发送信号，无法调整它的调度，也无法执行其它高级操作。如果需要这些特性，你需要自己添加。比如说，如果你需要终止线程，那么这个线程必须通过编程在某个特定点轮询来退出。你可以像下边这样把线程放入一个类中：

```python
class CountdownTask:
    def __init__(self):
        self._running = True

    def terminate(self):
        self._running = False

    def run(self, n):
        while self._running and n > 0:
            print('T-minus', n)
            n -= 1
            time.sleep(5)

c = CountdownTask()
t = Thread(target=c.run, args=(10,))
t.start()
c.terminate() # Signal termination
t.join()      # Wait for actual termination (if needed)
```

如果线程执行一些像I/O这样的阻塞操作，那么通过轮询来终止线程将使得线程只见的协调变得非常棘手。比如，如果一个线程一致阻塞在一个I/O操作上，它就永远无法返回，也就无法检查自己是否已经结束了。要正确处理这些问题，你需要利用超时循环来小心操作线程。例子如下：

```python
class IOTask:
    def terminate(self):
        self._running = False

    def run(self, sock):
        # sock is a socket
        sock.settimeout(5)        # Set timeout period
        while self._running:
            # Perform a blocking I/O operation w/ timeout
            try:
                data = sock.recv(8192)
                break
            except socket.timeout:
                continue
            # Continued processing
            ...
        # Terminated
        return
```

> 注意，由于全局解释锁(GIL)的原因，Python的线程被限制到同一时可只允许一个线程执行。所以，Python的线程更适用于处理I/O和其它需要并发执行的阻塞操作(比如等待I/O、等待从数据库获取数据等等)，而不是需要多处理并行的计算密集型任务。

有时你会看到下边这种通过继承`Thread`类来实现的线程：

```python
from threading import Thread

class CountdownThread(Thread):
    def __init__(self, n):
        super().__init__()
        self.n = n
    def run(self):
        while self.n > 0:

            print('T-minus', self.n)
            self.n -= 1
            time.sleep(5)

c = CountdownThread(5)
c.start()
```

尽管这样也可以工作，但这使得你的代码依赖于`threading`库，所以你的这些代码只能在线程上下文中使用。上文所写的那些代码、函数都是与`threading`库无关的，这样就使得这些代码可以被用在其它的上下文中，可能与线程有关，也可能与线程无关。比如，你可以通过`multiprocessing`模块在一个单独的进程中执行你的代码：

```python
import multiprocessing
c = CountdownTask(5)
p = multiprocessing.Process(target=c.run)
p.start()
```

再次重申，这段代码仅适用于CountdownTask类时以独立于实际的并发手段(多线程、多进程等等)实现的情况。

### 判断线程是否已经启动

你已经启动了一个线程，但是你想直到它不是真的已经开始运行了。线程的一个关键特性是每个线程都是独立运行且状态不可预测。如果程序中的其它线程需要通过判断某个线程的状态来确定自己下一步的操作，这时线程同步问题就会变得非常棘手。为了解决这些问题，我们需要使用`threading`库中的`Event`对象。`Event`对象包含一个可由线程设置的信号标志，它允许线程等待某些事件的发生。在初始情况下，`Event`对象中的信号标志被设置为`False`。如果有线程在等待一个`Event`对象，而这个`Event`对象的标志为假，那么这个线程将会被一致阻塞直至该标志为真。一个线程如果将一个`Event`对象的信号标志设置为真，那么它将忽略这个事件，继续执行。下面的代码展示如何使用`Event`来协调线程的启动：

```python
from threading import Thread, Event
import time

# Code to execute in an independent thread
def countdown(n, started_evt):
    print('countdown starting')
    started_evt.set()
    while n > 0:
        print('T-minus', n)
        n -= 1
        time.sleep(5)

# Create the event object that will be used to signal startup
started_evt = Event()

# Launch the thread and pass the startup event
print('Launching countdown')
t = Thread(target=countdown, args=(10,started_evt))
t.start()

# Wait for the thread to start
started_evt.wait()
print('countdown is running')
```

当你执行这段代码，"countdown is running"总是显示在"countdown starting"之后显示。这是由于使用`evetn`来协调线程，使得主线程要等到`countdown()`函数输出启动信息后，才能继续执行。

> Event()对象主要用于阻塞线程的。

> * Event().is_set()，判断内部线程标识状态，返回bool类型；
> * Event().set()，设置内部所有线程标识状态为`True`；
> * Event().clean()，重置内部线程标识为`False`；
> * Event().wait(*timeout=None*)，在`timeout`时间内会一直等待线程内部标识状态的改变；如果状态为`True`，则立即往下执行，不再等待余下的时间；否则一直等待`timeout`发生(注意，如果`timeout`发生了，则程序会自动将内部线程标识改为`True`，然后再继续往下执行)。如果没有设置`timeout`(像这样`Event().wait()`)，那么线程会一直处于阻塞状态。

我们将上面的代码改成如下：

```python
#!/usr/bin/env python

from threading import Thread, Event
import time
import logging

logging.basicConfig(level=logging.DEBUG, 
               format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

# Code to execute in an independent thread
def countdown(n, started_evt):
    print('countdown starting')
    while n > 0:
        if started_evt.is_set():
            return 0
        print('T-minus', n)
        n -= 1
        time.sleep(1)

def loop(started_evt):
    while not started_evt.is_set():
        started_evt.wait(1)
        logging.debug('loop wait for set => output Jasontom.')

def set_true(started_evt):
    started_evt.wait(10)
    started_evt.set()
    logging.debug('set true status => output ...')

# Create the event object that will be used to signal startup
started_evt = Event()

# Launch the thread and pass the startup event
print('Launching countdown')
t = Thread(target=countdown, args=(50,started_evt))
t.start()

t1 = Thread(target=loop, args=(started_evt,))
t1.start()

t2 = Thread(target=set_true, args=(started_evt,))
t2.start()

# Wait for the thread to start
started_evt.wait()
print('countdown is running')
```

上面的代码中我们定义了三个函数(`countdown`, `loop`, `set_true`)，并分别启动了三个线程；在`loop`和`countdown`函数中我们使用`started_evt.is_set()`一直在判断内部线程状态；而`set_true`函数内通过`started_evt.wait(10)`让内部线程阻塞**10s**后将状态改为`True`，释放所有阻塞状态的内部线程；我们来看看程序的执行结果：

```shell
Launching countdown
countdown starting
T-minus 50
2016-07-17 11:00:41,781 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 49
2016-07-17 11:00:42,783 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 48
T-minus 47
2016-07-17 11:00:43,786 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 46
2016-07-17 11:00:44,789 DEBUG [Thread-2] loop wait for set => output Jasontom.
2016-07-17 11:00:45,791 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 45
T-minus 44
2016-07-17 11:00:46,794 DEBUG [Thread-2] loop wait for set => output Jasontom.
2016-07-17 11:00:47,795 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 43
2016-07-17 11:00:48,796 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 42
2016-07-17 11:00:49,798 DEBUG [Thread-2] loop wait for set => output Jasontom.
T-minus 41
2016-07-17 11:00:50,781 DEBUG [Thread-3] set true status => output ...
countdown is running
2016-07-17 11:00:50,782 DEBUG [Thread-2] loop wait for set => output Jasontom.
```

`countdown`和`loop`两个函数一直循环的执行并一直在获取内部线程状态；当`set_true`函数在阻塞**10s**后将内部线程状态设置为`True`时，`countdown`和`loop`两个函数终止循环，执行结束。

`Event`对象最好单次使用，就是说，你创建一个`Event`对象，让某个线程等待这个对象，一旦这个对象被设置为真，你就应该丢弃它。尽管可以通过`clear()`方法来重置`Event`对象，但是很难确保安全的清理`Event`对象并对它重新赋值。很可能会发生错过事件、死锁或者其它问题(特别是，你无法保证重置`Event`对象的代码会在线程再次等待这个`Event`对象之前执行)。如果一个线程需要不停的重复使用`Event`对象，你最好使用`Condition`对象来代替。

我们先来看看`Condition`的方法说明：

> thread.Condition()	=> condition方法主要用于生产者/消费者模型（producer/consumer）；

> * Condition().wait()，线程挂起，直到收到一个notify通知才会被唤醒继续运行；
> * Condition().notify()，通知其他线程，那些挂起的线程接收到这个通知之后会继续往下执行；
> * Condition().notify_all()，如果 wait() 线程比较多，notify_all() 的作用就是通知所有线程；
> * Condition().acquire()，加锁
> * Condition().release()，解锁

> 注意：消费者应该先启动，让消费者一直处于 wait() 状态
> 注意：如果消费者启动在后，则会爆出 un-acquire lock 的错误

演示如下：

```python
import threading
import logging
import time

logging.basicConfig(level=logging.DEBUG, 
                    format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

def consumer(cond):
    with cond:
        cond.wait()
        logging.debug('consumer')
        cond.notify_all()
        cond.wait()
        logging.debug('yes.......')


def producer(cond):
    with cond:
        time.sleep(2)
        logging.debug('producer')
        cond.notify_all()
        cond.wait()
        logging.debug('est........')
        cond.notify_all()

if __name__ == '__main__':
    cond = threading.Condition()
    c1 = threading.Thread(target=consumer, args=(cond, ))
    c1.start()
    p = threading.Thread(target=producer, args=(cond, ))
    p.start()
```

输出如下：

```shell
[root@localhost local_scripts]# python thread_cond.py 
2016-07-17 17:57:54,029 DEBUG [Thread-2] producer
2016-07-17 17:57:54,030 DEBUG [Thread-1] consumer
2016-07-17 17:57:54,030 DEBUG [Thread-2] est........
2016-07-17 17:57:54,031 DEBUG [Thread-1] yes.......
```

上面的代码中，我们首先启动了两个线程分别指向`producer`和`consumer`，并传递了一个`Condition`对象。

我们再来看个有趣的例子：

```python
#!/usr/bin/env python
import logging
import threading
import time

logging.basicConfig(level=logging.DEBUG, 
                    format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

def kbg(name, cond):
    with cond:
        print('{0}: 一支穿云箭'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 山无棱, 天地合, 乃敢与君绝!'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 紫薇!!!!'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 是你'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 有钱吗? 借点'.format(name))
        cond.notify_all()

def xmg(name, cond):
    with cond:
        cond.wait()
        time.sleep(1)
        print('{0}: 千军万马来相见'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 海可枯, 石可烂, 激情永不散!'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 尔康!!!!'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 是我'.format(name))
        cond.notify_all()
        cond.wait()

        time.sleep(1)
        print('{0}: 滚'.format(name))

if __name__ == '__main__':
    cond = threading.Condition()

    xm_t = threading.Thread(target=xmg, args=('ximige', cond), name='xmg')
    xm_t.start()
    kb_t = threading.Thread(target=kbg, args=('kobage', cond), name='kbg')
    kb_t.start()
```

> `Event`对象的一个重要特点是当它被设置为`True`时会唤醒所有等待它的线程。如果你只想唤醒单个线程，最好是使用信号量或者`Condition`对象来代替。

> 当编写涉及到大量的线程间同步问题的代码时会让你痛不欲生。比较合适的方式使用队列来进行线程间通信或者每个把线程当作一个`Actor`，利用`Actor`模型来控制并发。

### 线程间通信

你的程序中有多个线程，你需要在这些线程只见安全的交换信息或数据。在Python中从一个线程向另一个线程发送数据最安全的方式可能就是适用`queue`库中的队列了。创建一个被多个线程共享的`Queue`对象，这些线程通过使用`put()`和`get()`操作来向队列中添加或删除元素。例如：

```python
from queue import Queue
from threading import Thread

# A thread that produces data
def producer(out_q):
    while True:
        # Produce some data
        ...
        out_q.put(data)

# A thread that consumes data
def consumer(in_q):
    while True:
# Get some data
        data = in_q.get()
        # Process the data
        ...

# Create the shared queue and launch both threads
q = Queue()
t1 = Thread(target=consumer, args=(q,))
t2 = Thread(target=producer, args=(q,))
t1.start()
t2.start()
```

`Queue`对象已经包含了必要的锁，所以你可以通过它在多个线程间安全的共享数据。当使用队列时，协调生产者和消费者的关闭问题可能会有一些麻烦。一个通用的解决方法时在队列中放置一个特殊的值，当消费者读到这个值的时候，终止执行。例如：

```python
#!/usr/bin/env python

import logging
from time import sleep
from queue import Queue
from threading import Thread

logging.basicConfig(level=logging.DEBUG, 
                    format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

_sentinel = object()

def producer(out_q):
    for i in range(10):
        out_q.put(i)
        logging.info('Producer: Queue put {0} ok. '.format(i))
        sleep(1)
    out_q.put(_sentinel)
    logging.info('Producer: Queue put {!r} ok. '.format('_sentinel'))

def consumer(in_q):
    while True:
        data = in_q.get()
        logging.info('Consumer: Queue get {0} ok. '.format(data))
        sleep(1)
        if data is _sentinel:
            in_q.put(_sentinel)
            logging.info('Consumer: Queue put {!r} ok. '.format('_sentinel'))
            break

q = Queue()
t1 = Thread(target=consumer, args=(q,), name='Consumer')
t2 = Thread(target=producer, args=(q,), name='Producer')
t1.start()
t2.start()
```

本列中有一个特殊的地方：消费者在读到这个特殊值之后立即又把它方会到队列中，将之传递下去。这样，所有监听这个队列的消费者线程就可以全部关闭了。尽管队列时最常见的线程间通信机制，但是仍然可以自己通过创建自己的数据结构并添加所需的锁和同步机制来实现线程间通信。最常见的方法是使用`Condition`变量来包装你的数据结构。下边这个例子演示了如何创建一个线程安全的优先级队列：

```python
import heapq
import threading

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._count = 0
        self._cv = threading.Condition()
    def put(self, item, priority):
        with self._cv:
            heapq.heappush(self._queue, (-priority, self._count, item))
            self._count += 1
            self._cv.notify()

    def get(self):
        with self._cv:
            while len(self._queue) == 0:
                self._cv.wait()
            return heapq.heappop(self._queue)[-1]
```

使用队列来进行线程间通信是一个单向、不确定的过程。通常情况下，你没有办法直到接收数据的线程是什么时候接收到的数据并开始工作的。不过队列对象提供了一些基本完善的特性，比如下面这个列子中的`task_done()`和`join()`：

```python
#!/usr/bin/env python

import logging
from time import sleep
from queue import Queue
from threading import Thread

logging.basicConfig(level=logging.DEBUG, 
                    format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

_sentinel = object()

def producer(out_q):
    for i in range(10):
        out_q.put(i)
        logging.info('Producer: Queue put {0} ok. '.format(i))
        sleep(1)
    out_q.put(_sentinel)
    logging.info('Producer: Queue put {!r} ok. '.format('_sentinel'))

def consumer(in_q):
    while True:
        data = in_q.get()
        # Indicate completion
        in_q.task_done()
        logging.info('Consumer: Queue get {0} and task done ok. '.format(data))
        sleep(1)
        if data is _sentinel:
            in_q.put(_sentinel)
            logging.info('Consumer: Queue put {!r} ok. '.format('_sentinel'))
            break

q = Queue()
t1 = Thread(target=consumer, args=(q,), name='Consumer')
t2 = Thread(target=producer, args=(q,), name='Producer')
t1.start()
t2.start()

# Wait for all produced items to be consumed
q.join()
```

如果一个线程需要在一个“消费者”线程处理完特定的数据项时立即得到通知，你可以把要发送的数据和一个`Event`放到一起使用，这样“生产者”就可以通过这个`Event`对象来监测处理的过程了。示例如下：

```python
#!/usr/bin/env python

import logging
from time import sleep
from queue import Queue
from threading import Thread
from threading import Event

logging.basicConfig(level=logging.DEBUG, 
                    format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

_sentinel = object()

def producer(out_q):
    for i in range(10):
        evt = Event()
        out_q.put((i, evt))
        logging.info('Producer: Queue put {0} ok. '.format(i))
        evt.wait()

def consumer(in_q):
    for _ in range(10):
        data, evt = in_q.get()
        logging.info('Consumer: Queue get {0} ok. '.format(data))
        sleep(1)
        evt.set()

q = Queue()
t1 = Thread(target=consumer, args=(q,), name='Consumer')
t2 = Thread(target=producer, args=(q,), name='Producer')
t1.start()
t2.start()
```

基于简单队列编写多线程程序在多数情况下是一个比较明智的选择。从线程安全队列的底层实现来看，你无需在你的代码中使用锁和其它底层的同步机制，这些只会把你的程序弄得乱七八糟。此外，使用队列这种基于消息的通信机制可以被扩展到更大的应用范畴，比如，你可以把你的程序放入多个进程甚至时分布式系统而无需改变底层的队列结构。**使用线程队列由一个需要注意的问题是，向队列中添加数据项时并不会复制此数据项，线程间通信实际上是在线程间传递对象引用。**如果你担心对象的共享状态，那你最好值传递不可修改的数据结构(如：整型、字符串或者元组)或者一个对象的深拷贝。例如：

```python
from queue import Queue
from threading import Thread
import copy

# A thread that produces data
def producer(out_q):
    while True:
        # Produce some data
        ...
        out_q.put(copy.deepcopy(data))

# A thread that consumes data
def consumer(in_q):
    while True:
        # Get some data
        data = in_q.get()
        # Process the data
        ...
```

现在我们演示下线程间传递对象的引用是怎么一回事。我们将上面第二段代码改成如下：

```python
#!/usr/bin/env python

import logging
from time import sleep
from queue import Queue
from threading import Thread
from threading import Event

logging.basicConfig(level=logging.DEBUG, 
                    format='%(asctime)s %(levelname)s [%(threadName)s] %(message)s')

_sentinel = object()

def producer(out_q, evt):
    for i in range(5):
        out_q.put((i, evt))
        logging.info('Producer: Queue put {0} ok. '.format(i))
        evt.wait()

def consumer(in_q):
    for _ in range(5):
        data, evt = in_q.get()
        logging.info('Consumer: Queue get {0} ok. '.format(data))
        sleep(1)
        evt.set()

q = Queue()
evt = Event()
t1 = Thread(target=consumer, args=(q,), name='Consumer')
t2 = Thread(target=producer, args=(q, evt), name='Producer')
t1.start()
t2.start()
```

程序运行结果如下：

```shell
[root@localhost local_scripts]# python queue_test.py 
2016-07-21 15:56:18,989 INFO [Producer] Producer: Queue put 0 ok. 
2016-07-21 15:56:18,990 INFO [Consumer] Consumer: Queue get 0 ok. 
2016-07-21 15:56:19,992 INFO [Producer] Producer: Queue put 1 ok. 
2016-07-21 15:56:19,992 INFO [Producer] Producer: Queue put 2 ok. 
2016-07-21 15:56:19,993 INFO [Producer] Producer: Queue put 3 ok. 
2016-07-21 15:56:19,993 INFO [Producer] Producer: Queue put 4 ok. 
2016-07-21 15:56:19,993 INFO [Consumer] Consumer: Queue get 1 ok. 
2016-07-21 15:56:20,997 INFO [Consumer] Consumer: Queue get 2 ok. 
2016-07-21 15:56:21,999 INFO [Consumer] Consumer: Queue get 3 ok. 
2016-07-21 15:56:23,002 INFO [Consumer] Consumer: Queue get 4 ok. 
[root@localhost local_scripts]# 
```

我们看到只有第一次的`put()`和`get()`如我们所愿正常的执行，这说明我们第一次传递的`Event`对象是成功写入到队列中去了；而后续的`put()`并没有写入`Event`对象，因为第一次的操作已经将`Event`对象引用改变了。

`Queue`对象提供一些在当前上下文很有用的附加特性。比如在创建`Queue`对象时提供可选的`size`参数来限制可以添加到队列中的元素数量。对于"生产者"与"消费者"速度有差异的情况，为队列中的元素数量添加上限是有意义的。比如，一个"生产者"产生项目的速度比"消费者消费"的速度快，那么使用固定大小的队列就可以在队列已满的时候阻塞队列，以免未预期的连锁效应扩散整个程序造成死锁或者程序运行失常。在通信的线程之间进行"流量控制"是一个看起来容易，实现起来困难的问题。如果你发现自己曾经试图通过摆弄队列大小来解决一个问题，这也许就标志着你的程序可能存在脆弱设计或者固有的可伸缩问题。`get()`和`put()`方法都支持非阻塞方式和设定超时，例如：

```python
import queue
q = queue.Queue()

try:
    data = q.get(block=False)
except queue.Empty:
    ...

try:
    q.put(item, block=False)
except queue.Full:
    ...

try:
    data = q.get(timeout=5.0)
except queue.Empty:
    ...
```

关于`Queue`对象的使用请参考：
- [个人笔记](https://github.com/tangjiaxing669/python_learning/blob/master/queue.md)。
- [官方站点](https://docs.python.org/3/library/queue.html?highlight=queue#module-queue)。

这些操作都可以用来避免当执行某些特定队列操作时发生无限阻塞的情况，比如，一个非阻塞的`put()`方法和一个固定大小的队列一起使用，这样当队列已满时就可以执行不同的代码。比如输出一条日志信息并丢弃。

```python
def producer(q):
    ...
    try:
        q.put(item, block=False)
    except queue.Full:
        log.warning('queued item %r discarded!', item)
```

如果你试图让消费者线程在执行像`q.get()`这样的操作时，超时自动终止以便检查终止标志，你应该使用`q.get()`的可选参数`timeout`，如下：

```python
_running = True

def consumer(q):
    while _running:
        try:
            item = q.get(timeout=5.0)
            # Process item
            ...
        except queue.Empty:
            pass
```

最后，有`q.qsize()`，`q.full()`，`q.empty()`等实用方法可以获取一个队列的当前大小和状态。但要注意，这些方法都不是安全的。可能你对一个队列使用`empty()`判断出这个队列为空，但同时另外一个线程可能已经向这个队列中插入一个数据项。所以，你最好不要在你的代码中使用这些方法。
