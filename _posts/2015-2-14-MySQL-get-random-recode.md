---
layout: post
title: MySQL随机获取记录的方法
category: MySQL
tags: [mysql, MySQL Workbeanch]
---

我们想随机从mysql表中获取一条或者多条记录，该如何操作呢？

一、使用rand()方法

	select * from t order by rand() limit 1;

rand()方法为每一行记录产生一个随机值；order by将会对整个表的所有记录进行排序。这种方法在表比较小的情况下效果很好，但是在数据量比较多的情况下会很慢，因为mysql需要对所有表中所有的记录进行排序。


二、使用inner join

首先获取当前表最大的id，使用max(id)获取，然后乘以一个随机数rand()，得到一个随机的id：
	
	rand()*max(id)

使用函数ceil(rand()*max(id))进行取整。也可以使用其他函数：
round(x)： 四舍五入
floor(x)：舍去小数
ceil(x) 和ceiling(x) ： 小数部分有值就进一，向上去整。

获取随机id的sql为
	
	select ceil(rand()*max(id)) from t;

但是这么做会失去了max函数的优势，可是使用子句：

	select ceil(rand()*(select max(id) from t)) from t;

这就就解决了性能问题。

然后我们就根据当前id获取记录
	
	select * from t where id=(select ceil(rand()*(select max(id) from t)) from t);

注意：这是最明显的做法，不过也是错误的做法。因为where子句中的select语句会对外围select获取的每一个记录执行一次。

我们需要确定一个随机的id

	select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;

这样就能每次获取一个随机的记录了。

如果我们想获取多条记录，可以把limt 1改成limt n，但是这样获取的多条记录都是连续的，如果想获取不连续的多条记录，可以结合程序执行多次sql语句；或者根据程序使用max(id)生成多个随机的id，然后直接根据id获取记录。




参考文章链接：[http://jan.kneschke.de/projects/mysql/order-by-rand/](http://jan.kneschke.de/projects/mysql/order-by-rand/)

