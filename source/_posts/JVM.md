---
toc: true
title: JVM
date: 2017-10-18 14:12:54
tags: [jvm,java虚拟机]
---

### 一、类初始化
我们常说的类初始化分为三个阶段，类加载、连接、和初始化。类加载把class文件载入内存，类连接进行内存分配、初始化为静态变量赋值。
<!--more-->
#### 类的加载
类的加载指将类的class文件载入内存，并为之创建一个java.lang.Class对象。可以从下面几种来源加载类。
- 本地文件系统
- JAR包
- 网络
- 把一个java文件动态编译，并执行加载
类加载完成后，JVM就为其生成一个java.lang.Class对象，通过这个对象就可操作这个类。

#### 类的连接
将类的二进制文件合并到jre中，连接阶段会为变量分配内存并赋初始值。

#### 类初始化
<font color="red">**在初始化阶段，java虚拟机执行类的初始化语句，为类的静态变量赋予初始值。静态变量的语句，和静态代码块都被看做类的初始化语句，java虚拟机会按照初始化语句在类中的先后顺序来依次执行他们。**</font>
类的初始化时机：
1. **主动使用**：（6种情况）
	- 创建类的实例 ` new Test() `
	- 访问某个类或接口的静态变量，或者对该静态变量赋值
		- ```int b = Test.a; Test.b = a; ```
	- 调用类的静态方法
	- 反射
	- 初始化一个类的子类，那么这个类会先被初始化
	- java虚拟机启动时被标明为启动类的类

2. 除了以上6种情况为**主动使用**，其余使用java类的方式都被看做是对类的**被动使用**，都不会导致类的初始化。

#### 接口的特殊性
当java虚拟机初始化一个类时，会先初始化它的父类，但是**这条规则不适用于接口**。
- <font color="red">在初始化一个类时，并不会初始化它实现的接口</font>
- <font color="red">在初始化一个接口时，并不会初始化它的父接口</font>

因此，一个父接口并不会因为它的子接口或者实现类的初始化而初始化，只有当程序首次使用特定接口的静态变量时，才会导致该接口的初始化。
#### final类型的静态变量
- 如果一个静态变量的值是一个编译时的常量，就不会对类型进行初始化（类的static块不执行）
- 如果一个静态变量的值是一个非编译时的变量，即只有运行时会有确定的初始值，则就会对这个类型进行初始化（类的static块执行）

#### ClassLoader
ClassLoader类的loadClass()知识加载类，不初始化类。

### 二、类加载器
类加载器负责将.class文件加载到内存，并为其创建java.lang.Class对象，这个对象就代表这个类。
在java中，通过包名+类名来唯一标识一个类，而在JVM中，要用类加载器实例+包名+雷鸣 来唯一标识一个类。
在JVM中，类加载器是层次结构的，从上当下依次为根加载器（BootStrapLoader）、扩展类加载器（ExtentionLoader）、应用类加载器（ApplicaitonLoader）和用户自定义类加载器。
#### 1. 根加载器（BootStrapLoader）
负责加载JAVA核心类，底层由C/C++实现，不是java.lang.Classloader的子类
#### 2. 扩展类加载器（ExtentionLoader）
负责加载来自JRE的扩展目录（\jre\lib\ext\ 或者 java.ext.dirs 系统属性指定的目录）中JAR包中的类，，我们也可以将自己的类放到这个目录作为扩展类加载。
#### 系统类加载器
也称为应用类加载器，负责加载下面几种类，
- JVM启动时加载来自 java命令的 -classpath选项的JAR包
- java.lang.path系统属性
- CLASSPATH环境变量

#### 类的加载机制
- 全盘负责
当一个类的加载器加载一个CLass时，改class依赖的其他class也由这个加载器加载。
- 父类委托

JVM加载一个Class时，会先使用其父类加载器来加载，所以一直迭代到最上层的加载器，一个类会最先由BootstrapLoader尝试加载，如果失败则由extensionLoader尝试加载，再失败则由systemLoader尝试加载，最后还失败则由自定义的类加载器来加载，如果依然失败，就会抛出错误。 父类委托机制可以防止类被重复加载，也更安全。
![Alt text](./1503387415981.png)
但是Tomcat采用了完全相反的机制，先通过默认类加载器加载，失败再找父类加载器加载。
![Alt text](./1503387471723.png)
#### 缓存机制
缓存机制保证加载过的Class被缓存起来，当家在新类时，先进缓存查询是否已经加载，只有缓存中没有的时候才进行加载，这样会显著提高性能。





