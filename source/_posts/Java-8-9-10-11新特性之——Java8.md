---
toc: true
title: 'Java 8,9,10,11新特性之——Java8'
date: 2018-11-09 09:25:14
tags: [java]
---

北京时间 9 月 26 日，Oracle 官方宣布 Java 11 正式发布。这是 Java 大版本周期变化后的第一个长期支持版本，非常值得关注。吃饭的家伙都已经更新到11了，而我还在使用Java7，实在是看不下去了。俗话说公益善其事，必先利其器。对于Java本身的特性必须要好好掌握，这个系列文章将介绍Java8至11每一个版本的新特性。本篇讲解Java8。
<!--more-->

## 函数式接口
函数式接口就是一个具有一个方法的普通接口。像这样的接口，可以被隐式的转换为lambda表达式。java.lang.Runnable和java.util.concurrent.Callable是函数式接口最典型的两个例子。在实际应用中	，函数式接口很容易出错：如果某个人在接口定义中增加了另一个方法这时这个接口，那么编译就会出错。为了不让开发者容易弄出错，java8增加了一种特殊的注解`@FunctionalInterface`（Java8中所有类库的函数式接口都添加了这个注解）。
函数式接口的定义：
```java
@FunctionalInterface
interface FunctionalMethod {
    void method();
}
```
需要注意的一件事是：**默认方法和静态方法并不影响函数式接口的契约，可以任意使用**
```java
@FunctionalInterface
public interface FunctionalMethod {
    void method();
    static void staticMethod(){
        System.out.println("static method");
    }
    default void defaultMethod(){
        System.out.println("default method");
    }
}
```
使用方法：
```java
@Test
    public void test(){
        FunctionalMethod functionalMethod = ()-> System.out.println("hello world");
        functionalMethod.method();
        functionalMethod.defaultMethod();
        FunctionalMethod.staticMethod();
    }
```
## Lambda表达式
Java是一流的面向对象语言，但是在Java里将普通的方法像参数一样传值并不简单，为此，Java 8增加了一个语言及的新特性，名为**Lambda表达式**。
### Lambda 表达式简介
Lambda 表达式是一种匿名函数(对 Java 而言这并不完全正确，但现在姑且这么认为)，简单地说，它是没有声明的方法，也即没有访问修饰符、返回值声明和名字。
Java中的Lambda表达式通常使用`(argument)->{body}`语法书写，例如：
```java
(arg1, arg2...) -> { body }

(type1 arg1, type2 arg2...) -> { body }
```
以下是Lambda表达式的例子：
```java
(int a, int b) -> {  return a + b; }

() -> System.out.println("Hello World");

(String s) -> { System.out.println(s); }

() -> 42

() -> { return 3.1415 };
```
### Lambda 表达式的结构
首先看一下Lambda表达式的语法结构
- 一个 Lambda 表达式可以有零个或多个参数
- 参数的类型既可以明确声明，也可以根据上下文来推断。例如：`(int a)`与`(a)`效果相同
- 所有参数需包含在圆括号内，参数之间用逗号相隔。例如：`(a, b) `或` (int a, int b) `或` (String a, int b, float c)`
- 空圆括号代表参数集为空。例如：`() -> 42`
- 当只有一个参数，且其类型可推导时，圆括号`（）`可省略。例如：`a -> return a*a`
- Lambda 表达式的主体可包含零条或多条语句
- 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
- 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空
### 什么是函数式接口
在上文我们已经介绍过了**函数式接口**
### Lambda 表达式举例
**Runnable**
```java
// Java 8之前：
new Thread(new Runnable() {
    @Override
    public void run() {
    System.out.println("Before Java8, too much code for too little to do");
    }
}).start();

//Java 8方式：
new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
```
**使用lambda表达式对列表进行迭代**
```java
// Java 8之前：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
for (String feature : features) {
    System.out.println(feature);
}

// Java 8之后：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
features.forEach(n -> System.out.println(n));
 
// 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示，
// 看起来像C++的作用域解析运算符
features.forEach(System.out::println);
```
**Java 8中使用lambda表达式的Map和Reduce示例**
```java
// 不使用lambda表达式为每个订单加上12%的税
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    System.out.println(price);
}
 
// 使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);
```
### Lambda 表达式与匿名类的区别
既然lambda表达式即将正式取代Java代码中的匿名内部类，那么有必要对二者做一个比较分析。一个关键的不同点就是关键字 this。匿名类的 this 关键字指向匿名类，而lambda表达式的 this 关键字指向包围lambda表达式的类。另一个不同点是二者的编译方式。Java编译器将lambda表达式编译成类的私有方法。使用了Java 7的 invokedynamic 字节码指令来动态绑定这个方法。

## 接口默认方法和静态方法
在 java 8 之前，接口与其实现类之间的 耦合度 太高了（tightly coupled），当需要为一个接口添加方法时，所有的实现类都必须随之修改。默认方法解决了这个问题，它可以为接口添加新的方法，而不会破坏已有的接口的实现。这在 lambda 表达式作为 java 8 语言的重要特性而出现之际，为升级旧接口且保持向后兼容（backward compatibility）提供了途径。

默认方法与普通方法的不同之处在于抽象方法必须要求实现，但是默认方法则没有这个要求。相反，每个接口都必须提供一个所谓的默认实现，这样所有的接口实现者将会默认继承它（如果有必要的话，可以覆盖这个默认实现）。让我们看看下面的例子：
```java
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or 
    // may not implement (override) them.
    default String notRequired() { 
        return "Default implementation"; 
    }        
}
         
private static class DefaultableImpl implements Defaulable {
}
     
private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}
```
Defaulable接口用关键字default声明了一个默认方法notRequired()，Defaulable接口的实现者之一DefaultableImpl实现了这个接口，并且让默认方法保持原样。Defaulable接口的另一个实现者OverridableImpl用自己的方法覆盖了默认方法。

Java 8带来的另一个有趣的特性是接口可以声明（并且可以提供实现）静态方法。
```java
private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create( Supplier< Defaulable > supplier ) {
        return supplier.get();
    }
}
```
下面的一小段代码片段把上面的默认方法与静态方法黏合到一起。
```java
public static void main( String[] args ) {
    Defaulable defaulable = DefaulableFactory.create( DefaultableImpl::new );
    System.out.println( defaulable.notRequired() );
         
    defaulable = DefaulableFactory.create( OverridableImpl::new );
    System.out.println( defaulable.notRequired() );
}
```

## 方法引用
在前面我们介绍了java8的lambda表达式，而java的方法引用提供了非常有用的语法，可以直接引用已有Java类或对象的方法和构造器，与lambda联合使用，方法引用可以使语言的构造更紧凑简洁，减少冗余代码。

Java支持4中不同的方法引用，分别是：
- 类名::构造方法
- 类名::静态方法
- 类名::无参普通方法
- 对象名::普通方法（方法参数与lambda表达式参数类型需一致）
```java
public static class Car {
    public static Car create( final Supplier< Car > supplier ) {
        return supplier.get();
    }              
         
    public static void collide( final Car car ) {
        System.out.println( "Collided " + car.toString() );
    }
         
    public void follow( final Car another ) {
        System.out.println( "Following the " + another.toString() );
    }
         
    public void repair() {   
        System.out.println( "Repaired " + this.toString() );
    }
}
```
### 类名::构造方法
```java
final Car car = Car.create( Car::new );
final List< Car > cars = Arrays.asList( car );
```
### 类名::静态方法
```java
cars.forEach( Car::collide );
```
### 类名::无参普通方法
```java
cars.forEach( Car::repair );
```
### 对象名::普通方法（方法参数与lambda表达式参数类型需一致）
```java
final Car police = Car.create( Car::new );
cars.forEach( police::follow );
```

## Stream
Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。

本节完全参考 [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)

Stream不是集合元素，它不是数据结构不保存数据，它是和算法和计算有关的，它像是更高版本的Iterator。Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。而和迭代器又不同的是，Stream 可以并行化操作，迭代器只能命令式地、串行化操作。

Stream 的另外一大特点是，数据源本身可以是无限的。

当我们使用一个流的时候，通常包括三个基本步骤：

获取一个数据源（source）→ 数据转换→执行操作获取想要的结果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道，如下图所示。
![Alt text](https://app.yinxiang.com/shard/s15/res/b3741ad3-080e-4a76-89c7-d8f9441d2cde/1541692063211.png)

**流的生成**
从 Collection 和数组

**流的操作**
流的操作类型分为两种：
- **Intermediate**：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- **Terminal**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。

还有一种操作被称为 short-circuiting。用以指：
- 对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的 Stream，但返回一个有限的新 Stream。
- 对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。

关于流的更多操作请参考：
 [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)

## 其他特性
**重复注解**
**更好的类型推断机制**
**Optional**
**Date/Time API**
**Base64**
**并发**
等等

Java8主要功能为lambda表达式和Streams，有关java8更多的细节可以查询其他资料。

> 参考文献
> 1. [深入浅出 Java 8 Lambda 表达式](http://blog.oneapm.com/apm-tech/226.html)，OneAPM，OneAPM
> 2. [Java8 lambda表达式10个示例](http://www.importnew.com/16436.html), ImportNew, lemeilleur
> 3. [Java8中拥有默认方法实现的函数式接口和抽象类区别？王爵nice](https://www.zhihu.com/question/68474140)，知乎，王爵nice
> 4. [Java 8新特性终极指南](http://www.importnew.com/11908.html#defaultAndStaticMethod)，ImportNew，刘家财
> 5. [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)，IBM，陈 争云, 占 宇剑, 和 司 磊
