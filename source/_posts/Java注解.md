---
toc: true
title: Java注解
date: 2018-04-02 22:35:05
tags: [注解]
---

### 1. 概念
**注解是那些插入到源代码中使用其他工具可以对其进行处理的标签。这些工具可以在源码层次上进行操作，或者可以处理编译器在其中放置了注解的类文件。**
<!--more-->
看完了概念你理解了吗？注解本身并不能影响代码的执行，它能带来的影响是其他tools或Java编译器产生的。在这里我们只需要知道**注解**就相当于一个**标签**。

注解不会改变程序的编译方式。Java编译器对于包含注解和不包含注解的代码会生成相同的虚拟机指令。

### 2. 用法
注解的使用范围很广泛，我现在只是初步了解了注解的用法。在这儿摘抄一段《Java核心技术卷二》原文：
>下面是关于注解的一些可能的用法：
>- 附属文件的自动生成，例如部署描述符或者bean信息类。
>- 测试、日志、事务语义等代码的自动生成。

### 3. 注解语法

#### 注解定义
注解是由接口来定义的：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnotationTest{
	int age() default 18;
	String name() default "小明";
}
```
每个元素声明都具有下面这种形式：
`type elementName();`
或者
`type elementName() default value`
注解接口中的元素声明实际上是方法声明。一个注解接口的方法不能有任何参数和任何throws语句，并且它们也不能是泛型的。
注解元素的类型可以是下列之一：
- Java8种基本类型
- String
- Class（具有一个可选的类型参数，例如`Class<? extends MyClass>`）
- enum 类型
- 注解类型
- 由上述类型组成的数组（由数组组成的数组不是合法的元素类型）

既然一个注解元素可以是另一个注解，那么就可以创建出任意复杂的注解。例如：
`@BugReport(ref=@Reference(id="123"))`
*注意：在注解中引入循环依赖是一种错误。*

#### 注解使用
在Java中，注解是当作一个`修饰符`来使用的，它被置于被注解项之前，中间没有分号。（修饰符就是诸如`public`和`static`之类的关键词）。除了方法外，还可以注解类、成员以及局部变量，这些注解可以存在于任何可以放置一个像`public`或者`static`这样的修饰符的地方。

```java
    @AnnotationTest(age = 20, name="小红")
    public String toString() {
        return "test";
    }
```
有两个特殊的快捷方式可以用来简化注解。
如果没有指定元素，要么是因为注解中没有任何元素，要么是因为所有元素都是用默认值，那么你就不需要使用圆括号了。例如：
`@AnnotationTest`
和下面的注解是一样的
`@AnnotationTest(age =18, name = "小明")`
这样的注解又称为**标记注解**。
另外一种快捷方式是**单值注解**。如果一个元素具有特殊的名字`value`，并且没有指定其他元素，那么你就可以忽略掉这个元素名以及等号。例如：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OneValue{
	String value();
}
```
那么我们可以将这个注解写成：
`@OneValue("一个")`
代替：
`@OneValue(value = "一个")`

一个项可以有多个注解，只要他们属于不同的类型即可。（在JDK1.8以前当注解一个特定项的时候，不能多次使用同一注解类型。但是在1.8之后新增`@Repeatable`注解，可以重复使用）例如
```java
@OneValue("一")
@Onevalue("二")
public void test(){}
```
就是一种编译期错误。
如果你一定要多次使用一个注解，那么设计一个注解，它的值是一个由更简单的注解组成的数组：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OneValues{
	Onevalue[] OneValues();
}
```
```java
@Onevalues({@Onevalue("一"), @OneValue("二")})
public void test(){}
```

### 4. 标准注解
Java SE在java.lang、java.lang.annotation和javax.annotation包中定义了大量的注解接口。其中四个是**元注解**，用于描述注解接口的行为属性，还有一些其他的预置注解方便你注解你的源代码中的项。

#### 元注解
元注解是什么意思呢？

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。

如果难于理解的话，你可以这样理解。元注解也是一张标签，但是它是一张特殊的标签，它的作用和目的就是给其他普通的标签进行解释说明的。

元标签有 `@Retention、@Documented、@Target、@Inherited、@Repeatable` 5 种。

##### @Retention
Retention 的英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。
它的取值如下： 
- RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。 
- RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。 
- RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

##### @Documented
顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去。

##### @Target
Target 是目标的意思，@Target 指定了注解运用的地方。

你可以这样理解，当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。

类比到标签，原本标签是你想张贴到哪个地方就到哪个地方，但是因为 @Target 的存在，它张贴的地方就非常具体了，比如只能张贴到方法上、类上、方法参数上等等。@Target 有下面的取值
- ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
- ElementType.CONSTRUCTOR 可以给构造方法进行注解
- ElementType.FIELD 可以给属性进行注解
- ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
- ElementType.METHOD 可以给方法进行注解
- ElementType.PACKAGE 可以给一个包进行注解
- ElementType.PARAMETER 可以给一个方法内的参数进行注解
- ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

##### @Inherited
Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。 
说的比较抽象。代码来解释。
```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}

@Test
public class A {}

public class B extends A {}
```
注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。

可以这样理解：

老子非常有钱，所以人们给他贴了一张标签叫做富豪。

老子的儿子长大后，只要没有和老子断绝父子关系，虽然别人没有给他贴标签，但是他自然也是富豪。

老子的孙子长大了，自然也是富豪。

这就是人们口中戏称的富一代，富二代，富三代。虽然叫法不同，好像好多个标签，但其实事情的本质也就是他们有一张共同的标签，也就是老子身上的那张富豪的标签。
【示例】
```java
import java.lang.annotation.*;

@Documented
@Target(ElementType.TYPE)
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodInfo {
    public int age() default 20;
    public String name() default "小明";
}
```
```java
@MethodInfo
public class AnnotationExample {
}
```
```java
public class InheritedExample extends AnnotationExample {
}
```
【Test.java】
```java
package annotation;

/**
 * author: jifang
 * date: 18-4-2 下午9:51
 */

public class Test {
    public static void main(String[] args) {
        MethodInfo info = InheritedExample.class.getAnnotation(MethodInfo.class);
        System.out.println("age: " + info.age());
        System.out.println("name: " + info.name());
    }
}
```
运行`main`方法，得到结果：
```
age: 20
name: 小明
```

##### @Repeatable
Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。
举一个例子你就懂了，可以看这个链接：https://blog.csdn.net/z69183787/article/details/54602994


还有一些Java预置的注解如`@Deprecated、@Override、@SuppressWarnings、@SafeVarargs`等，有兴趣可以自行查阅。

### 5. 注解的提取：反射

注解通过反射获取。首先可以通过 Class 对象的 `isAnnotationPresent()` 方法判断它是否应用了某个注解。
```java
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```
然后通过 getAnnotation() 方法来获取 Annotation 对象。
```java
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
```
或者是 getAnnotations() 方法。
`public Annotation[] getAnnotations() {}`
前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解。

如果获取到的 Annotation 如果不为 null，则就可以调用它们的属性方法了。比如
```java
import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MethodInfo {
    public int age() default 20;
    public String name() default "小明";
}
```
```
@MethodInfo
public class AnnotationExample {
}
```
```java
import java.lang.annotation.Annotation;

/**
 * author: jifang
 * date: 18-4-2 下午9:51
 */

public class Test {
    public static void main(String[] args) {
        MethodInfo info = InheritedExample.class.getAnnotation(MethodInfo.class);
        System.out.println("age: " + info.age());
        System.out.println("name: " + info.name());
        Annotation[] annotations = AnnotationExample.class.getAnnotations();
        System.out.println(annotations.length);
    }
}
```
结果：
```bash
age: 20
name: 小明
1
```
需要注意的是，如果一个注解要在运行时被成功提取，那么`@Retention(RetentionPolicy.RUNTIME) `是必须的。


### 6. 源码级注解处理
明天再研究一下补充。


> 参考文献
> [1] [秒懂，Java 注解 （Annotation）你可以这样学](https://blog.csdn.net/briblue/article/details/73824058)
> [2] [java8 新增的@Repeatable注解](https://blog.csdn.net/z69183787/article/details/54602994)
> [3] 《Java核心技术卷二》