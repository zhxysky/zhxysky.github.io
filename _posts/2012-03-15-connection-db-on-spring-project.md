---
layout: post
title: Spring mvc项目连接数据库
category: spring
tags: [Spring, DB]
---


使用spring连接mysql数据库。

1.添加数据库驱动，数据库连接池，页面访问等需要用到的jar包。在pom.xml中添加如下依赖：
	
	<!-- 用于spring访问数据库 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.6</version>
		</dependency>
		<dependency>
			<groupId>taglibs</groupId>
			<artifactId>standard</artifactId>
			<version>1.1.2</version>
		</dependency>
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
		</dependency>



2.在applicationContext.xml中配置数据源dataSource。

（1）在src/main/resources目录下添加数据库连接的基本信息jdbc.properties文件。

	dataSource.driverClassName=com.mysql.jdbc.Driver
	dataSource.jdbcUrl=jdbc:mysql://127.0.0.1/msm
	dataSource.username=root
	dataSource.password=root
	dataSource.initialPoolSize=10
	dataSource.minPoolSize=5
	dataSource.maxPoolSize=15
	dataSource.checkoutTimeout=1000

（2）在applicationContext.xml中配置数据源和访问数据库的模板：
	
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
		<context:property-placeholder location="classpath:/jdbc.properties" />
		
		<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
			<property name="driverClass" value="${dataSource.driverClassName}" />
			<property name="jdbcUrl" value="${dataSource.jdbcUrl}" />
			<property name="user" value="${dataSource.username}" />
			<property name="password" value="${dataSource.password}" />
			<property name="initialPoolSize" value="${dataSource.initialPoolSize}" />
			<property name="minPoolSize" value="${dataSource.minPoolSize}" />
			<property name="maxPoolSize" value="${dataSource.maxPoolSize}" />
			<!-- 当连接池用完时客户端调用getConnection()后等待获取新连接的时间，超时后将抛出  SQLException,如设为0则无限期等待。单位毫秒。Default: 0 -->
			<property name="checkoutTimeout" value="${dataSource.checkoutTimeout}" />
		</bean>
		
		
		<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
			<property name="dataSource" ref="dataSource" />
		</bean>
		
	</beans>


3.创建数据库表tb_user。首先，你需要建立自己的数据库，本文使用msm数据库。然后使用如下sql添加数据：

		/*
		Navicat MySQL Data Transfer

		Source Server         : localhost
		Source Server Version : 50510
		Source Host           : 127.0.0.1:3306
		Source Database       : msm

		Target Server Type    : MYSQL
		Target Server Version : 50510
		File Encoding         : 65001

		Date: 2014-08-15 16:02:31
		*/

		SET FOREIGN_KEY_CHECKS=0;

		-- ----------------------------
		-- Table structure for `tb_user`
		-- ----------------------------
		DROP TABLE IF EXISTS `tb_user`;
		CREATE TABLE `tb_user` (
		  `id` int(11) NOT NULL,
		  `username` varchar(100) NOT NULL,
		  PRIMARY KEY (`id`)
		) ENGINE=InnoDB DEFAULT CHARSET=utf8;

		-- ----------------------------
		-- Records of tb_user
		-- ----------------------------
		INSERT INTO `tb_user` VALUES ('1', '张三');
		INSERT INTO `tb_user` VALUES ('2', '里斯');
		INSERT INTO `tb_user` VALUES ('3', 'tony');
		INSERT INTO `tb_user` VALUES ('4', 'jack');


4.创建基本bean类User。
	
	package com.zhxy.bean;

	public class User {

		private int id;
		private String name;

		public int getId() {
			return id;
		}

		public void setId(int id) {
			this.id = id;
		}

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}

		@Override
		public String toString() {
			return "User [id=" + id + ", name=" + name + "]";
		}

	}	

5.创建访问数据库的DAO类。

接口：
	
	package com.zhxy.dao;

	import java.util.List;

	import com.zhxy.bean.User;

	public interface UserDao {

		List<User> getUserList();
	}


实现类；

	package com.zhxy.dao.impl;

	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.util.List;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.jdbc.core.JdbcTemplate;
	import org.springframework.jdbc.core.RowMapper;
	import org.springframework.stereotype.Repository;

	import com.zhxy.bean.User;
	import com.zhxy.dao.UserDao;

	@Repository
	public class UserDaoImpl implements UserDao {

		@Autowired
		private JdbcTemplate jdbcTemplate;

		@Override
		public List<User> getUserList() {
			String sql = "select * from tb_user";
			return this.jdbcTemplate.query(sql, new RowMapper<User>() {
				@Override
				public User mapRow(ResultSet rs, int rowNum) throws SQLException {
					return rsToObj(rs);
				}
			});
		}

		protected User rsToObj(ResultSet rs) throws SQLException {
			User user = new User();
			user.setId(rs.getInt("id"));
			user.setName(rs.getString("username"));
			return user;
		}

	}


6.在Action中获取数据并传递到页面。现在我们在IndexAction添加一个映射，用于访问数据库信息的。

		@RequestMapping("/userList.do")
		public String userList(ModelMap model) {
			List<User> userList = userDao.getUserList();
			model.addAttribute("userList", userList);
			model.addAttribute("msg", "success");
			return "userlist";
		}

可以看到结果是返回到userlist.jsp页面的，所以要在WEB-INF/jsp 下面新建页面userlist.jsp用于显示数据。

		<%@ page language="java" contentType="text/html; charset=UTF-8"
		    pageEncoding="UTF-8"%>
		<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
		<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
		<html>
		<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<title>user list</title>
		</head>
		<body>
			${msg}<br>
			
			<c:forEach var="user" items="${userList }">
				${user.id } : ${user.name } <br>
			</c:forEach>
		</body>
		</html>


7.现在我们重新运行msm工程，访问http://localhost:8080/msm/userList.do ，即可看到显示出从数据库获取到的数据。Y(^_^)Y