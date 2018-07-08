---
toc: true
title: 《Spring实战》面向切面的Spring
date: 2018-04-05 10:06:10
tags: [Spring,aop]
---


在软件开发中,散布于应用中多处的功能被称为横切关注点(cross-cutting concern)。通常来讲,这些横切关注点从概念上是与应用的业务逻辑相分离的(但是往往会直接嵌入到应用的业务逻辑之中)。把这些横切关注点与业务逻辑相分离正是面向切面编程(AOP)所要解决的问题。

我们介绍了如何使用依赖注入(DI)管理和配置我们的应用对象。DI有助于应用对象之间的解耦,而AOP可以实现横切关注点与它们所影响的对象之间的解耦。
<!--more-->
### 1. 什么是面向切面编程
#### 1.1 定义AOP术语
**通知(Advice)**
切面的工作被称为通知，通知定义了切面是什么以及何时使用。

Spring切面可以应用5种类型的通知：
- 前置通知(Before):在目标方法被调用之前调用通知功能；
- 后置通知(After):在目标方法完成之后调用通知,此时不会关心方法的输出是什么;
- 返回通知(After-returning):在目标方法成功执行之后调用通知;
- 异常通知(After-throwing):在目标方法抛出异常后调用通知;
- 环绕通知(Around):通知包裹了被通知的方法,在被通知的方法调用之前和调用之后执行自定义的行为。

**连接点(Join point)**
我们的应用可能也有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中,并添加新的行为。

**切点(Poincut)**
如果说通知定义了切面的“什么”和“何时”的话,那么切点就定义了“何处”。切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称,或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。有些AOP框架允许我们创建动态的切点,可以根据运行时的决策(比如方法的参数值)来决定是否应用通知。

**切面(Aspect)**
切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么,在何时和何处完成其功能。

**引入(Introduction)**
引入允许我们向现有的类添加新方法或属性。例如,我们可以创建一个Auditable通知类,该类记录了对象最后一次修改时的状态。这很简单，只需一个方法，setLastModified(Date)，和一个实例变量来保存这个状态。然后,这个新方法和实例变量就可以被引入到现有的类中,从而可以在无需修改这些现有的类的情况下,让它们具有新的行为和状态。

**织入(Weaving)**
织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入:
- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器(ClassLoader),它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入(load-time weaving,LTW)就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下,在织入切面时,AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

#### 1.2 Spring对AOP的支持
Spring提供了4种类型的AOP支持：
- 基于代理的经典Spring AOP;
- 纯POJO切面;
- @AspectJ注解驱动的切面;
- 注入式AspectJ切面(适用于Spring各版本)。

前三种都是Spring AOP实现的变体,Spring AOP构建在动态代理基础之上,因此,Spring对AOP的支持局限于方法拦截。

### 2. 通过切点来选择连接点
在Spring AOP中,要使用AspectJ的切点表达式语言来定义切点。
关于Spring AOP的AspectJ切点,最重要的一点就是Spring仅支持AspectJ切点指示器(pointcutdesignator)的一个子集。
![Alt text](https://app.yinxiang.com/shard/s15/res/32c0c7c3-3903-458a-9f51-40dfc1843123/1522831830709.png)
在Spring中尝试使用AspectJ其他指示器时,将会抛出IllegalArgument-Exception异
常。当我们查看如上所展示的这些Spring支持的指示器时,注意只有execution指示器是实际执行匹配的,而其他的指示器都是用来限制匹配的。这说明execution指示器是我们在编写切点定义时最主要使用的指示器。在此基础上,我们使用其他指示器来限制所匹配的切点。

#### 2.1 编写切点
**exeution()**
格式：
`execution(返回类型 类名(参数类型)[异常类型])`
返回类型：必需，`*`表示所有返回类型
类名：必需，`a.*.*`表示`a`包下的所有类的所有方法，`set*`表示以`set`开头的方法。
参数类型：必需，`(..)`代表所有参数,`(*)`代表一个参数,`(*,String)`代表第一个参数为任何值,第二个为String类型.

示例1：
![Alt text](https://app.yinxiang.com/shard/s15/res/f93d95bc-1687-491d-af85-32efd6953518/1522890366397.png)

示例2：
![Alt text](https://app.yinxiang.com/shard/s15/res/bfa34aee-1c92-4041-8e05-e1ac59b6391a/1522890430573.png)
请注意我们使用了“&&”操作符把execution()和within()指示器连接在一起形成与(and)关系(切点必须匹配所有的指示器)。类似地,我们可以使用“||”操作符来标识或
(or)关系,而使用“!”操作符来标识非(not)操作。因为“&”在XML中有特殊含义,所以在Spring的XML配置里面描述切点时,我们可以使用and来代替“&&”。同样,or和not可以分别用来代替“||”和“!”。

除了以上的切点指示器，Spring引入了一个新的`bean()`指示器，它允许我们在切点表达式中使用bean的ID来标识`bean`

### 3. 使用注解定义切面
`@Aspect`：应用在类上声明该类为一个切面。
【示例】：
```java
@Aspect
@Component
public class AopLog {
    private static final Logger LOG = LoggerFactory
            .getLogger(AopLog.class);

    @Pointcut("execution( * com.stepbystep.spring4.samples.aop.*.*(..))")
    public void aspect(){}

    @Around("aspect()")
    public Object around(JoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            LOG.info("=======before {}",joinPoint.getSignature().getName());
            Object proceed = ((ProceedingJoinPoint) joinPoint).proceed();
            LOG.info("=======after {}",joinPoint.getSignature().getName());
            long end = System.currentTimeMillis();
                LOG.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms!");
            return proceed;
        } catch (Throwable e) {
            LOG.error(e.getMessage(),e);
            long end = System.currentTimeMillis();
            LOG.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms with exception : " + e.getMessage());
            throw e;
        }
    }
}
```
AspectJ提供了五个注解来定义通知：
![Alt text](https://app.yinxiang.com/shard/s15/res/cc518a43-f3f0-4c98-a539-03a26d6d3bf8/1522891975846.png)
这五个表达式接收`execution()`等切点表达式。


`@Pointcut`注解：能够在一个@AspectJ切面内定义可重用的切点。

关于这个新的通知方法,你首先注意到的可能是它接受ProceedingJoinPoint作为参数。这个对象是必须要有的,因为你要在通知中通过它来调用被通知的方法。通知方法中可以做任何的事情,当要将控制权交给被通知的方法时,它需要调用ProceedingJoinPoint的proceed()方法。

如果你就此止步的话,Audience只会是Spring容器中的一个bean。即便使用了AspectJ注解,但它并不会被视为切面,这些注解不会解析,也不会创建将其转换为切面的代理。

如果你使用JavaConfig配置，可以在配置类的类级别上加上`@EnableAspectJAutoProxy`注解启用自动代理功能。

假如你在Spring中要使用XML来装配bean的话,那么需要使用Spring aop命名空间中的`<aop:aspectj-autoproxy>`元素。下面的XML配置展现了如何完成该功能。

不管你是使用JavaConfig还是XML,AspectJ自动代理都会为使用@Aspect注解的bean创建一个代理,这个代理会围绕着所有该切面的切点所匹配的bean。

**处理通知中的参数**
如果切面所通知的方法确实有参数该怎么办呢？切面能访问和使用传递给被通知方法的参数吗？
答案是可以的。我们可以使用`@args(参数名称)`注解。它表明传递给playTrack()方法的int类型参数也会传递到通知中去。`参数名称`也与切点方法签名中的参数想匹配。切点定义中的参数与切点方法中的参数名称是一样的,这样就完成了从命名切点到通知方法的参数转移。

示例：
```java
@Aspect
public class TrackCounter {

    @Pointcut(
            "execution(* soundsystem.CompactDisc.playTrack(int)) && args(trackNumber)"
    )
    public void trackPlayed(int trackNumber) {
    }

    @Before("trackPlayed(trackNumber)")
    public void countTrack(int trackNumber){
        ...
    }
}
```

### 4. 通过注解引入新的功能
参考：https://blog.csdn.net/u010502101/article/details/76944753


