---
layout: post
title: ubuntu下安装nginx
category: linux
tags: [linux, ubuntu, nginx]
---


最初要安装nginx，发现configure使用不了，所以先安装configure


安装zlib-devel,要更换软件源(http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8)，然后在命令行执行一下命令：
	
	apt-get update
	apt-get upgrade --show-upgraded
	apt-get install libssl-dev  
	(# sudo apt-get install openssl
	# sudo apt-get install libssl-dev)
	/etc/apt/sources.list


下面开始安装nginx：

	root@ubuntu:/opt# cd nginx-1.2.4/
	root@ubuntu:/opt/nginx-1.2.4# ./configure --prefix=/opt/nginx --with-pcre=/opt/pcre-8.31 --with-http_stub_status_module --with-http_gzip_static_module
	make
	make install


进入/opt/nginx，下面有四个目录 conf，html，logs，sbin
进入sbin，运行命令 
	
	nginx

即可启动nginx。浏览器访问ip可以看到welcome to nginx！ 


也可用一下命令启动nginx：

	nginx -c /opt/nginx/conf/nginx.conf

或者在nginx目录下运行： 
	
	./sbin/nginx