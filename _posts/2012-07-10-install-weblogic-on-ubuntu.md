---
layout: post
title: ubuntu下安装weblogic
category: linux
tags: [linux, weblogic]
---


本文使用weblogic的bin文件进行安装。

1.首先赋予bin文件可执行的权限

	chmod a+x filename.bin

然后执行  

	./filename.bin -mode=console

此时出现安装步骤界面，根据需要进行选择索引或者下一步，一直进行即可。

2.找到安装过程中指定的中间件安装目录，本文路径为 root/Oracle/Middleware.
该目录下的文件有：

	[root@localhost Middleware]# ls
	coherence_3.7        jdk160_29  modules  oepe_12.1.1.0.1  patch_oepe101  registry.dat  user_projects  wlserver_12.1
	domain-registry.xml  logs       ocm.rsp  patch_ocp371     patch_wls1211  registry.xml  utils

进入上面目录wlserver_12.1：

	[root@localhost Middleware]# cd wlserver_12.1/
	[root@localhost wlserver_12.1]# ls
	common  endorsed  inventory  L10N  server  sip  uninstall\

进入common/bin目录：

	[root@localhost wlserver_12.1]# cd common/
	[root@localhost common]# ls
	bin  deployable-libraries  derby  eval  lib  nodemanager  quickstart  templates  wlst
	[root@localhost common]# cd bin
	[root@localhost bin]# ls
	commEnv.sh         config.sh  setPatchEnv.sh  startManagedWebLogic.sh  unpack.sh   wlscontrol.sh   wlst.sh
	config_builder.sh  pack.sh    startDerby.sh   stopDerby.sh             upgrade.sh  wlsifconfig.sh
	[root@localhost bin]# 

运行config.sh文件进行weblogic的配置（一般选择默认就可以）：

	[root@localhost bin]# sh config.sh

无法实例化 GUI, 默认进入控制台模式。


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	欢迎使用:
	-------------

	在创建和扩展域之间选择。根据您的选择,  配置向导将引导您完成生成新域或扩展现有域的步骤。

	 ->1|创建新的 WebLogic 域
	    |    在您的项目目录中创建 WebLogic 域。  

	   2|扩展现有的 WebLogic 域
	    |    使用此选项可以向现有域添加新组件以及修改配置设置。 


	输入要选择的索引号 或 [退出][下一步]>  

> 按回车(Enter)



	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	选择域源:
	-------------

	选择要从中创建域的源。可以通过 在所需的组件中选择或在现有域模板列表中选择来创建域。

	 ->1|选择 Weblogic Platform 组件
	    |    您可以选择希望在域中支持的 Weblogic 组件。 

	   2|选择定制模板
	    |    如果要使用现有模板, 请选择此选项。 此模板可以是使用模板构建器创建的定制模板。 


	输入要选择的索引号 或 [退出][上一步][下一步]> 

> 按回车(Enter)


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	应用程序模板选择:
	-------------------------

	    可用模板
	    |_____Basic WebLogic Server Domain - 12.1.1.0 [wlserver_12.1]x
	    |_____Basic WebLogic SIP Server Domain - 12.1.1.0 [wlserver_12.1] [2] 
	    |_____WebLogic Advanced Web Services for JAX-RPC Extension - 12.1.1.0 [wlserver_12.1] [3] 
	    |_____WebLogic Advanced Web Services for JAX-WS Extension - 12.1.1.0 [wlserver_12.1] [4] 


	输入与方括号中完全相同的数字以切换选择 或 [退出][上一步][下一步]> 

> 按回车(Enter)


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	编辑域信息:
	----------------

	    | Name |    Value    |
	   _|______|_____________|
	   1| *名称: | base_domain |


	输入以下内容的值 "名称" 或 [退出][上一步][下一步]> 
> 按回车(Enter)

注意：这里可以输入1然后编辑Value值。本文选择默认。


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	为此域选择目标域目录:
	-------------------------------

	    "目标位置" = [输入新值或使用默认值 "/root/Oracle/Middleware/user_projects/domains"]


	输入新值 目标位置 或 [退出][上一步][下一步]> 

> 输入  /root/Oracle/Middleware/test_projects/domains Enter

注意：这里可以选择project的路径，由于我已经有一个user_projects，所有我换了一个名称。直接输入目标位置就可以。


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	配置管理员用户名和口令:
	----------------------------------

	创建一个要分配到管理员角色的用户。 此用户是用于启动开发模式服务器的默认管理员。

	    |   Name   |                  Value                  |
	   _|__________|_________________________________________|
	   1|   *名称:   |                weblogic                 |
	   2|  *用户口令:  |                                         |
	   3| *确认用户口令: |                                         |
	   4|   说明:    | This user is the default administrator. |

	使用以上值或选择另一选项:
	    1 - 修改 "名称"
	    2 - 修改 "用户口令"
	    3 - 修改 "确认用户口令"
	    4 - 修改 "说明"


	    ** CFGFWK-60050:  User "weblogic" 的属性 "ConfirmUserPassword" 无效。
	    ** CFGFWK-64014:  该属性值是必需的。


	输入要选择的选项编号 或 [退出][上一步][下一步]> 

> 2 按回车(Enter)

注意：这里我选择2要更改口令为wls12345


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	配置管理员用户名和口令:
	----------------------------------

	创建一个要分配到管理员角色的用户。 此用户是用于启动开发模式服务器的默认管理员。

	    "*用户口令:" = []


	输入新值 *用户口令: 或 [退出][重置][接受]> 

> 这里输入密码，但是不会显示 ，然后按回车(Enter)



	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	配置管理员用户名和口令:
	----------------------------------

	创建一个要分配到管理员角色的用户。 此用户是用于启动开发模式服务器的默认管理员。

	    |   Name   |                  Value                  |
	   _|__________|_________________________________________|
	   1|   *名称:   |                weblogic                 |
	   2|  *用户口令:  |                ********                 |
	   3| *确认用户口令: |                                         |
	   4|   说明:    | This user is the default administrator. |

	使用以上值或选择另一选项:
	    1 - 修改 "名称"
	    2 - 修改 "用户口令"
	    3 - 修改 "确认用户口令"
	    4 - 修改 "说明"
	    5 - 放弃更改

	输入要选择的选项编号 或 [退出][上一步][下一步]>
> 3 按回车(Enter)

注意：这里输入3要确认密码


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	配置管理员用户名和口令:
	----------------------------------

	创建一个要分配到管理员角色的用户。 此用户是用于启动开发模式服务器的默认管理员。

	    "*确认用户口令:" = []


	输入新值 *确认用户口令: 或 [退出][重置][接受]> 这里输入确认口令，
>这里输入确认口令，按回车(Enter)


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	配置管理员用户名和口令:
	----------------------------------

	创建一个要分配到管理员角色的用户。 此用户是用于启动开发模式服务器的默认管理员。

	    |   Name   |                  Value                  |
	   _|__________|_________________________________________|
	   1|   *名称:   |                weblogic                 |
	   2|  *用户口令:  |                ********                 |
	   3| *确认用户口令: |                ********                 |
	   4|   说明:    | This user is the default administrator. |

	使用以上值或选择另一选项:
	    1 - 修改 "名称"
	    2 - 修改 "用户口令"
	    3 - 修改 "确认用户口令"
	    4 - 修改 "说明"
	    5 - 放弃更改

	以下几项都默认值就可以：


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	域模式配置:
	----------------

	为此域启用开发或生产模式。 

	 ->1|开发模式

	   2|生产模式


	输入要选择的索引号 或 [退出][上一步][下一步]>

> 按回车(Enter)


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	Java SDK 选择:
	----------------

	 ->1|Sun SDK 1.6.0_29 @ /root/Oracle/Middleware/jdk160_29
	   2|其他 Java SDK


	输入要选择的索引号 或 [退出][上一步][下一步]>
> 按回车(Enter)


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	选择可选配置:
	-------------------

	   1|管理服务器 [ ]
	   2|受管服务器, 集群和计算机 [ ]
	   3|RDBMS 安全存储 [ ]

	输入要选择的索引号 或 [退出][上一步][下一步]>
> 按回车(Enter)


	<----------------------------------------------------------------- Fusion Middleware 配置向导 ----------------------------------------------------------------->

	正在创建域...

	0%          25%          50%          75%          100%
	[------------|------------|------------|------------]
	[***************************************************]


	**** 域创建成功! ****

	[root@localhost bin]# 

下面启动Weblogic服务：
由于在SSH控制台直接启动的话，控制台没没法使用。有一下两种方法可以解决该问题：
首先进入Oracle/Middleware/user_projects/domains/base_domain/目录,执行

	vi startWeblogic.sh 

编辑启动文件，添加如下两行：	
	WLS_USER="weblogic"
	WLS_PW="wls12345"
保存退出，如下所示。

	[root@localhost base_domain]# vi startWebLogic.sh 
	#!/bin/sh

	# WARNING: This file is created by the Configuration Wizard.
	# Any changes to this script may be lost when adding extensions to this configuration.

	DOMAIN_HOME="/root/Oracle/Middleware/user_projects/domains/base_domain"

	${DOMAIN_HOME}/bin/startWebLogic.sh $*
	WLS_USER="weblogic"
	WLS_PW="wls12345"

：wq（保存退出）


1.运行命令 ./startWeblogic.sh,启动之后 ctrl+z，然后运行命令bg指定在后台运行。

	[root@localhost base_domain]# ./startWebLogic.sh 
	ctrl+z
	[root@localhost base_domain]# bg

2.运行命令nohup ./startWeblogic.sh &，然后ctrl+c断开ssh，不会关闭Weblogic。

	[root@localhost base_domain]# nohup ./startWebLogic.sh &
	[1] 3273
	[root@localhost base_domain]# nohup: appending output to “nohup.out”

	[root@localhost base_domain]# 

