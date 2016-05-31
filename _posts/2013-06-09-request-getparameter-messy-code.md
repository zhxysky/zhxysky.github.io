---
layout: post
title: request.getParameter乱码
category: web
tags: [web]
---

在web工程下，碰到一个乱码问题，就是form提交汉字的时候，后台用request.getParameter("parameterName")获取到的是乱码，查看页面，已经设置了 
	
	<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

和

	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

，在后台使用request.getCharacterEncoding()获取到的是null，而使用response.getCharacterEncoding() 获取到的是ISO-8859-1，竟然不一致。
google之后才知道，是因为没有设置forceEncoding，导致request和response编码不一致。在web.xml中，添加以下filter，并且把该filter作为web.xml中第一个filter。使所有请求都通过该filter过滤。

	<filter>
		<filter-name>SetCharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>SetCharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
