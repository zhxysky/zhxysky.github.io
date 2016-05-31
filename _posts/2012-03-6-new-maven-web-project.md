---
layout: post
title: 新建maven web工程
category: maven
tags: [maven, java]
---



一、新建maven project。


1.选择左上角 File ---> new ---> other --> maven Project 。如下图：

![maven project](/images/maven/1.jpg "maven project")
	
	
2.选择next，进入下一步，选择工作区。

![maven project](/images/maven/2.jpg "maven project")
	
	
3.选择next。出现选择maven工程类型的界面。此处我们想要一个web工程，所以选择maven-archetype-webapp。如下：
	
	
![maven project](/images/maven/3.jpg "maven project")


4.选择next。填写一下内容。Group Id相当于项目组织唯一的标识符，实际对应JAVA的包的结构，是main目录里java的目录结构。Archetype 
Id就是项目的唯一的标识符，实际对应项目的名称，就是项目根目录的名称。此处填写的如下：

![maven project](/images/maven/4.jpg "maven project")


5.点击Finish，可以看到我们的工程已经建好如下：

![maven project](/images/maven/5.jpg "maven project")

6.由于工程里面目录还不全，需要我们手动补充一下。

（1）在src下面的main目录上右击，新建一个Foler，名为java，形成 src/main/java目录结构。我们在此目录下编写我们的代码。

（2）在src/main/java下建立我们的package，形成我们的包目录。

结果如下图所示：

![maven project](/images/maven/6.jpg "maven project")

至此，我们的maven工程建立起来。