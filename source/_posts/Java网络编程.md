---
toc: true
title: Java网络编程
date: 2018-03-26 22:56:32
tags: [Java网络编程,socket,JavaMail]
---

这一篇博客将介绍怎么进行Java网络编程，首先我们回顾网络的基本概念，然后进一步介绍Java socket，并演示网络客户端和服务器是如何实现的，最后将介绍如果通过Java程序发送E-mail，以及如何从Web服务器获得信息。
<!--more-->
### 1. Socket
#### 1.1 连接到服务器
在Linux和windows操作系统中都预装了`telnet`工具，它对我们调试网络程序非常有帮助。
首先尝试下面的命令：
`telnet time-A.timefreq.bldrdoc.gov 13`
在你的控制台将返回下面一行类似的信息：
`58203 18-03-26 15:01:27 50 0 0 631.0 UTC(NIST) * `
如果你得到上面这行信息，说明你已经连接到了大多数UNIX计算机都支持的“当日时间”服务

**socket概念**
在计算机网络课程中我们学习了ISO七层协议，其中讲述了socket是指ip地址加port端口号组成的通道，当通道连接之后我们就可以使用socket进行数据传输。

下面这个程序的作用和telnet工具是一样的，即连接到某个端口并打印出它所找到的信息。
```java
package socket;

import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;
import java.util.Scanner;

/**
 * 这个类的作用与telnet工具是一致的，即连接到某个端口并打印出它所找到的信息
 * author: jifang
 * date: 18-3-26 下午2:01
 */

public class SocketTest {
    public static void main(String[] args) throws IOException {
        //打开一个套接字
        try (Socket s = new Socket("time-A.timefreq.bldrdoc.gov", 13)) {
            s.setSoTimeout(10000);
            //一旦套接字被打开，就可以得到一个InputStream对象，使用它进行读取数据
            InputStream in = s.getInputStream();
            Scanner sc = new Scanner(in);
            while (sc.hasNextLine()){
                System.out.println(sc.nextLine());
            }
        }
    }
}

```
**套接字超时**
从套接字读取信息时，在有数据可以访问之前，读操作将会被阻塞。如果此时主机不可达，那么应用将要等待很长的时间，并且因为受底层操作系统的限制而最终会导致超时。
对于不同的应用，应当确定合理的超时值。调用`setSoTimeout`方法设置超时时间（单位：毫秒）。
`Socket s = new SOcket(...)`
`s.setSoTimeout(10000)` # 设置超时时间为10秒
如果已经为套接字设置了超时时间，并且之后的读操作和写操作在没有完成之前就超过了时间限制，那么这些操作就会抛出`SocketTimeoutException`异常。你可以捕获这个异常，并对超时做出反应。

另外还有一个超时问题是必须解决的。下面这个构造器：
`Socket s = new Socket(String host, int port)`
会一直无限期的阻塞下去，知道建立了到达主机的初始连接为止。
可以先通过建立一个无连接的套接字，然后再使用一个超时来进行连接的方法解决这个问题。
`Socket s = new Socket();`
`s.connect(new InetSocketAddress(host, port), timeout);`

**InetAddress**
因特网地址类，使用它在主机名和因特网地址之间进行转换。
静态的`getByName(String host)`方法可以返回代表某个主机的InetAddress对象，使用`getAddress`返回对应的ip地址字节数组。
使用`InetAddress.getAllByName()`获得所有的主机。
```java
package socket;

import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * author: jifang
 * date: 18-3-26 下午2:11
 */

public class InetAddressTest {
    public static void main(String[] args) throws UnknownHostException {
        if (args.length>0){
            String host = args[0];
            InetAddress[] addresses = InetAddress.getAllByName(host);
            for (InetAddress a:addresses)
                System.out.println(a);
        }
        else
        {
            InetAddress localHostAddress = InetAddress.getLocalHost();
            System.out.println(localHostAddress);
        }
    }
}

```
###  2. Client/Server通信
#### 2.1 实现客户端
现在我们实现一个简单的服务器，它可以向客户端发送信息。
整个代码的大致情况：
1. 通过输入数据流从客户端接受一个命令
2. 解码这个客户端命令
3. 收集客户端所请求的信息
4. 通过输出数据流发送信息给客户端

【代码清单】
```java
package socket;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Scanner;

/**
 * author: jifang
 * date: 18-3-26 下午2:20
 */

public class EchoServer {
    public static void main(String[] args) throws IOException {
        try(ServerSocket s = new ServerSocket(8189)){
            try (Socket incoming = s.accept()){
                InputStream in = incoming.getInputStream();
                OutputStream out = incoming.getOutputStream();
                try (Scanner sc = new Scanner(in)){
                    PrintWriter pw = new PrintWriter(out, true);
                    pw.println("hello! Enter BYE to exit.");
                    boolean done = false;
                    while (!done && sc.hasNextLine()){
                        String line = sc.nextLine();
                        pw.println("Echo: "+line);
                        if (line.trim().equals("BYE"))done=true;
                    }
                }
            }
        }

    }
}

```
运行这个程序，然后使用`telnet`或再次运行之前的程序`SocketTest`连接本地`8189`端口，当你的客户端连接到这个端口后会立马接收到这条信息：
`Hello! Enter BYE to exit.`
之后客户端的输入便是服务器返回的数据。
```bash
jifang@jifang:~/workspace/javastudy/java/src/main/java/socket% telnet localhost 8189
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
hello! Enter BYE to exit.
1
Echo: 1
2
Echo: 2
BYE
Echo: BYE
Connection closed by foreign host.
```
#### 2.2 为多个客户端服务
前面的例子中的简单服务器存在一个问题：服务器同一时间只能为一个客户端服务。我们访问各种网站一般都可以处理成千上万个请求，所以我们来写一个多线程的服务器。

每当程序建立一个新的套接字连接，也就是说当调用accept时，将启动一个新的线程来处理服务器与客户端之间的连接，而主程序将立即返回并等待下一个连接。
【ThreadedEchoServer.java】
```java
package socket;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * author: jifang
 * date: 18-3-26 下午2:35
 */

public class ThreadedEchoServer {
    public static void main(String[] args)  {
        try {
            int i = 1;
            ServerSocket s = new ServerSocket(8189);
            while (true){
                Socket incoming = s.accept();
                System.out.println("Spawning "+i);
                ThreadEchoHandler r = new ThreadEchoHandler(incoming);
                Thread thread = new Thread(r);
                thread.start();
                i++;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
【ThreadEchoHandler.java】
```java
package socket;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;
import java.util.Scanner;

/**
 * author: jifang
 * date: 18-3-26 下午2:30
 */

public class ThreadEchoHandler implements Runnable {

    private Socket incoming;

    public ThreadEchoHandler(Socket incoming) {
        this.incoming = incoming;
    }

    @Override
    public void run() {
        try {
            InputStream inStream = incoming.getInputStream();
            OutputStream outStream = incoming.getOutputStream();
            Scanner sc = new Scanner(inStream);
            PrintWriter pw = new PrintWriter(outStream,true);
            pw.println("Hello! Enter BYE to exit.");
            boolean done = false;
            while (!done && sc.hasNextLine()){
                String line = sc.nextLine();
                pw.println("Echo: "+line);
                if (line.trim().equals("BYE"))done=true;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                incoming.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
在这个程序中我们为每个连接生成一个单独的线程。这种方法并不能满足高性能服务器的要求。为使服务器实现更高的吞吐量，可以使用java.nio包中的一些特性，我们将在下一篇博客中讲述**NIO**。

### 3. URL和URLConnection
为了更高级别的处理网络服务，我们将讨论专门用于此目的的Java类库中的各个类。
#### 3.1 URI、URL、URN
#### 3.2 使用URLConnection获取信息
如果想从某个Web资源获取更多的信息，那么应该使用URLConnection类，通过它能够得到比基本的URL类更多的控制功能。

操作步骤：
1. 调用URL类中的`openConnection`方法获得URLConection对象：
`URLConnection connection = url.openConnection()`
2. 使用以下方法设置 HTTP Header属性：
`setDoInput`
`SetDoOutput`
`setIfmodifiedSince`
`setUseCaches`
`setReadTimeout`
...
3. 调用connect方法链接远程资源：
`connection.connect()`
除了与服务器建立套接字连接外，该方法还可用于向服务器查询头信息（header）
4. 与服务器建立连接后，你可以查询头信息。getHeaderFieldKey和getHeaderField两个方法枚举了消息头的所有字段。`getHeaderFields`方法返回一个包含了消息头中所有字段的标准Map对象。
`getContentType`
`getContentLength`
`getContentEncoding`
`getDate`
...
5. 最后，访问资源数据。使用`getInputStream`方法获取一个输入流用以读取信息。

**注意：** *默认情况下URLConnection发送一个HTTP GET请求到web服务器。如果你想发送一个HTTP POST请求，要调用URLConnection.setDoOutput(true)方法，如下：* 
```java
URL url = new URL("http://jenkov.com");
URLConnection urlConnection = url.openConnection();
urlConnection.setDoOutput(true);
```
*一旦你调用了setDoOutput(true)，你就可以打开URLConnection的OutputStream，如下：*
```java
OutputStream output = urlConnection.getOutputStream();
```
你可以使用这个OutputStream向相应的HTTP请求中写任何数据，但你要记得将其转换成URL编码。
```java
output.print(name1+"="+URLEncoder.encode(value1,"UTF-8")+"&");
output.print(name2+"="+URLEncoder.encode(value2,"UTF-8"));
output.close()
```

### 4. 发送Email

过去，编写程序通过创建到SMTP专用的端口25来发送邮件是一个很简单的事。SMTP用于描述E-mail消息的格式。一旦连接到服务器，就可以发送一个邮件报头（采用SMTP格式）。

操作过程：
1. 打开一个到达主机的套接字
```java
Socket s = new Socket("mail.youserver.com",25); // 25 is SMTP
PrintWriter out = new PrintWriter(s.getOutputStream());
```
2. 发送一下信息到输出流
```
HELO sending host
MAIL FROM: sender e-mail address
RCPT TO: recipient e-mail address
DATA
Subject: subject
(blank line)
mail message(邮件内容)

QUIT
```
SMTP规范规定，每一行都要以`\r`再紧跟一个`\n`来结尾。
SMTP之前总是路由任何人的e-mail，在垃圾邮件泛滥的今天，大多数服务器都内置了检查功能，并且只接收来自授信用户或IP地址范围的请求。其中，认证是通过安全套接字连接来实现的，非常冗长乏味。所以我们使用JavaMailAPI在Java程序中发送email。

这里以QQ邮箱来演示如何用JavaMail发送一封邮件
#### 4.1 先启用QQ邮箱里POP3/STMP服务，生成授权码
QQ邮箱-->账户-->POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务-->开启POP3/SMTP服务
发送短信验证通过之后可以看到授权码
![Alt text](https://app.yinxiang.com/shard/s15/res/a836d720-dcca-42c8-86b1-d21ea60395bc/1522074780596.png)
![Alt text](https://app.yinxiang.com/shard/s15/res/e5374a0f-acba-41e0-9aa8-cd2a6ccbdd3c/1522074797444.png)
#### 4.2 导入JavaMail jar包
我使用的maven进行jar包管理
```xml
<dependency>
   <groupId>javax.mail</groupId>
   <artifactId>mail</artifactId>
   <version>1.4.7</version>
</dependency>
```
![Alt text](https://app.yinxiang.com/shard/s15/res/d2fa064e-d387-427b-b21d-e04b2cc25c83/1522074945736.png)
#### 4.3 代码实现
【MailTest】
```java
package socket;

import com.sun.mail.util.MailSSLSocketFactory;

import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.security.GeneralSecurityException;
import java.util.List;
import java.util.Properties;

/**
 * author: jifang
 * date: 18-3-26 下午5:07
 */

public class MailTest {
    public static void main(String[] args) throws IOException, MessagingException {
        Properties props = new Properties();
        try (InputStream in = Files.newInputStream(Paths.get("src/main/java/socket","mail.properties"))){
            // 1. 创建参数配置, 用于连接邮件服务器的参数配置
            props.load(in);
            // PS: 某些邮箱服务器要求 SMTP 连接需要使用 SSL 安全认证 (为了提高安全性, 邮箱支持SSL连接, 也可以自己开启),
            //     如果无法连接邮件服务器, 仔细查看控制台打印的 log, 如果有有类似 “连接失败, 要求 SSL 安全连接” 等错误,
            //     打开下面 /* ... */ 之间的注释代码, 开启 SSL 安全连接。
            // SMTP 服务器的端口 (非 SSL 连接的端口一般默认为 25, 可以不添加, 如果开启了 SSL 连接,
            //                  需要改为对应邮箱的 SMTP 服务器的端口, 具体可查看对应邮箱服务的帮助,
            //                  QQ邮箱的SMTP(SLL)端口为465或587, 其他邮箱自行去查看)
            /*MailSSLSocketFactory sf = new MailSSLSocketFactory();
            sf.setTrustAllHosts(true);
            props.put("mail.smtp.ssl.enable", "true");
            props.put("mail.smtp.ssl.socketFactory", sf);*/

        } catch (IOException e) {
            e.printStackTrace();
        }
        List<String> lines = Files.readAllLines(Paths.get(args[0]), Charset.forName("utf-8"));

        String from = lines.get(0);
        String to = lines.get(1);
        String subject = lines.get(2);
        StringBuilder builder = new StringBuilder();
        for (int i=3;i<lines.size();i++){
            builder.append(lines.get(i)+"\n");
        }

        // 2. 根据配置创建会话对象, 用于和邮件服务器交互
        Session mailSession = Session.getDefaultInstance(props);
        // 3. 创建一封邮件
        MimeMessage message = new MimeMessage(mailSession);
        // 发件人
        message.setFrom(new InternetAddress(from, "jifang", "UTF-8"));
        // 收件人（可以增加多个收件人(RecipientType.To)、抄送(RecipientType.CC)、密送(RecipientType.BCC)）
        message.addRecipient(Message.RecipientType.TO, new InternetAddress(to,"jifang","UTF-8"));
        //主题
        message.setSubject(subject);
        //邮件内容
        message.setText(builder.toString());
        // 4. 根据 Session 获取邮件传输对象
        Transport tr = mailSession.getTransport();
        try{
            // 5. 使用 邮箱账号 和 密码 连接邮件服务器, 这里认证的邮箱必须与 message 中的发件人邮箱一致, 否则报错
            //
            //    PS_01: 成败的判断关键在此一句, 如果连接服务器失败, 都会在控制台输出相应失败原因的 log,
            //           仔细查看失败原因, 有些邮箱服务器会返回错误码或查看错误类型的链接, 根据给出的错误
            //           类型到对应邮件服务器的帮助网站上查看具体失败原因。
            //
            //    PS_02: 连接失败的原因通常为以下几点, 仔细检查代码:
            //           (1) 邮箱没有开启 SMTP 服务;
            //           (2) 邮箱密码错误, 例如某些邮箱开启了独立密码;
            //           (3) 邮箱服务器要求必须要使用 SSL 安全连接;
            //           (4) 请求过于频繁或其他原因, 被邮件服务器拒绝服务;
            //           (5) 如果以上几点都确定无误, 到邮件服务器网站查找帮助。
            //
            //    PS_03: 仔细看log, 认真看log, 看懂log, 错误原因都在log已说明。
            tr.connect("smtp.qq.com","1579461369","pjyfztifsbcthbad");
            // 6. 发送邮件, 发到所有的收件地址, message.getAllRecipients() 获取到的是在创建邮件对象时添加的所有收件人, 抄送人, 密送人
            tr.sendMessage(message,message.getAllRecipients());
        }finally {
            // 7. 关闭连接
            tr.close();
        }

    }
}
```
【mail.properties】
```xml
mail.transport.protocol=smtps
mail.smtps.auth=true
# 发件人邮箱的 SMTP 服务器地址, 必须准确, 不同邮件服务器地址不同, 一般(只是一般, 绝非绝对)格式为: smtp.xxx.com
# QQ邮箱的 SMTP 服务器地址为: smtp.qq.com
mail.smtps.host=smtp.qq.com
```
【message.txt】
```
xxx@qq.com
yyy@qq.com
测试
你已经收到了邮件！
```
运行`java MailTest message.txt`（message.txt可随意放置，如果你不知道相对路径可以写绝对路径）
接下来你可以查看收件箱，你就会发现收到邮件啦。


> 参考文献
> [1] 《Java核心技术卷二》
> [2] [基于JavaMail的Java邮件发送：简单邮件发送](https://blog.csdn.net/xietansheng/article/details/51673073)
> [3] [用JavaMail通过QQ邮箱来发送邮件（第一篇博客，备忘）](https://www.cnblogs.com/xyzq/p/5704358.html)
