---
layout: post
title: maven使用本地jar包
category: maven
tags: [maven, java]
---

一般我们在使用maven添加jar的时候，都是直接在pom.xml中添加依赖,变可从中央仓库中下载该jar包。但是有时候偏偏中央仓库中没有我们需要的jar包，或者需要使用我们自己的jar包的时候，该怎么办呢？

解决办法就是把本地的jar包添加到本地仓库中，然后在pom.xml中添加依赖即可。具体如下：

本文以添加ZHConverter.jar为例。

1.把需要添加的jar文件ZHConverter.jar放到一个文件夹下面，比如放到lib下面。lib可以在任何地方。

2.在该文件夹下面添加pom.xml文件，主要用于定义jar文件的坐标及其相应的依赖代码。本文添加的jar包不需要依赖别的jar包，所以省略依赖的配置。pom.xml具体内容如下：
	
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
		<modelVersion>4.0.0</modelVersion>
		<groupId>com.spreada</groupId>
		<artifactId>zhconverter</artifactId>
		<version>1.0</version>
		<packaging>jar</packaging>
		<name>ZHConverter</name>
		<description>zh converter.</description>
	</project>

3.在lib文件夹下面 Shift+右键，选择在此处打开命令行，即弹出在该目录下的命令行。

4.在命令行输入一下安装代码：
	
	mvn install:install-file -Dfile=ZHConverter.jar -DgroupId=com.spreada -DartifactId=zhconverter -Dversion=1.0 -Dpackaging=jar

回车，可看到安装信息以及是否成功。成功以后，可以在本地仓库相应的文件夹下面找到jar文件。

5.在工程的pom.xml中添加依赖：

		<dependency>
			<groupId>com.spreada</groupId>
			<artifactId>zhconverter</artifactId>
			<version>1.0</version>
		</dependency>

即可正常使用该jar了。 ^_^
