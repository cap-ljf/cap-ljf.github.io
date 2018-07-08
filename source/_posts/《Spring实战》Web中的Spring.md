---
toc: true
title: 《Spring实战》Web中的Spring
date: 2018-04-10 10:58:29
tags: [Spring MVC]
---

在上一篇博客中我们简单介绍了如何搭建一个最简单的Spring MVC，现在我们来学习一下Spring的其他Web功能。为了方便学习，我将自己学习时敲下的源码放到了github上：https://github.com/cap-ljf/Spittr
<!--more-->
本文内容对应《Spring实战》第二部分 Web中的Spring 内容：
- JavaConfig配置Spring MVC
- Spring MVC数据绑定
- JSR 303数据校验
- Spring Web视图渲染之Thymeleaf
- JavaConfig配置自定义Servlet和Filter
- 文件上传
- 统一异常处理


### 1. JavaConfig配置Spring MVC
[《Spring实战》搭建Spring MVC](http://cap-ljf.top/2018/04/08/%E3%80%8ASpring%E5%AE%9E%E6%88%98%E3%80%8B%E6%90%AD%E5%BB%BASpring-MVC/)

### 2. 数据绑定入门
[[Spring MVC] - SpringMVC的各种参数绑定方式](https://www.cnblogs.com/HD/p/4107674.html)
[SpringMVC数据绑定入门](https://www.imooc.com/learn/558)

### 3. JSR 303数据校验
在了解了数据绑定之后，大部分时候我们都需要校验接受到的数据。有种处理校验的方式非常初级，那就是在controller或service方法中添加代码来检查值的合法性，如果值不合法的话，就将注册表单重新显示给用户。但是这样的校验逻辑会弄乱我们的controller方法。从Spring3.0开始，在SpringMVC中提供了对Java校验API的支持（JSR-303）。

使用JSR 303需要两个jar包依赖。
```xml
		<!-- JSR303校验 -->
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.1.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.3.6.Final</version>
        </dependency>
```
**Bean Validation中内置的constraint**
![Alt text](https://app.yinxiang.com/shard/s15/res/e7d4563c-a6bc-4a99-926a-55d052fc28ec/1523324310124.png)
**Hibernate Validator 附加的 constraint**
![Alt text](https://app.yinxiang.com/shard/s15/res/9b50d558-e179-465d-818f-ff0a415cf77e/1523324344858.png)

有些时候，你可能需要更复杂的constraint，Bean Validation 提供扩展 constraint 的机制。可以通过两种方法去实现：
- 组合现有的 constraint 来生成一个更复杂的 constraint
```java
@NotNull
@Size(min = 5, max = 10)
private String username;
```
- 自定义constraint。
定义的`IntegerRange`注解。自定义constraint注解，message、groups和payload三个属性是必须定义的。
```java
package com.jifang.spittr.annotation;

import com.jifang.spittr.validator.IntegerRangeValidator;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = IntegerRangeValidator.class)
public @interface IntegerRange {
    String message() default "校验失败";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    int min();
    int max();
}
```
定义验证类
```java
package com.jifang.spittr.validator;

import com.jifang.spittr.annotation.IntegerRange;
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
/**
 * author: jifang
 * date: 18-4-9 下午11:08
 */
public class IntegerRangeValidator implements ConstraintValidator<IntegerRange, Integer> {
    private Integer min;
    private Integer max;

    public void initialize(IntegerRange constraintAnnotation) {
        min = constraintAnnotation.min();
        max = constraintAnnotation.max();
    }

    public boolean isValid(Integer integer, ConstraintValidatorContext constraintValidatorContext) {
        boolean b = integer > min && integer < max ? true : false;
        return b;
    }
}
```
ConstraintValidator使用了泛型，有两个类型参数。第一个类型是对应的initialize方法的参数类型（约束注解类型），第二个类型是对应的isValid方法的第一个参数类型。


### 4.Spring Web视图渲染之Thymeleaf
将控制器中请求处理的逻辑和视图中的渲染实现解耦是Spring MVC的一个重要特性。
Spring自带了13个视图解析器，能够将逻辑视图名转换为物理实现。
![Alt text](https://app.yinxiang.com/shard/s15/res/eb8d5dab-3036-4eb0-9eb5-a2a40b2b8b14/1523325208325.png)
![Alt text](https://app.yinxiang.com/shard/s15/res/f304df0e-d997-4cfb-aa65-8cca5dcc6a82/1523325213803.png)
在[上一篇博文](http://cap-ljf.top/2018/04/08/%E3%80%8ASpring%E5%AE%9E%E6%88%98%E3%80%8B%E6%90%AD%E5%BB%BASpring-MVC/)我们介绍了`InternalResourceViewResolver`将视图解析为JSP。这一次我们使用新的项目`Thymeleaf`来配置实现视图解析。
```java
/**
     * 配置生成模板解析器
     *
     * @return
     */
    @Bean
    public TemplateResolver templateResolver() {
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver();
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setTemplateMode("HTML5");
        return templateResolver;
    }

    /**
     * 生成模板引擎并为模板引擎注入模板解析器
     *
     * @param templateResolver
     * @return
     */
    @Bean
    public SpringTemplateEngine templateEngine(TemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    /**
     * 生成视图解析器并为解析器注入引擎
     *
     * @param templateEngine
     * @return
     */
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }
```
需要注意的是ThymeleafViewResolver bean中注入了一个对SpringTemplate Engine bean的引用。SpringTemplateEngine会在Spring中启用Thymeleaf引擎,用来解析模板,并基于这些模板渲染结果。可以看到,我们为其注入了一个TemplateResolver bean的引用。

Thymeleaf在很大程度上就是HTML文件,与JSP不同,它没有什么特殊的标签或标签库。要使用Thymeleaf的标签库，需要声明Thymeleaf的命名空间
```
<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml" 
xmlns:th="http://thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Spittr</title>
    <link rel="stylesheet"
          type="text/css"
          th:href="@{/resources/style.css}"/>
</head>
<body>
<h1>Welcome to Spitter</h1>
<a th:href="@{/spittles}">Spittles</a>
<a th:href="@{/spitter/register}">Register</a>
<br/>
<a th:href="@{/databind}">databind</a>
<a th:href="@{/upload}">upload</a>
</body>
</html>
```
关于Thymeleaf可以参考
http://www.cnblogs.com/hjwublog/p/5051732.html#autoid-11-0-0

### 5.JavaConfig配置自定义Servlet和Filter
实现了WebApplicationInitializer接口的类会被自动加载。我们可以在`onStartup`中添加自定义的Servlet和Filter。
```java
package com.jifang.spittr.config;

import com.jifang.spittr.filter.MyFilter;
import com.jifang.spittr.servlet.MyServlet;
import org.springframework.web.WebApplicationInitializer;
import javax.servlet.*;
import java.util.EnumSet;

/**
 * 通过实现WebApplicationInitializer来配置额外的Servlet和Filter
 * author: jifang
 * date: 18-4-9 下午3:39
 */

public class MyServletInitializer implements WebApplicationInitializer {
    public void onStartup(ServletContext servletContext) throws ServletException {
        //配置自己的Servlet
        ServletRegistration.Dynamic myServlet = servletContext.addServlet("myServlet", MyServlet.class);
        myServlet.setLoadOnStartup(1);
        myServlet.addMapping("/myServlet/*");
        //配置自己的Filter
        FilterRegistration.Dynamic myFilter = servletContext.addFilter("myFilter", MyFilter.class);
        myFilter.addMappingForServletNames(EnumSet.of(DispatcherType.REQUEST), true, "myServlet");
    }
}
```

### 6.文件上传
一般表单提交所形成的请求结果是很简单的,就是以“&”符分割的多个name-value对。尽管这种编码形式很简单,并且对于典型的基于文本的表单提交也足够满足要求,但是对于传送二进制数据,如上传图片,就显得力不从心了。与之不同的,multipart格式的数据会将一个表单拆分为多个部分(part),每个部分对应一个输入域。在一般的表单输入域中,它所对应的部分中会放置文本型数据,但是如果上传文件的话,它所对应的部分可以是二进制。

#### 6.1 MultiPart形式的数据
Multipart格式数据会将一个表单拆分为多个部分(part)，每个部分对应一个输入域。在一般的表单输入域中，它对应的部分会放置文本型数据，如果是文件上传形式，它对应的部分可以是二进制。
#### 6.2 Multipart/form-data请求方式
示例：
```
<form action="/spittles/upload.do" method="POST" th:object="${spittr}" enctype="multipart/form-data">
    <label>Profile Picture</label>:
    <input type="file" name="profilePicture" accept="image/jpeg,image/png,image/gif"/><br/>
    <input type="submit" value="save"/>
</form>
```
#### 6.3 SpringMVC处理Multipart数据
xml配置可以参考：https://www.cnblogs.com/maying3010/p/6679974.html
下面介绍JavaConfig配置：
**配置multipart解析器**
DispatcherServlet并没有实现任何解析multipart请求数据的功能。它将该任务委托给了Spring中MultipartResolver策略接口的实现,通过这个实现类来解析multipart请求中的内容。从Spring 3.1开始,Spring内置了两个MultipartResolver的实现供我们选择:
- CommonsMultipartResolver:使用Jakarta Commons FileUpload解析multipart请求;
- StandardServletMultipartResolver:依赖于Servlet 3.0对multipart请求的支持(始于Spring 3.1)。

一般来讲,在这两者之间,StandardServletMultipartResolver可能会是优选的方案。它使用Servlet所提供的功能支持,并不需要依赖任何其他的项目。如果我们需要将应用部署到Servlet 3.0之前的容器中,或者还没有使用Spring 3.1或更高版本,那么可能就需要CommonsMultipartResolver了。
兼容Servlet 3.0的StandardServletMultipartResolver没有构造器参数,也没有要设置的属性。
```java
	@Bean
    public MultipartResolver multipartResolver() {
        return new StandardServletMultipartResolver();
    }
```
我们不能在Spring中配置StandardServletMultipartResolver的限制条件，而是要在Servlet中指定multipart的配置。

如果我们采用Servlet初始化类的方式来配置DispatcherServlet的话，这个初始化类应该已经实现了WebApplicationInitializer，那我们可以在`ServletRegistration.Dynamic`上调用`setMultipartConfig()`方法，传入MultipartConfigElement实例。如果我们配置DispatcherServlet的Servlet初始化类继承了AbstractAnnotationConfigDispatcherServletInitializer类的话，那么我们不会直接创建DispatcherServlet实例并将其注册到Servlet上下文中。这样的话,将不会有对Dynamic Servlet registration的引用供我们使用了。但是,我们可以通过重载customizeRegistration()方法(它会得到一个Dynamic作为参数)来配置multipart的具体细节:
```java
/**
     * 配置multipart的具体细节
     *
     * @param registration
     */
    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setMultipartConfig(new MultipartConfigElement("/home/jifang/spittr/uploads", 2097152, 4194304, 0));
    }
```
除了临时路径的位置,其他的构造器所能接受的参数如下:
- 上传文件的最大容量(以字节为单位)。默认是没有限制的。
- 整个multipart请求的最大容量(以字节为单位),不会关心有多少个part以及每个part的大小。默认是没有限制的。
- 在上传的过程中,如果文件大小达到了一个指定最大容量(以字节为单位),将会写入到临时文件路径中。默认值为0,也就是所有上传的文件都会写入到磁盘上。

**处理multipart请求**
1. 使用byte[] 数组
2. 使用上传文件的原始byte比较简单但是功能有限。因此,Spring还提供了MultipartFile接口,它为处理multipart数据提供了内容更为丰富的对象。
```java
public interface MultipartFile extends InputStreamSource {
    String getName();

    String getOriginalFilename();

    String getContentType();

    boolean isEmpty();

    long getSize();

    byte[] getBytes() throws IOException;

    InputStream getInputStream() throws IOException;

    void transferTo(File var1) throws IOException, IllegalStateException;
}
```
在controller里保存上传的文件
```java
@RequestMapping("upload.do")
    public String upload(@RequestPart("profilePicture") MultipartFile profilePicture) throws IOException {
        File file = new File("/home/jifang/spittr/uploads/");
        if (!file.exists() && !file.isDirectory()) {
            file.mkdir();
        }
        profilePicture.transferTo(new File("/home/jifang/spittr/uploads/" + profilePicture.getOriginalFilename()));
        return "home";
    }
```

### 7. 异常处理
不管请求成功或失败，Servlet请求的输出都是一个Servlet相应。异常必须要以某种方式转换为响应。

Spring提供了多种方式将异常转换为响应：
- 特定的Spring异常将会自动映射为指定的HTTP状态码;
- 异常上可以添加@ResponseStatus注解,从而将其映射为某一个HTTP状态码;
- 在方法上可以添加@ExceptionHandler注解,使其用来处理异常。

#### 7.1 将异常对象映射为HTTP状态码（@ResponseStatus）
Spring默认会将自身抛出的异常自动映射到合适的状态码，如下是一些示例：
![Alt text](https://app.yinxiang.com/shard/s15/res/df5202a1-79e7-4677-956f-234e1879e86f/1523328457188.png)
也可以通过@ResponseStatus注解将自定义异常映射为HTTP状态码。
```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Spittle Not Found")
public class SpittleNotFoundException extends RuntimeException {
}
```
在引入@ResponseStatus注解之后,如果控制器方法抛出SpittleNotFoundException异常的话,响应将会具有404状态码,这是因为Spittle Not Found。

#### 7.2 本地处理@ExceptionHandler
`@ExceptionHandler(SpittleNotFoundException.class)`，当抛出
SpittleNotFoundException异常的时候,将会委托该方法来处理。
```java
	@ExceptionHandler(SpittleNotFoundException.class)
    public String spittleNotFoundHandler() {
        return "error/spittleNotFound";
    }
```
这个方法现在只能处理它所属类的方法抛出的异常，如果要处理程序中所有的会抛出该异常的controller，那么可以使用通知（@ControllerAdvice）
#### 7.3 全局处理@ControllerAdvice
在带有@ControllerAdvice注解的类中,以上所述的这些方法会运用到整个应用程序所有控制器中带有@RequestMapping注解的方法上。
 @ControllerAdvice注解本身已经使用了@Component,因此@ControllerAdvice注解所标注的类将会自动被组件扫描获取到,就像带有@Component注解的类一样。
```java
@ControllerAdvice
public class AppWideExceptionHandler {
    @ExceptionHandler(SpittleNotFoundException.class)
    public String spittleNotFoundHandler() {
        return "error/spittleNotFound";
    }
}
```


> 参考文献
> [1] [Java数据校验(Bean Validation / JSR303)](https://www.cnblogs.com/pixy/p/5306567.html)
> [2] 《Spring实战》4th