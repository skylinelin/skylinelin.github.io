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



### 案例

*注意三步走*

导包、注解、配置文件



**Eureka注册中心**

```xml
<!-- 导入依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
//启动类添加注解
@EnableEurekaServer
```

```yaml
# application.yml
server:
  port: 10086
spring:
  application:
    name: eureka01
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10086/eureka
  instance:
    prefer-ip-address: true
    ip-address: 127.0.0.1
```



**服务提供者**

```xml
<!-- 导入依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```java
//启动类添加注解
@EnableDiscoveryClient
```

```yaml
# application.yml
server:
  port: 8081
spring:
  application:
    name: user-service
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
    ip-address: 127.0.0.1
```



**消费者**

消费者和服务提供者配置相同（除端口和名称不同），因为消费者同样也是服务提供者。



**测试**

```java
@RestController
@RequestMapping("/get")
public class UserGetController {
    @Autowired
    protected RestTemplate restTemplate;
    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/{id}")
    public User queryById(@PathVariable("id") Integer id){
        //根据服务id获取实例
        List<ServiceInstance> instances = discoveryClient.getInstances("user-service");
        ServiceInstance instance = instances.get(0);
        String url = "http://"+instance.getHost()+":"+instance.getPort()+"/user/"+id;
        System.out.println(url);
        User user = restTemplate.getForObject(url,User.class);
        return user;
    }
}
```

