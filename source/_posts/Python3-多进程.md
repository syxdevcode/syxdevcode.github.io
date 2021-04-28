---
title: Python3-多进程
date: 2021-04-28 15:49:35
tags:
- Python3
- Python
categories: Python3
---

## Process 类

`multiprocessing` 模块通过创建一个 Process 对象然后调用它的 start() 方法来生成进程，Process 与 threading.Thread API 相同。

进程对象，表示在单独进程中运行的活动。

```py
multiprocessing.Process(group=None, target=None, name=None, 
args=(), kwargs={}, *, daemon=None)
```

参数：
* **group**：仅用于兼容 threading.Thread，应该始终是 None。
* **target**：由 run() 方法调用的可调用对象。
* **name**：进程名。
* **args**：目标调用的参数元组。
* **kwargs**：目标调用的关键字参数字典。
* **daemon**：设置进程是否为守护进程，如果是默认值 None，则该标志将从创建的进程继承。
<!--more-->
multiprocessing.Process 对象具有如下方法和属性。

* **run()**：进程具体执行的方法。
* **start()**：启动进程。
* **join([timeout])**：如果可选参数 timeout 是默认值 None，则将阻塞至调用 join() 方法的进程终止；如果 timeout 是一个正数，则最多会阻塞 timeout 秒。
* **name**：进程的名称。
* **is_alive()**：返回进程是否还活着。
* **daemon**：进程的守护标志，是一个布尔值。
* **pid**：返回进程 ID。
* **exitcode**：子进程的退出代码。
* **authkey**：进程的身份验证密钥。
* **sentinel**：系统对象的数字句柄，当进程结束时将变为 ready。
* **terminate()**：终止进程。
* **kill()**：与 terminate() 相同，但在 Unix 上使用 SIGKILL 信号。
* **close()**：关闭 Process 对象，释放与之关联的所有资源。

```py
from multiprocessing import Process
import time
import os

def target():
    time.sleep(2)
    print('子进程ID：', os.getpid())

if __name__ == '__main__':
    print('主进程ID：', os.getpid())
    ps = []
    for i in range(10):
        p = Process(target=target)
        p.start()
        ps.append(p)
    for p in ps:
        p.join()
```

进程池对象。

```py
multiprocessing.pool.Pool([processes[, initializer[, initargs[, maxtasksperchild[, context]]]]])
```

参数：

* **processes**：工作进程数目，如果 processes 为 None，则使用 os.cpu_count() 返回的值。
* **initializer**：如果 initializer 不为 None，则每个工作进程将会在启动时调用 initializer(*initargs)。
* **maxtasksperchild**：一个工作进程在它退出或被一个新的工作进程代替之前能完成的任务数量，为了释放未使用的资源。
* **context**：用于指定启动的工作进程的上下文。

有如下两种方式向进程池提交任务：

* **apply(func[, args[, kwds]])**：阻塞方式。
* **apply_async(func[, args[, kwds[, callback[, error_callback]]]])**：非阻塞方式。

```py
import multiprocessing
import time

def target(p):
    print('t')
    time.sleep(2)
    print(p)

if __name__ == "__main__":
    pool = multiprocessing.Pool(processes=5)
    for i in range(3):
        p = 'p%d' % (i)
        # 阻塞式
        pool.apply(target, (p, ))
        # 非阻塞式
        # pool.apply_async(target, (p, ))
    pool.close()
    pool.join()
```

## 进程间交换数据

### 管道

`multiprocessing.Pipe([duplex])` 返回一对 Connection 对象 (conn1, conn2) ， 分别表示管道的两端；如果 duplex 被置为 True (默认值)，那么该管道是双向的，否则管道是单向的。

```py
from multiprocessing import Pipe, Process


def setData(conn, data):
    conn.send(data)


def printData(conn):
    print(conn.recv())


if __name__ == "__main__":
    data = 'python'
    # 创建管道返回管道的两端
    conn1, conn2 = Pipe()
    p1 = Process(target=setData, args=(conn1, data,))
    p2 = Process(target=printData, args=(conn2,))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

### 队列

`multiprocessing.Queue([maxsize])` 返回一个共享队列实例。具有如下方法：

* **qsize()**：返回队列的大致长度。
* **empty()**：如果队列是空的，返回 True，反之返回 False。
* **full()**：如果队列是满的，返回 True，反之返回 False。
* **put(obj[, block[, timeout]])**：将 obj 放入队列。
* **put_nowait(obj)**：相当于 put(obj, False)。
* **get([block[, timeout]])**：从队列中取出并返回对象。
* **get_nowait()**：相当于 get(False)。
* **close()**：指示当前进程将不会再往队列中放入对象。
* **join_thread()**：等待后台线程。
* **cancel_join_thread()**：防止进程退出时自动等待后台线程退出。

```py
from multiprocessing import Queue, Process

def setData(q, data):
    q.put(data)

def printData(q):
    print(q.get())

if __name__ == "__main__":
    data = 'python'
    q = Queue()
    p1 = Process(target=setData, args=(q, data,))
    p2 = Process(target=printData, args=(q,))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

## 进程间同步

多进程之间不共享数据，但共享同一套文件系统，像访问同一个文件、同一终端打印，如果不进行同步操作，就会出现错乱的现象。

所有在 threading 存在的同步方式，multiprocessing 中都有类似的等价物，如：锁、信号量等。

不加锁

```py
from multiprocessing import Process
import os
import time


def target():
    print('p%s is start' % os.getpid())
    time.sleep(2)
    print('p%s is end' % os.getpid())


if __name__ == '__main__':
    for i in range(3):
        p = Process(target=target)
        p.start()
```

加锁：

```py
from multiprocessing import Process, Lock
import os
import time


def target(lock):
    lock.acquire()
    print('p%s is start' % os.getpid())
    time.sleep(2)
    print('p%s is end' % os.getpid())
    lock.release()


if __name__ == '__main__':
    lock = Lock()
    for i in range(3):
        p = Process(target=target, args=(lock,))
        p.start()
```

## 进程间共享状态

### 共享内存

`multiprocessing.Value(typecode_or_type, *args, lock=True)` 返回一个从共享内存上创建的对象。参数：

* `typecode_or_type`：返回的对象类型。
* `*args`：传给类的构造函数。
* `lock`：如果 lock 值是 True（默认值），将会新建一个递归锁用于同步此值的访问操作；如果 lock 值是 Lock、RLock 对象，那么这个传入的锁将会用于同步这个值的访问操作；如果 lock 是 False，那么对这个对象的访问将没有锁保护，也就是说这个变量不是进程安全的。

`multiprocessing.Array(typecode_or_type, size_or_initializer, *, lock=True)` 从共享内存中申请并返回一个数组对象。

* `typecode_or_type`：返回的数组中的元素类型。
* `size_or_initializer`：如果参数值是一个整数，则会当做数组的长度；否则参数会被当成一个序列用于初始化数组中的每一个元素，并且会根据元素个数自动判断数组的长度。
* `lock`：如果 lock 值是 True（默认值），将会新建一个递归锁用于同步此值的访问操作；如果 lock 值是 Lock、RLock 对象，那么这个传入的锁将会用于同步这个值的访问操作；如果 lock 是 False，那么对这个对象的访问将没有锁保护，也就是说这个变量不是进程安全的。

使用 Value 或 Array 将数据存储在共享内存映射中。

```py
from multiprocessing import Process, Value, Array

def setData(n, a):
    n.value = 1024
    for i in range(len(a)):
        a[i] = -a[i]

def printData(n, a):
    print(num.value)
    print(arr[:])

if __name__ == '__main__':
    num = Value('d', 0.0)
    arr = Array('i', range(5))
    print(num.value)
    print(arr[:])
    print('-----------------------')
    p = Process(target=setData, args=(num, arr))
    p.start()
    p.join()
    print(num.value)
    print(arr[:])
```

### 服务进程

参考：[multiprocessing.Manager()](https://docs.python.org/zh-cn/3/library/multiprocessing.html#multiprocessing.sharedctypes.multiprocessing.Manager)

由 Manager() 返回的管理器对象控制一个服务进程，该进程保存 Python 对象并允许其他进程使用代理操作它们。

Manager() 返回的管理器支持类型包括：list、dict、Namespace、Lock、RLock、Semaphore、BoundedSemaphore、Condition、Event、Barrier、Queue、Value 和 Array。

```py
from multiprocessing import Process, Manager

def setData(d, l):
    d[1] = '1'
    d[0.5] = None
    l.reverse()

if __name__ == '__main__':
    with Manager() as manager:
        d = manager.dict()
        l = manager.list(range(5))
        print(d)
        print(l)
        print('-----------------------')
        p = Process(target=setData, args=(d, l))
        p.start()
        p.join()
        print(d)
        print(l)
```

参考：

[Python 进阶（二）：多进程](https://ityard.blog.csdn.net/article/details/104219452)

[multiprocessing --- 基于进程的并行](https://docs.python.org/zh-cn/3/library/multiprocessing.html#the-process-class)