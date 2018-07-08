---
toc: true
title: java I/O流
date: 2018-03-21 14:16:43
tags: [java,I/O,流]
---


### 1. 概述
IO流简单来说就是Input和Output流，IO流主要是用来处理设备之间的数据传输，Java对于数据的操作都是通过流实现，而java用于操作流的对象都在IO包中。
<!--more-->
### 2. 分类
按操作数据分为：字节流和字符流。 如：`Reader`和`InpurStream`
按流向分：输入流和输出流。如：`InputStream`和`OutputStream`
	
	顺序访问：相当于链表，想要读取第5个位置的字节，必须按顺序读取第1\2\3\4位置的字节
	随机访问：相当于数组，想要读取第5个位置的字节，可以直接读取到

![IO流家族](https://app.yinxiang.com/shard/s15/res/2c5ca3f2-2db4-45f3-acb9-4515c3257ae1)


### 3. N点（notice point）
使用了缓冲区是否一定要flush 
【BufferedReader源码】
```java
public class BufferedReader extends Reader {

    private Reader in;

    private char cb[];
    private int nChars, nextChar;

    private static final int INVALIDATED = -2;
    private static final int UNMARKED = -1;
    private int markedChar = UNMARKED;
    private int readAheadLimit = 0; /* Valid only when markedChar > 0 */

    /** If the next character is a line feed, skip it */
    private boolean skipLF = false;

    /** The skipLF flag when the mark was set */
    private boolean markedSkipLF = false;

    private static int defaultCharBufferSize = 8192;
    private static int defaultExpectedLineLength = 80;
    /**
	 * 省略很多源码
	 */
}
```
可以看到`defaultCharBufferSize`是8192，单位是char，即8192 Byte
如果文件size大于8192 Byte，而且写入的时候不及时flush，就会发生数据丢失。
由于`close()`方法会执行`flush()`，所以我写得测试代码并没有发生数据丢失的情况，而我一行的大小却明显大于8192字节，由于源码看得有点痛苦，我大致认为是`readline()`方法多次调用buffer，一次读满了就append到返回字符串后面。

流close的顺序
【示例代码：】
```java
package stream;

import java.io.*;

public class InputStream {
    public static void main(String[] args) throws IOException {
        FileReader fr = new FileReader("/home/jifang/Desktop/a.txt");
        BufferedReader br = new BufferedReader(fr);
        String str;
        FileWriter fw = new FileWriter("/home/jifang/Desktop/b.txt");
        BufferedWriter bw = new BufferedWriter(fw);
        while ((str=br.readLine())!=null){
            bw.write(str);
        }
        fr.close();
        br.close();
        fw.close();
        bw.close();
    }
}
```
【运行出现异常】
```java
Exception in thread "main" java.io.IOException: Stream closed
	at sun.nio.cs.StreamEncoder.ensureOpen(StreamEncoder.java:45)
	at sun.nio.cs.StreamEncoder.write(StreamEncoder.java:118)
	at java.io.OutputStreamWriter.write(OutputStreamWriter.java:207)
	at java.io.BufferedWriter.flushBuffer(BufferedWriter.java:129)
	at java.io.BufferedWriter.close(BufferedWriter.java:265)
	at stream.InputStream.main(InputStream.java:18)
```
### 4. 流操作基本规律
在阅读了[这篇博客](https://www.cnblogs.com/xll1025/p/6418766.html)之后，他提出的这一点我感觉很好用，也方便记忆，但是博主不允许转载，我也就不贴了。有兴趣的可以点链接看看。

使用流的思路：
1. 明确源和目的
2. 明确操作数据类型（字符或字节）
3. 明确设备（键盘、内存、文件、网络）
4. 是否使用buffer

**BufferReader的威力**
《Java核心技术 卷二》表1-7
| 方法      |    时间 |
| :-------- | --------:|
| 普通输入流  | 110秒 |
|带缓冲的输入流|9.9秒|
|随机访问文件|162秒|
|内存映射文件|7.2秒|
相信大家看完这个对比之后能直观的感受到缓冲带来的效率提升。

> 参考文献
> [1] [inputStream关闭了，还有必要关闭InputStreamReader和BufferedReader吗？](https://segmentfault.com/q/1010000005970993)
> [2] [JAVA IO流最详解](https://www.cnblogs.com/xll1025/p/6418766.html)
> [3] 《Java核心技术 卷二》