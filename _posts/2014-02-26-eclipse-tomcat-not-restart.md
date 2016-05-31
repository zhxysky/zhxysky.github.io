---
layout: post
title: Eclipse下修改java文件不重启tomcat
category: IDE
tags: [java, IDE]
---


每次在Spring Tool Suite里面修改完java类之后，tomcat都会自动重启，并且报错，浪费不少时间，觉得要解决一下这个问题。

搜了不少资料，都要修改tomcat各种配置，还有工程配置，很是麻烦。

后来在一个论坛里找到一种很简单的方法，记录如下：

打开sts中Servers工程里对应你使用的tomcat文件夹，找到server.xml文件 打开 拖到最后 修改 
	
	<Context docBase="test" path="/test" reloadable="false" source="org.eclipse.jst.jee.server:t8"/>

把reloadable值改为false，，然后启动tomcat的时候选择debug模式，即可看到随时修改的java文件随时生效。

