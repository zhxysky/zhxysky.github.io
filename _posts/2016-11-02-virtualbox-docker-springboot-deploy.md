---
layout: post
title: 虚拟机Virtualbox环境下Docker部署springboot项目
category: docker
tags: [docker]
---


现有一台安装了docer的虚拟机，系统为Ubuntu 16.04 LTS (GNU/Linux 4.4.0-45-generic x86_64)。docker版本如下：

	zhxy@zhxyu:~$ docker --version
	Docker version 1.12.1, build 23cf638


想使用docker部署一个最简单的springboot应用。[应用代码点击此处](http://www.baidu.com)

首先，在应用根目录下添加Dockerfile文件，内容如下：


	FROM maven:3.3.3
	MAINTAINER zhxysky

	##此处端口设置了跟应用tomcat启动端口相同（tomcat启动端口见application.properties文件）
	EXPOSE 8081

	##把要运行的jar包放到一个目录下面（随意的目录，便于运行时目录填写）。 后续可以把生成jar包的过程也放到该文件里面
	WORKDIR temp
	##把打好的jar包添加到temp目录
	ADD target/docker-demo-spring-boot-0.0.1-SNAPSHOT.jar /temp/docker-demo-spring-boot-0.0.1-SNAPSHOT.jar

	## 运行 java -jar 命令，启动服务
	CMD ["java","-jar","/temp/docker-demo-spring-boot-0.0.1-SNAPSHOT.jar"]


之后，在应用根目录下，运行maven命令，对应用编译，打包

	mvn clean compile package


然后，把整个应用所有文件，**包括target文件夹**，一起放到虚拟机上。此处放置的位置是

	/home/zhxy

在该位置执行docker build命令，生成docker镜像：

	docker build docker-demo-spring-boot/ -t docker-demo:6.0

其中，build后面跟应用的目录，-t表示tag，后面跟镜像名称和标签
结果如下：
	
	zhxy@zhxyu:~$ docker build docker-demo-spring-boot/ -t docker-demo:6.0
	Sending build context to Docker daemon 13.61 MB
	Step 1 : FROM maven:3.3.3
	 ---> baadc9c8b0ce
	Step 2 : MAINTAINER zhxysky
	 ---> Using cache
	 ---> 789dcb4d9651
	Step 3 : EXPOSE 8081
	 ---> Using cache
	 ---> 9ddc782cd61d
	Step 4 : WORKDIR temp
	 ---> Using cache
	 ---> 6b56fa334530
	Step 5 : ADD target/docker-demo-spring-boot-0.0.1-SNAPSHOT.jar /temp/docker-demo-spring-boot-0.0.1-SNAPSHOT.jar
	 ---> Using cache
	 ---> 0d80bc7a57ae
	Step 6 : CMD java -jar /temp/docker-demo-spring-boot-0.0.1-SNAPSHOT.jar
	 ---> Using cache
	 ---> 0e1ce48770a7
	Successfully built 0e1ce48770a7


使用docker images查看生成的镜像：

	zhxy@zhxyu:~$ docker images
	REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
	docker-demo            5.0                 0e1ce48770a7        47 minutes ago      692.1 MB
	docker-demo            6.0                 0e1ce48770a7        47 minutes ago      692.1 MB

其中5.0为我之前生成的镜像。

然后就可以运行该镜像了。运行命令为：

	docker run -p 8081:8081 -t docker-demo:6.0

-p表示发布一个容器的端口，映射到主机。此处映射8081端口到虚拟机的8081端口。

	zhxy@zhxyu:~$ docker run -p 8081:8081 -t docker-demo:6.0
	docker: Error response from daemon: driver failed programming external connectivity on endpoint loving_shirley (81a9aee1305e4ba6995612a508f16e8a4cf148414fb2cb1bd1526977fad0a5b1): Bind for 0.0.0.0:8081 failed: port is already allocated.

我在运行该命令的时候，提示了错误：8081端口已经被占用，那是因为我已经启动docker-demo:5.0镜像的一个容器。使用docker ps -a命令可查询已经启动的容器：

	zhxy@zhxyu:~$ docker ps -a
	CONTAINER ID        IMAGE                  COMMAND                  CREATED              STATUS                   PORTS                    NAMES
	d7e299e9bb80        docker-demo:6.0        "java -jar /temp/dock"   About a minute ago   Created                                           loving_shirley
	f97971ee5526        docker-demo:5.0        "java -jar /temp/dock"   59 minutes ago       Up 59 minutes            0.0.0.0:8081->8081/tcp   admiring_panini

现在先停止占用8081端口的容器，然后再启动docker-demo:6.0对应的那个容器：

	zhxy@zhxyu:~$ docker stop f97971ee5526
	f97971ee5526
	zhxy@zhxyu:~$ docker start d7e299e9bb80
	d7e299e9bb80
	zhxy@zhxyu:~$ 

stop和start后面跟的都是容器的ID。

现在用虚拟机外用浏览器访问http://127.0.0.1:8081/，可以看到返回内容
	
	Hello! Docker!

运行成功。 ^_^

