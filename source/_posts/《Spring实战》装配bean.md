---
toc: true
title: 《Spring实战》装配bean
date: 2018-04-03 23:16:31
tags: [Spring]
---

创建应用对象之间协作关系的行为（类的依赖）通常称为装配（wiring），这也是依赖注入的本质。
<!--more-->
### 1. Spring配置的可选方案
在 Spring 中装配 bean 有多种方式。Spring 容器负责创建应用程序中的 bean 并通过 DI 来协调这些对象之间的关系。但是,作为开发人员,你需要告诉 Spring 要创
建哪些 bean 并且如何将其装配在一起。当描述 bean 如何进行装配时, Spring 具有非常大的灵活性,它提供了三种主要的装配机制:
- 在XMl中进行显示配置
- 在Java中进行显示配置
- 隐式的bean发现机制和自动装配

Spring 的配置风格是可以互相搭配的,所以你可以选择使用 XML 装配一些 bean ,使用 Spring 基于 Java 的配置( JavaConfig )来装配另一些 bean ,而将剩余的bean 让 Spring 去自动发现。

**即便如此,我的建议是尽可能地使用自动配置的机制。显式配置越少越好。当你必须要显式配置 bean 的时候(比如,有些源码不是由你来维护的,而当你需要为这些代码配置 bean 的时候),我推荐使用类型安全并且比 XML 更加强大的 JavaConfig 。最后,只有当你想要使用便利的 XML命名空间,并且在 JavaConfig 中没有同样的实现时,才应该使用 XML 。**

### 2. 自动化装配Bean（隐式）
#### 2.1 创建可被发现的Bean
尽管你会发现这些显式装配技术非常有用,但是在便利性方面,最强大的还是 Spring 的自动化配置。如果 Spring 能够进行自动化装配的话,那何苦还要显式地将这些 bean 装配在一起呢?

Spring从两个角度来实现自动化装配：
- 组件扫描（component scanning）: Spring会自动发现应用上下文中所创建的Bean
- 自动装配（autowiring）: Spring自动满足bean之间的依赖

组件扫描和自动装配组合在一起就能发挥出强大的威力,它们能够将你的显式配置降低到最少。

`@component`：这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建bean。

不过，组件扫描默认是不启用的。我们还需要显示配置一下Spring，从而命令它去寻找带有`@component`注解的类，并为其创建bean。

有两种方式启用组件扫描：
1. @ComponentScan 注解启用了组件扫描
```java
@Configuration
@ComponentScan
public class CDPlayerConfig{
}
```
类 CDPlayerConfig 通过 Java 代码定义了 Spring 的装配规则。
`@ComponentScan`：这个注解能够在Spring中启用组件扫描。如果没有其他配置的话, @ComponentScan 默认会扫描与配置类相同的包。
2. 通过XML启用组件扫描
使用 Spring context 命名空间的 <context:component-scan> 元素。
```
<context:component-scan base-package="soundsystem"/>
```
`context:component-scan> `元素会有与`@ComponentScan `注解相对应的属性和子元素。

**测试组件扫描能够发现`CompactDisc`**
```
@Run
```
使用了 Spring 的 `SpringJUnit4ClassRunner` ,以便在测试开始的时候自动创建 Spring 的应用上下文。
`@ContextConfiguration`会告诉它需要在CDPlayerConfig中加载配置。

#### 2.2 为组件扫描的bean命名
Spring 应用上下文中所有的 bean 都会给定一个 ID 。在前面的例子中,尽管我们没有明确地为 SgtPeppers bean 设置 ID ,但 Spring 会根据类名为其指定一个 ID 。具体来讲,这个 bean 所给定的 ID 为 sgtPeppers ,也就是将类名的第一个字母变为小写。

如果想为这个 bean 设置不同的 ID ,你所要做的就是将期望的 ID 作为值传递给 @Component 注解。比如：
```java
@Component("longlyHeartsClub")
public class SgtPeppers implements CompactDisc{
...
}
```
也可以使用`@Named`注解替代`@component`，但是建议使用`@component`
#### 2.3 设置组件扫描的基础包
`@ComponentScan("soundsystem")`
`@ComponentScan(basePackages="soundsystem")`
`@ComponentScan(basePackages={"soundsystem","video"})`
`@ComponentScan(basePackageClasses={CDplayer.class,DVDPlayer.class})`

#### 2.4 通过为 bean 添加注解实现自动装配
简单来说,自动装配就是让 Spring 自动满足 bean 依赖的一种方法,在满足依赖的过程中,会在 Spring 应用上下文中寻找匹配某个 bean 需求的其他 bean 。为了声明要进行自动装配,我们可以借助 Spring 的 @Autowired 注解。

`@Autowired`：这个注解可以用在类的任何方法上。这表明当这行某个方法时Spring会创建参数对应的Bean。进行依赖注入。它也可以用在类变量上，这样就相当于是使用在该类变量的`setter`方法上。

不管是构造器、 Setter 方法还是其他的方法, Spring 都会尝试满足方法参数上所声明的依赖。假如有且只有一个 bean 匹配依赖需求的话,那么这个 bean 将会被装配进来。

如果没有匹配的 bean ,那么在应用上下文创建的时候, Spring 会抛出一个异常。为了避免异常的出现,你可以将 @Autowired 的 required 属性设置为 false 。将 required 属性设置为 false 时, Spring 会尝试执行自动装配,但是如果没有匹配的 bean 的话, Spring 将会让这个 bean 处于未装配的状态。但是,把 required 属性设置为 false 时,你需要谨慎对待。如果在你的代码中没有进行 null 检查的话,这个处于未装配状态的属性有可能会出现 NullPointerException 。

如果有多个 bean 都能满足依赖关系的话, Spring 将会抛出一个异常,表明没有明确指定要选择哪个 bean 进行自动装配。

`@Inject` 和 `@Autowired`有相同的功能。

### 3. 通过Java代码装配bean
尽管在很多场景下通过组件扫描和自动装配实现 Spring 的自动化配置是更为推荐的方式,但有时候自动化配置的方案行不通,因此需要明确配置 Spring 。比如说,你想要将第三方库中的组件装配到你的应用中,在这种情况下,是没有办法在它的类上添
加 @Component 和 @Autowired 注解的,因此就不能使用自动化装配的方案了。

#### 3.1 创建配置类

创建 JavaConfig 类的关键在于为其添加 @Configuration 注解, @Configuration 注解表明这个类是一个配置类,该类应该包含在 Spring 应用上下文中如何创建 bean 的细节。
`@Configuration`:表明这个类是一个配置类,该类应该包含在 Spring 应用上下文中如何创建 bean 的细节。

#### 3.2 声明简单的bean
要在 JavaConfig 中声明 bean ,我们需要编写一个方法,这个方法会创建所需类型的实例,然后给这个方法添加 @Bean 注解。例如：
```java
@Bean
  public CompactDisc compactDisc() {
    return new SgtPeppers();
  }
```
`@Bean`： 注解会告诉 Spring 这个方法将会返回一个对象,该对象要注册为 Spring 应用上下文中的 bean 。方法体中包含了最终产生 bean 实例的逻辑。

默认情况下, bean 的 ID 与带有 @Bean 注解的方法名是一样的。在本例中, bean 的名字将会是 sgtPeppers 。如果你想为其设置成一个不同的名字的话,那么可以重命名该方法,也可以通过 name 属性指定一个不同的名字:
```java
  @Bean(name="othername")
  public CompactDisc compactDisc() {
    return new SgtPeppers();
  }
```

#### 3.3 借助JavaConfig实现注入
我们前面所声明的 CompactDisc bean 是非常简单的,它自身没有其他的依赖。但现在,我们需要声明 CDPlayerbean ,它依赖于 CompactDisc 。在 JavaConfig 中,要如何将它们装配在一起呢?
1. 引用创建 bean 的方法
```java
@Bean
  public CDPlayer cdPlayer() {
    return new CDPlayer(compactDisc());
  }
```
因为`compactDisc()`方法添加了`@Bean`注解，Spring会拦截所有对它的调用，并确保直接返回该方法所创建的bean。而不是每次都对其进行实际的调用。（即bean都是单例）

可以看到,通过调用方法来引用bean的方式有点令人困惑。其实还有一种理解起来更为简单的方式:
```java
  @Bean
  public CDPlayer cdPlayer(CompactDisc compactDisc) {
    return new CDPlayer(compactDisc);
  }
```

### 4. 通过XML装配bean
尽管Spring长期以来确实与XML有着关联,但现在需要明确的是,XML不再是配置Spring的唯一可选方案。Spring现在有了强大的自动化配置和基于Java的配置,XML不应该再是你的第一选择了。

不过,鉴于已经存在那么多基于XML的Spring配置,所以理解如何在Spring中使用XML还是很重要的。但是,我希望本节的内容只是用来帮助你维护已有的XML配置,在完成新的Spring工作时,希望你会使用自动化配置和JavaConfig。

#### 4.1 创建XML配置规范
在使用XML为Spring装配bean之前,你需要创建一个新的配置规范。在使用avaConfig的时候,这意味着要创建一个带有@Configuration注解的类,而在XML配置中,这意味着要创建一个XML文件,并且要以`<beans>`元素为根。
![Alt text](https://app.yinxiang.com/shard/s15/res/3447fe93-de26-4eb3-b358-7bc65b51e8c1/1522765974699.png)

#### 4.2 声明一个简单的`<bean>`
`<bean>`元素元素类似于JavaConfig中的@Bean注解。我们可以按照如下的方式声明`CompactDisc` bean:
`<bean class="soundsystem.SgtPeppers" />`
这里声明了一个很简单的bean,创建这个bean的类通过class属性来指定的,并且要使用全限定的类名。

因为没有明确给定ID,所以这个bean将会根据全限定类名来进行命名。在本例中,bean的ID将会是“soundsystem.SgtPeppers#0”。其中,“#0”是一个计数的形式,用来区分相同类型的其他bean。如果你声明了另外一个SgtPeppers,并且没有明确进行标识,那么它自动得到的ID将会是“soundsystem.SgtPeppers#1”。

尽管自动化的bean命名方式非常方便,但如果你要稍后引用它的话,那自动产生的名字就没有多大的用处了。因此,通常来讲更好的办法是借助id属性,为每个bean设置一个你自己选择的名字:
`<bean id="compactDisc" class="soundsystem.SgtPeppers" />`

#### 4.3 借助构造器注入初始化bean
在Spring XML配置中,只有一种声明bean的方式:使用`<bean>`元素并指定class属性。Spring会从这里获取必要的信息来创建bean。但是,在XML中声明DI时,会有多种可选的配置方案和风格。具体到构造器注入,有两种基本的配置方案可供选择：
- `<constructor-arg>`元素
- 使用Spring 3.0所引入的c-命名空间

**构造器注入bean引用**
```xml
<bean id="cdPlayer" class="soundsystem.cdPlayer">
	<constructor-arg ref="compactDisc" />
</bean>
```
![Alt text](https://app.yinxiang.com/shard/s15/res/4259a7f5-f0b6-4863-93ff-b508cdf8d824/1522766746766.png)

使用`<constructor-arg>`元素进行构造器参数的注入

#### 4.4 设置属性
`<property>`元素为属性的Setter方法所提供的功能与`<constructor-arg>`元素为构造器所提供的功能是一样的。
![Alt text](https://app.yinxiang.com/shard/s15/res/b365723c-f3c5-43d7-934e-088cd6a472f0/1522767233977.png)

### 5. 导入和混合配置
在典型的Spring应用中,我们可能会同时使用自动化和显式配置。即便你更喜欢通过
JavaConfig实现显式配置,但有的时候XML却是最佳的方案。

**关于混合配置,第一件需要了解的事情就是在自动装配时,它并不在意要装配的bean来自哪里。自动装配的时候会考虑到Spring容器中所有的bean,不管它是在JavaConfig或XML中声明的还是通过组件扫描获取到的。**

#### 5.1 在JavaConfig中引用XML配置

1. `A` JavaConfig类导入 `B` JavaConfig类：使用`@Import`注解导入。
```java
@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```
2. 在JavaConfig中引用XML配置，使用`@ImportResource`注解，示例同上。

#### 5.2 在XML配置中引用JavaConfig
1. 两个XML配置文件。使用`<import resource="">`标签
`<import resource="cdplayer-config.xml"/>`
2. 在XML配置中引用JavaConfig。使用`<bean>`标签。
`<bean class="soundsystem.CDconfig" />`

好了。在这一个博客中我们看到了在Spring中装配bean的三种方式：自动化配置、基于Java的显示配置和基于XML的显示配置。

建议使用自动化配置或JavaConfig配置，如果你是在维护一个旧系统，这里也介绍了如何进行混合配置。明天见。


