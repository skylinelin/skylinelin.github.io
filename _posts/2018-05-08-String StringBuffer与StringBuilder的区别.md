---
layout:     post
title:      String,StringBuffer与StringBuilder的区别
subtitle:   内存、线程、继承、差异
date:       2018-05-08
author:     skylinelin
header-img: resource_img/java/String/head.jpeg
catalog: true
tags:
    - 内存
    - 可变or不可变
    - 线程安全
    - 应用范围
---

> 白衣苍狗几干回，惟有溪山长不改。

## String 类

字符串广泛应用 在Java 编程中，在 Java 中字符串属于对象，Java 提供了 String 类来创建和操作字符串。

> String的值是**不可变**的，这就导致每次对String的操作都会生成**新的String对象**，这样不仅效率低下，而且大量浪费有限的内存空间。

![内存图](/resource_img/java/String/nct.png)



初始String值为“hello”，然后在这个字符串后面加上新的字符串“world”，这个过程是需要重新在栈堆内存中开辟内存空间的，最终得到了“hello world”字符串也相应的需要开辟内存空间。

这样高频率的开辟内存空间，使得内存极大的浪费。

**解决上面问题，Google引入了两个新的类——StringBuffer类和StringBuild类来对此种变化字符串进行处理。**



## **StringBuffer 类 和 StringBuilder 类**

> StringBuffer 类 和 StringBuilder 类的对象进行多次修改而不产生新的未使用对象。

StringBuilder 类在 Java 5 中被提出，它和 StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。

由于 StringBuilder 相较于 StringBuffer 有速度优势，**所以多数情况下建议使用 StringBuilder 类**。然而在应用程序要求线程安全的情况下，则必须使用 StringBuffer 类。

#### 继承关系图

![继承关系](/resource_img/java/String/jcgx.png)



## 三者区别

|                  |     String     | StringBuffer | StringBuilder |
| :--------------: | :------------: | :----------: | :-----------: |
|   **是否可变**   |     不可变     |     可变     |     可变      |
| **线程是否安全** |       /        |     安全     |    不安全     |
|     **速度**     | 慢（浪费空间） |      慢      |      快       |



## 应用范围

1. 操作数据量少使用String；
2. 多线程操作字符串缓冲区下操作大量数据 StringBuffer；
3. 单线程操作字符串缓冲区下操作大量数据 StringBuilder。



> 结尾