---
layout: post
title: Spring MVC REST URL
category: srping
tags: [srping, mvc]
---

在msm项目中，之前是使用.do结尾的url，现在我们把.do去掉，改成restful风格的url。改变如下：

1.在web.xml中，把url-pattern的内存改成 / [注意是 / 不是 /*]
	
	<servlet-mapping>
		<servlet-name>spring</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

2.在spring-mvc.xml 中配置静态文件路径映射，以免spring拦截静态文件：

	<!-- 访问静态资源的，定位到static目录 -->
	<mvc:resources location="/static/" mapping="/static/**" />

3.去掉action中RequestMapping的value只的.do后缀，同时可以添加参数。IndexController实例如下：

	@RequestMapping("/hello/{name}")
	public @ResponseBody String hello(@PathVariable String name) {
		System.out.println("hello "+name);
		logger.info("hello info.");
		return "hello "+name;
	}

4.运行之后访问如下地址：http://localhost:8080/msm/hello/zhangsan 可以看到返回内容。

注：如果出现404错误，可能是在spring-mvc.xml中没有配置 &lt;mvc:annotation-driven /&gt;

5.我们把静态文件放到static目录下，由于配置了静态文件映射，所以不会被拦截。如下访问：
	
	http://localhost:8080/msm/static/f1.jpg

详细信息请参见[源代码](https://github.com/zhxysky/msm)