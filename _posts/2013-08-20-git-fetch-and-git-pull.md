---
layout: post
title: git fetch 和 git pull
category: git
tags: [git]
---

git fetch和git pull都是从远程分支获取最新版本到本地，区别就是git pull会自动与本地代码合并。

#####使用git fetch获取master分支最新内容并合并到本地
	
	###获取更新
	git fetch origin master
	###比较本地分支与origin/master 分支的差别
	git log -p master..origin/master 
	###合并
	git merge origin/master

#####git pull获取最新内容并合并到本地
	
	git pull origin master

相当于git fetch和git merge，但在实际中git fetch更为安全一些，因为我们可以根据实际情况进行合并。