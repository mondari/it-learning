[TOC]

## 值传递与引用传递

值传递是传递变量的值，引用传递时传递对象的引用地址，对于这类问题，应该结合 JVM 内存模型中的虚拟机栈和堆来看。

[JAVA中值传递和引用传递的三种情况](https://blog.csdn.net/zhzhao999/article/details/53449504)

[java中的值传递和引用传递问题](https://www.cnblogs.com/coderising/p/5697986.html)

## 类加载的过程

一般来说，我们把 Java 的类加载过程分为三个主要步骤：加载、链接、初始化

加载（Loading）：是将 Java 字节码数据从不同的数据源读取到 JVM 中，并映射为 JVM 认可的数据结构（Class 对象）。这里的数据源可能是各种各样的形态，如 jar 文件、class 文件，甚至是网络数据源等；如果输入数据不是 ClassFile 的结构，则会抛出 ClassFormatError。加载阶段是用户参与的阶段，我们可以自定义类加载器，去实现自己的类加载过程。
链接（Linking）：这是核心的步骤，简单说是把原始的类定义信息平滑地转化入 JVM 运行的过程中。
初始化阶段（initialization）：这一步真正去执行类初始化的代码逻辑，包括静态字段赋值，以及静态初始化块执行，编译器在编译阶段就会把这部分逻辑整理好，父类型的初始化逻辑优先于当前类型的逻辑。

双亲委派模型，简单说就是当类加载器（Class-Loader）试图加载某个类型的时候，除非父加载器找不到相应类型，否则尽量将这个任务代理给当前加载器的父加载器去做。使用委派模型的目的是避免重复加载 Java 类型。

## JVM 内存模型

- 程序计数器（Program Counter Register）是**线程私有**的一块内存区域，表示当前线程所执行的字节码的行号指示器。
- Java 虚拟机栈（Java Virtual Machine Stacks）是**线程私有**的一块内存区域，线程执行的每个 Java 方法在虚拟机栈中都会创建一个栈帧（Stack Frame ），用于存放方法入参、局部变量（包括对象的引用）等数据。
- 本地方法栈（Native Method Stacks）与虚拟机栈的作用相同，只不过前者是 Java 方法，而后者是本地方法。
- Java 堆（Java Heap）是**线程共享**的一块内存区域，用于存放对象实例、数组。另外需要注意一点，垃圾回收是作用在堆上的。**堆分为新生代（Eden和Survivor）和老生代**
- 方法区（Method Area）是**线程共享**的一块内存区域，用于存放已被虚拟机加载的类信息、类变量、常量等数据。
- 运行时常量池（Runtime Constant Pool），它是方法区的一部分，用于存放编译期生成的各种字面量和符号引用。

## 堆和栈的区别

- 堆需要手动申请和释放，栈由系统自动分配和释放
- 堆存放对象的实例、数组等数据，栈存放方法的入参、局部变量等数据
- 堆有垃圾回收，栈没有

## JVM 监控工具

[JVM性能调优监控工具专题一：JVM自带性能调优工具（jps,jstack,jmap,jhat,jstat,hprof)](https://www.iteye.com/blog/josh-persistence-2161848)

## jps：打印 Java 进程状态信息

jps -l 打印 Java 进程状态信息，-m 打印 main 方法的入参，-v 打印 JVM 的参数

```shell
[root@10 ~]# jps -l
24226 arthas-demo.jar
25598 sun.tools.jps.Jps
[root@10 ~]# jps -lm
24226 arthas-demo.jar
25609 sun.tools.jps.Jps -lm
[root@10 ~]# jps -lv
24226 arthas-demo.jar
25621 sun.tools.jps.Jps -Dapplication.home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.222.b10-1.el7_7.x86_64 -Xms8m
```



### jstack：查看 Java 进程内线程的堆栈信息，诊断死锁

下面的结果可以看出该进程有 8 个线程

```shell
[root@10 ~]# jstack 24226
2019-09-26 13:38:36
Full thread dump OpenJDK 64-Bit Server VM (25.222-b10 mixed mode):

"Attach Listener" #8 daemon prio=9 os_prio=0 tid=0x00007f7354001000 nid=0x5f86 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007f737c13e800 nid=0x5eaa runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007f737c13b800 nid=0x5ea9 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007f737c12d000 nid=0x5ea8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007f737c12a800 nid=0x5ea7 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007f737c101000 nid=0x5ea6 in Object.wait() [0x00007f736c3de000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000e3408ed8> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x00000000e3408ed8> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007f737c0fc000 nid=0x5ea5 in Object.wait() [0x00007f736c4df000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000e3406c00> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000e3406c00> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"main" #1 prio=5 os_prio=0 tid=0x00007f737c04b800 nid=0x5ea3 waiting on condition [0x00007f738362b000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at java.lang.Thread.sleep(Thread.java:340)
        at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
        at demo.MathGame.main(MathGame.java:17)

"VM Thread" os_prio=0 tid=0x00007f737c0f2800 nid=0x5ea4 runnable

"VM Periodic Task Thread" os_prio=0 tid=0x00007f737c141000 nid=0x5eab waiting on condition

JNI global references: 5
```



### jmap：查看 Java 进程堆内存使用状况

一般使用 `jmap -heap pid` 命令来查看，另外还会使用 `jmap -dump:format=b,file=dumpFileName pid` 命令将进程内存 dump 到文件中，再配合 jhat 或 VisualVM 分析。

### jstat：JVM 统计监控工具，查看 GC 信息

```
jstat pid [interval[s|ms] [count]] ]
```

interval 是采样时间间隔。count 是采样数目。比如下面输出的是GC信息，采样时间间隔为250ms，采样数为4：

```shell
root@ubuntu:/# jstat -gc 21711 250 4 
S0C    S1C    S0U    S1U      EC       EU        OC         OU       PC     PU    YGC     YGCT    FGC    FGCT     GCT   
192.0  192.0   64.0   0.0    6144.0   1854.9   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
192.0  192.0   64.0   0.0    6144.0   1972.2   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
192.0  192.0   64.0   0.0    6144.0   1972.2   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
192.0  192.0   64.0   0.0    6144.0   2109.7   32000.0     4111.6   55296.0 25472.7    702    0.431   3      0.218    0.649
```



### *图形化监控工具：jconsole，jmc，jvisualvm



## 找出某个Java进程中最耗费CPU的线程并打印堆栈信息

1. `jps -l` 查看 Java 进程的 pid
2. `top -Hp pid` ，从中查看 TIME+ 列最大的线程的 pid
3. `printf '%x\n' thread_pid` 将线程的 pid 转为 16 进制
4. `jstack pid | grep thread_pid` 查看线程的堆栈信息

## *[Minor GC、Major GC、Full GC的区别](https://youzhixueyuan.com/the-difference-between-minor-gc-major-gc-full-gc.html)