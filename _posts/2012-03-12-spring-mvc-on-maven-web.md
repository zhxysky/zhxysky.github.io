---
layout: post
title: 在maven web工程上配置Spring MVC
category: Spring
tags: [Spring, maven, java]
---


在[前面一篇](http://zhxysky.github.io/maven/2012/03/06/new-maven-web-project/) 新建的maven web项目里面，我们添加Spring MVC的配置。


一、首先，我们把配置spring需要用的jar文件添加一下。在pom.xml中添加用到的jar文件的依赖，即可自动下载jar包。本文pom.xml用用到的jar依赖如下：
	
		<properties>
			<spring.version>3.1.0.RELEASE</spring.version>
		</properties>

		<dependencies>
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>3.8.1</version>
				<scope>test</scope>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-core</artifactId>
				<version>${spring.version}</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>${spring.version}</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${spring.version}</version>
			</dependency>
		</dependencies>


二、在src/main/resources目录下面添加配置文件applicationContext.xml 和 spring-mvc.xml。其中applicationContext.xml用于配置基本bean类。spring-mvc.xml用于填写spring mvc的一些配置。
	

applicationContext.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd  
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd  
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

		<!-- spring bean config. -->
	</beans>



spring-mvc.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="
	        http://www.springframework.org/schema/beans     
	        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	        http://www.springframework.org/schema/context 
	        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

		<!-- spring mvc config. -->

		<!-- 配置自动扫描范围 -->
		<context:component-scan base-package="com.zhxy.controller" />

	 	<!-- 配置视图目录信息 -->
		<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
			<property name="prefix">
				<value>/WEB-INF/jsp/</value>
			</property>
			<property name="suffix">
				<value>.jsp</value>
			</property>
		</bean>

	</beans>

spring-mvc.xml中配置用于显示的页面的目录是WEB-INF/jsp,所以需要在WEB-INF下新建jsp目录。

然后添加web.xml的配置。如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
	http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
		<display-name>Archetype Created Web Application</display-name>

		<context-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath*:applicationContext.xml</param-value>
		</context-param>

		<listener>
			<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
		</listener>

		<servlet>
			<servlet-name>spring</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<!-- 为spring指定配置文件 -->
			<init-param>
				<param-name>contextConfigLocation</param-name>
				<param-value>classpath*:spring-mvc.xml</param-value>
			</init-param>
			<load-on-startup>1</load-on-startup>
		</servlet>

		<servlet-mapping>
			<servlet-name>spring</servlet-name>
			<url-pattern>*.do</url-pattern>
		</servlet-mapping>

	</web-app>



三、现在我们可以新建一个Action类，其实就是一个普通的类，不过用于处理请求，所以常以Action结尾。此处我们新建IndexAction类。
首先，新建一个package名为com.zhxy.controller，用于存放控制类。然后，在该包下面新建一个类IndexAction.
		
	package com.zhxy.controller;

	import org.springframework.stereotype.Controller;
	import org.springframework.web.bind.annotation.RequestMapping;

	@Controller
	public class IndexAction {

		@RequestMapping("/hello.do")
		public String hello() {
			System.out.println("hello......");
			return "hello";
		}
	}

这里我们使用@Controller让IndexAction表明该类是一个控制类，又使用@RequestMapping来指定可以处理的请求如/hello.do，如果访问/hello.do就会执行hello方法。return “hello”则会返回到WEB-INF/jsp/hello.jsp页面


现在，我们先在index.jsp页面上添加一个按钮，作为触发hello.do的连接
	
		<a href="hello.do">Click Here</a>

然后，我们在jsp目录下新建一个hello.jsp页面作为显示结果的页面，内容可以随意填写。


四、至此，我们的spring mvc环境已经搭建完毕。可以运行一下该工程访问hello.do看效果。我是运行在tomcat上的，然后访问http://localhost:8080/msm/，点击连接访问hello.do,跳转到hello.jsp，说明spring 处理了我们的请求，并跳转到相应的页面。

项目源码请查看<https://github.com/zhxysky/msm>