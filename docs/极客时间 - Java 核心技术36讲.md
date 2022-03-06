## 模块一 Java 基础

## 模块二 Java 进阶

## 线程篇

## [第15讲 | synchronized和ReentrantLock有什么区别呢？](https://time.geekbang.org/column/article/8799)

synchronized 和 ReentrantLock 有什么区别？

什么是线程安全？

## [第16讲 | synchronized底层如何实现？什么是锁的升级、降级？](https://time.geekbang.org/column/article/9042)

锁的升级和降级？

偏斜锁（Biased Locking）、轻量级锁（Lightweight Lock）和重量级锁（Heavyweight Lock）？

synchronized 底层如何实现？

ReentrantReadWriteLock 和 StampedLock 的使用？

什么是自旋锁？自旋锁的使用场景？

## [第17讲 | 一个线程两次调用start()方法会出现什么情况？](https://time.geekbang.org/column/article/9103)

一个线程两次调用 start() 方法会出现什么情况？

线程生命周期的状态变化是怎样的？

什么是线程？

线程的基本操作方法介绍

**ThreadLocal 源码分析**

**一个简单的 HelloWorld 程序会创建几个线程？**

## [第22讲 | AtomicInteger底层实现原理是什么？如何在自己的产品代码中应用CAS操作？](https://time.geekbang.org/column/article/9788)

AtomicInteger 原理？

什么是 CAS（Compare And Swap） ？

CAS 的使用场景？

AQS（AbstractQueuedSynchronizer）的原理及应用？

## JVM 篇

## [第23讲 | 请介绍类加载过程，什么是双亲委派模型？](https://time.geekbang.org/column/article/9946)

类加载的过程？类加载分为三步：Loading（加载），Linking（链接）和 Initializing（初始化）

参考：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html

什么是双亲委派模型？

双亲委派模型的缺点？

类加载过程中静态变量和静态常量如何加载？

类加载器有哪些？

如何自定义类加载器？

如何降低类加载的开销？

什么是 Jar Hell 问题，如何解决？

## [第24讲 | 有哪些方法可以在运行时动态生成一个Java类？](https://time.geekbang.org/column/article/10076)

字节码操纵技术的应用？

- 动态代理
- 各种 Mock 框架
- ORM 框架
- IOC 容器
- 部分 Profiler 工具，或者运行时诊断工具等
- 生成形式化代码的工具

## [第25讲 | 谈谈JVM内存区域的划分，哪些区域可能发生OutOfMemoryError?](https://time.geekbang.org/column/article/10192)

## [第33讲 | 后台服务出现明显“变慢”，谈谈你的诊断思路？](https://time.geekbang.org/column/article/11651)

- **系统性能分析**：CPU、内存和 IO（磁盘 IO 和网络 IO） 是主要关注项。

  top、free、iostat

- **JVM 性能分析**：

  - 利用 JMC、JConsole 等工具进行运行时监控。
  - 利用各种工具，在运行时进行堆转储分析，或者获取各种角度的统计数据（如[jstat](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html) -gcutil 分析 GC、内存分带等）。
  - GC 日志等手段，诊断 Full GC、Minor GC，或者引用堆积等。

- **对应用进行 Profiling**：JFR(Java Flight Recorder) + JMC(Java Mission Control)

  一般不建议在生产系统进行 Profiling，大多数是在性能测试阶段进行。建议使用 JFR 配合 [JMC](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) 来做 Profiling，因为它是从 Hotspot JVM 内部收集底层信息，并经过了大量优化，性能开销非常低，通常是低于 **2%** 的。

