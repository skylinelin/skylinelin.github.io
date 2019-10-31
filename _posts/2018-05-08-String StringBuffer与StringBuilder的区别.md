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

## String

字符串广泛应用 在Java 编程中，在 Java 中字符串属于对象，Java 提供了 String 类来创建和操作字符串。

> String的值是**不可变**的，这就导致每次对String的操作都会生成**新的String对象**，这样不仅效率低下，而且大量浪费有限的内存空间。

![内存图](/resource_img/java/String/nct.png)

