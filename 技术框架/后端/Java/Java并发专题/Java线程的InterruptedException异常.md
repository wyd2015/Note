# Java线程的InterruptedException异常

## 场景

网课期间的一个关于终止线程执行的例子，代码如下：

```java
@Test
public void test1(){
    Thread t1 = new Thread(() -> {
        while (!Thread.interrupted()) {
            System.out.println(Thread.currentThread().getName()+" is running ...");
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().getName()+"线程异常");
                e.printStackTrace();
            }
        }
    }, "t1");
    t1.start();

    try {
        Thread.sleep(10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    t1.interrupt();
    System.out.println("中断线程");
}
```

但在执行后发现，`t1.interrupt()`方法并未打断while循环。执行结果如下：

```sh
"C:\Program Files\Java\jdk1.8.0_231\bin\java.exe" ...
t1 is running ...
中断线程
t1线程异常
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at org.example.thread.ThreadTest.lambda$test1$1(ThreadTest.java:7)
	at java.lang.Thread.run(Thread.java:748)
t1 is running ...# 这里可以看出，发生异常后，线程依然还在执行

Process finished with exit code 0
```

## 问题

### 1. 为什么发生异常后，线程又执行了一次？

这里需要知道对不同状态的线程执行Interrupt()方法的后果，这里先列出对sleep状态的线程执行interrupt()方法的结果：  

当执行到当在sleep中的线程被t1调用interrupt方法时，t1会放弃睡眠状态，并抛出InterruptedException异常，这样一来，线程的控制权就交给了捕获这个异常的线程t1，t1拿到线程执行权后，因为while循环的条件此时仍为true，所以还会继续执行打印语句。

### 2. interrupt方法为何没生效？

当一个方法后面声明可能抛出`InterruptedException`异常时，说明这个是`可能会花一点时间，但最终可以取消`的方法。

#### 2.1 可能抛出`InterruptedException`异常的方法有：

- `java.lang.Object`类的`wait()`方法；
- `java.lang.Thread`类的`sleep()`方法；
- `java.lang.Thread`类的`join()`方法。

#### 2.2 需要花点时间的方法：

- 执行`wait()`方法的线程，会进入等待区等待被`notify/notifyAll`，在等待期间，线程不活动；
- 执行`sleep()`方法的线程，会暂停执行参数内所设置的时间；
- 执行`join`方法的线程，会等待指定的线程结束为止。

#### 2.3 可以取消的方法：

因为需要花时间的操作会降低程序的响应速度，所以可能会取消/中途放弃这个方法。这里主要是通过`interrupt()`方法来取消。

interrupt()方法是Thread实例的方法，在执行的时候并不需要获取Thread实例的锁定，任何线程在任何时刻，都可以通过线程实例来调用其他线程的interrupt()方法。

##### 2.3.1. sleep()方法和interrupt()方法

当在sleep状态的线程被调用interrupt方法时，会短暂放弃睡眠状态，并抛出`InterruptedException`异常，并将线程控制权交给捕获`InterruptedException`异常的catch块。

```java
/**
  * Causes the currently executing thread to sleep (temporarily cease
  * execution) for the specified number of milliseconds, subject to
  * the precision and accuracy of system timers and schedulers. The thread
  * does not lose ownership of any monitors.
  *
  * @param  millis
  *         the length of time to sleep in milliseconds
  *
  * @throws  IllegalArgumentException
  *          if the value of {@code millis} is negative
  *
  * @throws  InterruptedException
  *          if any thread has interrupted the current thread. The
  *          <i>interrupted status</i> of the current thread is
  *          cleared when this exception is thrown.
  *			 如果当前处于sleep状态的线程被中断，会抛出此异常，并且清除当前线程的中断状态。
  */
public static native void sleep(long millis) throws InterruptedException;
```



##### 2.3.2 wait()方法和interrupt()方法

当线程调用wait方法后，线程会进入等待区，并解除锁定。当对wait状态的线程调用interrupt方法时，需要先重新获取锁定，再抛出`InterruptedException`异常，且在获取锁定之前，无法抛出`InterruptedException`异常。

```java
/**
  * Causes the current thread to wait until either another thread invokes the
  * {@link java.lang.Object#notify()} method or the
  * {@link java.lang.Object#notifyAll()} method for this object, or a
  * specified amount of time has elapsed.
  * <p>
  * 当前线程必须拥有当前共享资源的锁
  * The current thread must own this object's monitor.
  * <p>
  * This method causes the current thread (call it <var>T</var>) to
  * place itself in the wait set for this object and then to relinquish
  * any and all synchronization claims on this object. Thread <var>T</var>
  * becomes disabled for thread scheduling purposes and lies dormant
  * until one of four things happens:
  * <ul>
  * <li>Some other thread invokes the {@code notify} method for this
  * object and thread <var>T</var> happens to be arbitrarily chosen as
  * the thread to be awakened.
  * <li>Some other thread invokes the {@code notifyAll} method for this
  * object.
  * <li>Some other thread {@linkplain Thread#interrupt() interrupts}
  * thread <var>T</var>.
  * <li>The specified amount of real time has elapsed, more or less.  If
  * {@code timeout} is zero, however, then real time is not taken into
  * consideration and the thread simply waits until notified.
  * </ul>
  * The thread <var>T</var> is then removed from the wait set for this
  * object and re-enabled for thread scheduling. It then competes in the
  * usual manner with other threads for the right to synchronize on the
  * object; once it has gained control of the object, all its
  * synchronization claims on the object are restored to the status quo
  * ante - that is, to the situation as of the time that the {@code wait}
  * method was invoked. Thread <var>T</var> then returns from the
  * invocation of the {@code wait} method. Thus, on return from the
  * {@code wait} method, the synchronization state of the object and of
  * thread {@code T} is exactly as it was when the {@code wait} method
  * was invoked.
  * <p>
  * A thread can also wake up without being notified, interrupted, or
  * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
  * occur in practice, applications must guard against it by testing for
  * the condition that should have caused the thread to be awakened, and
  * continuing to wait if the condition is not satisfied.  In other words,
  * waits should always occur in loops, like this one:
  * <pre>
  *     synchronized (obj) {
  *         while (&lt;condition does not hold&gt;)
  *             obj.wait(timeout);
  *         ... // Perform action appropriate to condition
  *     }
  * </pre>
  * (For more information on this topic, see Section 3.2.3 in Doug Lea's
  * "Concurrent Programming in Java (Second Edition)" (Addison-Wesley,
  * 2000), or Item 50 in Joshua Bloch's "Effective Java Programming
  * Language Guide" (Addison-Wesley, 2001).
  *
  * <p>If the current thread is {@linkplain java.lang.Thread#interrupt()
  * interrupted} by any thread before or while it is waiting, then an
  * {@code InterruptedException} is thrown.  This exception is not
  * thrown until the lock status of this object has been restored as
  * described above.
  *
  * <p>
  * Note that the {@code wait} method, as it places the current thread
  * into the wait set for this object, unlocks only this object; any
  * other objects on which the current thread may be synchronized remain
  * locked while the thread waits.
  * <p>
  * This method should only be called by a thread that is the owner
  * of this object's monitor. See the {@code notify} method for a
  * description of the ways in which a thread can become the owner of
  * a monitor.
  *
  * @param      timeout   the maximum time to wait in milliseconds.
  * @throws  IllegalArgumentException      if the value of timeout is
  *               negative.
  * @throws  IllegalMonitorStateException  if the current thread is not
  *               the owner of the object's monitor.
  * @throws  InterruptedException if any thread interrupted the
  *             current thread before or while the current thread
  *             was waiting for a notification.  The <i>interrupted
  *             status</i> of the current thread is cleared when
  *             this exception is thrown.
  *				当前线程处于wait状态时，如果任一线程调用interrupt方法来中断当前线程，当前线程的中断   *			 状态会被清除，并抛出异常。
  * @see        java.lang.Object#notify()
  * @see        java.lang.Object#notifyAll()
  */
public final native void wait(long timeout) throws InterruptedException;
```



##### 2.3.3 join()方法和interrupt()方法

当线程以join状态等待其它线程结束时，同样可以使用interrupt方法取消，但join方法不需要获取锁定，因此与sleep一样，会马上跳到catch程序块。

### 3. interrupt方法干了什么？

interrupt方法其实只是改变了中断状态。而sleep、wait、join这些方法内部，会不断检查中断状态的值，从而自己抛出`InterruptedException`异常。所以，如果在线程进行其他处理时调用了它的interrupt方法，线程不会抛出`InterruptedException`异常，只有当线程遇到sleep、wait、join方法时，才会抛出`InterruptedException`异常。如果没有调用sleep、wait、join方法，或没有在线程里自己检查中断状态，自己抛出`InterruptedException`异常，那么`InterruptedException`异常是不会被抛出来的。

### 4. 线程中断方法比较

- isInterrupt()：线程实例方法，用来检查线程中断状态；

  ```java
  /**
    * Tests whether this thread has been interrupted.  The <i>interrupted
    * status</i> of the thread is unaffected by this method.
    *
    * <p>A thread interruption ignored because a thread was not alive
    * at the time of the interrupt will be reflected by this method
    * returning false.
    *
    * @return  <code>true</code> if this thread has been interrupted;
    *          <code>false</code> otherwise.
    * @see     #interrupted()
    * @revised 6.0
    */
  public boolean isInterrupted() {
      return isInterrupted(false);
  }
  ```

- interrupted()：Thread类的静态方法，不需要具体的线程实例即可被调用，用于检查并清除线程的中断状态。

  ```java
  /**
    * Tests whether the current thread has been interrupted.  The
    * <i>interrupted status</i> of the thread is cleared by this method.  In
    * other words, if this method were to be called twice in succession, the
    * second call would return false (unless the current thread were
    * interrupted again, after the first call had cleared its interrupted
    * status and before the second call had examined it).
    *
    * <p>A thread interruption ignored because a thread was not alive
    * at the time of the interrupt will be reflected by this method
    * returning false.
    *
    * @return  <code>true</code> if the current thread has been interrupted;
    *          <code>false</code> otherwise.
    * @see #isInterrupted()
    * @revised 6.0
    */
  public static boolean interrupted() {
      return currentThread().isInterrupted(true);
  }
  ```

## 解决

线程之所以没有中断，通过上面的分析，可以得到结论：sleep操作影响了线程的中断状态，因此，不要使用sleep操作来代表业务逻辑就行了，即：

```java
@Test
public void test1(){
    Thread t1 = new Thread(() -> {
        while (!Thread.interrupted()) {
            System.out.println(Thread.currentThread().getName()+" is running ...");
        }
    }, "t1");
    t1.start();

    try {
        Thread.sleep(10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    t1.interrupt();
    System.out.println("中断线程");
}
```



