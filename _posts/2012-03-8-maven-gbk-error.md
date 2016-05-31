---
layout: post
title: maven编码GBK的不可映射字符
category: maven
tags: [maven]
---


最近遇到maven工程在编译或者打包的时候出现[错误: 编码GBK的不可映射字符]的错误，这是因为m2默认的是GBK的编码格式，而我们的项目使用的是UTF-8的格式。


解决办法：在pom.xml 的properties属性里添加project.build.sourceEncoding节点，并设置为UTF-8。如下：

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
	</properties>
