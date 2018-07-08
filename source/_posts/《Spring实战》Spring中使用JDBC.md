---
toc: true
title: 《Spring实战》Spring中使用JDBC
date: 2018-04-11 09:45:43
tags: [JDBC,DataSource]
---

为了避免持久化的逻辑分散到应用的各个组件中，最好将数据访问的功能放到一个或多个专注于此项任务的组件中。这样的组件通常称为数据访问对象（Data acess  object, DAO）或 Repository。
<!--more-->
### 1. 配置数据源
Spring上下文中配置数据源bean提供了多种方式：
- 通过JDBC驱动程序定义的数据源
- 通过JNDI查找数据源配置
- 连接池的数据源

这里介绍第一和第三种方式。
#### 通过JDBC驱动程序定义的数据源
在Spring中,通过JDBC驱动定义数据源是最简单的配置方式。Spring提供了三个这样的数据源类(均位于org.springframework.jdbc.datasource包中)供选择:
- DriverManagerDataSource:在每个连接请求时都会返回一个新建的连接。与DBCP的BasicDataSource不同,由DriverManagerDataSource提供的连接并没有进行池化管理;
- SimpleDriverDataSource:与DriverManagerDataSource的工作方式类似,但是它直接使用JDBC驱动,来解决在特定环境下的类加载问题,这样的环境包括OSGi容器;
- SingleConnectionDataSource:在每个连接请求时都会返回同一个的连接。尽管SingleConnectionDataSource不是严格意义上的连接池数据源,但是你可以将其视为只有一个连接的池。
这些数据源的配置与DBCP的配置类似。
```java
    /**
     * 基于JDBC驱动的数据源
     * @return
     */
    @Bean
    public DataSource dataSource(){
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:~/test");
        dataSource.setUsername("sa");
        dataSource.setPassword("");;
        return dataSource;
    }
```
与具备池功能的数据源相比,唯一的区别在于这些数据源bean都没有提供连接池功能,所以没有可配置的池相关的属性。


#### 使用数据源连接池
常用的主流开源数据库连接池有C3P0、DBCP、Tomcat Jdbc Pool、BoneCP、Druid等

**C3p0**: 开源的JDBC连接池，实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有Hibernate、Spring等。单线程，性能较差，适用于小型系统，代码600KB左右。

**DBCP** (Database Connection Pool):由Apache开发的一个Java数据库连接池项目， Jakarta commons-pool对象池机制，Tomcat使用的连接池组件就是DBCP。单独使用dbcp需要3个包：common-dbcp.jar,common-pool.jar,common-collections.jar，预先将数据库连接放在内存中，应用程序需要建立数据库连接时直接到连接池中申请一个就行，用完再放回。单线程，并发量低，性能不好，适用于小型系统。

**Druid**：Druid是Java语言中最好的数据库连接池，Druid能够提供强大的监控和扩展功能，是一个可用于大数据实时查询和分析的高容错、高性能的开源分布式系统，尤其是当发生代码部署、机器故障以及其他产品系统遇到宕机等情况时，Druid仍能够保持100%正常运行。主要特色：为分析监控设计；快速的交互式查询；高可用；可扩展；Druid是一个开源项目，源码托管在github上。

以DBCP连接池为例：
Java配置，连接池形式的DataSource Bean可以声明如下：
```java
    /**
     * Druid数据库连接池
     *
     * @return
     */
    @Bean
    public DruidDataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        /*dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:~/test");
        dataSource.setUsername("sa");
        dataSource.setPassword("");*/
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/spittr");
        dataSource.setUsername("root");
        dataSource.setPassword("");
        dataSource.setInitialSize(5);
        dataSource.setMaxActive(10);
        return dataSource;
    }
```

前四个属性是配置BasicDataSource所必需的。属性driverClassName指定了JDBC驱
动类的全限定类名，需要提前将对应数据库的驱动依赖jar包放置在classpath下。
注释配置的是H2内存数据库，显式配置的是MySQL数据库。

#### 使用嵌入式的数据源
**嵌入式数据库（embedded database）**：每次重启应用或运行测试的时候，都能够重新填充测试数据。多用于开发和测试环境，它会预先加载一组测试数据。

**使用jdbc命名空间配置嵌入式数据库**
```xml
<jdbc:embedded-database id="dataSource" type="H2">
	<jdbc:script localtion="classpath:xxx.sql"/>
	<jdbc:script localtion="classpath:yyy.sql"/>
</jdbc:embedded-database>
```
在`<jdbc:embedded-database>`中,我们可以不配置也可以配置多个`<jdbc:script>`元素来搭建数据库。

**使用Java配置嵌入式数据库**
```java
    @Bean
    public DataSource dataSource3(){
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("")
                .addScript("")
                .build();
    }
```

#### 使用profile选择数据源
我们已经学习了多种配置数据源的方式，在正常的开发任务中，我们有不同的环境，也就有不同的数据源。

例如,对于开发期来说,`<jdbc:embedded-database>`元素是很合适的,而在QA环境中,你可能希望使用DBCP的BasicDataSource,在生产部署环境下,可能需要使用`<jee:jndi-lookup>`。

我们之前讨论的装配Bean profile特性恰好用在这儿。
```java
    /**
     * Druid数据库连接池
     *
     * @return
     */
    @Profile("prod")
    @Bean
    public DruidDataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        /*dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:~/test");
        dataSource.setUsername("sa");
        dataSource.setPassword("");*/
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/spittr");
        dataSource.setUsername("root");
        dataSource.setPassword("");
        dataSource.setInitialSize(5);
        dataSource.setMaxActive(10);
        return dataSource;
    }

    /**
     * 配置Jdbc模板类
     *
     * @param druidDataSource
     * @return
     */
    @Bean
    public JdbcTemplate jdbcTemplate(DruidDataSource druidDataSource) {
        return new JdbcTemplate(druidDataSource);
    }

    /**
     * 基于JDBC驱动的数据源
     * @return
     */
    @Profile("qa")
    @Bean
    public DataSource dataSource2(){
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:~/test");
        dataSource.setUsername("sa");
        dataSource.setPassword("");;
        return dataSource;
    }

    /**
     * 嵌入式数据库
     * @return
     */
    @Profile("dev")
    @Bean
    public DataSource dataSource3(){
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("")
                .addScript("")
                .build();
    }
```

### 2. 使用JDBC模板
Spring将数据访问过程中固定的和可变的部分明确划分为两个不同的类：模板(template)和回调(callback)。模板管理过程中固定的部分，而回调处理自定义的数据访问代码。

**繁琐的JDBC代码**
```java
    public static void fun() throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            //操作5（查看李四余额）
            sql = "select balance from account where name=?";
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, "ls");
            ResultSet rs = pstmt.executeQuery();
            rs.next();
            double balance = rs.getDouble(1);
            //如果李四余额为负数，那么回滚到指定保存点
            if(balance < 0) {
                System.out.println("张三，你上当了！");
            }
        } catch(Exception e) {
            //回滚事务
            if(con != null) {
                try {
                    con.rollback();
                } catch(SQLException ex) {}
            }
            throw new RuntimeException(e);
        } finally {
            //关闭
            if (!con.isClosed()){
                con.close();
            }
        }
    }
```
可以看到，我们每次都要进行connection创建，statement创建，执行语句，获取结果，一次关闭连接这些样板代码。

Spring将数据访问的样板代码抽象到模板类之中。Spring为JDBC提供了三个模板类供选择：
- **JdbcTemplate**:最基本的Spring JDBC模板,这个模板支持简单的JDBC数据库访问功能以及**基于索引参数**的查询;
- **NamedParameterJdbcTemplate**:使用该模板类执行查询时可以将值**以命名参数的形式**绑定到SQL中,而不是使用简单的索引参数;
- **SimpleJdbcTemplate**:该模板类利用Java 5的一些特性如自动装箱、泛型以及可变参数列表来简化JDBC模板的使用。

以前,在选择哪一个JDBC模板的时候,我们需要仔细权衡。但是从Spring 3.1开始,做这个决定变得容易多了。SimpleJdbcTemplate已经被废弃了,其Java 5的特性被转移到了JdbcTemplate中,并且只有在你需要使用命名参数的时候,才需要使用NamedParameterJdbcTemplate。这样的话,对于大多数的JDBC任务来说,JdbcTemplate就是最好的可选方案。

**配置JdbcTemplate**
```java
    @Bean
    public JdbcTemplate jdbcTemplate(DruidDataSource druidDataSource) {
        return new JdbcTemplate(druidDataSource);
    }
```

JdbcTemplate主要提供一下五类方法：
- execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句；
- update方法：用于执行新增、修改、删除等语句；
- batchUpdate方法用于执行批处理相关语句；
- query方法及queryForXXX方法：用于执行查询相关语句；
- call方法：用于执行存储过程、函数相关语句。

**RowMapper接口**
我们可以实现RowMapper接口的RowMapper()方法，这个方法会传入ResultSet和包含行号的整数。在实习类中创建自定义对象并填充对应的值。



> 参考文献
> [1] 《Spring实战》4th
> [2] [Spring-jdbc：JdbcTemplate使用简介](https://blog.csdn.net/u013468917/article/details/52217954)
> [3] [Spring之jdbcTemplate：查询的三种方式（单个值、单个对象、对象集合）](https://www.cnblogs.com/gongxr/p/8053010.html)

