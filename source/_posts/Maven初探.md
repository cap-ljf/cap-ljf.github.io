---
toc: true
title: maven入门
comments: true
date: 2017-10-23 10:56:10
tags: maven
---

今天整理一下对于Maven的学习。和学习其他知识一样，我在看完各种关于Maven的介绍以及文档之后，按照what，how，why来温习。
<!--more-->
### 1. What（Maven是什么？）
Maven是一个强大的Java项目构建工具，构建工具：将软件项目构建相关的过程自动化的工具。

构建一个软件项目通常包含以下一个或多个过程：
- 生成源码（如果项目使用自动生成源码）；
- 从源码生成项目文档；
- 编译源码；
- 将编译后的源码打包成JAR文件或者ZIP文件；
- 将打包好的代码安装到服务器、仓库或者其他的地方；

##### 安装Maven
[【项目管理和构建】——Maven下载、安装和配置（二）](http://blog.csdn.net/jiuqiyuliang/article/details/45390313)

##### Maven概览-核心概念
Maven的中心思想是**POM文件**（项目对象模型）。POM文件是以XML文件的形式表述项目的资源，如源码、测试代码、依赖（用到的外部Jar包）等。POM文件应该位于项目的根目录下。

下图说明了Maven是如何使用POM文件的，以及POM文件的主要组成部分：
![](http://ifeve.com/wp-content/uploads/2014/06/maven-overview-1.png)

##### 构建生命周期、构建阶段、构建目标
Maven的构建过程被分解为构建生命周期、阶段和目标。一个构建周期由一系列的构建阶段组成，每一个构建阶段由一系列的构建目标组成。当你运行Maven的时候，你会传入一条命令。这条命令就是构建生命周期、阶段或目标的名字。如果执行一个生命周期，该生命周期内的所有构建阶段都会被执行。如果执行一个构建阶段，在预定义的构建阶段中，所有处于当前构建阶段之前的阶段也都会被执行。  

##### 依赖和仓库
Maven执行时，其中一个首要目标就是检查项目的依赖。依赖是你的项目用到的jar文件（java库）。如果在本地仓库中不存在该依赖，则Maven会从中央仓库下载并放到本地仓库。本地仓库只是你电脑硬盘上的一个目录。你可以根据需要制定本地仓库的位置。你也可以指定下载依赖的远程仓库的地址。

##### 插件
构建插件可以向构建阶段中增加额外的构建目标。如果Maven标准的构建阶段和目标无法满足项目构建的需求，你可以在POM文件里增加插件。Maven有一些标准的插件供选用，如果需要你可以自己实现插件。

##### 配置文件
*不同的开发环境会使用不同的配置*
配置文件用于以不同的方式构建项目。比如，你可能需要在本地环境构建，用于开发和测试，你也可能需要构建后用于开发环境。这两个构建过程是不同的。在POM文件中增加不同的构建配置，可以启用不同的构建过程。当运行Maven时，可以指定要使用的配置。

##### 父pom
所有的Maven pom文件都继承自一个父pom。如果没有指定父pom，则该pom文件继承自根pom。pom文件的继承关系如下图所示：
![](http://ifeve.com/wp-content/uploads/2014/06/super-pom.png)
可以让一个pom文件显式地继承另一个pom文件。这样，可以通过修改公共父pom文件的设置来修改所有子pom文件的设置。在pom文件的起始处指定父pom，例如：
```xml
<project xmlns=”http://maven.apache.org/POM/4.0.0″
xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance”
xsi:schemaLocation=”http://maven.apache.org/POM/4.0.0
http://maven.apache.org/xsd/maven-4.0.0.xsd”>
<modelVersion>4.0.0</modelVersion>

<parent>
<groupId>org.codehaus.mojo</groupId>
<artifactId>my-parent</artifactId>
<version>2.0</version>
<relativePath>../my-parent</relativePath>
</parent>

<artifactId>my-project</artifactId>
…
</project>
```
子pom文件的设置可以覆盖父pom文件的设置，只需要在子pom文件里指定新的设置即可。

##### Maven配置文件
Maven配置文件

Maven有两个配置文件。配置文件里的设置，对所有的pom文件都是有效的。比如，你可以配置：

本地仓库的路径；
当前的编译配置选项
等等
配置文件名为settings.xml，两个配置文件分别为：

+ Maven安装目录中：$M2_HOME/conf/settings.xml
+ 用户主目录中：${user.home}/.m2/settings.xml

两个配置文件都是可选的。如果两个文件都存在，则用户目录下的配置会覆盖Maven安装目录中的配置。

##### Maven运行
[maven常用命令行及解释](http://blog.csdn.net/phantomes/article/details/8110779)

### 2. How(Maven具体使用)
##### Maven安装目录
四个文件夹：
bin：包含mvn运行的脚本,m2.conf配置文件
boot：boot目录包含一个类加载器的框架
conf：是配置文件目录
lib：类库

##### Maven目录结构
Maven有一个标准的目录结构。如果你在项目中遵循Maven的目录结构，就无需在pom文件中指定源代码、测试代码等目录。
以下为最重要的目录：
```xml
- src
  - main
    - java
    - resources
    - webapp
  - test
    - java
    - resources

- target
```
[Maven标准目录结构介绍](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)
src目录是源代码和测试代码的根目录。main目录是应用的源代码目录。test目录是测试代码的目录。main和test下的java目录，分别表示应用的java源代码和测试代码。

resources目录包含项目的资源文件，比如应用的国际化配置的属性文件等。

如果是一个web项目，则webapp目录为web项目的根目录，其中包含如WEB-INF等子目录。

target目录是由Maven创建的，其中包含编译后的类文件、jar文件等。当执行maven的clean目标后，target目录会被清空。

##### POM文件
[《Maven官方文档》POM文件](http://ifeve.com/maven-pom/)
[Maven之（七）pom.xml配置文件详解](http://blog.csdn.net/u012152619/article/details/51485297)

##### 外部依赖
Maven的外部依赖指的是不在Maven的仓库（包括本地仓库、中央仓库和远程仓库）中的依赖（jar包）。它可能位于你本地硬盘的某个地方，比如web应用的lib目录下。这里的“外部”是对Maven仓库系统而言的，不仅仅是对项目而言的。大部分的外部依赖都是针对项目的，很少的外部依赖是针对仓库系统的（即不在仓库中）。

配置外部依赖的示例如下：
<pre><code>
&lt;dependency&gt;
  &lt;groupId&gt;mydependency&lt;/groupId&gt;
  &lt;artifactId&gt;mydependency&lt;/artifactId&gt;
  &lt;scope&gt;system&lt;/scope&gt;
  &lt;version&gt;1.0&lt;/version&gt;
  &lt;systemPath&gt;${basedir}\war\WEB-INF\lib\mydependency.jar&lt;/systemPath&gt;
&lt;/dependency&gt;
 </code></pre>
groupId和artifactId为依赖的名称，即API的名称。scope属性为system。systemPath属性为jar文件的路径。${basedir}为pom文件所在的目录，路径中的其它部分是相对于该目录而言的。

##### 快照依赖
快照依赖指的是那些还在开发中的依赖（jar包）。与其经常地更新版本号来获取最新版本，不如你直接依赖项目的快照版本。快照版本的每一个build版本都会被下载到本地仓库，即使该快照版本已经在本地仓库了。总是下载快照依赖可以确保本地仓库中的每一个build版本都是最新的。

在pom文件的最开头（设置groupId和artifactId的地方），在版本号后追加-SNAPSHOT，则告诉Maven你的项目是一个快照版本。如：
<pre><code>&lt;version&gt;1.0-SNAPSHOT&lt;/version&gt;    </code></pre>
可以看到加到版本号后的-SNAPSHOT。

在配置依赖时，在版本号后追加-SNAPSHOT表明依赖的是一个快照版本。如：
<pre><code>&lt;dependency&gt;
    &lt;groupId&gt;com.jenkov&lt;/groupId&gt;
    &lt;artifactId&gt;java-web-crawler&lt;/artifactId&gt;
    &lt;version&gt;1.0-SNAPSHOT&lt;/version&gt;
&lt;/dependency&gt;</code></pre>
追加在version后的-SNAPSHOT告诉Maven这是一个快照版本。

可以在Maven配置文件中设置快照版本下载的频率。

##### Maven仓库
Maven仓库就是存储jar包和一些元数据信息的目录。其中的元数据即pom文件，描述了该jar包属于哪个项目，以及jar包所需的外部依赖。该元数据信息使得Maven可以递归地下载所有的依赖，直到整个依赖树都下载完毕并放到你的本地仓库中。
Maven有三种类型的仓库：

本地仓库
中央仓库
远程仓库
Maven根据以上的顺序去仓库中搜索依赖。首先是本地仓库，然后是中央仓库，最后，如果pom文件中配置了远程仓库，则会去远程仓库中查找。

下图说明了三种仓库的类型以及位置：
![](http://ifeve.com/wp-content/uploads/2014/06/maven-repo-types-loc-300x185.png)
本地仓库

本地仓库就是开发者电脑上的一个目录。该仓库包含了Maven下载的所有依赖。一般来讲，一个本地仓库为多个不同的项目服务。因此，Maven只需下载一次，即使有多个项目都依赖它（如junit）。

通过mvn install命令可以将你自己的项目构建并安装到本地仓库中。这样，你的其它项目就可以通过在pom文件将该jar包作为外部依赖来使用。

Maven的本地仓库默认在你本机的用户目录下。不过，你可以在Maven的配置文件中修改本地仓库的路径。Maven的配置文件也在用户目录下(user-home/.m2)，文件名为settings.xml。以下示例为本地仓库指定其它的路径：
<pre><code>&lt;settings&gt;
    &lt;localRepository&gt;
        d:\data\java\products\maven\repository
    &lt;/localRepository&gt;
&lt;/settings&gt;</code></pre>
##### 中央仓库

Maven的中央仓库由Maven社区提供。默认情况下，所有不在本地仓库中的依赖都会去这个中央仓库查找。然后Maven会将这些依赖下载到你的本地仓库。访问中央仓库不需要做额外的配置。

##### 远程仓库

远程仓库是位于web服务器上的一个仓库，Maven可以从该仓库下载依赖，就像从中央仓库下载依赖一样。远程仓库可以位于Internet上的任何地方，也可以是位于本地网络中。

远程仓库一般用于放置组织内部的项目，该项目由多个项目共享。比如，由多个内部项目共用的安全项目。该安全项目不能被外部访问，因此不能放在公开的中央仓库下，而应该放到内部的远程仓库中。

远程仓库中的依赖也会被Maven下载到本地仓库中。

可以在pom文件里配置远程仓库。将以下的xml片段放到属性之后：
<pre><code>&lt;repositories&gt;
   &lt;repository&gt;
       &lt;id&gt;jenkov.code&lt;/id&gt;
       &lt;url&gt;http://maven.jenkov.com/maven2/lib&lt;/url&gt;
   &lt;/repository&gt;
&lt;/repositories&gt;</code></pre>

##### 生命周期
一个完整的项目构建过程通常包括清理、编译、测试、打包、集成测试、验证、部署等步骤，Maven从中抽取了一套完善的、易扩展的生命周期。Maven的生命周期是抽象的，其中的具体任务都交由插件来完成。Maven为大多数构建任务编写并绑定了默认的插件，如针对编译的插件：maven-compiler-plugin。用户也可自行配置或编写插件。
**1. 三套生命周期**
Maven定义了三套生命周期：clean、default、site，每个生命周期都包含了一些阶段（phase）。三套生命周期相互独立，但各个生命周期中的phase却是有顺序的，且后面的phase依赖于前面的phase。执行某个phase时，其前面的phase会依顺序执行，但不会触发另外两套生命周期中的任何phase。
**1.1 clean生命周期**
1.	pre-clean    ：执行清理前的工作；
2.	clean    ：清理上一次构建生成的所有文件；
3.	post-clean    ：执行清理后的工作

**1.2 default生命周期**
default生命周期是最核心的，它包含了构建项目时真正需要执行的所有步骤。
<ol>
<li>validate</li>
<li>initialize</li>
<li>generate-sources</li>
<li>process-sources</li>
<li>generate-resources</li>
<li>process-resources &nbsp; &nbsp;：复制和处理资源文件到target目录，准备打包；</li>
<li>compile &nbsp; &nbsp;：编译项目的源代码；</li>
<li>process-classes</li>
<li>generate-test-sources</li>
<li>process-test-sources</li>
<li>generate-test-resources</li>
<li>process-test-resources</li>
<li>test-compile &nbsp; &nbsp;：编译测试源代码；</li>
<li>process-test-classes</li>
<li>test &nbsp; &nbsp;：运行测试代码；</li>
<li>prepare-package</li>
<li>package &nbsp; &nbsp;：打包成jar或者war或者其他格式的分发包；</li>
<li>pre-integration-test</li>
<li>integration-test</li>
<li>post-integration-test</li>
<li>verify</li>
<li>install &nbsp; &nbsp;：将打好的包安装到本地仓库，供其他项目使用；</li>
<li>deploy &nbsp; &nbsp;：将打好的包安装到远程仓库，供其他项目使用；</li>
</ol>
**1.3 site生命周期**
<ol>
<li>pre-site</li>
<li>site &nbsp; &nbsp;：生成项目的站点文档；</li>
<li>post-site</li>
<li>site-deploy &nbsp; &nbsp;：发布生成的站点文档</li>
</ol>

##### 插件
Maven的核心文件很小，主要的任务都是由插件来完成。定位到：%本地仓库%\org\apache\maven\plugins，可以看到一些下载好的插件：
![](http://images.cnitblog.com/i/293735/201407/012038215279940.png)
具体参考的这篇文章，写作风格真棒，很漂亮的排版。
[Maven入门指南⑦：Maven的生命周期和插件](http://www.cnblogs.com/luotaoyeah/p/3819001.html)

##### maven 命令

> 编译源代码：mvn compile
> 编译测试代码：mvn test-compile
> 运行测试：mvn test (会运行前面的三个命令)
> 打包：mvn package
> 部署到本地：mvn install 
> 在远程部署jar：mvn deploy
> 清除产生的项目：mvn clean
> 窥探SuperPom：mvn help:effective-pom
> maven依赖树：mvn dependency:tree
> 打包的时候过滤test：mvn clean package -Pdev -Dmaven.test.skip=true

##### maven的生命周期
- 构建生命周期
	- clean
		- Pre-clean 做移除准备
		- Clean 移除jar
		- Post-clean 移除target
	- default（项目构建真正需要执行的所有步骤：23个阶段，执行某个执行阶段前的阶段都会被执行，避免重复工作）		
		- pre-resource
		- compile
		- test-compile
		- test
		- package
		- install
		- deploy 
	- site （项目所有的资源文件放在一个站点服务器上，用户可以通过站点访问我们的api文档等）
		- Pre-site
		- Site
		- Post-site
		- Site-deploy
	
##### shapshot和release版本
snapshot 快照版会经常更新，maven会实时下载
release 稳定版
防止版本号的滥用，以及及时的协作更新

##### 依赖
type:
scope:
optional:
exclusions:
依赖特性：依赖传递、
依赖冲突：路径优先、声明优先
依赖排除：dependencymanagement、execlusion

##### maven的聚合与继承
聚合：使用一个命令同时构建多个模块
< modules > < package >pom< >
继承： 子模块可以使用父模块的依赖

##### maven测试
快捷键？？？



---

### 3. WHY（为什么要使用maven）
首先，为什么有maven？构建是程序员每天要做的工作，而且相当长的时间花在了这上面，而maven使这系列的工作完全自动化。 我们一直在寻找避免重复的方法，设计的重复，文档的重复，编码的重复，构建的重复等，maven是跨平台的，最大的消除了构建的重复。

maven的其他优势：
1.  maven不仅是构建工具，它还是依赖管理工具和项目管理工具，提供了中央仓库，能够帮我们自动下载构件。
2.  为了解决的依赖的增多，版本不一致，版本冲突，依赖臃肿等问题，它通过一个坐标系统来精确地定位每一个构件（artifact）。
3.  还能帮助我们分散在各个角落的项目信息，包括项目描述，开发者列表，版本控制系统，许可证，缺陷管理系统地址。
4.  maven还为全世界的java开发者提供了一个免费的中央仓库，在其中几乎可以找到任何的流行开源软件。通过衍生工具(Nexus),我们还能对其进行快速搜索
5.  maven对于目录结构有要求，约定优于配置，用户在项目间切换就省去了学习成本。

Maven提供了开发人员构建一个完整的生命周期框架。开发团队可以自动完成项目的基础工具建设，Maven使用标准的目录结构和默认构建生命周期。

在多个开发团队环境时，Maven可以设置按标准在非常短的时间里完成配置工作。由于大部分项目的设置都很简单，并且可重复使用，Maven让开发人员的工作更轻松，同时创建报表，检查，构建和测试自动化设置。

> **参考文献：**
> http://www.cnblogs.com/luotaoyeah/p/3819001.html Maven入门指南⑦：Maven的生命周期和插件
> http://ifeve.com/maven-1/ Maven入门指南（一）
> http://ifeve.com/maven-2/ Maven入门指南（二）
> http://blog.csdn.net/nancy_feng/article/details/38148625 maven学习-----maven的优势






