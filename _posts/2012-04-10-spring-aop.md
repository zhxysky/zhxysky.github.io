---
layout: post
title: Spring AOP简单实例
category: Spring
tags: [Spring, aop]
---






本文通过一个Spring AOP最简单的例子，来记录一下AOP的使用方法，以供以后参考。

我在之前的项目实例[msm](https://github.com/zhxysky/msm)基础上添加aop以实现日志的记录。步骤如下：

一、添加jar文件
	
	<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjrt</artifactId>
			<version>1.8.2</version>
	</dependency>
	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjweaver</artifactId>
		<version>1.8.2</version>
	</dependency>


二、添加对Spring aop的支持，在spring-mvc.xml中添加如下配置：
	
	 <aop:aspectj-autoproxy />


三、编写切面类LogAspect.java


	package com.zhxy.aop;

	import org.aspectj.lang.JoinPoint;
	import org.aspectj.lang.ProceedingJoinPoint;
	import org.aspectj.lang.annotation.After;
	import org.aspectj.lang.annotation.Around;
	import org.aspectj.lang.annotation.Aspect;
	import org.aspectj.lang.annotation.Before;
	import org.springframework.stereotype.Component;

	/**
	 * 定义切面
	 * @author zhxy
	 *
	 */

	@Component
	@Aspect
	public class LogAspect {

		@After("execution(* com.zhxy.service.*.*(..))")
		public void doAfter(JoinPoint jp) {
			System.out.println("do after method:" + jp.getTarget().getClass().getName() + "."
					+ jp.getSignature().getName());
		}

		@Around("execution(* com.zhxy.service.*.*(..))")
		public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
			long time = System.currentTimeMillis();
			Object retval = pjp.proceed();
			time = System.currentTimeMillis() - time;
			System.out.println("process time :" + time + " ms.");
			System.out.println();
			return retval;
		}

		@Before("execution(* com.zhxy.service.*.*(..))")
		public void doBefore(JoinPoint jp) {
			System.out.println("do before method:" + jp.getTarget().getClass().getName() + "."
					+ jp.getSignature().getName());
		}

		public void doThrowing(JoinPoint jp, Throwable ex) {
			System.out.println("method " + jp.getTarget().getClass().getName() + "." + jp.getSignature().getName()
					+ " throw exception");
			System.out.println(ex.getMessage());

		}
		
	}


四、修改以前的结构，添加service层接口和实现类，并把Controller中对dao的直接引用改成调用service层方法。

接口UserService.java

	package com.zhxy.service;

	import java.util.List;

	import com.zhxy.bean.User;

	public interface UserService {

		List<User> getUserList();
		
		public User getUserById(int userId);
		
	}


实现类UserServiceImpl.java

	package com.zhxy.service.impl;

	import java.util.List;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;

	import com.zhxy.bean.User;
	import com.zhxy.dao.UserDao;
	import com.zhxy.service.UserService;

	@Service
	public class UserServiceImpl implements UserService {

		@Autowired
		private UserDao userDao;
		@Override
		public List<User> getUserList() {
			return userDao.getUserList();
		}
		@Override
		public User getUserById(int userId) {
			System.out.println("get user by id."+userId);
			return null;
		}

	}


IndexAction中修改对dao的使用。

	package com.zhxy.controller;

	import java.util.List;

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Controller;
	import org.springframework.ui.ModelMap;
	import org.springframework.web.bind.annotation.RequestMapping;

	import com.zhxy.bean.User;
	import com.zhxy.service.UserService;

	@Controller
	public class IndexAction {

		Logger logger = LoggerFactory.getLogger(IndexAction.class);
		
		@Autowired
		private UserService userService;
		
		@RequestMapping("/hello.do")
		public String hello() {
			System.out.println("hello......");
			logger.info("hello info.");
			return "hello";
		}
		
		
		@RequestMapping("/userList.do")
		public String userList(ModelMap model) {
			List<User> userList = userService.getUserList();
			userService.getUserById(1);
			model.addAttribute("userList", userList);
			model.addAttribute("msg", "success");
			return "userlist";
		}
	}


现在我们再次运行该项目，访问userList.do，可以看到控制台打印出通过LogAspect记录的一些信息：

	do before method:com.zhxy.service.impl.UserServiceImpl.getUserList
	do after method:com.zhxy.service.impl.UserServiceImpl.getUserList
	process time :219 ms.

	do before method:com.zhxy.service.impl.UserServiceImpl.getUserById
	get user by id.1
	do after method:com.zhxy.service.impl.UserServiceImpl.getUserById
	process time :0 ms.

表明我们通过切面编程记录日志成功。^_^


代码已更新到[msm](https://github.com/zhxysky/msm)项目中。