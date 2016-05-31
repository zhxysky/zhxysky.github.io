---
layout: post
title: MySQL大数据分页查询性能优化
category: MySQL
tags: [mysql, MySQL Workbeanch]
---


MySQL中limit关键字用于数据分页查询，使用方法为
	
	limit ?,?

其中第一个?表示要查询的数据的起始位置，第二个‘ ?’ 表示页面大小。


一、使用limit start,pageSize 查询数据

我们使用MySQL进行分页查询的时候都会使用如下的方式：

	select * from message limit 0,20 order by id desc;

这样确实是可以查询到数据，但是如果数据量特别大的时候，比如有1000000数据的时候，这样查询可能比较慢了。现在有message表有一百万数据，一些查询语句的情况如下图：

	14:47:06	select * from message limit 500000,20	20 row(s) returned	0.219 sec / 0.000 sec

	14:47:06	select * from message limit 800000,20	20 row(s) returned	0.359 sec / 0.000 sec

	14:47:07	select * from message limit 1000000,20	20 row(s) returned	0.436 sec / 0.000 sec


可以看出随着起始记录的增加，时间也随着增大，这说明limit的查询时间跟起始位置有很大关系。而且数据量更多的时间，时间还会更长。所以limit语句对于查询数据量很大的表效果并不是很好。

可是对于数据量很大的表进行查询是避免不了的问题，该如何解决呢？

二、对limit分页进行优化

由于id字段是主键，包含了默认的主键索引，那么我们利用覆盖索引来加速分页查询。

我们查询1000000之后的一页的id，如下：

	15:07:07	select id from message limit 1000000,20	20 row(s) returned	0.266 sec / 0.000 sec

可是看到时间缩短了将近一半。如果我们要查询所有列，可以使用id>=的形式，还可以利用join，情况如下：

（1） 利用id>=的格式
	
	15:14:10	select * from message where id >=(select id from message limit 500000,1) limit 20	20 row(s) returned	0.125 sec / 0.000 sec

	15:14:10	select * from message where id >=(select id from message limit 800000,1) limit 20	20 row(s) returned	0.202 sec / 0.000 sec

	15:14:10	select * from message where id >=(select id from message limit 1000000,1) limit 20	20 row(s) returned	0.250 sec / 0.000 sec

（2） join形式
	
	15:18:37	select * from message m join(select id from message limit 500000,20) a on m.id=a.id LIMIT 0, 300	20 row(s) returned	0.140 sec / 0.000 sec

	15:18:37	select * from message m join(select id from message limit 800000,20) a on m.id=a.id LIMIT 0, 300	20 row(s) returned	0.203 sec / 0.000 sec

	15:18:37	select * from message m join(select id from message limit 1000000,20) a on m.id=a.id LIMIT 0, 300	20 row(s) returned	0.249 sec / 0.000 sec


可以看出两种形式效率上差不多，但是相比于limit ?,?的形式，随时数据量的增多，效率提升的会更加明显。

参考文章链接：[http://www.cnblogs.com/lyroge/p/3837886.html](http://www.cnblogs.com/lyroge/p/3837886.html)

