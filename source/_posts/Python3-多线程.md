---
title: Python3-多线程
date: 2021-04-26 14:47:53
tags:
- Python3
- Python
categories: Python3
---

## 线程简介

每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

每个线程都有他自己的一组CPU寄存器，称为线程的上下文，该上下文反映了线程上次运行该线程的CPU寄存器的状态。

指令指针和堆栈指针寄存器是线程上下文中两个最重要的寄存器，线程总是在进程得到上下文中运行的，这些地址都用于标志拥有线程的进程地址空间中的内存。

* 线程可以被抢占（中断）。
* 在其他线程正在运行时，线程可以暂时搁置（也称为睡眠） -- 这就是线程的退让。

线程可以分为:

* 内核线程：由操作系统内核创建和撤销。
* 用户线程：不需要内核支持而在用户程序中实现的线程。

Python3 线程中常用的两个模块为：
* _thread
* threading(推荐使用)

<!--more-->
`thread` 模块已被废弃。用户可以使用 `threading` 模块代替。所以，在 Python3 中不能再使用"thread" 模块。为了兼容性，Python3 将 thread 重命名为 "_thread"。

Python中使用线程有两种方式：函数或者用类来包装线程对象。

函数式：调用 `_thread` 模块中的 `start_new_thread()` 函数来产生新线程。语法如下:

```py
_thread.start_new_thread ( function, args[, kwargs] )
```

参数说明:

* function - 线程函数。
* args - 传递给线程函数的参数,他必须是个tuple类型。
* kwargs - 可选参数。

```py
#!/usr/bin/python3

import _thread
import time

# 为线程定义一个函数


def print_time(threadName, delay):
    count = 0
    while count < 5:
        time.sleep(delay)
        count += 1
        print("%s: %s,%s" % (threadName, time.ctime(time.time()), count))


# 创建两个线程
try:
    _thread.start_new_thread(print_time, ("Thread-1", 2, ))
    _thread.start_new_thread(print_time, ("Thread-2", 4, ))
except:
    print("Error: 无法启动线程")

while 1:
    pass
```

## threading模块

Python3 通过两个标准库 `_thread` 和 `threading` 提供对线程的支持。

_thread 提供了低级别的、原始的线程以及一个简单的锁，它相比于 threading 模块的功能还是比较有限的。

`threading` 模块的直接方法和属性：

* **threading.enumerate()** 以列表形式返回当前所有存活的 threading.Thread 对象。
* **threading.active_count()** 返回当前存活的 threading.Thread 对象，等于 len(threading.enumerate())。
* **threading.current_thread()** 返回当前对应调用者控制的 threading.Thread 对象，如果调用者的控制线程不是利用 threading 创建，则会返回一个功能受限的虚拟线程对象。
* **threading.get_ident()** 返回当前线程的线程标识符，它是一个非零的整数，其值没有直接含义，它可能会在线程退出，新线程创建时被复用。
* **threading.main_thread()** 返回主线程对象，一般情况下，主线程是 Python 解释器开始时创建的线程。
* **threading.stack_size([size])** 返回创建线程时用的堆栈大小，可选参数 size 指定之后新建线程的堆栈大小，size 值需要为 0 或者最小是 32768（32KiB）的一个正整数，如不指定 size，则默认为 0。
* **threading.get_native_id()** 返回内核分配给当前线程的原生集成线程 ID，其值是一个非负整数。
* **threading.TIMEOUT_MAX** 指定阻塞函数（如：Lock.acquire()， Condition.wait() …）中形参 timeout 允许的最大值，传入超过这个值的 timeout 会抛出 OverflowError 异常。

Python 守护线程基本概念

* 守护线程：当一个线程被标记为守护线程时，Python 程序会在剩下的线程都是守护线程时退出，即等待所有非守护线程运行完毕；守护线程在程序关闭时会突然关闭，可能会导致资源不能被正确释放的的问题，如：已经打开的文档等。

* 非守护线程：通常我们创建的线程默认就是非守护线程，Python 程序退出时，如果还有非守护线程在运行，程序会等待所有非守护线程运行完毕才会退出。

### 线程对象

```py
threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
```

参数说明:

* **group**：通常默认即可，作为日后扩展 ThreadGroup 类实现而保留。
* **target**：用于 run() 方法调用的可调用对象，默认为 None。
* **name**：线程名称，默认是 Thread-N 格式构成的唯一名称，其中 N 是十进制数。
* **args**：用于调用目标函数的参数元组，默认为 ()。
* **kwargs**：用于调用目标函数的关键字参数字典，默认为 {}。
* **daemon**：设置线程是否为守护模式，默认为 None。

threading.Thread 的方法和属性。

* **start()**：启动线程。
* **run()**：线程执行具体功能的方法。
* **join(timeout=None)**：当 timeout 为 None 时，会等待至线程结束；当 timeout 不为 None 时，会等待至 timeout 时间结束，单位为秒。
* **is_alive()**：判断线程是否存活。
* **getName()**：返回线程名。
* **setName()**：设置线程名。
* **isDaemon()**：判断线程是否为守护线程。
* **setDaemon()**：设置线程是否为守护线程。
* **name**：线程名。
* **ident**：线程标识符。
* **daemon**：线程是否为守护线程。

**实例化 threading.Thread:**

```py
import threading
import time

def target(sleep):
    time.sleep(sleep)
    print('当前线程为:', threading.current_thread().name, ' ', 'sleep:', sleep)

if __name__ == '__main__':
    t1 = threading.Thread(name='t1', target=target, args=(1,))
    t2 = threading.Thread(name='t2', target=target, args=(2,))
    t1.start()
    t2.start()
    print('主线程结束')
```

**继承 threading.Thread**

```py
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, sleep, name):
        super().__init__()
        self.sleep = sleep
        self.name = name

    def run(self):
        time.sleep(self.sleep)
        print('name：' + self.name)

if __name__ == '__main__':
    t1 = MyThread(1, 't1')
    t2 = MyThread(1, 't2')
    t1.start()
    t2.start()
```

## 锁对象

**threading.Lock**

实现原始锁对象的类，一旦一个线程获得一个锁，会阻塞随后尝试获得锁的线程，直到它被释放，通常称其为互斥锁，它是由 `_thread` 模块直接扩展实现的。方法：

* acquire(blocking=True, timeout=-1)：可以阻塞或非阻塞地获得锁，参数 blocking 用来设置是否阻塞，timeout 用来设置阻塞时间，当 blocking 为 False 时 timeout 将被忽略。
* release()：释放锁。
* locked()：判断是否获得了锁，如果获得了锁则返回 True。

**threading.RLock**

可重入锁（也称递归锁）类，一旦线程获得了重入锁，同一个线程再次获取它将不阻塞，重入锁必须由获取它的线程释放。方法：

* acquire(blocking=True, timeout=-1)：可以阻塞或非阻塞地获得锁。
* release()：释放锁。

```py
import threading

# 创建锁
lock = threading.Lock()
a = 5

def oper(b):
    # 获取锁
    lock.acquire()
    global a
    a = a - b
    a = a + b
    # 释放锁
    lock.release()

def target(b):
    for i in range(1000000):
        oper(b)

if __name__ == '__main__':
    m = 5
    while m > 0:
        t1 = threading.Thread(target=target, args=(1,))
        t2 = threading.Thread(target=target, args=(2,))
        t1.start()
        t2.start()
        t1.join()
        t2.join()
        print(a)
        m = m - 1
```

## 条件对象

**threading.Condition(lock=None)** 实现条件对象的类

方法：
* **acquire(*args)**：请求底层锁。
* **release()**：释放底层锁。
* **wait(timeout=None)**：等待直到被通知或发生超时。
* **wait_for(predicate, timeout=None)**：等待直到条件计算为 True，predicate 是一个可调用对象且它的返回值可被解释为一个布尔值。
* **notify(n=1)**：默认唤醒一个等待该条件的线程。
* **notify_all()**：唤醒所有正在等待该条件的线程。

使用条件对象的典型场景是将锁用于同步某些共享状态的权限，那些关注某些特定状态改变的线程重复调用 wait() 方法，直到所期望的改变发生；对于修改状态的线程，它们将当前状态改变为可能是等待者所期待的新状态后，调用 notify() 方法或者 notify_all() 方法。

```py
import time
import threading

# 创建条件对象
c = threading.Condition()
privilege = 0

def getPri():
    global privilege
    c.acquire()
    c.wait()
    print(privilege)
    c.release()

def updPri():
    time.sleep(5)
    c.acquire()
    global privilege
    privilege = 1
    c.notify()
    c.release()

if __name__ == '__main__':
    t1 = threading.Thread(target=getPri)
    t2 = threading.Thread(target=updPri)
    t1.start()
    t2.start()
```

## 信号量对象

一个信号量管理一个内部计数器，该计数器因 acquire() 方法的调用而递减，因 release() 方法的调用而递增。 计数器的值永远不会小于零；当 acquire() 方法发现计数器为零时，将会阻塞，直到其它线程调用 release() 方法。

**threading.Semaphore(value=1)** 信号量实现类，可选参数 value 赋予内部计数器初始值，默认值为 1。

* **acquire(blocking=True, timeout=None)**：获取一个信号量，参数 blocking 用来设置是否阻塞，timeout 用来设置阻塞时间。
* **release()**：释放一个信号量，将内部计数器的值增加1。

```py
import threading

# 创建信号量对象
s = threading.Semaphore(10)

a = 5
def oper(b):
    # 获取信号量
    s.acquire()
    global a
    a = a - b
    a = a + b
    # 释放信号量
    s.release()

def target(b):
    for i in range(1000000):
        oper(b)

if __name__ == '__main__':
    m = 5
    while m > 0:
        t1 = threading.Thread(target=target, args=(1,))
        t2 = threading.Thread(target=target, args=(2,))
        t1.start()
        t2.start()
        t1.join()
        t2.join()
        print(a)
        m = m - 1
```

## 事件对象

**threading.Event** 一个线程发出事件信号，其他线程等待该信号。

方法：

* **is_set()**：当内部标志为 True 时返回 True。
* **set()**：将内部标志设置为 True。
* **clear()**：将内部标志设置为 False。
* **wait(timeout=None)**：阻塞线程直到内部变量为 True。

```py
import time
import threading

# 创建事件对象
event = threading.Event()

def dis_class():
    time.sleep(5)
    event.wait()
    print('同学们下课了')

def bell():
    time.sleep(3)
    print('下课铃声响了')
    event.set()

if __name__ == '__main__':
    t1 = threading.Thread(target=bell)
    t2 = threading.Thread(target=dis_class)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

参考：

[Python3 多线程](https://www.runoob.com/python3/python3-multithreading.html)

[threading --- 基于线程的并行](https://docs.python.org/zh-cn/3.10/library/threading.html#)