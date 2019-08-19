---
layout: post
title: "Java类加载机制"
date: 2019-07-17 10:51:51
categories: Java
---

## 1.什么是类的加载？

Java虚拟机把描述类的数据从class文件中加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的的Java类型，这就是虚拟机的加载机制。class文件由类的装载器装载后，在JVM中将形成一份描述class结构的元信息对象，通过该元信息对象可以获取class的结构信息：如构造函数，属性和方法等，Java允许用户借由这个class相关的元信息对象间接调用class对象的功能。

![jvm体系结构](https://niubility.org.cn/assets/images/JVM0.jpeg)

备注：方法区和堆线程共享,java栈，本地方法栈和程序计数器线程私有。
