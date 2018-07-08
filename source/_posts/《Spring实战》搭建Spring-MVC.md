---
toc: true
title: 《Spring实战》搭建Spring MVC
date: 2018-04-08 16:19:01
tags: [Spring,web,Spring MVC]
---

本文采用Java将`DispatcherServlet`配置在`Servlet`容器中，而不会再使用`web.xml`文件。

### 1. Spring MVC启动流程
![Spring MVC启动流程图](https://app.yinxiang.com/shard/s15/res/683c4f95-8bd2-42f5-9024-8519c1b9a121/1523191585434.png)
<!--more-->
#### 1.1 

### 2. 搭建web项目

#### 2.1 使用maven创建webapp项目
maven是一款管理java项目依赖、编译、发布的工具。我们可以使用Idea或直接使用maven命令来创建`Spittr`项目。
- maven 命令
`mvn archetype:generate  -DarchetypeCatalog=local -DgroupId=com.jifang -DartifactId=spittr -DarchetypeArtifactId=maven-archetype-webapp `

注意`-DarchetypeCatalog=local`这里使用的是local而不是remote，因为使用remote需要去远程仓库下载，速度很慢，使用local很快。
- Idea 使用maven创建webapp项目
![Alt text](https://app.yinxiang.com/shard/s15/res/31c0786e-fa14-4cfe-93a8-95f8d1191cda/1523173940956.png)
![Alt text](https://app.yinxiang.com/shard/s15/res/0721e144-8736-488a-8c49-d31a5a0dabae/1523173969014.png)
填入`groupId`和`ArtifactId`即可，同样的可以在Idea的settings中将maven运行参数设置一下：
![Alt text](https://app.yinxiang.com/shard/s15/res/8cfc7e4e-7feb-4462-a5de-6fa1c954704f/1523174186104.png)

#### 2.2 配置DispatcherServlet
`DispatcherServlet`是Spring MVC的核心。按照传统方式，像DispatcherServlet这样的Servlet会配置在web.xml文件中，这个文件会放到应用的war包里。但是，借助于Servlet 3规范和Spring 3.1的功能增强，我们可以使用JavaConfig方式配置。

```java
package com.jifang.spittr.config;

import com.jifang.spittr.config.RootConfig;
import com.jifang.spittr.config.WebConfig;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * author: jifang
 * date: 18-4-6 上午11:11
 */

public class SpittrWebAppInitlizer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{RootConfig.class};
    }

    /**
     * 指定配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{WebConfig.class};
    }

    /**
     * 将DispatcherServlet映射到"/"
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/*"};
    }
}keyikandao
```
这里有两个方法`getRootConfigClasses`和`getServletConfigClasses`，而且他们返回了两个Class类型的数组，每一个都包含独特的一个类，分别是`RootConfig`和`WebConfig`。我们先来看看这两个类都是什么样的。
【RootConfig.java】
```java
package com.jifang.spittr.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

/**
 * author: jifang
 * date: 18-4-6 上午11:25
 */

@Configuration
@ComponentScan(basePackages = {"com.jifang.spittr"},
    excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, value = EnableWebMvc.class)})
public class RootConfig {
}
```
【WebConfig.java】
```java
package com.jifang.spittr.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

/**
 * 最小但可用的Spring MVC配置
 * author: jifang
 * date: 18-4-6 上午11:26
 */
@Configuration
@EnableWebMvc
@ComponentScan("com.jifang.spittr.web")
public class WebConfig extends WebMvcConfigurerAdapter{

	/**
     * 配置JSP视图解析器
     * @return
     */
    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        //使得可以在JSP页面中通过${ }访问容器中的bean。
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }

	/**
     * 配置静态资源的处理
     * @param configurer
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```
SpringMVC 项目启动时配置的东西还有很多，HandlerException 异常处理，数据校验等，这些Spring提供的抽象类WebMvcConfigurerAdapter中已经实现好了，我们在项目中直接继承就行了。


rootConfig应该扫描除了@Controller注解以外的注解，webConfig专注扫描的@Controller注解。容器启动时加载顺序是 先RootConfig 后WebConfig，RootConfig先加载时会创建Controller层的单例，紧接着WebConfig加载时会检测相应的Bean是否已经创建，若父容器（RootConfig）已经创建了该Bean，子容器（WebConfig）就不会再去创建了。关于`RootConfig`和`WebConfig`这两个类之间的区别可以参考我的下一篇文章：[AbstractAnnotationConfigDispatcherServletInitializer类方法getRootConfigClasses和getServletConfigClasses区别](http://cap-ljf.top/2018/04/08/AbstractAnnotationConfigDispatcherServletInitializer%E7%B1%BB%E6%96%B9%E6%B3%95getRootConfigClasses%E5%92%8CgetServletConfigClasses%E5%8C%BA%E5%88%AB/)

#### 2.3 定义最简单的Controller
```java
package com.jifang.spittr.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

/**
 * author: jifang
 * date: 18-4-6 下午2:07
 */
@Controller
@RequestMapping({"/","/homepage"})
public class HomeController {
    @RequestMapping(method = RequestMethod.GET)
    public String home(){
        return "home";
    }
}
```

包结构：
![Alt text](https://app.yinxiang.com/shard/s15/res/70895f8c-2de6-485f-8ace-fe1b53a60c18/1523176016984.png)
*框起来的是必须的*

pom文件：
```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
    <!-- Spring相关 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
  </dependencies>
```
配置好tomcat启动项目，运行之后可以看到Hello World！

> 参考文献
> [1] [零配置简单搭建SpringMVC 项目](http://www.cnblogs.com/beiyan/p/5942741.html)
> [2] 《Spring实战》4th





