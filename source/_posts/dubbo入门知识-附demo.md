---
toc: true
title: 'dubbo入门知识,附demo'
date: 2018-11-03 10:45:23
tags: [dubbo]
---

[dubbo官方文档](http://dubbo.apache.org/zh-cn/docs/user/quick-start.html)
关于dubbo产生的背景不再赘述。本文讨论如何在spring中集成dubbo，使用dubbo进行服务注册、服务发现、远程调用，dubbo的使用注意事项，及demo示例。

## 1、架构
![Alt text](https://app.yinxiang.com/shard/s15/res/d7a1fa30-a283-4f1c-99eb-476f90df4990/1540556051260.png?search=dubbo)
<!--more-->
**节点角色说明**
| 节点      |     角色说明 |
| :-------- | --------:|
| Provider    |   暴露服务的服务提供方 |
|Consumer|调用远程服务的服务消费方|
|Registry|服务注册与发现的注册中心|
|Monitor|统计服务的调用次数和调用时间的监控中心|
|Container|服务运行容器|
**调用关系说明**
0. 服务容器负责启动，加载，运行服务提供者
1. 服务提供者在启动时，向注册中心注册自己提供的服务
2. 服务消费者在启动时，向注册中心订阅自己所需的服务
3. 注册中心返回服务提供者地址列表给消费者，如有变更，注册中心将基于长连接推送变更数据给消费者。
4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## 2、demo
### 父子工程
使用IDEA创建一个Maven父子工程
#### Dubbo（Father）
父工程`pom.xml`：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ljf</groupId>
    <artifactId>dubbo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>dubbo-provider</module>
        <module>dubbo-consumer</module>
    </modules>
    <packaging>pom</packaging>

    <properties>
        <spring.version>4.3.16.RELEASE</spring.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>${spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.jboss.netty</groupId>
                <artifactId>netty</artifactId>
                <version>3.2.10.Final</version>
            </dependency>
            <dependency>
                <groupId>io.netty</groupId>
                <artifactId>netty-all</artifactId>
                <version>4.1.25.Final</version>
            </dependency>
            <dependency>
                <groupId>org.javassist</groupId>
                <artifactId>javassist</artifactId>
                <version>3.20.0-GA</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>dubbo</artifactId>
                <version>2.6.4</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
#### dubbo-provider（son）
**pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo</artifactId>
        <groupId>com.ljf</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbo-provider</artifactId>
    <packaging>war</packaging>

    <name>dubbo-provider Maven Webapp</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jboss.netty</groupId>
            <artifactId>netty</artifactId>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
        </dependency>
    </dependencies>

</project>
```
**定义服务接口：**
```java
public interface DemoService {
    String sayHello(String name);
}
```
**实现服务接口**
```java
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHello(String name) {
        return "Hello "+name;
    }
}
```
**用Spring配置声明暴露服务**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"/>

    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234"/>

    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.ljf.dubbo.service.DemoService" ref="demoService"/>

    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="com.ljf.dubbo.service.impl.DemoServiceImpl"/>
</beans>
```
**加载Spring配置**
```java
public class Provider {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("provider.xml");
        context.start();
        System.in.read();
    }
}
```
#### dubbo-consumer（son）
**pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbo</artifactId>
        <groupId>com.ljf</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbo-consumer</artifactId>
    <packaging>war</packaging>

    <name>dubbo-consumer Maven Webapp</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
        </dependency>
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jboss.netty</groupId>
            <artifactId>netty</artifactId>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
        </dependency>
        <dependency>
            <groupId>com.ljf</groupId>
            <artifactId>dubbo-provider</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```
**用Spring配置引用远程服务**
consumer.xml：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />

    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />

    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="com.ljf.dubbo.service.DemoService" />
</beans>
```
**加载Spring配置，并调用远程服务**
```java
public class Consumer {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("consumer.xml");
        context.start();
        DemoService demoService = (DemoService) context.getBean("demoService");
        System.out.println(demoService.sayHello("world"));
    }
}
```
#### 控制台输出结果
**provider**
```bash
...
十月 31, 2018 4:59:00 下午 com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol info
信息:  [DUBBO] disconnected from /192.168.56.1:52966,url:dubbo://192.168.56.1:20880/com.ljf.dubbo.service.DemoService?anyhost=true&application=hello-world-app&bind.ip=192.168.56.1&bind.port=20880&channel.readonly.sent=true&codec=dubbo&dubbo=2.0.2&generic=false&heartbeat=60000&interface=com.ljf.dubbo.service.DemoService&methods=sayHello&pid=239880&side=provider&timestamp=1540976329055, dubbo version: 2.6.4, current host: 192.168.56.1
```
**consumer**
```bash
十月 31, 2018 4:58:59 下午 com.alibaba.dubbo.config.AbstractConfig info
信息:  [DUBBO] Refer dubbo service com.ljf.dubbo.service.DemoService from url multicast://224.5.6.7:1234/com.alibaba.dubbo.registry.RegistryService?anyhost=true&application=consumer-of-helloworld-app&check=false&dubbo=2.0.2&generic=false&interface=com.ljf.dubbo.service.DemoService&methods=sayHello&pid=240992&register.ip=192.168.56.1&remote.timestamp=1540976329055&side=consumer&timestamp=1540976338851, dubbo version: 2.6.4, current host: 192.168.56.1
Hello world
十月 31, 2018 4:59:00 下午 com.alibaba.dubbo.config.DubboShutdownHook info
信息:  [DUBBO] Run shutdown hook now., dubbo version: 2.6.4, current host: 192.168.56.1
```

### 问题
1. `Configuration problem: Unable to locate Spring NamespaceHandler for XML schema namespace [http://dubbo.apache.org/schema/dubbo]`
解决方法 https://github.com/apache/incubator-dubbo/issues/1739
2. dubbo 缺省依赖问题
```
java.lang.NoClassDefFoundError: io/netty/channel/EventLoopGroup
	at com.alibaba.dubbo.qos.protocol.QosProtocolWrapper.startQosServer(QosProtocolWrapper.java:95)
	at com.alibaba.dubbo.qos.protocol.QosProtocolWrapper.export(QosProtocolWrapper.java:59)
	at com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper.export(ProtocolListenerWrapper.java:55)
```
通过`mvn dependency:tree`命令分析，dubbo缺省依赖以下三方库：
```
[INFO] --- maven-dependency-plugin:2.1:tree (default-cli) @ dubbo-provider ---
[INFO] com.ljf:dubbo-provider:war:1.0-SNAPSHOT
[INFO] +- com.alibaba:dubbo:jar:2.6.4:compile
[INFO] |  \- org.javassist:javassist:jar:3.20.0-GA:compile
[INFO] +- org.jboss.netty:netty:jar:3.2.10.Final:compile
[INFO] \- io.netty:netty-all:jar:4.1.25.Final:compile
```
可以看到，只是依赖于`javassist`而已。但实际上还依赖于两个netty包：
```
[INFO] +- org.jboss.netty:netty:jar:3.2.10.Final:compile
[INFO] \- io.netty:netty-all:jar:4.1.25.Final:compile
```
## 3、async dubbo
参考 [Async Dubbo](https://blog.csdn.net/oooyooo/article/details/49254201)
## 4、schema配置参考
所有配置分为三大类：
- 服务发现：表示该配置项用于服务的注册与发现，目的是让消费方找到提供方。
- 服务治理：表示该配置项用于治理服务间的关系，或为开发测试提供便利条件。
- 性能调优：表示该配置项用于调优性能，不同的选项对性能会产生影响。
- 所有配置最终都将转换为 URL [3] 表示，并由服务提供方生成，经注册中心传递给消费方。
`<dubbo:service>`
`<dubbo:reference>`
`<dubbo:application>`
## 5、多协议
推荐使用dubbo协议，在dubbo文档中介绍了各协议的性能对比。
Dubbo 缺省协议采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。

反之，Dubbo 缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。
![Alt text](https://app.yinxiang.com/shard/s15/res/df2cf7de-5a2c-4863-b5df-0432e40d8982/1541035833300.png?search=dubbo)
### 约束
- 参数及返回值需实现Serializable接口
- 参数及返回值不能自定义实现List、Map、Number、Date、Calendar等接口，只能使用jdk自带的实现，因为hessian会做特殊处理，自定义实现的属性值会丢失。
- Hessian 序列化，只传成员属性值和值的类型，不传方法或静态变量，兼容情况 
![Alt text](https://app.yinxiang.com/shard/s15/res/6057190e-184d-401a-ac9c-31c3949ab00d/1541036163856.png?search=dubbo)
（客户端应该是指消费者，服务器端指提供者）
接口增加方法，对客户端无影响，如果该方法不是客户端需要的，客户端不需要重新部署。输入参数和结果集中增加属性，对客户端无影响，如果客户端并不需要新属性，不用重新部署。

输入参数和结果集属性名变化，对客户端序列化无影响，但是如果客户端不重新部署，不管输入还是输出，属性名变化的属性值是获取不到的。

总结：服务器端和客户端对领域对象并不需要完全一致，而是按照最大匹配原则。
### 配置
配置协议：
`<dubbo:protocol name="dubbo" port="20880" />`
配置默认协议：
`<dubbo:provider protocol="dubbo" />`
设置服务协议：
`<dubbo:service protocol="dubbo" />`
多端口：
```xml
<dubbo:protocol id="dubbo1" name="dubbo" port="20880" />
<dubbo:protocol id="dubbo2" name="dubbo" port="20881" />
```
配置协议选项：
```xml
<dubbo:protocol name=“dubbo” port=“9090” server=“netty” client=“netty” codec=“dubbo” serialization=“hessian2” charset=“UTF-8” threadpool=“fixed” threads=“100” queues=“0” iothreads=“9” buffer=“8192” accepts=“1000” payload=“8388608” />
```
多连接配置：
Dubbo 协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。
```xml
<dubbo:service connections="1"/>
<dubbo:reference connections="1"/>
```
- `<dubbo:service connections="0">` 或 `<dubbo:reference connections="0">` 表示该服务使用 JVM 共享长连接。缺省
- `<dubbo:service connections="1">` 或 `<dubbo:reference connections="1">` 表示该服务使用独立长连接。
- `<dubbo:service connections="2">` 或`<dubbo:reference connections="2">` 表示该服务使用独立两条长连接。
为防止被大量连接撑挂，可在服务提供方限制大接收连接数，以实现服务提供方自我保护。
`<dubbo:protocol name="dubbo" accepts="1000" />`

## 6、多注册中心
推荐使用Multicast 注册中心
### Multicast 注册中心
Multicast 注册中心不需要启动任何中心节点，只要广播地址一样，就可以互相发现。
![Alt text](https://app.yinxiang.com/shard/s15/res/2597ed3f-c8b0-43a7-8703-cc4a35a13ec6/1541078593920.png?search=dubbo)
0. 提供方启动时广播自己的地址
1. 消费方启动时广播订阅请求
2. 提供方收到订阅请求时，单播自己的地址给订阅者，如果设置了 unicast=false，则广播给订阅者
3. 消费方收到提供方地址时，连接该地址进行 RPC 调用。
组播受网络结构限制，只适合小规模应用或开发阶段使用。组播地址段: 224.0.0.0 - 239.255.255.255
**配置**
`<dubbo:registry address="multicast://224.5.6.7:1234" />`
或
`<dubbo:registry protocol="multicast" address="224.5.6.7:1234" />`
为了减少广播量，Dubbo 缺省使用单播发送提供者地址信息给消费者，如果一个机器上同时启了多个消费者进程，消费者需声明 unicast=false，否则只会有一个消费者能收到消息：
`<dubbo:registry address="multicast://224.5.6.7:1234?unicast=false" />`
或
```xml
<dubbo:registry protocol="multicast" address="224.5.6.7:1234">
    <dubbo:parameter key="unicast" value="false" />
</dubbo:registry>
```
### Zookeeper注册中心
Zookeeper 是 Apacahe Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用 。
![Alt text](https://app.yinxiang.com/shard/s15/res/8d64daf4-c1c0-4a50-b6e8-a423a3966546/1541123904178.png?search=dubbo)
流程说明：

- 服务提供者启动时: 向 /dubbo/com.foo.BarService/providers 目录下写入自己的 URL 地址
- 服务消费者启动时: 订阅 /dubbo/com.foo.BarService/providers 目录下的提供者 URL 地址。并向 /dubbo/com.foo.BarService/consumers 目录下写入自己的 URL 地址
- 监控中心启动时: 订阅 /dubbo/com.foo.BarService 目录下的所有提供者和消费者 URL 地址。
支持以下功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过` <dubbo:registry username="admin" password="1234" /> `设置 zookeeper 登录信息
- 可通过` <dubbo:registry group="dubbo" />` 设置 zookeeper 的根节点，不设置将使用无根树
- 支持 * 号通配符` <dubbo:reference group="*" version="*" />`，可订阅服务的所有分组和所有版本的提供者

**使用**
在provider和consumer添加zookeeper依赖：
```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.3.3</version>
</dependency>
```
Dubbo支持zkclient和curator两种用法
这里介绍zkclient。
provider和consumer添加zkclient maven依赖：
```xml
<dependency>
                <groupId>com.github.sgroschupf</groupId>
                <artifactId>zkclient</artifactId>
                <version>0.1</version>
            </dependency>
```

`<dubbo:registry client="zkclient" address>`

## 7、推荐用法
### 在Provider上尽量多配置Consumer端属性
原因如下：
- 作服务的提供者，比服务使用方更清楚服务性能参数，如调用的超时时间、合理的重试次数等
- 在 Provider 配置后，Consumer 不配置则会使用 Provider 的配置值，即 Provider 配置可以作为 Consumer 的缺省值 [1]。否则，Consumer 会使用 Consumer 端的全局设置，这对于 Provider 是不可控的，并且往往是不合理的

Provider 上尽量多配置 Consumer 端的属性，让 Provider 实现者一开始就思考 Provider 服务特点、服务质量等问题。
示例：
```xml
<dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService"
    timeout="300" retry="2" loadbalance="random" actives="0" />
 
<dubbo:service interface="com.alibaba.hello.api.WorldService" version="1.0.0" ref="helloService"
    timeout="300" retry="2" loadbalance="random" actives="0" >
    <dubbo:method name="findAllPerson" timeout="10000" retries="9" loadbalance="leastactive" actives="5" />
<dubbo:service/>
```
在 Provider 上可以配置的 Consumer 端属性有：
1. timeout 方法调用超时
2. retries 失败重试次数，缺省是 2 
3. loadbalance 负载均衡算法 ，缺省是随机 random。还可以有轮询 roundrobin、最不活跃优先 leastactive 等
4. actives 消费者端，最大并发调用限制，即当 Consumer 对一个服务的并发调用到上限后，新调用会阻塞直到超时，在方法上配置 dubbo:method 则并发限制针对方法，在接口上配置 dubbo:service，则并发限制针对服务

### Provider 上配置合理的 Provider 端属性
```xml
<dubbo:protocol threads="200" /> 
<dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService"
    executes="200" >
    <dubbo:method name="findAllPerson" executes="50" />
</dubbo:service>
```
Provider 上可以配置的 Provider 端属性有：
1. threads 服务线程池大小
2. executes 一个服务提供者并行执行请求上限，即当 Provider 对一个服务的并发调用达到上限后，新调用会阻塞，此时 Consumer 可能会超时。在方法上配置 dubbo:method 则并发限制针对方法，在接口上配置 dubbo:service，则并发限制针对服务
### 配置管理信息
目前有负责人信息和组织信息用于区分站点。有问题时便于的找到服务的负责人，至少写两个人以便备份。负责人和组织的信息可以在注册中心的上看到。

应用配置负责人、组织：
`<dubbo:application owner=”ding.lid,william.liangf” organization=”intl” />
`
service配置负责人：
`<dubbo:service owner=”ding.lid,william.liangf” />
`
reference配置负责人：
`<dubbo:reference owner=”ding.lid,william.liangf” />
`
若没有配置 service 和 reference 的负责人，则默认使用 dubbo:application 设置的负责人。

### 配置 Dubbo 缓存文件
提供者列表缓存文件：
`<dubbo:registry file=”${user.home}/output/dubbo.cache” />
`
注意：

1. 应用可以根据需要调整缓存文件的路径，保证这个文件不会在发布过程中被清除；
2. 如果有多个应用进程，注意不要使用同一个文件，避免内容被覆盖；

该文件会缓存注册中心列表和服务提供者列表。配置缓存文件后，应用重启过程中，若注册中心不可用，应用会从该缓存文件读取服务提供者列表，进一步保证应用可靠性。
### 监控配置
1. 使用固定端口暴露服务，而不要使用随机端口；这样在注册中心推送有延迟的情况下，消费者通过缓存列表也能调用到原地址，保证调用成功。
## 8、附录：demo
github：[dubbo示例](https://github.com/cap-ljf/javastudy/tree/master/dubbo)
> 参考文献
> [Async dubbo](https://blog.csdn.net/oooyooo/article/details/49254201)
> http://dubbo.apache.org/zh-cn/

