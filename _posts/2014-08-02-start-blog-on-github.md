---
layout: post
title: 搭建github博客
category: github
tags: [github, blog]
---

github工作过程：提交特定格式的文章，通过github的jekyll转化成网页形式显示出来

一、创建项目

1.在github上新建一个repository作为博客主目录，假定叫：tblog

2.使用本地github工具clone下tblog目录

3.使用git shell工具进入tblog目录

4.在tblog目录下面创建一个没有父节点的分支gh-pages： 

    $ git checkout --orphan gh-pages

以后所有动作都在该分支下完成

注：以下创建文件和目录的过程可以使用命令行或者直接在目录里进行

二、创建配置文件

1.在根目录下（tblog）创建一个名为_config.
yml的文本文件，作为jekyll的配置文件，在里面加入如下内容：

	baseurl: /tblog


三、创建文件模板

1.在根目录(tblog)下创建一个_layout目录，用于存放模板： $ mkdir _layouts

2.进入该目录，创建一个default.html文件，作为blog的默认模板，并在该文件中添加一下内容：

{% raw %}
    <!DOCTYPE html>
    <html>
	<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>{{ page.title }}</title>
    </head>
    <body>
    {{ content }}
    </body>
    </html>
{% endraw %}

四、创建文件

1.在根目录下创建一个_posts目录，用户存放文章

	$ mkdir _posts

2.进入该目录，创建一个普通的文本文件，文件名假定为：2014-08-01-start.html(注意，文件名必须为"年-月-日-文章标题.后缀名"的格式。如果网页代码采用html格式，后缀名为html；如果采用markdown格式，后缀名为md。)
在文件中添加以下内容：

{% raw %}
    ---
    layout: default
    title: 你好，世界
    ---
    <h2>{{ page.title }}</h2>
    <p>我的第一篇文章</p>
    <p>{{ page.date | date_to_string }}</p>
{% endraw %}

五、创建首页

在根目录下面创建一个index.html文件

	{% raw %}
	---
	layout: default
	title: 我的Blog
	---
	<h2>{{ page.title }}</h2>
	<p>最新文章</p>
	<ul>
	　　{% for post in site.posts %}
	　　　　<li>{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
	　　{% endfor %}
	</ul>
	{% endraw %}


六、发布文章

1.先把内容加入本地库
 	
	$ git add .
	$ git commit -m "first post"

 	由于github上已经存在该库，所以不用创建新库，直接提交代码

	$ git push origin gh-pages

七、访问博客：http://usernamne.github.io/tblog/  其中usernane换成自己的用户名


参考：[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)