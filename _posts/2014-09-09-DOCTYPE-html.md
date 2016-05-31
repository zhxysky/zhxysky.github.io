---
layout: post
title: html <!DOCTYPE>标签
category: html
tags: [html,doctype]
---

1.最近发现页面在ie下显示的时候总是有各种各样的问题，其中一点就是在一个IE浏览器模式下面显示正常，在另外一个浏览器模式下不正常（一般IE 9下不正常）。一直不知道什么原因，很是困扰。通过各种查找，发现页面DOCTYPE标签前面存在一些内容，还有空行等，这就是问题所在了。原来html页面的DOCTYPE前面是不允许有任何内容的。修改页面去掉DOCTYPE前面可能会显示的内容之后，发现显示正常。

2.还有一个问题就是有些页面在IE下显示的时候，浏览器模式和文档模式不匹配，也会导致显示不正常。后来发现，一般不匹配的页面，都没有写DOCTYPE标签，原来这是最终的原因，加上DOCTYPE标签，果然显示正常。


注：<!DOCTYPE>声明没有结束标签。<!DOCTYPE> 声明对大小写不敏感。

3.常用的DOCTYPE声明：

##### HTML 5
	
	<!DOCTYPE html>

##### HTML 4.01 Strict

该 DTD 包含所有 HTML 元素和属性，但不包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。

	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

##### HTML 4.01 Transitional

该 DTD 包含所有 HTML 元素和属性，包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。

	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

##### HTML 4.01 Frameset

该 DTD 等同于 HTML 4.01 Transitional，但允许框架集内容。

	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">

##### XHTML 1.0 Strict

该 DTD 包含所有 HTML 元素和属性，但不包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。必须以格式正确的 XML 来编写标记。
	
	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

##### XHTML 1.0 Transitional

该 DTD 包含所有 HTML 元素和属性，包括展示性的和弃用的元素（比如 font）。不允许框架集（Framesets）。必须以格式正确的 XML 来编写标记。
	
	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

##### XHTML 1.0 Frameset

该 DTD 等同于 XHTML 1.0 Transitional，但允许框架集内容。

	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">

##### XHTML 1.1

该 DTD 等同于 XHTML 1.0 Strict，但允许添加模型（例如提供对东亚语系的 ruby 支持）。

	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

