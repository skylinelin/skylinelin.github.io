---
layout:     post
title:      Spring Cloud
subtitle:   分布式、微服、负载均衡
date:       2019-11-20
author:     skylinelin
header-img: resource_img/spring/SpringCloud/head.jpg
catalog: true
tags:
    - Eureka
    - Zuul
    - Ribbon
    - Feign
    - Hystix
---

> 不是一种技术，而是一个集合。

---

## Spring Cloud常用组件

- Eureka
- zuul
- Ribbon
- Feign
- Hystix

---

## Eureka注册中心

![](/resource_img/spring/SpringCloud/Eurekayl.png)



- **Eureka**：就是服务注册中心（可以是一个集群），对外暴露自己的地址
- **提供者**：启动后向Eureka注册自己信息（地址，提供什么服务）
- **消费者**：向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新
- **心跳（续约）**：提供者定期通过http方式向Eureka刷新自己的状态