---
layout: post
title: shiro框架多个注解问题
category: 权限
tags: [权限, shiro]
---


Shiro的认证注解处理是有内定的处理顺序的，如果有个多个注解的话，前面的通过了会继续检查后面的，若不通过则直接返回，处理顺序依次为（与实际声明顺序无关）：
	
	RequiresRoles
	RequiresPermissions
	RequiresAuthentication
	RequiresUser
	RequiresGuest

例如：你同时声明了RequiresRoles和RequiresPermissions，那就要求拥有此角色的同时还得拥有相应的权限。


@RequiresRoles和RequiresPermissions注解可以使用多个roles或者permissions，默认逻辑为AND，也就是具备所有才能访问。也可以通过logical修改逻辑关系，示例如下：

属于user角色
	
	@RequiresRoles("user")

必须同时属于user和admin角色

	@RequiresRoles({"user","admin"})

属于user或者admin之一。

	@RequiresRoles(value={"user","admin"},logical=Logical.OR)

除此之外还有以下类似权限的注解：

	@RequiresPermissions("index:hello")
	@RequiresPermissions({"index:hello","index:world"})
	@RequiresPermissions(value={"index:hello","index:world"},logical=Logical.OR)



参考：<http://my.oschina.net/myaniu/blog/137205>