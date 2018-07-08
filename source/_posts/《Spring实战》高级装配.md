---
toc: true
title: 《Spring实战》高级装配
date: 2018-04-04 15:11:57
tags: [Spring, DI]
---


- Spring profile
- 条件化的bean声明
- 自动装配及歧义性
- bean的作用域
- SpEL
<!--more-->
### 1. 环境与profile

在正常开发过程中，我们会有不同的环境。例如：
- 开发环境（dev）：也就是每个人在电脑上进行开发的环境。
- Beta环境（beta）：即测试环境，QA环境，在测试机器上运行的环境。
- 线上环境（prod）：也称为生产环境。即在线上机器上跑的环境。

这里的环境指的是类似于数据库，ng等资源。

#### 1.1 配置profile bean
**在JavaConfig中配置profile**
`@Profile("dev")`：这个注解可以应用在类或方法上。如果应用于类级别上，他会告诉Spring这个配置类中的bean在相应dev profile 激活时才会创建。如果dev profile没有激活的话，那么带有`@Bean`注解的方法都会被忽略。

在Spring 3.1中,只能在类级别上使用@Profile注解。不过,从Spring 3.2开始,你也可以在方法级别上使用@Profile注解,与@Bean注解一同使用。这样的话,就能将这两个bean的声明放到同一个配置类之中。

没有指定profile的bean始终都会被创建,与激活哪个profile没有关系。

**在XML中配置profile**
使用标签：
```xml
<beans profile="dev">
	这里定义相应的bean
</beans>
```

### 2. 激活profile

Spring在确定哪个profile处于激活状态时，需要依赖两个独立的属性：`spring.profiles.active`和`spring.profiles.default`。如果设置了`spring.profiles.active`属性的话,那么它的值就会用来确定哪个profile是激活的。但如果没有设置`spring.profiles.active`属性的话,那Spring将会查找`spring.profiles.default`的值。如果`spring.profiles.active`和`spring.profiles.default`均没有设置的话,那就没有激活的profile,因此只会创建那些没有定义在profile中的bean。

有多种方法来设置这两种属性：
- 作为DispatcherServlet的初始化参数;
- 作为Web应用的上下文参数;
- 作为JNDI条目;
- 作为环境变量;
- 作为JVM的系统属性;
- 在集成测试类上,使用@ActiveProfiles注解设置。


**在Web应用的web.xml文件中设置默认的profile**

```xml
# 为web应用上下文设置默认的profile
<context-param>
	<param-name>spring.profiles.default</param-name>
	<param-value>dev</param-value>
</context-param>
# 为Servlet设置默认的profile
<servlet>
	<servlet-name>appServlet</servlet-name>
	<servlet-class>
		org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<init-param>
		<param-name>spring.profile.default</param-name>
		<param-value>dev</param-value>
	</init-param>
</servlet>
...
```

**使用profile进行测试**
当运行集成测试时,通常会希望采用与生产环境(或者是生产环境的部分子集)相同的配置进行测试。但是,如果配置中的bean定义在了profile中,那么在运行测试时,我们就需要有一种方式来启用合适的profile。

Spring提供了@ActiveProfiles注解,我们可以使用它来指定运行测试时要激活哪个profile。在集成测试时,通常想要激活的是开发环境的profile。例如,下面的测试类片段展现了使用@ActiveProfiles激活dev profile:
```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(classes=DataSourceConfig.class)
  @ActiveProfiles("dev")
  public static class DevDataSourceTest {
  }
```

### 2. 条件化bean
假设你希望一个或多个bean只有在应用的类路径下包含特定的库时才创建。或者我们希望某个bean只有当另外某个特定的bean也声明了之后才会创建。我们还可能要求只有某个特定的环境变量设置之后,才会创建某个bean。

Spring4引入了一个新的注解：
`@Conditional`：它可以用到带有@Bean注解的方法上。如果给定的条件计算结果为true,就会创建这个bean,否则的话,这个bean会被忽略。
```java
  @Bean
  @Conditional(MagicExistsCondition.class)
  public MagicBean magicBean() {
    return new MagicBean();
  }
```
可以看到,@Conditional中给定了一个Class,它指明了条件——在本例中,也就是MagicExistsCondition。@Conditional将会通过Condition接口进行条件对比:
```java
public class MagicExistsCondition implements Condition {

  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    Environment env = context.getEnvironment();
    return env.containsProperty("magic");
  }
  
}
```
设置给@Conditional的类可以是任意实现了Condition接口的类型。可以看出来,这个接口实现起来很简单直接,只需提供matches()方法的实现即可。如果matches()方法返回true,那么就会创建带有@Conditional注解的bean。如果matches()方法返回
false,将不会创建这些bean。

*提示：`@Profile`注解源码使用了`@Conditional`注解来实现*

### 3. 处理自动装配的歧义性

仅有一个bean匹配所需的结果时,自动装配才是有效的。如果不仅有一个bean能够匹配结果的话,这种歧义性会阻碍Spring自动装配属性、构造器参数或方法参数。

Spring提供了多种可选方案来解决这样的问题。你可以将可选bean中的某一个设为首选(primary)的bean,或者使用限定符(qualifier)来帮助Spring将可选的bean的范围缩小到只有一个bean。

`@Primary`：在声明bean的时候,通过将其中一个可选的bean设置为首选(primary)bean能够避免自动装配时的歧义性。当遇到歧义性的时候,Spring将会使用首选的bean,而不是其他可选的bean。实际上,你所声明就是“最喜欢”的bean。

**限定自动装配的bean**

`@Qualifier`注解是使用限定符的主要方式。它可以与@Autowired和@Inject协同使用,在注入的时候指定想要注入进去的是哪个bean。

更准确地讲,`@Qualifier("iceCream")`所引用的bean要具有String类型的“iceCream”作为限定符。如果没有指定其他的限定符的话,所有的bean都会给定一个默认的限定符,这个限定符与bean的ID相同。因此,框架会将具有“iceCream”限定符的bean注入到setDessert()方法中。这恰巧就是ID为iceCream的bean,它是IceCream类在组件扫描的时候创建的。

限定符和bean id 是不同的概念，但是默认的限定符就是bean id。

### 4. Bean的作用域
在默认情况下,Spring应用上下文中所有bean都是作为以单例(singleton)的形式创建的。也就是说,不管给定的一个bean被注入到其他bean多少次,每次所注入的都是同一个实例。

Spring定义了多种作用域,可以基于这些作用域创建bean,包括:
- 单例(Singleton):在整个应用中,只创建bean的一个实例。
- 原型(Prototype):每次注入或者通过Spring应用上下文获取的时候,都会创建一个新的bean实例。
- 会话(Session):在Web应用中,为每个会话创建一个bean实例。
- 请求(Rquest):在Web应用中,为每个请求创建一个bean实例。

`@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)`：这个注解能够更改bean的作用域。这里,使用ConfigurableBeanFactory类的SCOPE_PROTOTYPE常量设置了原型作用域。你当然也可以使用`@Scope("prototype")`,但是使用SCOPE_PROTOTYPE常量更加安全并且不易出错。

`@Scope`可以应用在组件扫描和Java配置中。
在XML配置中：
`<bean id="xxx" class="xxx" scope="prototype">`

会话和请求作用域指Spring为Web应用中的每个会话或请求创建一个对应的bean。

