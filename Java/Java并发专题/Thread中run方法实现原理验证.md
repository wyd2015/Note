---
title: 'Thread中run方法实现原理验证'
date: 2019-07-10 21:32:08
categories: 'java'
tag: 'Thread', 'run'
---
# Java中的线程与Linux操作系统线程的关系
每个线程都有一个在进程中唯一的线程标识符，用一个数据类型 `pthread_t` 表示，该数据类型在Linux中就是一个无符号长整型数据。

## 创建新线程
默认情况下，线程在被创建时要被赋予一定的属性，这个属性存放在数据类型 `pthread_attr_t` 中，它包含了线程的调度策略、堆栈的信息、join/detach的状态等。
```c
#include <pthread.h>

/**
 * 方法参数说明：
 * thread: 线程标识符，不是由用户指定，而是由 pthread_create 函数在创建时将新的线程标识符放到这个变量中；
 * attr: 指定线程的属性，可以用 NULL 表示默认属性；
 * start_routine: 指定线程开始运行时需要执行的函数，Java Thread中的run方法通过它进行调用；
 * arg: start_routine函数运行所需要的参数，是一个无类型的指针。
 *
 * 方法返回值说明
 * 若创建成功，返回0；如果出错，则返回错误编号。
 */
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                    void *(*start_routine) (void *), void *arg);

//初始化线程
int pthread_attr_init(pthread_attr_t *attr);

//销毁线程
int pthread_attr_destroy(pthread_attr_t *attr);
```
## 结束线程的几种情况
当以下几种情况发生时，线程结束：
- 线程运行的函数执行了 `return` 语句，也即线程任务已完成；
- 线程调用了 `pthread_eixt` 函数；
- 其它线程调用 `pthread_cancel` 函数结束这个线程；
- `进程`调用 `exec()` 、`exit()` ，线程结束；
- `main()` 函数先结束了，且 main()自己没有调用 `pthread_exit` 来等待所有线程完成任务。

### 一个线程结束，并不意味着与该线程有关的所有信息都会消失，即`僵尸线程`问题。
```c
//retval: 由用户指定的参数，此方法完成后可以通过这个参数获得线程的退出状态
void pthread_exit(void *retval);

//被线程调用，用于取消统一进程中的其他线程，这个待取消的线程由thread参数进行指定，
//如果操作成功，则返回0，否则返回对应的错误码。
int pthread_cancel(pthread_t thread);
```

## 对线程的阻塞
阻塞是线程之间同步的一种方法。

创建一个线程时，要赋予它一定的属性，这其中就包括 `joinable`、`detachable` 的属性，只有被声明成 `joinable` 的线程，
才可以被其它线程 join 。
```c
/**
 * pthread_join函数会让调用它的线程等待 threadid线程运行结束之后再执行。
 * 
 * value_ptr: 存放了其它线程的返回值。
 * 
 * 一个可以被join（阻塞）的线程，仅仅可以被其它的一个线程join，如果同时有多个线程尝试 join 
 * 同一个线程时，最终结果是未知的，线程不能join它自身。
 */
int pthread_join(pthread_t threadid, void *value_ptr);
```
**僵尸线程**： 已经退出了 joinable 状态的线程，但还在等待其它线程调用 pthread_join 来 join阻塞它，以收集
它的退出信息。如果没有其他线程调用 pthread_join方法来阻塞它的话，被它占用的系统资源不会被释放，比如堆栈。  
如果main函数需要长时间运行，且创建pthread_detach，这样等它运行结束，资源就会得到释放。  
>一个线程被使用 pthread_detach处理后，就不能再被改成 joinable 状态了。  
总之，被创建的线程都应该使用 pthread_join 或 pthread_detach 处理掉，防止出现僵尸线程。
```c
pthread_detach (threadid)
pthread_attr_setdetachstate (attr,detachstate)
pthread_attr_getdetachstate (attr,detachstate)
```
## 堆栈管理
POSIX标准没有规定一个线程的堆栈大小. 安全可移植的程序不会依赖于具体实现默认的堆栈限制，而是显式地调用 pthread_attr_setstacksize 来分配足够的堆栈空间.
```c
pthread_attr_getstacksize (attr, stacksize)
pthread_attr_setstacksize (attr, stacksize)
pthread_attr_getstackaddr (attr, stackaddr)
pthread_attr_setstackaddr (attr, stackaddr)
```
## 其它函数
```c
//返回当前线程的id
pthread_self ();

//比较两个线程的id，id不同，返回0；否则返回一个非零值。
pthread_equal (thread1,thread2);
```

# 互斥锁 Mutex
## Mutex作用
Mutex常被用来保护那些可以被多个线程访问的共享资源。
## 使用方法
1. 创建一个互斥锁，即声明一个 pthread_mutex_t 类型的数据，然后初始化，只有初始化之后才能使用；
2. 多个线程尝试锁定这个互斥锁；
3. 只有一个成功锁定互斥锁，成为互斥锁的拥有者，然后进行一些指令操作；
4. 拥有者释放互斥锁；
5. 其他线程尝试锁定互斥锁，重复上面的3~4过程；
6. 最后互斥锁被显式调用 pthread_mutex_destroy来销毁。

## 初始化互斥锁的两种方式
### 1. 利用已经定义的常量进行初始化
```c
pthread_mutex_t clock = PTHREAD_MUTEX_INITIALIZER;
```
### 2. 调用 pthread_mutex_init(mutex, attr)函数进行初始化
当多个线程同时尝试锁定一个互斥锁时，失败的那些线程，如果是用 pthread_mutex_lock 函数，那它会被阻塞，
直到这个互斥锁被释放，它们再继续竞争；

如果是用 pthread_mutex_trylock函数，那么失败者只会返回一个错误。