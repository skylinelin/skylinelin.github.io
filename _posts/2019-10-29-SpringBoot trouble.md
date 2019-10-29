---
layout:     post
title:      SpringBoot trouble
subtitle:   一些一些小坎坷
date:       2019-10-29
author:     skylinelin
header-img: resource_img/spring/SpringBoot/head.jpg
catalog: true
tags:
    - spring
    - springboot
    - error
    - warning
---

> winter has come,can spring be far behind?

---

## 问题一：使用@ConfigurationProperties注解将yaml注入bean问题

### 两个实体类

**Person.java**

```java
@Data
@ToString
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

**Dog.java**

```java
@Data
@ToString
public class Dog {

    private String name;
    private Integer age;
}
```

### yml文件

**application.yml**

```yaml
person:
  lastName: zhangsan
  age: 18
  boss: false
  birth: 2019/10/29
  maps: {key1: v1,k2: 12}
  lists:
    - lisi
    - wangwu
    - zhaoliu
  dog:
    name: 小狗
    age: 2
```

### 导入依赖

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-configuration-processor</artifactId>
     <optional>true</optional>
</dependency>
```



> **注意：** 
>
> - 使用@ConfigurationProperties的时候要保证该类可以被扫描（例如：加上@Component注解或者其它注解）；
> - 依赖的导入；
> - yml文件格式的正确。

---



## 问题二：@Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件yml还是properties他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；