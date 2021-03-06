---
layout: post
title: "Jvm内存结构"
date: 2019-08-21 18:52:51
categories: Java
---

jvm内存结构

## 1.概述

Java虚拟机在运行Java程序时，把它所管理的内存划分为若干个不同的数据区域，主要包括以下五个部分：程序计数器、Java堆、Java虚拟机栈、方法区和本地方法栈。

![jvm内存结构](https://niubility.org.cn/assets/images/jvm201908196.png)

## 2.程序计数器（Program Counter Register）

一块较小的内存空间，它是当前线程所执行的字节码的行号指示器，字节码解释器工作时通过改变计数器的值来选择下一条需要执行的字节码指令，分支、跳转、循环等基础功能都要依赖它实现。```每个线程都有一个独立的程序计数器，各线程间的计数器互不影响，因此该区域是线程私有的。```

当线程执行一个Java方法时，该计数器记录的是正在执行的虚拟机字节码指令的地址，当线程执行的是Native方法（调用本地操作系统方法）时，该计数器的值为空。另外，```该内存区域是唯一一个在Java虚拟机规范中没有规定任何OOM（内存溢出：OutMemoryError）情况的区域。```

## 3.Java虚拟机栈（Java Virtual Machine Stack）

该区域也是```线程私有```的，它的生命周期与线程相同。虚拟机描述的是Java方法执行的内存模型，每个方法被执行的时候都会同时创建一个栈桢，栈它是用于支持虚拟机进行方法调用和方法执行的数据结构。对于执行引擎来说，活动线程中，只有栈顶的栈桢是有效的，称为当前栈桢，这个栈桢所关联的方法称为当前方法，执行引擎所运行的所有字节码指令都只针对当前栈桢进行操作。栈桢用于存储局部变量、操作数栈、动态链接、方法返回地址和一些额外的附加信息。```在编译程序代码时，栈桢需要多大局部变量表、多深的操作数栈都已经完全确定，并且写入方法表的code属性之中。因此一个栈桢需要分配多少内存，不会收到程序运行器变量数据的影响，仅仅取决于具体的虚拟机实现。```

> 在Java虚拟机规范中，该区域规定了两种异常情况：

- 1.如果线程请求的栈深度大于虚拟机所允许的深度，将抛出```StackOverflowError```异常.
- 2.如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出```OutOfMemoryError```异常。

> 栈深度StackOverflowError异常测试

``` java
/**
 * 栈深度StackOverflowError异常测试
 * <p>
 * create on 2019-08-22 by gaoxinzhong
 **/
public class VmStackSOF {
    private static int index = 1;

    public void increment() {
        index++;
        increment();
    }

    public static void main(String[] args) {
        VmStackSOF vmStackSOF = new VmStackSOF();
        try {
            vmStackSOF.increment();
        } catch (StackOverflowError error) {
            System.out.println("stack deep : " + index);
            error.printStackTrace();
        }
    }
}
```

> 虚拟机栈OutOfMemmoryError异常 慎重测试！

``` java
/**
 * 虚拟机栈OutOfMemmoryError异常
 * <p> 
 * -Xms20m -Xmx20m
 * create on 2019-08-22 by gaoxinzhong
 **/
public class VmStackOOM {

    public void stackLeakByThread() {
        while (true) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true) {

                    }
                }
            });
            thread.start();
        }
    }

    public static void main(String[] args) {
        VmStackOOM vmStackOOM = new VmStackOOM();
        try {
            vmStackOOM.stackLeakByThread();
        } catch (OutOfMemoryError outOfMemoryError) {
            outOfMemoryError.printStackTrace();
        }

    }
}
```

### 3.1 局部变量表

局部变量表是一组变量值存储空间，```用于存放方法参数和方法内部定义的局部变量，其中存放的数据的类型是编译期可知的各种基本数据类型、对象引用（reference）和returnAddress类型（它指向了一条字节码指令的地址）。局部变量表所需的内存空间在编译期间完成分配，即在Java程序被编译成Class文件时，就确定了所需分配的最大局部变量表的容量。当进入一个方法时，这个方法需要在栈中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。```

局部变量表的容量以变量槽（Slot）为最小单位。在虚拟机规范中并没有明确指明一个Slot应占用的内存空间大小（允许其随着处理器、操作系统或虚拟机的不同而发生变化），```一个Slot可以存放一个32位以内的数据类型：boolean、byte、char、short、int、float、reference和returnAddresss。reference是对象的引用类型，returnAddress是为字节指令服务的，它执行了一条字节码指令的地址。对于64位的数据类型（long和double），虚拟机会以高位在前的方式为其分配两个连续的Slot空间。```

虚拟机通过索引定位的方式使用局部变量表，索引值的范围是从0开始到局部变量表最大的Slot数量，对于32位数据类型的变量，索引n代表第n个Slot，对于64位的，索引n代表第n和第n+1两个Slot。

在方法执行时，虚拟机是使用局部变量表来完成参数值到参数变量列表的传递过程的，如果是实例方法（非static），则局部变量表中的第0位索引的Slot默认是用于传递方法所属对象实例的引用，在方法中可以通过关键字“this”来访问这个隐含的参数。其余参数则按照参数表的顺序来排列，占用从1开始的局部变量Slot，参数表分配完毕后，再根据方法体内部定义的变量顺序和作用域分配其余的Slot。

局部变量表中的Slot是可重用的，方法体中定义的变量，作用域并不一定会覆盖整个方法体，如果当前字节码PC计数器的值已经超过了某个变量的作用域，那么这个变量对应的Slot就可以交给其他变量使用。```这样的设计不仅仅是为了节省空间，在某些情况下Slot的复用会直接影响到系统的而垃圾收集行为。```

### 3.2 操作数栈

操作数栈又常被称为操作栈，```操作数栈的最大深度也是在编译的时候就确定了。32位数据类型所占的栈容量为1,64为数据类型所占的栈容量为2。当一个方法开始执行时，它的操作栈是空的，在方法的执行过程中，会有各种字节码指令（比如：加操作、赋值运算等）向操作栈中写入和提取内容，也就是入栈和出栈操作。```

java虚拟机的```解释执行引擎称为“基于栈的执行引擎”，其中所指的“栈”就是操作数栈。因此我们也称Java虚拟机是基于栈的，这点不同于Android虚拟机，Android虚拟机是基于寄存器的。```

基于栈的指令集```最主要的优点是可移植性强，主要的缺点是执行速度相对会慢些；而由于寄存器由硬件直接提供，所以基于寄存器指令集最主要的优点是执行速度快，主要的缺点是可移植性差。```

### 3.3 动态链接

每个栈帧都包含一个指向运行时常量池（在方法区中，后面介绍）中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接。Class文件的常量池中存在有大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用为参数。这些符号引用，一部分会在类加载阶段或第一次使用的时候转化为直接引用（如final、static域等），称为静态解析，另一部分将在每一次的运行期间转化为直接引用，这部分称为动态连接。

### 3.4 方法返回地址

当一个方法被执行后，有两种方式退出该方法：执行引擎遇到了任意一个方法返回的字节码指令或遇到了异常，并且该异常没有在方法体内得到处理。无论采用何种退出方式，在方法退出之后，都需要返回到方法被调用的位置，程序才能继续执行。方法返回时可能需要在栈帧中保存一些信息，用来帮助恢复它的上层方法的执行状态。一般来说，方法正常退出时，调用者的PC计数器的值就可以作为返回地址，栈帧中很可能保存了这个计数器值，而方法异常退出时，返回地址是要通过异常处理器来确定的，栈帧中一般不会保存这部分信息。

方法退出的过程实际上等同于把当前栈帧出站，因此退出时可能执行的操作有：恢复上层方法的局部变量表和操作数栈，如果有返回值，则把它压入调用者栈帧的操作数栈中，调整PC计数器的值以指向方法调用指令后面的一条指令。

## 4.本地方法栈（Native Method Stack）

本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

## 5.Java堆（Java Heap）

Java Heap是Java虚拟机所管理的内存中最大的一块，它是所有线程共享的一块内存区域。几乎所有的对象实例和数组都在这类分配内存。Java Heap是垃圾收集器管理的主要区域，因此很多时候也被称为“GC堆”。从内存回收角度看，可分为新生代和老年代。而新生代又可分为 Eden 区、From Survivor 区、To Survivor 区等。

Java 堆的实现，既可以实现为固定的，也可以是扩展的。当前虚拟机都按照可扩展来实现，通过 -Xmx 和 -Xms 控制堆大小。

根据Java虚拟机规范的规定，Java堆可以处在物理上不连续的内存空间中，只要逻辑上是连续的即可。如果在堆中没有内存可分配时，并且堆也无法扩展时，将会抛出OutOfMemoryError异常。 

> 堆OutOfMemmoryError异常

``` java
import java.util.ArrayList;
import java.util.List;

/**
 * 堆OutOfMemmoryError异常
 * <p>
 * -Xms20m -Xmx20m
 * create on 2019-08-22 by gaoxinzhong
 **/
public class HeapOOM {
    public static void main(String[] args) {
        List<HeapOOM> heapOOMS = new ArrayList<>();
        while (true) {
            heapOOMS.add(new HeapOOM());
        }
    }
}
```

## 6.方法区（Method Area）

方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java堆区分开来。

对于习惯在HotSpot虚拟机上开发和部署程序的开发者来说，很多人愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已。

Java虚拟机规范对这个区域的限制非常宽松，除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是有必要的。

根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

> 方法区中```运行时常量池```

运行时常量池是方法区的一部分。Class 文件中的常量池用于编译期生成的各种字面量和符号引用，这部分内容在类加载后被存入运行时常量池。
动态性是运行时常量池相对于 Class 文件常量池的一个重要特征，即不要求常量一定只有编译期才能产生，运行期间也可能将新的常量放入池中。
运行时常量池受到方法区内存的限制，如果常量池无法再申请内存，就会抛出 OutOfMemoryError 异常。

## 7.永久代和元空间

### 7.1 永久代

绝大部分 Java 程序员应该都见过 "java.lang.OutOfMemoryError: PermGen space "这个异常。这里的 “PermGen space”其实指的就是方法区。不过方法区和“PermGen space”又有着本质的区别。前者是 JVM 的规范，而后者则是 JVM 规范的一种实现，并且只有 HotSpot 才有 “PermGen space”，而对于其他类型的虚拟机，如 JRockit（Oracle）、J9（IBM） 并没有“PermGen space”。由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。最典型的场景就是，在 jsp 页面比较多的情况，容易出现永久代内存溢出。我们现在通过动态生成类来模拟 “PermGen space”的内存溢出：

> 普通类

```java 
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
> 永久代OutOfMemoryError Permgen space异常，jdk1.7测试，jdk1.8用元空间替换了永久代。

``` java
import java.io.File;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.ArrayList;
import java.util.List;

/**
 * 永久代OutOfMemoryError Permgen space异常
 * <p>
 * PermSize8m MaxPermSize8m
 * create on 2019-08-24 by gaoxinzhong
 **/
public class PermGenOom {

    public static void main(String[] args) {
        URL url = null;
        List<ClassLoader> classLoaderList = new ArrayList<ClassLoader>();
        try {
            url = new File("/Users/gaoxinzhong/git").toURI().toURL();
            URL[] urls = {url};
            while (true) {
                ClassLoader loader = new URLClassLoader(urls);
                classLoaderList.add(loader);
                loader.loadClass("ClassLoaderTest");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 7.2 元空间

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：

- **-XX:MetaspaceSize**：初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
- **-XX:MaxMetaspaceSize**：最大空间，默认是没有限制的。

除了上面两个指定大小的选项以外，还有两个与 GC 相关的属性：

- **-XX:MinMetaspaceFreeRatio**：在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集
- **-XX:MaxMetaspaceFreeRatio**：在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集

## 8.常见的OOM及原因

**java.lang.OutOfMemoryError:Java heap space**

java堆中主要用于存放各种对象实例。当堆中没有足够的空间分配给新的对象时，或者说达到了堆空间设置的最大空间限制，则会抛出此异常。

引起内存溢出的主要原因有：
- 流量访问大，超过设置的堆空间大小。
- 内存泄漏，不能被回收的对象消耗过多堆空间。

**java.lang.OutOfMemoryErro:Permgen space**

在jdk1.7中，HotSpot虚拟机使用永久代实现方法区，永久代较小，而且回收效率低，很容易出现内存溢出。

因此，jdk1.8中取消了永久代，使用元空间实现方法区，存放在本地内存中。

**java.lang.OutOfMemoryError:Metaspace**

方法区主要存储类的元信息，Hotspot元数据区。当元空间没有足够的空间分配给加载的类是，会抛出此异常。

引起元空间数据不足主要原因有：
- 加载的类太多，常见于jsp页面过多时。
- 元空间被实现在堆外，主要受到进程本身的限制，一般很难出现溢出。

## 总结

通过上面分析，大家应该大致了解了 JVM 的内存划分，也清楚了 JDK 8 中永久代向元空间的转换。不过大家应该都有一个疑问，就是为什么要做这个转换？所以，最后给大家总结以下几点原因：

- 字符串存在永久代中，容易出现性能问题和内存溢出。
- 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
- 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
- Oracle 可能会将HotSpot 与 JRockit 合二为一。