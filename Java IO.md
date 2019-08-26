[TOC]



## 字符流和字节流的区别

## BIO NIO AIO

BIO（Block IO）：同步阻塞IO。

NIO（New IO）：Java 1.4 加入的同步非阻塞IO。

AIO（Asynchronous IO）：JDK 7 加入的异步非阻塞IO，也就是 NIO 2。

BIO 复制文件的方式：

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

NIO 复制文件的方式：

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