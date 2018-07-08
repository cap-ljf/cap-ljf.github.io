---
toc: true
title: JSON
date: 2018-03-26 11:24:55
tags: [JSON]
---

## json基本概念
json的基本概念在官网已经解释的很清楚，我肯定没有官网讲的更好，请自行前往[Json官网](http://www.json.org/json-zh.html)查阅。
<!--more-->
## java怎么处理json
Java下常见的Json类库有Gson、JSO-lib和Jackson等，Jackson相对来说比较高效，在项目中主要使用Jackson进行JSON和Java对象转换。转换原理是序列化，序列化在之前已经写过。
### jackson
#### 概述
Jackson框架是基于Java平台的一套数据处理工具，被称为“最好的Java Json解析器”。
Jackson框架包含了3个核心库：core, annotations, databind。
Jackson版本： 1.x (目前版本从1.1~1.9)与2.x。1.x与2.x从包的命名上可以看出来，1.x的类库中，包命名以：org.codehaus.jackson.xxx开头，而2.x类库中包命令：com.fastxml.jackson.xxx开头
#### 准备工作
Jackson有1.x系列和2.x系列，2.x系列有3个jar包需要下载：
jackson-core-2.2.3.jar（核心jar包）
jackson-annotations-2.2.3.jar（该包提供Json注解支持）
jackson-databind-2.2.3.jar
一个maven依赖就够了
```xml
<dependency>
   <groupId>com.fasterxml.jackson.core</groupId>
   <artifactId>jackson-databind</artifactId>
   <version>2.5.3</version>
</dependency>
```
![Alt text](https://app.yinxiang.com/shard/s15/res/16accc9f-27b7-4be3-a4a4-b4be4e9fd9f5/1522028544170.png)

#### jackson处理json
Jackson提供了三种可选的Json处理方法：
1. Streaming API：效率最高，开销低、读写速度快，但编程复杂度高
2. Tree Model：最灵活
3. Data Binding：最常用
##### 1. DataBinding处理Json
**(1)java对象转换成Json**
```java
package json;

import java.util.Date;

/**
 * JSON序列化和反序列化使用的User类
 * author: jifang
 * date: 18-3-26 上午9:49
 */

public class User {
    private String name;
    private Integer age;
    private Date birthday;
    private String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", birthday=" + birthday +
                ", email='" + email + '\'' +
                '}';
    }
}
```
```java
package json;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**
 * author: jifang
 * date: 18-3-26 上午9:52
 */

public class JavaBeanSerializeToJson {
    public static void main(String[] args) throws IOException {
        User user = new User();
        user.setName("james");
        user.setAge(33);
        user.setEmail("james@nba.com");
        Date date = new Date();
        user.setBirthday(date);

        /**
         * ObjectMapper是JSON操作的核心，Jackson的所有JSON操作都是在ObjectMapper中实现。
         * ObjectMapper有多个JSON序列化的方法，可以把JSON字符串保存File、OutputStream等不同的介质中。
         * writeValue(File arg0, Object arg1)把arg1转成json序列，并保存到arg0文件中。
         * writeValue(OutputStream arg0, Object arg1)把arg1转成json序列，并保存到arg0输出流中。
         * writeValueAsBytes(Object arg0)把arg0转成json序列，并把结果输出成字节数组。
         * writeValueAsString(Object arg0)把arg0转成json序列，并把结果输出成字符串。
         */
        ObjectMapper mapper = new ObjectMapper();

        //java对象转JSON
        //输出结果：{"name":"james","age":33,"birthday":1522029481531,"email":"james@nba.com"}
        mapper.writeValue(new File("data1-1.json"),user);

        //Java集合转JSON
        //输出结果：[{"name":"james","age":33,"birthday":1522029481531,"email":"james@nba.com"}]
        List<User> users = new ArrayList<>();
        users.add(user);
        mapper.writeValue(new File("data1-2.json"),users);
    }
}

```
**(2)Json字符串反序列化为Java对象**
```java
package json;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;


import java.io.File;
import java.io.IOException;
import java.util.List;

/**
 * author: jifang
 * date: 18-3-26 上午10:00
 */

public class JsonDeserializeToJava {
    public static void main(String[] args) throws IOException {
        ObjectMapper mapper = new ObjectMapper();

        //Json转换成java对象
        //输出结果：User{name='james', age=33, birthday=Mon Mar 26 10:03:22 CST 2018, email='james@nba.com'}
        File json1 = new File("data1-1.json");
        //当反序列化json时，未知属性会引起的反序列化被打断，这里我们禁用未知属性打断反序列化功能，
        //因为，例如json里有10个属性，而我们的bean中只定义了2个属性，其它8个属性将被忽略
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        User user = mapper.readValue(json1, User.class);
        System.out.println(user);

        //Json转换成Java集合
        //输出结果：[User{name='james', age=33, birthday=Mon Mar 26 10:03:22 CST 2018, email='james@nba.com'}]
        File json2 = new File("data1-2.json");
        List<User> users = mapper.readValue(json2, new TypeReference<List<User>>(){});
        System.out.println(users);
    }
}

```
##### 2. Tree Model处理Json
**(1)java对象转换成Json**
这里的`User`类与上面的有所不同，把`email`类型改成`String[]`。
```java
package json;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.node.ArrayNode;
import com.fasterxml.jackson.databind.node.JsonNodeFactory;
import com.fasterxml.jackson.databind.node.ObjectNode;

import java.io.FileWriter;
import java.io.IOException;

/**
 * author: jifang
 * date: 18-3-26 上午9:11
 */

public class SerializationExampleTreeModel {
    public static void main(String[] args) throws IOException {
        //创建一个节点工厂,为我们提供所有节点
        JsonNodeFactory factory = new JsonNodeFactory(false);
        //创建一个json factory来写tree modle为json
        JsonFactory jsonFactory = new JsonFactory();
        //创建一个json生成器
        JsonGenerator generator = jsonFactory.createGenerator(new FileWriter("data2.json"));
        //注意，默认情况下对象映射器不会指定根节点，下面设根节点为country
        ObjectMapper mapper = new ObjectMapper();
        ObjectNode user = factory.objectNode();

        user.put("name","james");
        user.put("age",33);
        user.put("date", "2018-03-26");
        ArrayNode email = factory.arrayNode();
        email.add("james@nba.com").add("james@cav.com");
        user.set("email",email);
        mapper.configure(SerializationFeature.INDENT_OUTPUT,true);
        //输出结果：{"name":"james","age":33,"date":"2018-03-26","email":["james@nba.com","james@cav.com"]}
        mapper.writeTree(generator, user);
    }
}

```
**(2)Json字符串反序列化为Java对象**
```java
package json;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.File;
import java.io.IOException;
import java.util.Iterator;

/**
 * author: jifang
 * date: 18-3-26 上午10:54
 */

public class DeserializationExampleTreeModel1 {
    public static void main(String[] args) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        // Jackson提供一个树节点被称为"JsonNode",ObjectMapper提供方法来读json作为树的JsonNode根节点
        JsonNode node = mapper.readTree(new File("data2.json"));
        // 看看根节点的类型
        System.out.println("node JsonNodeType:" + node.getNodeType());
        // 是不是一个容器
        System.out.println("node is container Node ? " + node.isContainerNode());
        // 得到所有node节点的子节点名称
        System.out.println("---------得到所有node节点的子节点名称-------------------------");
        Iterator<String> fieldNames = node.fieldNames();
        while (fieldNames.hasNext()) {
            String fieldName = fieldNames.next();
            System.out.print(fieldName + " ");
        }
        System.out.println("\n-----------------------------------------------------");
        // asText的作用是有值返回值，无值返回空字符串
        JsonNode name = node.get("name");
        System.out.println("name: " + name.asText() + "\t JsonNodeType: " + name.getNodeType());
        JsonNode age = node.get("age");
        System.out.println("age: " + age.asText() + "\t JsonNodeType: " + age.getNodeType());
        JsonNode date = node.get("date");
        System.out.println("date: " + date.asText() + "\t JsonNodeType: " + date.getNodeType());
        JsonNode email = node.get("email");
        System.out.println("email: " + email + "\t JsonNodeType: " + email.getNodeType());
    }
}

```
再来看一下DeserializationExampleTreeModel2.java,本例中使用JsonNode.path的方法，path方法类似于DeserializationExampleTreeModel1.java中使用的get方法，

但当node不存在时,get方法返回null,而path返回MISSING类型的JsonNode
```java
package json;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.File;
import java.io.IOException;

/**
 * author: jifang
 * date: 18-3-26 上午11:07
 */

public class DeserializationExampleTreeModle2 {
    public static void main(String[] args) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        JsonNode node = mapper.readTree(new File("data2.json"));
        //path方法获取JsonNode时，当对象不存在时，返回MISSING类型的JsonNode
        JsonNode missingNode = node.path("test");
        if (missingNode.isMissingNode()){
            System.out.println("JsonNodeType: "+missingNode.getNodeType());
        }
        System.out.println("name: " + node.path("name").asText());
        JsonNode email = node.path("email");
        System.out.println("email: "+email);
    }
}

```
##### 3. Stream处理Json
**(1)java对象转换成Json**
```java
package json;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonGenerator;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

/**
 * author: jifang
 * date: 18-3-26 上午9:02
 */

public class StreamGeneratorJson {
    public static void main(String[] args) throws IOException {
        JsonFactory factory = new JsonFactory();
        JsonGenerator generator = factory.createGenerator(new FileWriter(new File("data3.json")));
        generator.writeStartObject();
        generator.writeFieldName("country_id");
        generator.writeString("China");
        generator.writeFieldName("provinces");
        generator.writeStartArray();
        generator.writeStartObject();
        generator.writeStringField("name", "Shanxi");
        generator.writeNumberField("population", 33750000);
        generator.writeEndObject();
        generator.writeEndArray();
        generator.writeEndObject();
        generator.close();
    }
}

```
**(2)Json字符串反序列化为Java对象**
```java
package json;

import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonToken;

import java.io.File;
import java.io.IOException;

/**
 * author: jifang
 * date: 18-3-26 上午8:51
 */

public class StreamParseJson {

    public static void main(String[] args) throws IOException {
        JsonFactory factory = new JsonFactory();
        JsonParser parser = factory.createParser(new File("data3.json"));
        while (!parser.isClosed()){
            JsonToken token = parser.nextToken();
            if (token ==null)break;
            if (JsonToken.FIELD_NAME.equals(token)&&"provinces".equals(parser.getCurrentName())){
                token = parser.nextToken();
                if (!JsonToken.START_ARRAY.equals(token))break;
                token = parser.nextToken();
                if (!JsonToken.START_OBJECT.equals(token)){
                    break;
                }
                while (true){
                    token = parser.nextToken();
                    if (token ==null){
                        break;
                    }
                    if (JsonToken.FIELD_NAME.equals(token) && "population".equals(parser.getCurrentName())){
                        token = parser.nextToken();
                        System.out.println(parser.getCurrentName()+":"+parser.getIntValue());
                    }
                }
            }
        }
    }
}

```
## 总结
引用csdn`博主(java_huashan)`的总结：
> 上面的例子中，分别用3种方式处理Json，我的体会大致如下：
> 
> Stream API方式是开销最低、效率最高，但编写代码复杂度也最高，在生成Json时，需要逐步编写符号和字段拼接json,在解析Json时，需要根据token指向也查找json值，生成和解析json都不是很方便，代码可读性也很低。
> Databinding处理Json是最常用的json处理方式，生成json时，创建相关的java对象，并根据json内容结构把java对象组装起来，最后调用writeValue方法即可生成json,
> 解析时，就更简单了，直接把json映射到相关的java对象，然后就可以遍历java对象来获取值了。
> TreeModel处理Json，是以树型结构来生成和解析json，生成json时，根据json内容结构，我们创建不同类型的节点对象，组装这些节点生成json。解析json时，它不需要绑定json到java bean，根据json结构，使用path或get方法轻松查找内容。

json就到这儿，如果大家对数据感兴趣的话还可以查看这篇文章[还在用JSON? Google Protocol Buffers 更快更小 (原理篇)](https://juejin.im/post/5ab08a5f6fb9a028e46e7770)
> 参考文献
> [1] [JackSon学习笔记(一)](https://blog.csdn.net/java_huashan/article/details/46375857)
> [2] [Java下利用Jackson进行JSON解析和序列化](https://www.cnblogs.com/winner-0715/p/6109225.html)
