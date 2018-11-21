---
toc: true
title: 'Java 8,9,10,11新特性之——Java9'
date: 2018-11-21 18:25:29
tags: [java9, 模块化]
---

java9于2017年9月21日发布,  离java8发布三年多之久发布的新版本, 其中最重要的更新就是 Java平台模块系统.

Java9的核心在于解决历史遗留问题, 为以后的jar包森林清理道路.
<!--more-->
## 什么是模块化
首先看一下[Oricle官网Java9模块化介绍博客](https://www.oracle.com/corporate/features/understanding-java-9-modules.html)对模块化的定义:
![Paul Deitel](https://app.yinxiang.com/shard/s15/res/e6a2147f-cf94-47dc-a08d-7b3232179ef7/1542677911543.png)
> Modularity adds a higher level of aggregation above packages. The key new language element is the module—a uniquely named, reusable group of related packages, as well as resources (such as images and XML files) and a module descriptor specifying
> - the module’s name
- the module’s dependencies (that is, other modules this module depends on)
- the packages it explicitly makes available to other modules (all other packages in the module are implicitly unavailable to other modules)
- the services it offers
- the services it consumes
- to what other modules it allows reflection

**译:**
模块化是在package(包)层次上添加的更高层次的聚合. Java9语言心得关键元素就是模块化, 模块化是由 一组唯一命名的可重用的包、资源(例如图片和XML文件)和一个模块描述符组成的集合。
模块描述符需要指定了：
- 模块名
- 模块的依赖
- 明确指出哪些模块能够访问本模块的具体包（默认模块内的所有包对其他模块都不可用）
- 模块提供的服务
- 模块消费的服务
- to what other modules it allows reflection（暂不明白）

## 为什么需要模块化
### 历史遗留问题
两个问题， 臃肿的jar包和混乱的jar包依赖关系。
**臃肿的jar包**
Java8的rt.jar为60兆之大，运行一个Hello world程序， 也需要这么一个庞然大物。我们依赖一个大的工具包，但我们可能只需要其中几个类的功能，之前的jar包机制是很僵硬的， 我们需要一种更加灵活的环境，能够按需分配
![Alt text](https://app.yinxiang.com/shard/s15/res/f63cfb43-d641-4263-ab88-855f305abee6/1542762570596.png)
**jar hell**
jar包之间的依赖可谓错综复杂， 我们需要更加清晰的依赖关系。
![Alt text](https://app.yinxiang.com/shard/s15/res/def9edd3-fd3b-42be-810b-d821f29ea31a/1542762727134.png)

### 模块化的目标
根据 JSR 376， Java SE平台模块化的关键目标是：
- 可靠的配置——模块化提供了一种方式明确的声明模块之间的依赖关系， 在编译时和运行时都能够识别这些依赖关系。系统可以遍历这些依赖关系，以确定包含支持你的应用程序所需的所有模块
- 强封装——仅当模块显示的使用exports导出模块时，模块中的包才可供其他模块访问。即使这样，另一个模块也不能使用这些包，除非它使用requires明确声明它需要其他模块的功能。这提高了平台的安全性，因为潜在的攻击者能访问的类更少了。您可能会发现考虑模块化可以帮助您提出更清晰，更合理的设计。
- 可扩展的Java平台——以前，Java平台是由大量的包组成的整体，使得开发、维护和扩展都很难。它不容易被子集化。现在这个平台被模块化成95个模块（这个数量可能会随着Java的发展而改变）。您可以创建一个由您想要的能够支持您的应用或者设备所需的模块的运行环境。例如，如果设备不支持GUI，则可以创建不包含GUI模块的运行时，从而显著减少运行时的大小。
- 更高德平台完整性——在Java9之前，可以在平台中使用许多不适合应用程序类使用的类。通过强大的封装，这些内部API可以使用该Java9模块化平台真正封装并隐藏在应用程序之外。如果您的代码依赖于这些内部API，这可能会导致这些历史代码迁移到java9模块化时出现问题。
- 改进的性能——JVM使用各种优化技术来提高应用程序性能。JSR 376表明，当事先知道所需类型仅位于特定模块中时，这些技术更有效。

## 简单入门模块化
示例demo链接在文章末尾。
首先安装好Java11，windows和linux系统都可以安装多个版本的jdk。

### 第一个例子，Greetings
首先查看greetings项目目录结构
```
└─src
    │  module-info.java
    │
    └─com.greetings
            Main.java
```
整个项目包含两个文件，一个是`module-info.java`模块文件，还有一个是`Main.java`主类文件。
查看一下`Main.java`和`module-info.java`的代码：
**Main.Java**
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Greetings!");
    }
}
```
**module-info.java**
```java
module com.greetings {}
```
模块文件使用`module`关键字定义了该模块的模块名（唯一的）`com.greetings`，因为这个`Main.java`中没有依赖别的模块的内容，也没有打算向外界暴露自己的服务，所以内容是空的。
现在我们编译这两个文件：
```
cd greetings //进入项目根目录
javac -d mods/com.greetings src/com.greetings/Main.java src/module-info.java //linux
javac -d mods/greetings src\com.greetings\Main.java src\module-info.java //windows
```
`-d`参数用来指定放置生成的类文件的位置，
```
java --module-path mods -m com.greetings/com.greetings.Main //linux
java --module-path mods -m com.greetings/com.greetings.Main //windows
```
其中`--module-path <模块路径>` windows使用`；`分隔的目录列表，其中每个目录都包含一个模块
将主类、源文件、`-jar<jar文件>`、`-m`或`--module <模块>/<主类>`后的参数作为参数传递到主类执行
执行上面命令之后输出：
```
Greetings!
```
### 第二个例子，Greetings world！
先看一下目录结构：
```
├─astro
│  └─src
│      │  module-info.java
│      │
│      └─com.astro
│              World.java
│
├─greetings
│  └─src
│      │  module-info.java
│      │
│      └─com.greetings
│              Main.java
```
同样先看代码内容：
**astro/World.java**
```java
public class World {
    public static String name() {
        return "world";
    }
}
```
World类中定义了一个静态方法`name()`，返回一个`World`字符串。
**astro/module-info.java**
```java
module com.astro {
    exports com.astro;
}
```
astro定义了模块名为`com.astro`，并且使用了`exports`提供了自身模块的功能。
**greetings/Main.java**
```java
package com.greetings;
import com.astro.World;
public class Main {
    public static void main(String[] args) {
        System.out.format("Greetings %s", World.name());
    }
}
```
Main.java引用了astro的World.name方法。
**greetings/module-info.java**
```java
module com.greetings {
    requires com.astro;
}
```
因为Main.java，使用了`requires`关键字声明了需要`com.astro`这个模块的服务。
编译：首先编译astro，然后编译greetings
```
javac -d mods/com.astro src/com.astro/World.java src/module-info.java //linux astro
javac -p ../astro/mods/ -d mods/com.greetings src/com.greetings/Main.java src/module-info.java //linux greetings

javac -d mods src\com.astro\World.java src\module-info.java //windows
javac -p ..\astro\mods -d mods src\com.greetings\Main.java src\module-info.java //windows
```
` --module-path <path>, -p <path>` 指定查找应用程序模块的位置
运行：
```java
java -p mods;..\astro\mods -m com.greetings/com.greetings.Main //windows
java -p mods;../astro/mods -m com.greetings/com.greetings.Main //linux 试了一下没成功
```
成功的输出了：
```
Greetings world
```
### 多模块编译
### 打包
```
jar -c -f mlib/com.astro.jar --module-version=1.0 -C mods . //linux
jar -c -f mlib/com.greetings.jar --main-class=com.greetings.Main --module-version=1.0 -C mods . //linux
```
`-c` 创建jar包
`-f` jar包文件名
`--module-version` 创建模块化jar或更新非模块化jar时的模块版本
`-C` 更改为指定目录并包含以下文件（所以最后使用了`-C mods .` 注意有一个英文点号）
运行
```
java -p mlib;..\astro\mlib -m com.greetings //windows
```
同样的输出Greetings world

### jlink
我们可以通过jlink构造自己的jre，前文提到的rt.jar很大，jlink可以帮助我们定制自己想要的运行环境。
使用jlink命令：
```
jlink --module-path mods;..\astro\mods --add-modules com.greetings --output executable //windows
```
完成，这样就把执行文件输出到了executable文件夹中，如果对以上的命令不熟悉可以通过`命令 --help`查看命令帮助
```
executable\bin\java --module-path mods --module com.greetings/com.greetings.Main //windows
```
上面我们使用了executable文件夹中的java命令，这个就是我们自定义的jre环境。
查看一下executable/lib中jrt-fs.jar的大小，仅105KB。

### jmods
jmod 可以对模块或 jar 包压缩的一个工具，但他的格式和 jar 包不一样。
直接在上个练习目录中试用命令就好。
```
jmod create --class-path mods;..\astro\mods greetings.jmod
```
然后查看一下：
```
D:\workspace\java9-module-demo\greetings>jmod list greetings.jmod
classes/module-info.class
classes/com/greetings/Main.class
classes/com/astro/World.class
```
更多的命令解释可以查看一下 jmod --help

最后附上github地址：https://github.com/cap-ljf/java9-module-demo/


参考文献：
> https://hacpai.com/article/1532571971058#toc_h3_9, 黑客派，pattyq
> https://www.oracle.com/corporate/features/understanding-java-9-modules.html，www.oracle.com，Paul Deitel
