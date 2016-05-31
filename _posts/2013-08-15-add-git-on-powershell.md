---
layout: post
title: 在PowerShell下安装git
category: git
tags: [git]
---

最近在学习使用git，发现从GitHub的界面工具中右击一个Repository，选择Open in git Shell打开的命令行跟git bash不是一个东西，但本人却比较喜欢那个工具。想直接打开却不知怎么办。后来发现如此打开的命令行是一个叫posh~git的东西，其路径中有个WindowsPowerShell目录，至此才发现有一个叫powershell.exe的东西，双击打开是一个命令行工具，运行git命令却报错，说git不是内部命令之类的。搜索发现，原来需要在PowerShell下安装post~git。
在post~git[项目主页](https://github.com/dahlbyk/posh-git)，我们可以看到有明确的安装步骤，在此简要记录，以供后续查阅。


1.确认你的PowerShell是否是2.0或者以上版本。你可以在PowerShell下使用命令 $PSVersionTable 或者 Get-Host查看。如图：

![查看PowerShell版面](/images/powershell/psversion.jpg "PSVersionTable")


![查看PowerShell版面](/images/powershell/getHost.jpg "Get-Host")


2.确认允许执行脚本。使用命令 Get-ExecutionPolicy 查看，如果结果是 RemoteSigned 或者  Unrestricted ，则表示允许执行脚本。如果不允许执行，那么就以管理员身份运行PowerShell，然后执行命名 

	 Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Confirm 

3.试一下在PoserShell下git命令是否可以运行，如果不能运行，就像开头所说的报错，则需要添加git的环境变量了。把 %ProgramFiles(x86)%\Git\cmd 添加到系统变量path的最后（注意跟前面的内容用“;”隔开）。我本机添加的是：;C:\Program Files (x86)\Git\cmd  。然后可以重启PowerShell试试git命令，应该会出现git命令的一些提示信息。

4.然后把posh-git仓库clone到本地：git@github.com:dahlbyk/posh-git.git

5.进入posh-git目录，运行   .\install.ps1  命令。

6.安装完毕。


现在再使用PowerShell运行git命令，完全没有问题了。只是，打开PowerShell命令行的时候，开头出现“警告: Could not find ssh-agent”的提示信息，原来是找不到ssh-agent.exe。该文件在C:\Program Files (x86)\Git\bin下面，所以我们就像刚才添加git环境变量一样把该路径添加到path的最后保存，再重启PowerShell就没有这个提示信息了。


现在终于可以简单明了的在PowerShell下使用git命令了。