## 线程的六种状态

参考 

[Thread.State (Java Platform SE 8 ) (oracle.com)](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.State.html)

https://blog.csdn.net/pange1991/article/details/53860651

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

## 线程池的优点

重复利用线程，减少线程频繁创建和销毁的开销

## 线程池有哪些?它们的区别和使用场景？

Java 的线程池有如下四种，其内部实现都是基于 ThreadPoolExecutor

```java
//单线程的线程池（适用于一个接着一个执行任务的场景）
ExecutorService newSingleThreadExecutor = Executors.newSingleThreadExecutor();
//固定大小的线程池（适用于执行少量耗时较长的任务）
ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(int nThreads);
//带缓存的线程池（适用于执行大量耗时较短的任务）
ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
//计划任务线程池（适用于执行周期性的任务）
ExecutorService newScheduledThreadPool = Executors.newScheduledThreadPool(int corePoolSize);
```

它们的构造函数如下

```java
/**
 * 单线程的线程池
 *
 * 线程池中保留的线程数以及允许的最大线程数都是1，额外的线程能够闲置的时间为0s
 * 任务队列的数据结构是 LinkedBlockingQueue
 *
 * @return
 */
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
           (new ThreadPoolExecutor(1, 1,
                                   0L, TimeUnit.MILLISECONDS,
                                   new LinkedBlockingQueue<Runnable>()));
}

/**
 * 固定大小的线程池
 *
 * 线程池中保留的线程数以及允许的最大线程数都是nThreads，额外的线程能够闲置的时间为0s
 * 任务队列的数据结构是 LinkedBlockingQueue
 *
 * @param nThreads
 * @return
 */
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

/**
 * 带缓存的线程池
 *
 * 线程池中保留的线程数是0，允许的最大线程数都是Integer的最大值，
 * 额外的线程能够闲置的时间为60s，任务队列的数据结构是 SynchronousQueue
 * 
 * @return
 */
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

/**
 * 任务计划线程池
 *
 * ScheduledThreadPoolExecutor 其实也是调用 ThreadPoolExecutor 的构造方法
 * 
 * 线程池中保留的线程数是corePoolSize，允许的最大线程数都是Integer的最大值，
 * 额外的线程能够闲置的时间为0s，任务队列的数据结构是 DelayedWorkQueue
 * 
 * @param corePoolSize
 * @return
 */
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

线程池的底层使用的是 ThreadPoolExecutor：

```java
/**
 * Creates a new {@code ThreadPoolExecutor} with the given initial
 * parameters and default thread factory and rejected execution handler.
 * It may be more convenient to use one of the {@link Executors} factory
 * methods instead of this general purpose constructor.
 *
 * @param corePoolSize 线程池保留的线程数
 *        the number of threads to keep in the pool, even
 *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
 * @param maximumPoolSize 线程池允许的最大线程数
 *        the maximum number of threads to allow in the
 *        pool
 * @param keepAliveTime 额外的线程能够闲置的时间
 *        when the number of threads is greater than
 *        the core, this is the maximum time that excess idle threads
 *        will wait for new tasks before terminating.
 * @param unit 时间单位
 *        the time unit for the {@code keepAliveTime} argument
 * @param workQueue 存放待执行任务的工作队列
 *        the queue to use for holding tasks before they are
 *        executed.  This queue will hold only the {@code Runnable}
 *        tasks submitted by the {@code execute} method.
 * @throws IllegalArgumentException if one of the following holds:<br>
 *         {@code corePoolSize < 0}<br>
 *         {@code keepAliveTime < 0}<br>
 *         {@code maximumPoolSize <= 0}<br>
 *         {@code maximumPoolSize < corePoolSize}
 * @throws NullPointerException if {@code workQueue} is null
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}

/**
 * ScheduledThreadPoolExecutor 其实也是调用 ThreadPoolExecutor 的构造方法
 * 
 * Creates a new {@code ScheduledThreadPoolExecutor} with the
 * given core pool size.
 *
 * @param corePoolSize the number of threads to keep in the pool, even
 *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
 * @throws IllegalArgumentException if {@code corePoolSize < 0}
 */
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

由上可知，线程池中常使用的队列是 LinkedBlockingQueue、SynchronousQueue 和 DelayedWorkQueue，另外可用于线程池的队列还有 ArrayBlockingQueue 和 PriorityBlockingQueue。



## [通过 execute(Runnable) 方法向线程池添加任务时线程池的执行流程](https://blog.csdn.net/wangwenhui11/article/details/6760474)： 

一开始，线程池中是没有工作线程的，新增任务时会创建线程，直到线程数等于 corePoolSize，之后的新增任务都会添加到工作队列中，除非工作队列满了。

- 如果当前线程池中线程的数量小于 corePoolSize，则创建新的线程来处理新增任务，即使线程池中的线程都处于空闲状态。
- 如果当前线程池中线程的数量等于 corePoolSize，则把任务放入缓冲队列里。
- 如果缓冲队列满了，且当前线程池中线程的数量小于 maximumPoolSize，则创建新的线程来处理新增任务
- 如果缓冲队列满了，且当前线程池中线程的数量等于 maximumPoolSize，则根据 handler 所指定的策略来拒绝处理此任务。

也就是说，任务的处理方式需要依次判断三个条件是否满了：核心线程数 corePoolSize、工作队列 workQueue、最大线程数 maximumPoolSize，如果三者都满了，则根据 handler 所指定的策略来拒绝处理此任务。

另外当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过 keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

## ThreadPoolExecutor 提供了四种拒绝策略

- ThreadPoolExecutor.AbortPolicy：默认策略，直接抛出 RejectedExecutionException 拒绝执行异常，简单粗暴。
- ThreadPoolExecutor.CallerRunsPolicy：直接调用任务的 run 方法去运行，简单粗暴，不过会阻塞主线程。
- ThreadPoolExecutor.DiscardPolicy：啥都不干，简单粗暴
- ThreadPoolExecutor.DiscardOldestPolicy：抛弃任务队列中最旧的任务也就是最先加入队列的任务，再把这个新任务添加进去。

## 线程相关的方法

Thread 中的方法

- Thread.sleep()：使当前线程睡眠一段时间。JVM内部实现。
- Thread.yield()：提示线程调度器当前线程可以让出它的CPU使用，调度器可以选择忽略该提示。JVM内部实现。yield 使妥协、让步的意思。
- interrupt()：中断该线程。如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。注意该方法不是 Thread.interrupted()，后者是静态方法，判断当前线程是否处于中断状态。
- join()：等待当前线程执行完，才能执行后面的代码。

至于 Object 中线程相关的方法 wait()、notify()、notifyAll() 见下面。

参考 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

## sleep() 和 wait() 的区别

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。
- wait() 只能在同步方法、同步代码块中使用，sleep 可在任何地方使用
- wait() 使线程进入 WAITING 状态，直到其它线程通过 notify() 或 notifyAll() 唤醒，sleep() 使线程进入 TIMED_WAITING 状态。

参考 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

## yeild() 

当前线程由运行状态进入就绪状态，不释放锁，只是将CPU资源让步给优先级相同的线程，但是无法保证一定能达到让步目的，因为当前线程有可能再次被CPU调度运行。

## *volatile 的作用？

volatile 有两个作用，一是保证共享变量的可见性，二是禁止指令重排（重排序优化）。

- 什么是变量的可见性：一个线程对共享变量进行修改，另一个线程能立即看到这个修改。

- **什么是指令重排序优化**：建议结合单例模式中的双重检查锁（DCL）的写法来看。

    实例化一个对象（比如： `Object o = new Objext` ）其实可以分为三个步骤：

    1. 分配内存空间
    2. 初始化对象（初始化成员变量等）
    3. 将内存空间的地址赋值给对象的引用

    但是由于操作系统可以对指令进行重排序，上面的过程也可能会变成如下过程：

    1. 分配内存空间
    2. 将内存空间的地址赋值给对象的引用
    3. 初始化对象

    即123这个步骤变为了132步骤，如果是这个流程，多线程环境下就可能将一个未进行初始化操作、其内部状态还不可用的对象引用暴露出来，从而导致不可预料的结果。

参考：[Java 并发编程：volatile的使用及其原理](https://www.cnblogs.com/paddix/p/5428507.html)

## volatile 可以取代 synchronize 吗？

volatile 无法保证原子性，无法取代。

## join()：**现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行？**

这个问题的目的是检测你对 join() 是否熟悉。

```java
class T1 extends Thread {
    @Override
    public void run() {
        System.out.println("T1");
    }
}

class T2 extends Thread {

    private T1 t1;

    T2(T1 t1) {
        this.t1 = t1;
    }

    @Override
    public void run() {
        try {
            t1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("T2");
    }
}

class T3 extends Thread {

    private T1 t1;
    private T2 t2;

    T3(T1 t1, T2 t2) {
        this.t1 = t1;
        this.t2 = t2;
    }

    @Override
    public void run() {
        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("T3");
    }
}

public class Test {

    public static void main(String[] args) {
        T1 t1 = new T1();
        T2 t2 = new T2(t1);
        T3 t3 = new T3(t1, t2);
        t1.start();
        t2.start();
        t3.start();
    }

}
```

执行结果：

```shell
T1
T2
T3
```



## notify() 和 notifyAll() 的区别

- notify() 唤醒等待该对象的锁的多个线程中的一个

- notify() 唤醒等待该对象的锁的所有线程，让他们去竞争该对象的锁
- 和 wait() 一样，都是 Object 的方法，都是 JVM 内部实现的

## 同步代码块、同步方法、同步锁

同步代码块和同步方法是通过 synchronized 实现的，同步锁是通过 Lock 去实现的。

同步代码块：

```java
public void method() {
    //不需同步的代码
    synchronized (obj) {
		// 需要同步的代码
    }
    //不需同步的代码
}
/**
 * 上面的 obj 就是同步监视器，也就是锁，可以为 this，也可以为类，比如 Example.class。
 * 如果用 this，则锁只作用于同一个对象上，如果调用两个对象上的同步代码块，则互不影响。
 * 如果用类，则调用两个对象上的同步代码块会进行同步。
 */
```

同步方法：

```java
public synchronized void method() {
    // 需要同步的代码
}
```

尽量使用同步代码块而不是同步方法，减少同步范围，从而减少锁竞争。

## 什么是线程安全？什么是线程安全问题？

是指**多线程访问同一数据产生不确定的结果**的问题。

如果数据**不是共享的**，或者**不是可修改的**，也就不存在线程安全问题。

PS：注意区分**线程安全**、**线程安全问题**这几个名词



**线程安全问题的本质是原子性、顺序性、可见性的问题。**

## 线程同步的方法有哪些？

1.使用同步方法

2.使用同步代码块

3.使用同步锁

4.使用volatile修饰变量

## 什么是悲观锁和乐观锁？

- 悲观锁：在对任意记录进行修改前，先要为这条记录加上排它锁，加锁成功后，才对这条记录进行修改，修改完成后才释放锁。

  适合场景：写线程多、读线程少、冲突几率大的场景

  比如：

  - MySQL 的读锁、写锁、行锁等
  - Java 的 synchronized 关键字

  

- 乐观锁：基于**版本号**或 **CAS(Compare And Swap)** 的一种锁，在对任意记录进行修改前，并不会真正的上锁。

  适合场景：读线程多、写线程少、冲突几率低的场景

  - 基于版本号：在对任意记录进行修改前，先读取这条记录的版本号，修改时候对比版本号是否一致，一致的情况下才对记录进行修改，并且版本号加一。版本号不一致则操作失败，之后进行事务回滚或提示。

      ```sql
      update table set name = 'Aron', version = version + 1 where id = #{id} and version = #{version};
      ```
      
      MyBatis-Plus 有乐观锁的实现
      
  - 基于 CAS：当多个线程尝试同时更新同一个变量时，只有其中一个线程能成功，而其它线程都失败，失败的线程可以再次尝试。

      `CAS` 操作中包含三个操作数 :

      - 进行比较的**预期原值** `A`
      - 拟写入的**新值** `B`
      - 需要读写的**内存位置** `V`

      如果内存位置 `V` 的值与预期原值 `A` 相匹配，那么处理器会自动将该位置值更新为新值 `B`，否则处理器不做任何操作。

      `CAS` 有效地说明了“我认为位置 `V` 应该包含值 `A`；如果包含该值，则将 `B` 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可”

      缺点：

      - ABA问题：比如说一个线程 `T1` 从内存位置 `V` 中取出 `A`，这时候另一个线程 `T2` 也从内存位置 `V` 中取出 `A`，并且进行了一些操作将其变成了 `B`，然后又将其变成 `A`，这时候线程 `T1` 进行 `CAS` 操作发现内存中仍然是 `A` ，然后 `T1` 操作成功。尽管线程 `T1` 的 `CAS` 操作成功，但可能存在潜藏的问题。

         



参考：

[乐观锁与悲观锁 — 深入理解Yii2.0](http://www.digpage.com/lock.html)

[乐观锁、悲观锁，这一篇就够了！ - Java成神之路](https://segmentfault.com/a/1190000016611415)

## synchronized 和 ReentrantLock 的区别

- synchronized 是 JVM 内部实现的，而 ReentrantLock 是 JDK 实现的。
- synchronized 会自动释放锁，而 ReentrantLock 需要手动释放。
- synchronized 的锁是非公平的，而 ReentrantLock 的锁支持公平和不公平，但是默认情况下是非公平的，可通过 new ReentrantLock(true) 切换为公平锁。
- ReentrantLock 支持非阻塞锁（获取不到则返回 false）、超时锁、中断锁，并支持绑定多个 Condition 对象。

总的来说，synchronized 有的功能， ReentrantLock 都有，并支持更多高级功能。所以，我们通常说 synchronized 是轻量级锁，而 ReentrantLock 是重量级锁。需要注意的是，JDK1.8 后，两者的性能已经没什么区别，所以如果不需要使用 ReentrantLock 的高级功能，推荐使用 synchronized。

参考 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

## 锁的状态有哪些？

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

参考 [Java 并发](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%B9%B6%E5%8F%91)

## *Java 8 对 syncronized 的优化：偏向锁、轻量级锁、重量级锁

Java 8 对 syncronized 进行了优化(JVM 内部对其进行了优化)，引入了偏向锁、轻量级锁和重量级锁的概念。

- 偏向锁：第一个获取锁对象的线程，在之后再次获取该锁时不需要进行同步操作

- 轻量级锁：当有第二个线程获取锁时，则升级为轻量级锁，竞争锁的线程不会阻塞，而是使用**自旋**的方式去获取锁

- 重量级锁：多个线程共同竞争锁时，则升级为重量级锁，使用**阻塞**的方式去获取锁。

参考：

[偏向锁、轻量级锁、重量级锁的升级以及区别](https://www.jianshu.com/p/9998a9db17f7)



  

## ReentrantLock 获取锁的方式

```java
// 阻塞式获取锁，成功则返回true，否则一直阻塞
reentrantLock.lock();
// 非阻塞式获取锁，成功则返回true，失败则返回false
reentrantLock.tryLock();
// 在指定的时间内获取锁，成功则返回true，失败则一直阻塞，直到超过指定时间返回false
reentrantLock.tryLock(long timeout, TimeUnit unit);
// 可中断获取锁，成功则返回true，否则一直阻塞，直到中断
reentrantLock.lockInterruptibly();
```

## 公平锁和非公平锁的区别

- 公平锁能保证：老的线程排队使用锁，新线程仍然排队使用锁。 
- 非公平锁保证：老的线程排队使用锁，**但是无法保证新线程抢占已经在排队的线程的锁**。

- 非公平锁的效率高于公平锁，因为非公平锁降低了线程被挂起的机率，后来的线程有一定机率逃离被挂起，避免了线程挂起的开销。

两者获取锁的源码解读如下

```java
/**
 * Performs non-fair tryLock.  tryAcquire is implemented in
 * subclasses, but both need nonfair try for trylock method.
 */
// 非公平锁获取锁的方法
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

/**
 * Fair version of tryAcquire.  Don't grant access unless
 * recursive call or no waiters or is first.
 */
// 公平锁获取锁的方法
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 公平锁多出了 hasQueuedPredecessors() 方法
        // 该方法保障了不论是新的线程还是已经排队的线程都顺序使用锁
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

参考 

- https://blog.csdn.net/zhilinboke/article/details/83104597 （写的非常好）
- https://blog.csdn.net/m47838704/article/details/80013056

## 什么是自旋锁？自旋锁的使用场景？

自旋锁： 线程获取锁的时候，如果锁被其他线程持有，则当前线程将循环等待，直到获取到锁。 

自旋锁适合线程占用锁的时间非常短的场景，因为这种情况下自旋的资源消耗会小于线程进入阻塞状态的上下文开销。

参考：[认真的讲一讲：自旋锁到底是什么](https://mp.weixin.qq.com/s/iIoKKjG8re1LCDlQdrfEJg)

## 什么是死锁？如何诊断死锁？如何避免死锁？

死锁是一种现象或状态，是指两个或多个线程之间**互相等待**对方释放所持有的锁而陷入无限等待状态。

一个死锁的例子：

```java
public class DeadLock extends Thread {
    private final String lock1;
    private final String lock2;

    public DeadLock(String name, String lock1, String lock2) {
        super(name);
        this.lock1 = lock1;
        this.lock2 = lock2;
    }

    @Override
    public void run() {
        synchronized (lock1) {
            // do something
            System.out.println(this.getName() + " obtained: " + lock1);

            // sleep for a while
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (lock2) {
                // do something
                System.out.println(this.getName() + " obtained: " + lock2);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        String lockA = "lockA";
        String lockB = "lockB";
        DeadLock t1 = new DeadLock("Thread1", lockA, lockB);
        DeadLock t2 = new DeadLock("Thread2", lockB, lockA);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }
}
```

结果如下：

```bash
Thread1 obtained: lockA
Thread2 obtained: lockB
```

使用 jstack 可以诊断死锁：

```bash
$ jstack <pid>
Found one Java-level deadlock:
=============================
"Thread2":
  waiting to lock monitor 0x000000001c7de178 (object 0x000000076b959558, a java.lang.String),
  which is held by "Thread1"
"Thread1":
  waiting to lock monitor 0x000000001c7de018 (object 0x000000076b959590, a java.lang.String),
  which is held by "Thread2"

Java stack information for the threads listed above:
===================================================
"Thread2":
        at com.mondari.DeadLock.run(DeadLock.java:28)
        - waiting to lock <0x000000076b959558> (a java.lang.String)
        - locked <0x000000076b959590> (a java.lang.String)
"Thread1":
        at com.mondari.DeadLock.run(DeadLock.java:28)
        - waiting to lock <0x000000076b959590> (a java.lang.String)
        - locked <0x000000076b959558> (a java.lang.String)

Found 1 deadlock.
```

当然也可以使用 Java 提供的标准管理 API，ThreadMXBean 来定位死锁：

```java
public static void main(String[] args) throws InterruptedException {
    ThreadMXBean mbean = ManagementFactory.getThreadMXBean();
    Runnable dlCheck = () -> {
        long[] threadIds = mbean.findDeadlockedThreads();
        if (threadIds != null) {
            ThreadInfo[] threadInfos = mbean.getThreadInfo(threadIds);
            System.out.println("Detected deadlock threads:");
            for (ThreadInfo threadInfo : threadInfos) {
                System.out.println(threadInfo.getThreadName());
            }
        }
    };

    ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    // 稍等 5 秒，然后每 10 秒进行一次死锁扫描
    scheduler.scheduleAtFixedRate(dlCheck, 5L, 10L, TimeUnit.SECONDS);

    // 死锁样例代码
}

```

结果如下：

```bash
Thread1 obtained: lockA
Thread2 obtained: lockB
Detected deadlock threads:
Thread2
Thread1
```

但是要注意的是，对线程进行快照本身是一个相对重量级的操作，要慎重选择频率和时机。



如何避免死锁？这里举两个方法

1. 设计好锁的获取顺序

2. 非阻塞式获取锁或带超时获取锁。

   ```java
   if (lock.tryLock() || lock.tryLock(timeout, unit)) {
       // ...
   }
   ```

   

极客时间版权所有: https://time.geekbang.org/column/article/9266



## 两个线程交替打印1-10

参考：[经典线程面试题-两个线程交替打印](https://blog.csdn.net/afsvsv/article/details/86521789)

使用 synchronized、wait、notify 即可实现：

1. - 首先定义一个公共变量 `i = 1` 
   - 再定义一个公共变量 `flag` 用来控制交替打印，`flag = true` 时，线程T1打印，`flag = false` 时，线程T2打印
   - 定义一个对象锁 `lock`

2. 线程T1判断 `flag = true` 时打印，打印完后设置 `flag = false` ，并 `notify` 线程T2。否则 `wait` 等待
3. 线程T2判断 `flag = false` 时打印，打印完后设置 `flag = true` ，并 `notify` 线程T1。否则 `wait` 等待



```java
Object lock = new Object();
// 这里之所以使用 Atomic 类是因为 Lambda 表达式中的变量需要是 final 修饰
AtomicBoolean flag = new AtomicBoolean(true);
AtomicInteger integer = new AtomicInteger();

new Thread(() -> {
    while (integer.get() <= 10) {
        synchronized (lock) {
            if (flag.get()) {
                System.out.println("thread-1: " + integer.getAndIncrement());
                flag.set(false);
                lock.notify();
            } else {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}).start();

new Thread(() -> {
    while (integer.get() <= 10) {
        synchronized (lock) {
            if (!flag.get()) {
                System.out.println("thread-2: " + integer.getAndIncrement());
                flag.set(true);
                lock.notify();
            } else {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}).start();

```



## 生产者消费者问题

参考 https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html

```java
public class Test {

    public static void main(String[] args) {
        BlockingQueue q = new ArrayBlockingQueue(2);
        Producer p = new Producer(q);
        Consumer c1 = new Consumer(q);
        Consumer c2 = new Consumer(q);
        new Thread(p).start();
        new Thread(c1).start();
        new Thread(c2).start();
    }

}

class Producer implements Runnable {
    private final BlockingQueue queue;

    Producer(BlockingQueue q) {
        queue = q;
    }

    @Override
    public void run() {
        try {
            while (true) {
                queue.put(produce());
            }
        } catch (InterruptedException ex) {
            //...
        }
    }

    Object produce() {
        System.out.println("生产");
        return "produce";
    }
}

class Consumer implements Runnable {
    private final BlockingQueue queue;

    Consumer(BlockingQueue q) {
        queue = q;
    }

    @Override
    public void run() {
        try {
            while (true) {
                consume(queue.take());
            }
        } catch (InterruptedException ex) {
            //...
        }
    }

    void consume(Object x) {
        System.out.println("消费");
    }
}
```



## Round-Robin 轮询调度算法

编写一个线程安全的负载均衡轮询调度算法。

```java
/**
 * 线程安全
 */
private static AtomicInteger next = new AtomicInteger();

public static void main(String[] args) {
    // 服务器列表
    List<Object> serverlist = Arrays.asList(0, 1, 2, 3, 4);

    for (int i = 0; i < 10; i++) {
        System.out.println(roundRobbin(serverlist));
    }

}

static Object roundRobbin(List<Object> list) {
	// 核心思想（类似于 HashMap 的散列思想）
    int i = next.getAndIncrement() % list.size();
    return list.get(i);

}
```

结果如下：

```bash
0
1
2
3
4
0
1
2
3
4
```

轮询调度算法假设所有服务器的处理能力都相同，不关心每台服务器的当前连接数和响应速度，当请求服务间隔时间变化比较大时，容易导致服务器间的负载不平衡。所以该算法只适合于服务器组中的所有服务器都有相同的软硬件配置并且平均服务请求相对均衡的情况。

参考：https://blog.csdn.net/wordwarwordwar/article/details/79953880



## ‌如何设计一个高并发系统

要设计一个高并发系统，需要具备以下条件

- 系统拆分：一个系统拆分成多个子系统，每个系统连接一个数据库
- 缓存：高并发请求**读多写少**，所以加缓存提高查询效率，保护后端存储
- MQ：限流，削峰，针对增删改请求
- 分库分表：降低单表单库的压力
- 主从复制，读写分离：MySQL 高可用
- ElasticSearch：