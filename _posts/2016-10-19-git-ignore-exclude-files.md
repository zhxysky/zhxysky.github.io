---
layout: post
title: git添加忽略文件的方法
category: git
tags: [git]
---

在Git版本控制里面，有一些是我们不想提交的文件，处理这类文件最常用的方法就是使用.gitignore文件，即把不想被控制的文件目录写到这个文件就可以了。如下：
	
	target/
	bin/
	*.class

	.classpath
	.project
	.settings

这样这些目录就不会被控制。

那如果有一些文件也不像让它出现在.gitignore文件里面，该如何处理呢？
在.git/info目录下，有一个exclude文件，把不想提交的文件写到该文件里面，就不会出现在版本控制里了。这个配置只会在单个项目下生效。

还有一种就是方法是全局配置，在.gitconfig文件里的[core]模块下配置excludsfile，如下：

	[core]
	excludesfile = filepath

也可是使用命令行配置：

	git config --global core.excludesfile filepath


