---
layout:     post
title:      SpringMVC中遇到的问题
subtitle:   问题、技巧、bug
date:       2019-10-15
author:     skylinelin
header-img: img/Spring.png
catalog: true
tags:
    - 问题
    - 技巧
    - bug
---

> 日常问题积累

---

## 1、SpringMVC中使用过滤器完成 PUT 请求或 DELETE 请求时，出现 405，logger日志打印正常

**解决：**

使用 Controller redirect(重定向)

```java
@RequestMapping(value = "/testRest/{id}", method = RequestMethod.PUT)
	public String testRestPut(@PathVariable("id") Integer id) {
		System.out.println("正在更新的订单id为(PUT)：" + id);
		return "redirect:/mvc/success";
	}

@RequestMapping("success")
	public String success() {
		return SUCCESS;
	}
```

## 2、POST、PUT、DELETE请求出现中文乱码

**解决：**

配置字符编码过滤器

```xml
<!-- 配置字符编码过滤器 -->
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```

> 注意：	如果中文有乱码，需要配置字符编码过滤器，且配置其他过滤器之前，否则不起作用。

