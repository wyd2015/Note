---
title: '并发关键字-synchronized'
Date: 2019-03-14 14:09:48
categories： 计算机原理
tags： CPU
---
1. synchronized关键字锁的是对象，不是代码块；
```java
public class Demo1 {
  private int count = 10;
  private Object object = new Object();

  public void test(){
      System.out.println("object["+count+"]=> "+object);
      synchronized (object){
        count--;
        System.out.println(Thread.currentThread().getName() + " count = " + count);
      }
  }

  /**
   * 这里由于只创建一个Demo1实例demo1，两个线程通过demo1调用test方法时，被锁的是在实例中创建的
   * 对象object，两个线程调用test方法时，被锁定的是同一个对象，线程操作安全
   */
  public static void main(String[] args) {
    Demo1 demo1 = new Demo1();
    new Thread(demo1::test, "test1").start();
    new Thread(demo1::test, "test2").start();
  }

//执行结果如下：
object[10]=> java.lang.Object@6e154e44
test1 count = 9
object[9]=> java.lang.Object@6e154e44
test2 count = 8
```

如果是下面的形式：
```java
  //这样被锁定的是两个不同Demo1实例中的object对象，两线程调用test方法不受影响。线程操作不安全
  public static void main(String[] args) {
    Demo1 demo1 = new Demo1();
    Demo1 demo2 = new Demo1();
    new Thread(demo1::test, "test1").start();
    new Thread(demo2::test, "test2").start();
  }
}

//执行结果如下：
object[10]=> java.lang.Object@688ee48d
object[10]=> java.lang.Object@332ba300
test2 count = 9
test1 count = 9
```

2. `synchronized(this){}`中，锁定的是当前类的实例。
```java
public class Demo2 {
  private int count = 10;

  public void test(){
    //synchronized(this)锁定的是当前类的实例,这里锁定的是Demo2类的实例
    synchronized (this){
      count--;
      System.out.println(Thread.currentThread().getName() + " count = " + count);
    }
  }

  //直接加在方法声明上，相当于是synchronized(this)
  public synchronized void test(){
    count--;
    System.out.println(Thread.currentThread().getName() + " count = " + count);
  }
}
```