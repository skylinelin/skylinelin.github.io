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





