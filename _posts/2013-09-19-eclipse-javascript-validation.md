---
layout: post
title: eclipse取消对js文件的验证
category: IDE
tags: [eclipse,javascript]
---

在使用ecpise开发的时候，经常遇到javascript文件报错，而且编译很慢。其实我们一般不需要验证js文件，那么怎么能取消对js的验证呢？步骤如下：

1.点击eclipse的工具栏Window ---> Preferences ---> JavaScript ---> Validator ---> Errors/Warnings,把Enable JavaScript semantic validation前面的√去掉，然后保存。

2.打开工程目录里面的.project文件，把里面的关于JavaScriptValidator的部分注释掉：如下

		<!-- 		<buildCommand> -->
		<!-- 			<name>org.eclipse.wst.jsdt.core.javascriptValidator</name> -->
		<!-- 			<arguments> -->
		<!-- 			</arguments> -->
		<!-- 		</buildCommand> -->

和

		<!-- 		<nature>org.eclipse.wst.jsdt.core.jsNature</nature> -->

3.复制出报错的js文件，删掉eclipse中的js文件，然后在将js文件复制进来，就OK了。