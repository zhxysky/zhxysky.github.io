---
layout: post
title: 给项目添加日志
category: log
tags: [Spring, log4j, java]
---


在[前面一篇文章](http://zhxysky.github.io/spring/2012/03/12/spring-mvc-on-maven-web/)，我们搭建了一个spring mvc的工程，现在我们来给该工程加上日志。

1.首先，我们先添加一个jar包。在pom.xml中，添加以下依赖：

		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>

其中两个版面信息添加到pom.xml 的properties元素里。

		<log4j.version>1.2.17</log4j.version>
		<slf4j.version>1.7.7</slf4j.version>


2.添加log4j.properties配置文件：

	# Root logger option
	log4j.rootLogger=DEBUG, stdout, file
	 
	# Redirect log messages to console
	log4j.appender.stdout=org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.Target=System.out
	log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
	log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
	 
	# Redirect log messages to a log file
	log4j.appender.file=org.apache.log4j.RollingFileAppender

	log4j.appender.file.File=/logs/msm.log
	log4j.appender.file.MaxFileSize=5MB
	log4j.appender.file.MaxBackupIndex=10
	log4j.appender.file.layout=org.apache.log4j.PatternLayout
	log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n


配置内容可自行修改。

3.在IndexAction中记录日志：

在IndexAction.java中添加属性logger。

	Logger logger = LoggerFactory.getLogger(IndexAction.class);

在hell()方法中记录日志；
	
	logger.info("hello info.");

最后运行程序，可以在控制台和输出文件里看到日志内容。
	
