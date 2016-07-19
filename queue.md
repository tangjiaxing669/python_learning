# queue

> 注意，再Python3中，`Queue`模块被重命名为`queue`。

`queue`模块实现了多生产者、多消费者队列。它特别适用于信息必须在多个线程间安全交换的多线程程序中。这个模块中的`Queue`类实现了所有必须的锁语义。

`queue`模块实现了三个队列，主要差别在于取得数据的顺序上。

> * FIFO(First In First Out，先进先出)，此队列中最早加入的会被最先得到。
> * LIFO(Last In First Out，后进先出)，此队列中最后加入的被最先得到(就像栈一样)。
> * 优先级队列，此队列中的数据会被保持有序状态(适用`heapq`模块)，拥有最小值的任务(优先级最高)被最先得到。

`queue`模块定义了下列的类和异常：

### class queue.Queue(maxsize=0)

构造一个FIFO队列。**maxsize**是个整数，指明了队列中能存放的数据个数的上限。一旦达到上线，插入会导致阻塞，直到队列中的数据被消费掉。如果**maxsize**小于或者等于0，那么队列大小没有限制。

### class queue.LifoQueue(maxsize=0)

构造一个LIFO队列。**maxsize**是个整数，指明了队列中能存放的数据个数的上限。一旦达到上线，插入会导致阻塞，直到队列中的数据被消费掉。如果**maxsize**小于或者等于0，那么队列大小没有限制。

### class queue.PriorityQueue(maxsize=0)

构造一个优先级队列。**maxsize**是个整数，指明了队列中能存放的数据个数的上限。一旦达到上线，插入会导致阻塞，直到队列中的数据被消费掉。如果**maxsize**小于或者等于0，那么队列大小没有限制。

拥有最小值的任务会被最先得到(`sorted(list(entries))[0]`的返回值即为拥有最小值的任务)。队列中的数据类型就是如`(priority_number, data)`这样的元组。

### exception queue.Empty

在空的`queue`对象上调用非阻塞的`get()`(或者`get_nowait()`)会抛出此异常。

### exception queue.Full

在满的`queue`对象上调用非阻塞的`put()`(或者`put_nowait()`)会抛出此异常。

# Queue对象

> 注意，`Queue`对象(`Queue`、`LifoQueue`和`PriorityQueue`)提供了下述的公共方法。

### Queue.qsize()

返回队列的近似大小。注意，队列大小大于0并不保证接下来的`get()`调用不会被阻塞，队列大小小于`maxsize`也不保证接下来的`put()`不会被阻塞。

### Queue.empty()

如果队列为空，返回`True`；否则返回`False`。如果`empty()`返回`True`并不保证接下来的`put()`调用不会被阻塞。类似的，如果`empty()`返回`False`也不能保证接下来的`get()`调用不会被阻塞。

### Queue.full()

如果哦队列是满的，则返回`True`，否则返回`False`。如果`full()`返回`True`并不能保证接下来的`get()`调用不会被阻塞。类似的，如果`full()`返回`False`并不能保证接下来的`put()`调用不会被阻塞。

### Queue.put(item[,block[, timeout]])

将`item`放入队列中。`block`默认为`True`；`timeout`默认为`None`；
如果`timeout`和`block`都没有被指定；当队列满的时候，`put()`会一直阻塞；
如果指定`timeout=1`并且`block=True`，那么当队列满的时候，`put()`会等待**1秒**超时后抛出`Queue.Full`异常；
如果指定`block=False`(当`block=False`时，`timeout`参数将被忽略，指定与不指定都没用)表示队列满时不等待超时直接抛出异常；

### Queue.put_nowait(item)

等同于`Queue.put(item, block=False)`；表示队列满时不等待直接抛出异常。

### Queue.get([block, [timeout]])

表示从队列中移除并返回一个数据。`block`默认为`True`；`timeout`默认为`None`；
如果`timeout`和`block`都没有被指定；当队列为空的时候，`get()`会一直阻塞；
如果指定`timeout=1`并且`block=True`，那么当队列为空的时候，`get()`会等待**1秒**超时后抛出`Queue.Empty`异常；
如果指定`block=False`(当`block=False`时，`timeout`参数将被忽略，指定与不指定都没用)表示队列为空时不等待超时直接抛出异常；

### Queue.get_nowait()

等同于`Queue.get(block=False)`；表示队列为空时不等待直接抛出异常。

### Queue.task_done()

意味着之前入队的一个任务已经完成。由队列的消费者线程调用。每一个`get()`调用得到一个任务，接下来的`task_done()`调用告诉队列该任务已经处理完毕。

如果当前一个`join()`正在阻塞，它将在队列中所有任务都处理完成时恢复执行(即每一个由`put()`调用入队的任务都有一个对应的`task_done()`调用)。

> 注意，如果该方法被调用的次数多于被放入队列中任务的个数，则会抛出`ValueError`异常。

### Queue.join()

阻塞调用线程，直到队列中所有任务都被处理掉。

> 注意，只要有数据被加入队列，未完成的任务数就会增加。当消费者线程调用`task_done()`（意味着消费者取得任务并完成任务），未完成的任务数就会减少。当未完成的任务数降到0，`join()`解除阻塞。
