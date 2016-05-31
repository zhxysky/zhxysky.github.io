---
layout: post
title: git撤销合并操作
category: git
tags: [git, revert, merge]
---

在使用git进行开发的过程中，我们经常使用多个分支开发不同的任务，然后进行合并。那么就会出现这种现象：分支br1合并到分支br2，br2在此基础上又有提交，此时发现br1的提交有问题，需要在br2中撤销br1的合并，那么该如何进行呢？

###撤销合并操作
在br2下使用git log可以看到提交的记录，其中有一个就是merge br1的记录。如下：

![br2 log](/images/git/br2-log.jpg "br2 log")

现在我们要撤销br2中对br1的合并，即撤销
	
	commit 2c6d59bafa5ac1ea8ffaf7e08f5eb2138bfe16bd
	Merge: 992d00a 93df2a3
	Author: zhxysky <zhxysky@qq.com>
	Date:   Thu Oct 30 16:23:04 2014 +0800

    Merge branch 'br1' into br2

    Conflicts:
        src/main/java/com/zhxy/controller/IndexAction.java

使用git revert 命令可以满足我们的需求。git revert命令格式为

	git revert -m parentNum mergerCommitId

其中parentNum是该合并操作合并的两个分支的序号，从1开始，在log的Merge一行可以看到两个父分支的id。

mergerCommitId是指当前操作的id。

在此处我们需要执行

	git revert -m 1 2c6d59ba

由于这两个分支修改的地方有冲突，所以合并和撤销过程中需要自己手动处理冲突。

执行结果如下：

![br2 revert](/images/git/br2-revert-merge.jpg "br2 revert merge")

使用git log可以看到多出一条撤销合并操作的记录：

![br2 revert result](/images/git/br2-revert-result.jpg "br2 revert result")


###撤销被撤销的合并操作
现在我们发现br1分支的操作还是需要留的，刚才的撤销操作太鲁莽，需要恢复，该怎么做呢？

答案是使用revert操作

![br2 revert](/images/git/revert-revert-merge.jpg "br2 revert revert")


参考：[http://git-scm.com/blog/2010/03/02/undoing-merges.html](http://git-scm.com/blog/2010/03/02/undoing-merges.html)
