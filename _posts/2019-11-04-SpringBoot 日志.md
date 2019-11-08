---
layout:     post
title:      SpringBoot 日志
subtitle:   门面、使用、整合、
date:       2019-11-04
author:     skylinelin
header-img: resource_img/spring/SpringBoot/head3.jpg
catalog: true
tags:
    - spring
    - springboot
    - 日志
    - 核心
    - SLF4J
    - Logback
---

> 不同日志框架如何使用与整合。

---

# 日志

## 日志框架种类

**市面上有很多的日志框架：**

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....



| 日志门面  （日志的抽象层）                                   | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ~~JCL（Jakarta  Commons Logging）~~    SLF4j（Simple  Logging Facade for Java）    **jboss-logging** | Log4j  JUL（java.util.logging）  Log4j2  **Logback** |

左边选一个门面（抽象层）、右边来选一个实现；

日志门面：  SLF4J；

日志实现：Logback；



SpringBoot：底层是Spring框架，Spring框架默认是用JCL；‘

​	**==SpringBoot选用 SLF4j和logback；==**







> 结尾