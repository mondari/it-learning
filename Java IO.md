[TOC]



## 字符流和字节流的区别

## 五种IO模型

|      | 阻塞       | 非阻塞   |
| ---- | ---------- | -------- |
| 同步 | 阻塞IO     | 非阻塞IO |
| 异步 | IO多路复用 | 异步IO   |

只有异步IO才是真正的异步，其它都是同步IO。

Java有三种IO，BIO是阻塞IO模型，NIO是IO多路复用模型，而AIO是异步IO模型。

任何IO过程中都包含**两个步䠫**：第一是**等待**；第二是**拷贝**。在实际的应用场景中等待消耗的时间往往都远远高于拷贝的时间，所以让IO更高效的**最核心办法是减少等待的时间。**

### 阻塞IO(blocking)

1. 用户线程发起系统调用（**recvfrom**），内核会去查看数据是否就绪，如果没有就绪就会等待数据就绪，此时用户线程处于阻塞状态，并交出CPU资源。
2. 当数据就绪之后，内核会将数据从内核拷贝到用户空间，并返回结果给用户线程，用户线程才会解除阻塞状态。

用户线程自始至终处于阻塞状态。

### 非阻塞IO(nonblocking)

1. 用户线程发起系统调用（**recvfrom**），内核会去查看数据是否就绪，如果没有就绪就会立刻返回 **ERROR**(EWOULDBLOCK)。
2. 用户线程收到**ERROR**就会知道是数据还没有准备好，之后它可以再次发起系统调用，不停去轮询数据是否准备就绪，这个阶段比较占用CPU资源。
3. 一旦数据准备就绪，并且内核收到用户线程发起的系统调用（**recvfrom**），就会将数据从内核拷贝到用户空间，并返回结果给用户线程。

用户线程只在**拷贝**阶段处于阻塞状态，在**等待**阶段处于非阻塞状态。

### IO多路复用(I/O multiplexing)

与非阻塞IO的区别是，前者是通过**select**去轮询所有注册到该select的IO数据有无准备好，如果都没有准备好，select就会阻塞；只要任意一个IO需要的数据准备好了，select就会返回，然后进程再通过recvfrom去读取；后者是通过**recvfrom**去轮询和读取。

IO多路复用通过一个线程就可以管理多个IO，适合连接数比较多的情况，特别是长连接，不用每个连接都创建一个线程。

类似select的函数有 poll, epoll, kqueue。

### 信号驱动IO(signal-driven I/O)

1. 用户线程创建**SIGIO**的信号处理程序，然后发起系统调用（**sigaction**），内核收到系统调用会直接返回。
2. 然后内核等待数据准备以后，会使用SIGIO信号通知用户线程。
3. 用户线程收到信号后再通过系统调用（**recvfrom**）去读取数据。

### 异步IO(asynchronous I/O)

1. 用户线程发起系统调用（**aio_read**），内核收到系统调用会立刻返回。
2. 然后内核等待数据准备以后，直接把数据从内核拷贝到用户空间，并发送信号通知用户线程本次IO操作已经完成。

和信号驱动模型有所不同的是，在信号驱动模型中，当用户线程接收到信号表示数据已经就绪，然后需要用户线程调用IO函数进行实际的读写操作；而在异步IO模型中，收到信号表示IO操作已经完成，不需要再在用户线程中调用IO函数进行实际的读写操作。

### 参考

https://blog.csdn.net/ocean_fan/article/details/79622956

https://blog.csdn.net/qq_36095679/article/details/89641867

## 零拷贝

## Reactor和Proactor

Reactor：IO多路复用

Proactor：异步IO

## BIO NIO AIO

BIO（Block IO）：同步阻塞IO。

NIO（New IO）：Java 1.4 加入的同步非阻塞IO。

AIO（Asynchronous IO）：JDK 7 加入的异步非阻塞IO，也就是 NIO 2。

### BIO 复制文件

```java
/**
 * 来自 org.springframework.util.StreamUtils
 */
public static int copy(InputStream in, OutputStream out) throws IOException {
	Assert.notNull(in, "No InputStream specified");
	Assert.notNull(out, "No OutputStream specified");

	int byteCount = 0;
	byte[] buffer = new byte[BUFFER_SIZE]; // BUFFER_SIZE is 4096
	int bytesRead = -1;
	while ((bytesRead = in.read(buffer)) != -1) {
		out.write(buffer, 0, bytesRead);
		byteCount += bytesRead;
	}
	out.flush();
	return byteCount;
}
```

### NIO 复制文件

```java
public static void copy(File source, File dest) throws
    IOException {
    try (FileChannel sourceChannel = new FileInputStream(source).getChannel();
                FileChannel targetChannel = new FileOutputStream(dest).getChannel()) {

        long transferred;
        for (long count = sourceChannel.size(); count > 0; count -= transferred) {
            transferred = sourceChannel.transferTo(sourceChannel.position(), count, targetChannel);
            sourceChannel.position(sourceChannel.position() + transferred);
        }
    }
}
```

