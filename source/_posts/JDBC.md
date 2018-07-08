---
toc: true
title: JDBC
date: 2018-03-29 15:36:01
tags: [JDBC,事务]
---


## 1. 什么是JDBC？

1996年，Sun公司发布了第一版的Java数据库连接（JDBC）API，使编程人员可以通过这个API接口连接到数据库，并使用结构化查询语言（SQL）完成对数据库的查找与更新。JDBC自此称为Java类库中最常用的API之一。
<!--more-->
概念：
- SQL：结构化查询语言，是一种特定的目的程序语言，用于管理关系数据库管理系统，或在关系流数据管理系统（RDSMS）中进行流处理。
- MySQL：原本是一个开放源代码的关系数据库管理系统，原开发者为瑞典的MySQL AB公司，该公司于2008年被昇阳微系统（Sun Microsystems）收购。2009年，甲骨文公司（Oracle）收购昇阳微系统公司，MySQL成为Oracle旗下产品。
- JDBC：Java Database Connectivity (JDBC) is an application programming interface (API) for the programming language Java, which defines how a client may access a database. It is Java based data access technology and used for Java database connectivity. It is part of the Java Standard Edition platform, from Oracle Corporation. It provides methods to query and update data in a database, and is oriented towards relational databases. A JDBC-to-ODBC bridge enables connections to any ODBC-accessible data source in the Java virtual machine (JVM) host environment.

## 2. JDBC配置
你需要有一个可获得其JDBC驱动程序的数据库软件。目前这方面有很多出色的软件可供选择，比如IBM DB2、Microsoft SQL Server、MySQL、Oracle和PostgreSQL。在这一节，我会以MySQL为例演示，在下一篇博客，我会介绍H2缓存数据库。

现在，假设你已经安装好了MySQL，有一个名称为`jifang`的`database`，并且有权限对这个数据库增删改查。

### 2.1 数据库URL
在连接数据库时，我们必须使用各种与数据库类型相关的参数，例如主机名、端口号和数据库名。

JDBC URL一般格式：
`jdbc:subprotocol:other stuff`
其中，`subprotocol`（子协议名）用于选择连接到数据库的具体驱动程序。`other stuff`（数据源名）参数的格式随所使用的`subprotocol`不同而不同。如果要了解具体格式，你需要查阅数据库供应商的相关文档。

#### 几种常见的数据库连接

**1. Oracle **
驱动：`oracle.jdbc.driver.OracleDriver` 
URL：`jdbc:oracle:thin:@host:port:dbname`
host：数据库所在的机器的地址； 
port：端口号，默认是1521

**2. MySQL**
驱动：`com.mysql.jdbc.Driver `
URL：`jdbc:mysql://host:port/dbname `
host：数据库所在的机器的名称； 
port：端口号，默认3306

**3. SQL Server**
驱动：`com.microsoft.jdbc.sqlserver.SQLServerDriver` 
URL：`jdbc:microsoft:sqlserver://<host><:port>;DatabaseName=<dbname> `
host：数据库所在的机器的名称； 
port：端口号，默认是1433

**4. DB2**
驱动：`com.ibm.db2.jdbc.app.DB2Driver `
URL：`jdbc:db2://<host><:port>/dbname` 
host：数据库所在的机器的名称； 
port：端口号，默认是5000

#### MySQL URL格式详解

这里我们主要参考MySQL的URL格式
`jdbc:mysql://[host][,failoverhost...][:port]/[database][?propertyName1][=propertyValue1][&propertyName2][=propertyValue2]...`

![Alt text](https://app.yinxiang.com/shard/s15/res/7df5d637-a933-43e8-98a6-e539d3b08fba/1522287325072.png)

### 2.2 驱动程序
你需要获得包含了你所使用的数据库的驱动程序的JAR文件。
获取方法：
1. 去官网[https://dev.mysql.com/downloads/connector/j/](https://dev.mysql.com/downloads/connector/j/)载JDBC Driver
解压之后获取`mysql-connector-java-5.1.46-bin.jar`这个jar包
2. 使用Maven仓库下载 
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.6</version>
</dependency>
```
如果使用第二种方式，直接就会放在项目的`classpath`下。如果是第一种，下载好了之后如何安装呢? 把下载 的文件解压后的 `mysql-connector-java-5.1.46-bin.jar`文件 copy到 %JAVA_HOME%/jre/lib/ext 下， %JAVA_HOME就是jdk的安装目录。
### 2.3 注册驱动器类
某些JDBC的JAR文件将自动注册驱动器类，在这种情况下，可以跳过这一部分所描述的手动注册步骤。

如果驱动程序JAR文件不支持自动注册，那就需要找出数据库提供商使用的JDBC驱动器类的名字。例如MySQL：
`com.mysql.jdbc.Driver`

通过使用`DriverManager`，可以使用两种方式来注册驱动器。一种方式是在Java程序中加载加载驱动器类，例如：
`Class.forName("com.mysql.jdbc.Driver");`
这条语句将使得驱动器类被加载，由此将执行可以注册驱动器的静态初始化器。

另一种方式是设置jdbc.drivers属性。可以用命令行参数来指定这个属性，例如：
`java -Djdbc.drivers=com.mysql.jdbc.Driver ProgramName`
或者在应用中用下面这样的调用来设置系统属性：
`System.setProperty("jdbc.Driver", "com.mysql.jdbc.Driver");`

### 2.4 连接到数据库
在Java程序中，我们可以在代码中打开一个数据库连接，例如：
```java
String url = properties.getProperty("jdbc.url");
String username = properties.getProperty("jdbc.username");
String password = properties.getProperty("jdbc.password");
Connection conn = DriverManager.getConnection(url,username,password);
```
驱动管理器遍历所有注册过的驱动程序，以便找到一个能够使用数据库URL中指定的子协议的驱动程序。

### 2.5 执行SQL语句

1. 执行SQL命令之前，首先需要创建一个`Statement`对象，
`Statement stat = conn.createStatement();`
2. SQL语句字符串
`String sql = "CREATE TABLE Greetings (Message CHAR(20))"`
3. 然后调用Statement接口中的`executeUpdate`方法
`stat.executeUpdate(sql)`
对于`INSERT DELETE UPDATE`等都使用`executeUpdate`方法，它返回受SQL命令影响的行数。对于`SELECT`使用`executeQuery`方法，它返回一个`ResultSet`类型的对象，可以通过它来每一行地迭代遍历所有查询结果。

**注意：**`ResultSet`接口的迭代协议与java.util.Iterator接口稍有不同。对于`ResultSet`接口，迭代器初始化时被设定在第一行之前的位置，必须调用`next()`方法把它移动到第一行。另外，它没有`hasNext()`方法，我们需要不断地调用`next()`，知道它返回`false`。

`ResultSet`中行的顺序是人任意的，除非你在`sql`使用了`ORDER BY`子句指定排序。

#### **管理连接、语句和结果集** 
每个`Connection`对象都可以创建一个或多个`Statement`对象。同一个`Statement`对象可以用于多个不想干的命令和查阅。但是一个`Statement`对象最多只能有一个打开结果集。
如果不你信，可以看下面代码：
```java
        ResultSet resultSet = statement.executeQuery("SELECT * FROM Greetings");
        ResultSet resultSet2 = statement.executeQuery("SELECT * FROM Greetings");
        if (resultSet2.next()){
            System.out.println(resultSet2.getString(1));
        }
        if (resultSet.next()){
            System.out.println(resultSet.getString(1));
        }
```
结果是：
```bash
Hello World!sdfsdf
Exception in thread "main" java.sql.SQLException: Operation not allowed after ResultSet closed
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:1055)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:956)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:926)
	at com.mysql.jdbc.ResultSetImpl.checkClosed(ResultSetImpl.java:768)
	at com.mysql.jdbc.ResultSetImpl.next(ResultSetImpl.java:7008)
	at jdbc.MySQLTest.runTest(MySQLTest.java:31)
	at jdbc.MySQLTest.main(MySQLTest.java:17)

Process finished with exit code 1
```
也就是说，同一个`Statement`对象会以最后一个打开的`ResultSet`有效，之前打开的都已经关闭了。

**使用完ResultSet、Statement或Connection对象后，应立即调用close方法。这些对象都是用了规模较大的数据结构和数据库存服务器上的有限资源**
如果Statement对象上有一个打开的结果集，那么调用`statement.close()`方法将自动关闭该结果集。同样的调用Connection类的close方法将关闭该连接上的所有语句。
下面是完整的代码：
```java
package jdbc;

import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.sql.*;
import java.util.Properties;

/**
 * author: jifang
 * date: 18-3-29 上午9:42
 */

public class MySQLTest {
    public static void main(String[] args) throws SQLException {
        runTest();
    }

    public static void runTest() throws SQLException {
        Connection connection = getConnection();
        Statement statement = connection.createStatement();
        statement.executeUpdate("CREATE TABLE Greetings (Message CHAR(20))");
        statement.executeUpdate("INSERT INTO Greetings VALUES ('Hello World!sdfsdf')");

        ResultSet resultSet = statement.executeQuery("SELECT * FROM Greetings");
        if (resultSet.next()){
            System.out.println(resultSet.getString(1));
        }

        statement.executeUpdate("DROP TABLE Greetings");
    }

    public static Connection getConnection() throws SQLException {
        Properties properties = new Properties();
        try {
            InputStream inputStream = Files.newInputStream(Paths.get("src/main/java/jdbc/", "database.properties"));
            properties.load(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        String drivers = properties.getProperty("jdbc.drivers");
        if (drivers!=null)System.setProperty("jdbc.drivers",drivers);
        String url = properties.getProperty("jdbc.url");
        String username = properties.getProperty("jdbc.username");
        String password = properties.getProperty("jdbc.password");

        return DriverManager.getConnection(url,username,password);
    }
}
```
【data.properties】
```xml
jdbc.drivers=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/jifang
jdbc.username=root
jdbc.password=xjj520520ljf
```

我们已经了解了怎么使用`Statement`执行sql语句。现在来看看它的大哥。

**PreparedStatement（预备语句）**
在预备查询语句中，每个宿主变量都用“？”来表示。如果存在一个以上的变量，那么在设置变量值时必须注意“？”的位置。例如我们要查询的SQL为：
`SELECT * FROM jifang where id = ?`
在执行预备语句之前，必须使用set方法将变量绑定到实际的值上。
`preparedStat.setInt(1, 233)`
第一个参数指的是需要设置的宿主变量的位置，位置1表示第一个“？”。第二个参数是赋予宿主变量的值。

如果想要重用已经执行过的预备查询语句，那么除非使用set方法或调用`clearParameters`方法，否则所有宿主变量的绑定都不会改变。
一旦为所有变量绑定了具体的值，就可以执行查询操作了。

**注意：**如果查询中涉及用户输入，那就还需要警惕**注入攻击**。因此，只有查询涉及变量时，才应该使用预备语句。

### 后续
除了以上介绍的内容，还有：
1. 读写LOB
2. 获取自动生成键
3. 可滚动和可更新结果集
4. 元数据
等内容，有兴趣可以查阅《Java核心编程卷二》





> 参考文献
> [1] 《Java核心技术卷二》 
> [2] [几种常见的数据库连接的URL写法](https://blog.csdn.net/u014726937/article/details/52786502)
> [3] [Mysql JDBC Url参数说明](http://elf8848.iteye.com/blog/1684414)



