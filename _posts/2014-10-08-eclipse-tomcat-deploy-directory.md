---
layout: post
title: eclipse指定工程部署目录
category: eclipse
tags: [eclipse,IDE]
---

在Spring source tool suite下使用tomcat部署工程的时候，总是部署到一个比较复杂的路径下面，比如

	D:\workspace\.metadata\.plugins\org.eclipse.wst.server.core\tmp0

这是因为我们在添加tomcat服务的时候使用了他们默认的部署路径，如下图所示：

![eclipse下tomcat默认部署路径](/images/eclipse/eclipse-deploy-default.jpg "default deploy directory")

可以看到这个配置是不能更改的。那么如何更改路径呢？其实eclipse在新建一个Server的时候是可以更改的，就是在添加一个新的server切没有添加工程到Server的时候。双击该Server打开配置页面，可以看到：

![eclipse下tomcat默认部署路径](/images/eclipse/eclipse-deploy-new.jpg "new deploy directory")

可以看到有三个选项可以选择：

1.使用eclipse工作区目录，即：D:\workspace\.metadata\.plugins\org.eclipse.wst.server.core\tmp0

2.使用tomcat的安装目录，即：D:\tools\apache-tomcat-6.0.18

3.使用自定义路径，各自选择即可。

接着通过修改Deploy path来设置工程部署的目录。本文中上面选择了第二种即tomcat安装目录，所以此处选择tomcat下的webapps。如下：

![eclipse下选择tomcat目录为默认部署路径](/images/eclipse/tomcat-deploy.jpg "tomcat deploy directory")


还有一种修改这些信息的方法就是修改VM arguments,步骤为:eclipse菜单栏Run---> Run Configurations,在弹出的窗口中左侧选择Apache Tomcat，右侧选择Arguments选项卡，可以看到下面有VM arguments配置，修改相应的信息即可。如下：

![eclipse下Vm arguments配置](/images/eclipse/run-configurations.jpg "Run Configurations")

本文使用的是Spring Tool Suite而不是eclipse进行截图的，不过步骤是一样的，不影响。

参考文章链接：[http://blog.csdn.net/tfy1332/article/details/22155425](http://blog.csdn.net/tfy1332/article/details/22155425)

