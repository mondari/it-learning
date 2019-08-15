[TOC]

## 进程与线程的区别

- **进程**：进程是正在运行的程序。
- **线程**：线程是进程的执行单元，是程序执行的最小单位。
- **两者的关系**：
  一个进程可以拥有多个线程，一个线程必须有一个父进程。线程拥有自己的堆栈，它与父进程的其他线程共享该进程所拥有的全部资源。

## 线程的六种状态

参考 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

重点关注线程的几种状态及其如何转换。

## 线程的创建方法

创建线程主要有3种方法：

1. 实现 Runnable 接口的 run() 方法（无返回值）
2. 实现 Callable 接口的 call() 方法（有返回值且能抛出异常）
3. 线程池创建线程

注意：

1. 继承 Thread 类与实现 Runnable 接口的区别

   Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口

2. 实现 Callable 接口与 Runnable 接口的区别

   Callable 执行的方法是 call() ，而 Runnable 执行的方法是 run()。
   call() 方法有返回值还能抛出异常， run() 方法则没有没有返回值，也不能抛出异常。

参考 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

## 线程池有哪些?

Java 的线程池有如下四种，其内部实现都是基于 ThreadPoolExecutor

```java
//单线程的线程池
ExecutorService newSingleThreadExecutor = Executors.newSingleThreadExecutor();
//固定大小的线程池
ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(int nThreads);
//带缓存的线程池
ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
//计划任务线程池
ExecutorService newScheduledThreadPool = Executors.newScheduledThreadPool(int corePoolSize);
```

## 线程相关的方法



## sleep() 和 wait() 的区别

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。
- wait() 使当前线程等待，直到其它线程通过 notify() 或 notifyAll() 唤醒，且wait() 只能用在同步方法或者同步控制块中，否则会抛出 IllegalMonitorStateException。
- sleep() 使当前线程休眠一段时间。

来自 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

## notify() 和 notifyAll() 的区别

- notify() 唤醒等待该对象的锁的多个线程中的一个

- notify() 唤醒等待该对象的锁的所有线程，让他们去竞争该对象的锁
- 和 wait() 一样，都是 Object 的方法，都是 JVM 内部实现的

## synchronized 和 ReentrantLock 的区别



来自 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

