---
layout: post
title: "Java类加载机制"
date: 2019-08-17 10:51:51
categories: Java
---

类加载机制

## 0.java内存模型

![jvm内存模型](https://niubility.org.cn/assets/images/jvm20190819.png)

## 1.什么是类的加载？

Java虚拟机把描述类的数据从class文件中加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的的Java类型，这就是虚拟机的加载机制。class文件由类的装载器装载后，在JVM中将形成一份描述class结构的元信息对象，通过该元信息对象可以获取class的结构信息：如构造函数，属性和方法等，Java允许用户借由这个class相关的元信息对象间接调用class对象的功能。

## 2.类的加载过程

![java类加载过程](https://niubility.org.cn/assets/images/jvm201908191.png)

类的加载过程包括加载、验证、准备、解析、初始化五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定（也称为动态绑定或晚期绑定）。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是相互交叉地混合进行的，通常在一个阶段执行的过程调用或激活另一个阶段。

### 2.1 加载 - 查找和加载class文件

查找并加载类的二进制数据(```.class```)是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：

- 0.通过一类的全限定名来获取其定义的二进制字节流。
- 1.将这个字节流所代表的静态存储结构转换为方法区的运行时数据结构。
- 2.在Java堆中生成一个代表这个类的```java.lang.Class```对象，作为堆方法区中这些数据的入口。

> 注意：二进制字节流并不只是单纯的从class文件中获取，比如它还可以从jar包中获取，从网络中获取（最典型的应用就是Applet)，由其他文件生成（jsp文件）等。

相对于类加载的其他阶段而言，在加载阶段（准确的说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。

加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个```java.lang.Class```类的对象，这样便可以通过该对象访问方法区中的这些数据。

> TODO 类加载器

### 2.2 验证 - 检查加载的class文件数据的正确性

验证的目的是为了确保class文件中的字节流包含的信息符合当前虚拟机的要求，而且不会危害虚拟机自身的安全。不同虚拟机对类验证的实现可能会有所不同，但大致都会完成一下四个阶段的验证：```文件格式、元数据、字节码和符号引用```

- **文件格式**：验证字节流是否符合class文件格式的规范，并且能被当前版本的虚拟机处理，该验证的主要目的是确保输入的字节流能正确的解析并存储于方法区之内。经过该阶段的验证后，字节流才会进入内存的方法区中进行存储，后面的三个验证都是基于方法区的存储结构进行的。
- **元数据**：对类的元数据信息进行语义校验（其实就是对类中的各数据类型进行语法校验），确保不存在不符合Java语法规范的元数据信息。
- **字节码**：该阶段验证的主要工作是进行数据流和控制流分析，对类的方法体进行校验分析，以保证被校验的类的方法在运行时不会做出危害虚拟机的安全行为。
- **符号引用**：这是最后一个阶段的验证，它发生在虚拟机将符号引用转化为直接引用的时候（解析阶段中发生该转化，下文会讲解），主要是对类自身以外的信息（常量池中的各种符号引用）进行匹配性的校验。

> 验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用```-Xverifynone```参数来关闭大部分类的验证措施，以缩短虚拟机类加载时间。

### 2.3 准备 - 为类的静态变量分配内存，并将其初始为默认值

准备阶段正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

- 0.此时进行内存分配的仅包括类变量（```static```），而不包括实例变量，实例变量会在对象实例化时随对象一块分配在Java堆中。
- 1.这里所设置的初始值通常情况下是数据类型默认的零值（如0，0L，null，false等），而不是被Java代码中显示的赋予的值。

假设一个类变量的定义为：```public static int value = 3;```

那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何Java方法，而把value赋值为3的```public static```指令是在程序编译后，存放于类构造器```<clinit>()```方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行。

> Java中所有的基本数据类型以及reference类型的默认零值：

![数据类型及默认值](https://niubility.org.cn/assets/images/jvm201908193.png)

> 这里还需要注意一下几点：
- 对基本数据类型、类变量（```static```）和全局变量，如果不显示地赋值而直接使用，则系统会为其赋予默认零值，而对于局部变量来说，在使用前必须显示地为其赋值，否则编译时不通过。
- 对于同时被static和final修饰的常量，必须在声明时就为其显示地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显示的为其赋值，也可以在类初始化时显示地为其赋值，总之，在使用前必须为其显示地赋值，系统不会为其赋予默认零值。
- 对于引用数据类型reference来说，如数组，对象引用等，如果没有对其进行显示地赋值而直接使用，系统都会为其默认赋予默认零值，即null。
- 如果数组在初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认零值。

- 2.如果类字段的字段属性表中存放在```ConstantValue```属性，即同时被final和static修饰，那么在准备阶段变量value就会被初始化为```ConstantValue```属性所指定的值。

假设上面的类变量value被定义为：```public staitc final int value = 3;```

编译时Javac将会为value生成```ConstantValue```属性，在准备阶段虚拟机根据```ConstantValue```的设置将value赋值为3。```我们可以理解为static final常量在编译期就将其结果放入了调用它的类的常量池中。```

### 2.4 解析 - 把类中的符号引用转换为直接引用

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。符号引用就是一组符号来描述目标，可以是任何字面量。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

对同一个符号引用进行多次解析请求时很常见的事情，虚拟机实现可能会对第一次解析的结果进行缓存（在运行时常量池中记录直接引用，并把常量标示为已解析状态），从而避免解析动作重复进行。

解析动作主要针对类或接口、字段、类方法、接口方法四类符号引用进行，分别对应于常量池中的CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info、CONSTANT_InterfaceMethodref_info四种常量类型。

- **0.类或接口解析**：判断所要转化成的直接引用是对数组类型，还是普通的对象类型的引用，从而进行不同的解析。
- **1.字段解析**：会先在本类中查找是否包含有简单名称和字段描述符都与目标相匹配的字段，如果有，则查找结束；如果没有，则会按照继承关系从上往下递归搜索该类所实现的各个接口和它们的父接口，还没有，则按照继承关系从上往下递归搜索其父类，直至查找结束，查找流程如下图所示：
![字段解析搜索流程](https://niubility.org.cn/assets/images/jvm201908192.png)

``` java
class Super {
    public static int value = 5;
    static {
        System.out.println("执行了super类静态代码语句块");
    }
}

class Father extends Super {
   public static int value = 6;
   static {
       System.out.println("执行了father类静态代码语句块");
   }
}

class Child extends Father {
    static {
        System.out.println("执行了child类静态代码语句块");
    }
}

public class StaticTest {
    public static void main(String [] args) {
        System.out.println(Child.value);
    }
}
```

执行结果如下：
- 执行了super类静态代码语句块
- 执行了father类静态代码语句块
- 6

如果注释掉father类中对value定义的那一行，则输出结果如下：
- 执行了super类静态代码语句块
- 5

分析如下：
> static变量发生在静态解析阶段，也即是初始化之前，此时已经将字段的符号引用转化为了内存引用，也便将它与对应的类关联在了一起，由于在子类中没有查找到与value相匹配的字段，那么value便不会与子类关联在一起，因此并不会触发子类的初始化。

- **2.类方法解析**：对类方法的解析于对字段解析的搜索步骤差不多，只是多了判断方法所处的是类还是接口的步骤，而且对类方法的匹配索索，实现搜索父类在搜索接口。
- **3.接口方法解析**：于类方法解析步骤类似，只是接口不会有父类，因此只递归搜索父接口就行。

### 2.5 初始化 - 类的静态变量，静态代码块执行初始化

初始化是类加载过程的最后一步，到了此阶段，才真正开始执行类定义中的Java程序代码。在准备阶段，类变量已经被赋过一次系统要求的初始值，而在初始化阶段，则是根据程序员通过程序指定的主观计划去初始化类变量和其他资源，或者可以从另一个角度表达：初始化阶段是执行类构造器```<clinit>()```方法的过程。

简单说明下```<clinit>()```方法执行规则

- 0.```<clinit>()```方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块中的语句合并产生，编译器收集的顺序是由语句在源文件中出现的顺序决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块中可以赋值，但不能访问。
- 1.```<clinit>()```与实例构造器```<init>()```方法（类的构造函数）不同，它不需要显示的调用父类构造器，虚拟机会保证在子类的```<clinit>()```方法执行之前，父类的```<clinit>()```方法已经执行完毕。因此，在虚拟机中第一个被执行```<clinit>()```方法的类肯定是```java.lang.Object```。
- 3.```<clinit>()```方法对于类或接口来说并不是必须的，如果一个类中没有静态语句块，也没有对类变量的赋值操作，那么编译器可以不为这个类生成```<clinit>()```方法。
- 4.接口中不能使用静态语句块，但仍然有类变量（final static）初始化的赋值操作，因此接口与类一样会生成```<clinit>()```方法。但是接口与类不同的是：执行接口的```<clinit>()```方法不需要先执行父接口的```<clinit>()```方法，只有当父接口中定义的变量被使用时，父接口才会被初始化。另外接口的实现在初始化也一样不会执行接口的```<clinit>()```方法。
- 5.虚拟机会保证一个类的```<clinit>()```方法在多线程环境中被正确的加锁和同步，如果个多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的```<clinit>()```方法，其他线程需要阻塞等待，直到活动线程执行```<clinit>()```方法完毕。如果在一个类的```<clinit>()```方法中有耗时很长的操作，那么就可能造成多个线程阻塞，在实际应用中这种阻塞往往是很隐蔽的。

给出一个简单的例子：

``` java
class Father {
    public static int value = 1;
    static {
        value = 2;
    }
}

class Child extends Father {
    public static int copyValue = value;
}

public class ClinitTest {
    public static void main(String [] args) {
        System.out.println(Child.copyValue);
    }
}
```

执行上面的代码，会打印出2，也就是说b的值被赋为了2。

我们来看得到该结果的步骤。

首先在准备阶段为类变量分配内存并设置类变量初始值，这样value和copyValue均被赋值为默认值0，而后再在调用```<clinit>（）```方法时给他们赋予程序中指定的值。当我们调用Child.copyValue时，触发Child的```<clinit>（）```方法，根据规则2，在此之前，要先执行完其父类Father的```<clinit>（）```方法，又根据规则1，在执行```<clinit>（）```方法时，需要按static语句或static变量赋值操作等在代码中出现的顺序来执行相关的static语句，因此当触发执行Father的```<clinit>（）```方法时，会先将value赋值为1，再执行static语句块中语句，将value赋值为2，而后再执行Child类的```<clinit>（）```方法，这样便会将copyValue的赋值为2.

如果我们颠倒一下Father类中```public static int value = 1;```语句和```static语句块```的顺序，程序执行后，则会打印出1。很明显是根据规则1，执行Father的```<clinit>（）```方法时，根据顺序先执行了static语句块中的内容，后执行了```public static int value = 1;```语句。

另外，在颠倒二者的顺序之后，如果在static语句块中对value进行访问（比如将value赋给某个变量），在编译时将会报错，因为根据规则1，它只能对value进行赋值，而不能访问。

## 3.类加载器

> 如何寻找类加载器？ 简单看一个例子：

``` java
/**
 * create on 2019-08-20 by gaoxinzhong
 **/
public class ClassLoaderTest {

    public static void main(String[] args) {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        System.out.println(loader);
        System.out.println(loader.getParent());
        System.out.println(loader.getParent().getParent());
    }
}
```
运行后，输出结果：

``` java
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@77afea7d
null
```

从上面的结果可以看出，并没有获取到```ExtClassLoader```的父Loader，原因是```Boostrap Loader```（启动类加载器）是用c++语言实现的，找不到一个确定的父Loader，于是就返回null。

![类加载器层次关系](https://niubility.org.cn/assets/images/jvm201908194.png)

> 注意这里父类加载器并不是通过继承关系实现，而是采用组合实现的。

### 3.1 虚拟机角度来讲，只存在两种不同的类加载器

- **启动（Boostrap）类加载器**：它使用c++实现（这里仅限于Hotspot，也就是JDK1.5之后默认的虚拟机，有很多其他的虚拟机是用Java语言实现的），是虚拟机自身的一部分。
- **其他类加载器**：这些类加载器都由Java语言实现，独立于虚拟机之外，并且全部继承自抽象类```java.lang.ClassLoader```，这些类加载器需要由启动类加载器加载到内存中之后才能去加载其他的类。

### 3.2 Java开发人员的角度来看，类加载器大致可以分为以下三类

- **启动（Boostrap）类加载器**：启动类加载器主要加载的是jvm自身需要的类，这个类加载使用c++实现，是虚拟机自身的一部分，它负责将```<JAVA_HOME>/lib```路径下的核心类库或```-Xbootclasspath```参数指定的路径下的jar包加载到内存中，注意由于虚拟机是按照文件名识别加载jar包的，如：rt.jar，如果文件名不被虚拟机识别，即使把jar包丢到lib目录下也是没有作用的（出于安全考虑，Boostrap启动类加载器只加载包名为java,javax,sun等开头的类）。
- **扩展（Extension）类加载器**：扩展类加载器是由Sun公司（已被Oracle收购）实现的```sun.misc.Launcher$ExtClassLoader```类，由Java语言实现，是Launch的静态内部类，它负责加载```<JAVA_HOME>/lib/ext```目录下或者由系统变量```-Djava.ext.dirs```指定路径下的类库，开发者可以直接使用标准扩展类加载器。
- **应用程序（Application）类加载器**：Sun公司实现的```sun.misc.Launcher$AppClassLoader```。它负责加载系统类路径```java -classpath```
或者```-D java.class.path```指定路径下的库，也就是我们经常用到的classpath路径，开发者可直接使用应用程序类加载，一般情况下该类加载器是程序默认的加载器，通过```ClassLoader#getSystemClassLoader()```方法可以获取到该类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器。因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的java class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点：

- 0.在执行非置信代码之前，自动验证数字签名。
- 1.动态地创建符合用户特定需要的定制化构建类。
- 2.从特定的场所取得java class，例如数据库中和网络中。

### 3.3 jvm类加载机制

- **全盘负责**：当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。
- **父类委托**：先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类。
- **缓存机制**：缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效。

### 3.4 自定义类加载器

> 自定义类加载器一般都继承自```ClassLoader```类，只需要重写```findClass()```方法即可。

``` java
package com.gaoxinzhong.classloader;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * create on 2019-08-21 by gaoxinzhong
 **/
public class UserClassLoader extends ClassLoader {

    private String root;

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        final byte[] bytes = loadClassByte(name);
        if (bytes == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, bytes, 0, bytes.length);
        }
    }

    private byte[] loadClassByte(String className) {
        String fileName = root + File.separatorChar + className.replace('.', File.separatorChar) + ".class";
        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        UserClassLoader userClassLoader = new UserClassLoader();
        userClassLoader.setRoot("/Users/gaoxinzhong/git");

        final Class<?> classLoaderTest = Class.forName("ClassLoaderTest", true, userClassLoader);
        final Object o = classLoaderTest.newInstance();

        System.out.println(o);
        System.out.println(o.getClass().getClassLoader()); // 打印由那个classLoader加载。
    }
}
```

> ClassLoaderTest.java 直接用javac编译放在某个目下，不要和```UserClassLoader```放在一起！！！

``` java
/**
 * create on 2019-08-21 by gaoxinzhong
 **/
public class ClassLoaderTest {

    private String say;

    public ClassLoaderTest() {
        this.say = "hello world";
    }

    public String getSay() {
        return say;
    }

    public void setSay(String say) {
        this.say = say;
    }

    public String toString() {
        return "[say:" + this.say + "]";
    }
}
```

> 自定义类加载器运行结果

``` java
[say:hello world]
com.gaoxinzhong.classloader.UserClassLoader@16b3fc9e
```
 
> ```自定义类加载器的核心在于对字节码文件的读取，如果是加密的字节码则需要对该类文件进行解密。还有几点需要注意：```
- 这里传递的文件名需要是类的全限定名，即：```package```路径```.Class```格式(com.xx.Xx.class)，因为```defineClass```方法是按照这种格式处理。
- 最好不要重写loadClass方法，因为这样容易破坏双亲委托模式。
- 这个类本身可以被```AppClassLoader```加载，不要和```UserClassLoader```（自定义类加载）放在一起！！！否则由于双亲委托机制的存在，会导致该类由```AppClassLoader```加载，而不是通过自定义的类加载器加载。

## 4.类的加载方式

> 类的加载有三种方式：

- 0.命令行启动应用的时候有jvm初始化加载。
- 1.通过Class.forName()方法动态加载。
- 2.通过ClassLoader.loadClass()方法动态加载。

``` java
import sun.jvm.hotspot.HelloWorld;

/**
 * create on 2019-08-20 by gaoxinzhong
 **/
public class ClassLoaderTest {

    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader loader = HelloWorld.class.getClassLoader();
        System.out.println(loader);
        // 使用ClassLoader.loadClass()来加载类，不会执行初始化块。
        System.out.println(loader.loadClass("ClassLoaderTest2"));
        // 使用Class.forName()来加载类，默认会执行静态代码块。
        System.out.println(Class.forName("ClassLoaderTest2"));
        // 使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块
        System.out.println(Class.forName("ClassLoaderTest2", false, loader));
    }
}

class ClassLoaderTest2 {
    static  {
        System.out.println("静态初始化块执行了。");
    }
}
```

> 分别切换加载方式，会有不同的输出结果。

**Class.forName()和ClassLoader.loadClass()区别**

- ```Class.forName()```：将类的.class文件加载到jvm之外，还会对类进行解释，并不会执行类的static块。
- ```ClassLoader.loadClass()```：只做一件事情，就是将.class文件加载到jvm中，不会执行static块的内容，只有在```newInstance```才会执行static块。
- ```Class.forname(name,initialize,loader)```：带参函数也可以控制是否加载static块。并且只有调用了```newInstance```方法调用构造函数，创建类的对象。


## 总结

整个类加载过程中，除了在加载阶段用户应用程序可以自定义类加载器参与外，其余所有的动作完全由虚拟机主导和控制。到了初始化才开始执行类中定义的Java程序代码（亦字节码），但这里的执行代码只是个开端，它仅限于```<clinit>()```方法。类加载过程主要是将class文件（准确地讲，应该是类的二进制字节流）加载到虚拟机内存中，真正执行字节码的操作，在加载完成后才真正开始。