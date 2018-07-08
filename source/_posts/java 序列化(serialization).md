---
toc: true
title: java 序列化(serialization)
date: 2018-03-21 14:22:47
tags: [transient,序列化]
---

当你需要存储相同类型的数据时，使用固定长度的记录格式是一个不错的选择。但是在面向对象程序中创建的对象很少全部都具有相同的类型。Java语言支持一种称为`对象序列化(object serialization)`的非常通用的机制，它可以将任何对象写出到流中，并在之后将其读回。
<!--more-->

## 1. 怎么序列化
对希望在对象流中存储或恢复的所有类都应该实现`Serialization`接口。
为了保存对象数据，首先需要打开一个`ObjectOutputStream`对象；
为了读回这些对象，首先需要获得一个`ObjectInputStream`对象。

【例】：
```
package stream;

import java.io.*;

/**
 * author: jifang
 * date: 18-3-21 上午10:12
 */

public class SerializationTest {
    static class Employee implements Serializable{
        public static final long serialVersionUID = -5088705208352347828L;

        String name;
        int age;
        Employee(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "Employee{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }

    static class Manager extends Employee{

        Manager(String name, int age) {
            super(name, age);
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
		// write()
        read();
    }

    public static void write() throws IOException {
        Employee harry = new Employee("harry hacker",19);
        Manager boss = new Manager("carl cracker",20);
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("employee.txt"));
        out.writeObject(harry);
        out.writeObject(boss);
    }

    public static void read() throws IOException, ClassNotFoundException {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("employee.txt"));
        Employee e1 = (Employee) in.readObject();
        Employee e2 = (Employee) in.readObject();
    }
}

```
为什么`Employee`要加上`serialization`字段呢？这个在`版本管理`会详细讲，在这里不加也行。
**注意：** *你只有在写对象时才能用writeObject/readObject方法，对于基本类型值，你需要使用诸如writeInt/readInt或writeDouble/readDouble这样的方法。*

在后台，是ObjectOutputStream在浏览对象的所有域，并存储它们的内容。
但是，由一种重要的情况需要考虑：当一个对象被多个对象共享，作为它们各自状态的一部分时，会发生什么呢？
答案是这个对象只被存储一次。让我们看看序列化机制： 
（每个对象都是用一个序列号保存的）
- 当你遇到的每一个对象引用都关联一个序列号
- 对于每个对象，当第一次遇到时，保存其对象数据到流中
- 如果某个对象之前已经保存过了，那么只需要表明它之前的序列号
- 对于流中的数据，在第一次遇到其序列号时，构建它，并使用流中数据初始化它，然后记录这个序列号和新对象之间的关联
- 当遇到与之前保存过的相同序列号的对象时，获取这个序列号相关的对象引用

## 2. 对象序列化的文件格式
这里涉及到枯燥的规范，类似于计算机网络中tcp等报文头格式。（有兴趣可以看《java核心技术 卷二 》P34）
在这里提一下：当存储一个对象时，其对应的类也必须存储。而在存储这个类信息中有`序列化的版本唯一的ID,它是数据域类型和方法签名的指纹`。
**指纹**是通过对类、超类、接口、域类型和方法签名按照规范方式排序，然后将安全散列算法（SHA）应用与这些数据而获得。

	SHA是一种可以为较大的信息块提供指纹的快速算法，不论最初的数据块尺寸有多大，这种指纹总是20个字节的数据包。它是通过在数据上执行一个灵巧的位操作序列而创建的，这个序列在本质上可以保证无论这些数据以何种方式发生变化，其指纹也都会跟着变化。但是，序列化机制只使用了SHA码的前8个字节作为类的指纹。即便这样，当类的数据域或方法发生变化时，其指纹跟着变化的可能性还是非常大。

对象流拒绝读入具有不同指纹的对象。

## 3. transient
当某些数据域不可以或不想被序列化时，可以将他们标记成`transient`，瞬时的域在对象被序列化时总是被跳过的。
**transient使用小结**
1. 一旦变量被`transient`标记，那么变量将不会被序列化，在之后回复时无法访问。
2. `transient`只能修饰变量，而不能修饰方法和类。注意，局部变量不可以被`transient`修饰。
3. 静态变量不管是否被`transient`修饰，都不会被序列化。
【示例】
```java
package stream;

import java.io.*;

/**
 * author: jifang
 * date: 18-3-21 下午1:23
 */

public class TransientTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("user.dat"));
//        User user = new User();
//        user.setVar3(3);
//        user.setVar2(2);
//        User.setVar1(1);
//        objectOutputStream.writeObject(user);
//        objectOutputStream.flush();
//        objectOutputStream.close();
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("user.dat"));
        User user2 = (User) objectInputStream.readObject();
        System.out.println(user2.getVar3()+" "+user2.getVar2());
    }
}

import java.io.Serializable;

public class User implements Serializable{
    private static final long serialVersionUID = 8294180014912103005L;

    public static int var1;
    public  int var2;
    public int var3;

    public static int getVar1() {
        return var1;
    }

    public static void setVar1(int var1) {
        User.var1 = var1;
    }

    public int getVar2() {
        return var2;
    }

    public void setVar2(int var2) {
        this.var2 = var2;
    }

    public int getVar3() {
        return var3;
    }

    public void setVar3(int var3) {
        this.var3 = var3;
    }
}

```
先执行注释掉部分，再执行反序列化，得到结果。
【输出结果】
```java
var1: 0
var2: 2
var3: 3
```
`var1`反序列化之后为0，很明显，被赋予了默认值。所以这个`static`静态变量没有被序列化。

## 4. 版本管理
如果使用序列化来保存对象，就需要考虑在程序演化时会有什么问题。这就涉及到类的不同版本。
而当类改变时，相应的指纹也会发生变化，我们上面已经提到，**对象流拒绝读入具有不同指纹的对象。**
例如上面的示例代码，如果我把`User`类中的`serialVersionUID`注释掉之后再执行，就会出现下面这个异常。
```java
Exception in thread "main" java.io.InvalidClassException: stream.User; local class incompatible: stream classdesc serialVersionUID = 8294180014912103005, local class serialVersionUID = -773535447984752566
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:687)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1880)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1746)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2037)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1568)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:428)
	at stream.TransientTest.main(TransientTest.java:21)
```
如果一个类具有名为`serialVersionUID`的静态数据成员，它就不需要再人工地计算其指纹，而只需直接使用这个值。一旦这个静态数据成员被置于某个类的内部，那么序列化系统就可以读入这个类的对象的不同版本。
如果这个类只有方法产生了变化，那么在读入新对象数据时是不会有任何问题的。但是如果数据域产生了变化，那么久可能会有问题。例如，旧对象可能比新对象拥有更多或更少的数据域，或者相同名字的数据域的类型不同。那么，对象流在转换时就只能尽力转换成这个类的当前版本。
它是怎么处理的呢：
1. 对象流会将这个类当前版本的数据域域流中版本的数据域进行比较，当然，对象流只考虑非`transient`和非`static`数据域。
2. 如果这两部分数据域之间名字匹配而类型不匹配，那么对象流不会尝试将一种类型转换成另一种类型。
3. 如果流中的对象具有当前版本所没有的数据域，那么对象流就忽略这些额外的数据。
4. 如果当前版本具有在流化对象中所没有的数据域，那么这些新添加的域将被设置成它们的默认值（object: null, number: 0, boolean: false）

书中这一章结尾提到了使用序列化来实现clone，但是效率却比实现clonable差很多。好了，java序列化就到这儿，下次见。

> 参考文献
> [1] [Java transient关键字使用小记](https://www.cnblogs.com/lanxuezaipiao/p/3369962.html)
>  [2] 《Java核心技术 卷二》




