---
layout: post
title: Spring4单元测试
category: Spring
tags: [Spring,Junit]
---

最近突然发现自己在开发过程中一直没有主动使用过Junit单元测试，今天想学习一下到底怎么在web项目下进行junit测试的。由于使用了spring 4的版本，查看文档发现竟然如此简单。记录如下：

以测试service层接口UserService为例，首先添加spring-test的jar包。在pom.xml中添加如下依赖（junit jar包版本由3.8变成了4.10）：
	
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-test</artifactId>
		<version>${spring.version}</version>
		<scope>compile</scope>
	</dependency>


测试代码如下：
	
	package com.zhxy.test;

	import java.util.List;

	import javax.annotation.Resource;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

	import com.zhxy.bean.User;
	import com.zhxy.service.UserService;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(locations={"classpath:applicationContext.xml","classpath:spring-mvc.xml"})
	public class UserServiceTest {

		@Resource
		UserService userService;
		
		@Test
		public void testGetUserList() {
			List<User> userList = userService.getUserList();
			for(User user : userList) {
				System.out.println(user.toString());
			}
		}
		
		@Test
		public void testGetUserById() {
			User user = userService.getUserById(3);
			System.out.println(user.toString());
		}
	}

把该程序run as ---> Junit Test，可以看到如下输出内容：

	do before method:com.zhxy.service.impl.UserServiceImpl.getUserList
	process time :173 ms.

	do after method:com.zhxy.service.impl.UserServiceImpl.getUserList
	User [id=1, name=张三]
	User [id=2, name=里斯]
	User [id=3, name=tony]
	User [id=4, name=jack]
	do before method:com.zhxy.service.impl.UserServiceImpl.getUserById
	get user by id.3
	process time :23 ms.

	do after method:com.zhxy.service.impl.UserServiceImpl.getUserById
	User [id=3, name=tony]

由于代码中配置了切面，所以会有一些其他的输出信息。代码请查看[msm项目源码](https://github.com/zhxysky/msm)。