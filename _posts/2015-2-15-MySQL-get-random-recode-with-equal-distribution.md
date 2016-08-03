---
layout: post
title: MySQL随机获取记录id平均分布
category: MySQL
tags: [mysql, MySQL Workbeanch]
---

随机获取记录的方式请查看：[MySQL随机获取记录的方法](http://zhxysky.com/mysql/2015/02/14/MySQL-get-random-recode.html)

为了使随机获取各个id的概率差不多，可以采用映射表方式：

1.新建一个表t，插入几条记录：

	CREATE TABLE `t` (
	  `id` SERIAL,
	  `random` varchar(60) DEFAULT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;

写一个存储过程插入数据：
	
		delimiter $$
		delete from t;
		drop procedure if exists init;
		create procedure init(IN count INT)
		begin
		declare i int default 1;
		loop1: while i <= count do 
			insert into t values(null,md5(rand()));
			set i  = i+1;
		end  while loop1 ;
		end $$
		delimiter ;

把以上命令直接贴到命令行：

		mysql> delimiter $$
		mysql> delete from t;
		    -> drop procedure if exists init;
		    -> create procedure init(IN count INT)
		    -> begin
		    -> declare i int default 1;
		    -> loop1: while i <= count do
		    -> insert into t values(null,md5(rand()));
		    -> set i  = i+1;
		    -> end  while loop1 ;
		    -> end $$
		Query OK, 0 rows affected (0.00 sec)

		Query OK, 0 rows affected (0.00 sec)

		Query OK, 0 rows affected (0.00 sec)

		mysql> delimiter ;
		mysql>
		mysql>

存储过程init有一个参数count，用来表示你要插入几条记录。其中，truncate不仅可以把表中的数据全部删除，而且可以重新定位自增的字段，使id从1开始。concat函数用来连接字符串。

执行过程：
		
		mysql> call init(5);
		Query OK, 1 row affected (0.03 sec)

		mysql> select * from t;
		+----+----------------------------------+
		| id | random                           |
		+----+----------------------------------+
		|  1 | ab15a38895845460b0be2979e84ca7aa |
		|  2 | 9119cee81bbe2310ae86b029da743a12 |
		|  3 | 901a98b3ddfbff06e0e3506cff89dc69 |
		|  4 | 5a8a4fb71e2862b80411e7c41915e4a8 |
		|  5 | 0018bcbaefff855215645ef78ba3a73a |
		+----+----------------------------------+
		5 rows in set (0.00 sec)

		mysql>


2.新建映射表tm，row_id列表示行号，从1开始顺序记录，t_id代表t表的id，把原有的记录id加入映射表。

	create table tm(
	row_id int not null primary key,
	t_id int not null 
	);

然后把t表现有记录插入到tm表。

	mysql> set @id=0;
	Query OK, 0 rows affected (0.00 sec)

	mysql> insert into tm select @id:= @id+1, id from t;
	Query OK, 5 rows affected (0.01 sec)
	Records: 5  Duplicates: 0  Warnings: 0

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |    1 |
	|      2 |    2 |
	|      3 |    3 |
	|      4 |    4 |
	|      5 |    5 |
	+--------+------+
	5 rows in set (0.00 sec)

	mysql>

只把现有数据加入tm表还不够，还需要在t表更新的时候同步更新tm表。

（1）t表插入新数据要同步到tm表

	delimiter $$
	drop trigger if exists insert_sync$$
	create trigger insert_sync 
	after insert on t for each row
	begin
		declare m int unsigned default 1;
		select max(row_id)+1 from tm into m;
		select ifnull(m,1) into m;
		insert into tm(row_id,t_id) values(m,NEW.id);
	end $$
	delimiter ;


	mysql> delimiter $$
	mysql> drop trigger if exists insert_sync$$
	Query OK, 0 rows affected, 1 warning (0.00 sec)

	mysql> create trigger insert_sync
	    -> after insert on t for each row
	    -> begin
	    -> declare m int unsigned default 1;
	    -> select max(row_id)+1 from tm into m;
	    -> select ifnull(m,1) into m;
	    -> insert into tm(row_id,t_id) values(m,NEW.id);
	    -> end $$
	Query OK, 0 rows affected (0.02 sec)

	mysql> delimiter ;
	mysql> insert into t values(null, md5(rand()));
	Query OK, 1 row affected (0.01 sec)

	mysql> select * from t;
	+----+----------------------------------+
	| id | random                           |
	+----+----------------------------------+
	|  1 | ab15a38895845460b0be2979e84ca7aa |
	|  2 | 9119cee81bbe2310ae86b029da743a12 |
	|  3 | 901a98b3ddfbff06e0e3506cff89dc69 |
	|  4 | 5a8a4fb71e2862b80411e7c41915e4a8 |
	|  5 | 0018bcbaefff855215645ef78ba3a73a |
	|  6 | 01bed73a87b18624746cb06dfdf54b38 |
	+----+----------------------------------+
	6 rows in set (0.00 sec)

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |    1 |
	|      2 |    2 |
	|      3 |    3 |
	|      4 |    4 |
	|      5 |    5 |
	|      6 |    6 |
	+--------+------+
	6 rows in set (0.00 sec)

	mysql>

可以看到插入t表新记录之后，tm表也同步添加了一条记录。

（2）t表更新记录id之后，修改对应的tm表的t_id。

	delimiter $$
	drop trigger if exists update_sync$$
	create trigger update_sync
	after update on t  for each row
	begin
		update tm set t_id=NEW.id where t_id=OLD.id;
	end$$
	delimiter ;

执行上述代码：
	
	mysql> delimiter $$
	mysql> drop trigger if exists update_sync$$
	Query OK, 0 rows affected, 1 warning (0.00 sec)

	mysql> create trigger update_sync
	    -> after update on t  for each row
	    -> begin
	    -> update tm set t_id=NEW.id where t_id=OLD.id;
	    -> end$$
	Query OK, 0 rows affected (0.04 sec)

	mysql> delimiter ;
	mysql>

首先查看tm表记录：
	
	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |   39 |
	|      2 |   41 |
	|      3 |   42 |
	|      4 |   43 |
	|      5 |   44 |
	...
	...

现在我们把t表id=41的记录改成id=40,并查看tm表结果：

	mysql> update t set id=40 where id=41;
	Query OK, 1 row affected (0.01 sec)
	Rows matched: 1  Changed: 1  Warnings: 0

	mysql> select * from t;
	+-----+----------------------------------+
	| id  | random                           |
	+-----+----------------------------------+
	|  39 | 1f2339e7e041815475e10da7bfec15ed |
	|  40 | f26b691d18b002cd663ec39b78d858cb |
	|  42 | c7e5cf4cfab0ec0c550f9a289bd44301 |
	|  43 | 8c4047a001d9b0a6e5064054e4e6b9df |

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |   39 |
	|      2 |   40 |
	|      3 |   42 |
	|      4 |   43 |
	|      5 |   44 |

可以看到触发器已生效。

（3）t表删除数据要更新tm表，以保证row_id是按顺序无空隙的。

	delimiter $$
	drop trigger if exists delete_sync $$
	create trigger delete_sync
	after delete on t for each row
	begin 
		
		select max(row_id)
		delete from tm where t_id=OLD.id;
		update tm set row_id=row_id-1 where t_id>OLD.id;
	end $$
	delimiter ;

把以上命令直接贴到命令行：

	mysql> delimiter $$
	mysql> drop trigger if exists delete_sync $$

	mysql> create trigger delete_sync
	    -> after delete on t for each row
	    -> begin
	    -> delete from tm where t_id=OLD.id;
	    -> update tm set row_id=row_id-1 where t_id>OLD.id;
	    -> end $$
	Query OK, 0 rows affected (0.02 sec)

	mysql> delimiter ;
	mysql>

然后删除一条记录，查看结果：

	mysql> delete from t where id=3;
	Query OK, 1 row affected (0.01 sec)

	mysql> select * from t;
	+----+----------------------------------+
	| id | random                           |
	+----+----------------------------------+
	|  1 | ab15a38895845460b0be2979e84ca7aa |
	|  2 | 9119cee81bbe2310ae86b029da743a12 |
	|  4 | 5a8a4fb71e2862b80411e7c41915e4a8 |
	|  5 | 0018bcbaefff855215645ef78ba3a73a |
	|  6 | 01bed73a87b18624746cb06dfdf54b38 |
	+----+----------------------------------+
	5 rows in set (0.00 sec)

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |    1 |
	|      2 |    2 |
	|      3 |    4 |
	|      4 |    5 |
	|      5 |    6 |
	+--------+------+
	5 rows in set (0.00 sec)

	mysql>

可以看到在从t表删除id=3的记录之后，tm表也删除了t_id=3的记录，但是row_id还是保持连续的，并没有中断row_id=3，这是因为tm从被删除的记录开始往后，所有的记录row_id都往前移动了一个。这种情况下，如果数据量特别大的话，肯定会有效率问题。我们可以修改一下delete_sync触发器的逻辑，不用把所有的记录都移动一遍row_id，只修改max(row_id)对应的记录的row_id为当前被删除的记录的row_ id即可，而如果被删除的是tm表max(row_id)对应的记录，则不用做其他更新操作。现在我们把触发器修改一下：

	delimiter $$
	drop trigger if exists delete_sync;
	create trigger delete_sync
	after delete on t for each row
	begin
		declare rid int default 0;
		declare maxid int default 0;
		select row_id from tm  where t_id=OLD.id into rid;
		select max(row_id) from tm into maxid;
		if rid = maxid then
			delete from tm where row_id=rid;
		else 
			delete from tm where t_id = OLD.id;
			update tm set row_id = rid where row_id = maxid;
		end if;

	end $$
	delimiter ;

执行以上代码：

	mysql> delimiter $$
	mysql> drop trigger if exists delete_sync;
	    -> create trigger delete_sync
	    -> after delete on t for each row
	    -> begin
	    -> declare rid int default 0;
	    -> declare maxid int default 0;
	    -> select row_id from tm  where t_id=OLD.id into rid;
	    -> select max(row_id) from tm into maxid;
	    -> if rid = maxid then
	    -> delete from tm where row_id=rid;
	    -> else
	    -> delete from tm where t_id = OLD.id;
	    -> update tm set row_id = rid where row_id = maxid;
	    -> end if;
	    ->
	    -> end $$
	Query OK, 0 rows affected, 1 warning (0.00 sec)

	Query OK, 0 rows affected (0.02 sec)

	mysql> delimiter ;
	mysql>

我重新添加了数据到t表，现在我们查看一下表的现状：
	
	mysql> delete from t;
	Query OK, 62 rows affected (0.02 sec)

	mysql> select * from tm;
	Empty set (0.00 sec)

	mysql> call init(5);
	Query OK, 1 row affected (0.03 sec)

	mysql> select * from t;
	+-----+----------------------------------+
	| id  | random                           |
	+-----+----------------------------------+
	| 127 | daf28686a09f5ee8ce2cf5ce58b12432 |
	| 128 | 7edcf63b20c63cef3beb288e7aff99cf |
	| 129 | 1e23dcfaf2e546c12dd9f8137ce4b6e4 |
	| 130 | cde3ed2538f8799d4ba9dc87deea8a88 |
	| 131 | 07ebf2775f4d6d5d3378a046fe94622b |
	+-----+----------------------------------+
	5 rows in set (0.00 sec)

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |  127 |
	|      2 |  128 |
	|      3 |  129 |
	|      4 |  130 |
	|      5 |  131 |
	+--------+------+
	5 rows in set (0.00 sec)

	mysql>

我们先删除t表id为128的记录，然后查看结果：
	
	mysql> delete from t where id=128;
	Query OK, 1 row affected (0.01 sec)

	mysql> select * from t;
	+-----+----------------------------------+
	| id  | random                           |
	+-----+----------------------------------+
	| 127 | daf28686a09f5ee8ce2cf5ce58b12432 |
	| 129 | 1e23dcfaf2e546c12dd9f8137ce4b6e4 |
	| 130 | cde3ed2538f8799d4ba9dc87deea8a88 |
	| 131 | 07ebf2775f4d6d5d3378a046fe94622b |
	+-----+----------------------------------+
	4 rows in set (0.00 sec)

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |  127 |
	|      2 |  131 |
	|      3 |  129 |
	|      4 |  130 |
	+--------+------+
	4 rows in set (0.00 sec)

	mysql>

可以看到tm表row_id最大的一个记录row_id=5现在变成了row_id=2,其他的记录都没有变化。然后再删除t表id=130的记录，因为该记录在tm表是row_id最大的记录。
	
	mysql> delete from t where id=130;
	Query OK, 1 row affected (0.01 sec)

	mysql> select * from t;
	+-----+----------------------------------+
	| id  | random                           |
	+-----+----------------------------------+
	| 127 | daf28686a09f5ee8ce2cf5ce58b12432 |
	| 129 | 1e23dcfaf2e546c12dd9f8137ce4b6e4 |
	| 131 | 07ebf2775f4d6d5d3378a046fe94622b |
	+-----+----------------------------------+
	3 rows in set (0.00 sec)

	mysql> select * from tm;
	+--------+------+
	| row_id | t_id |
	+--------+------+
	|      1 |  127 |
	|      2 |  131 |
	|      3 |  129 |
	+--------+------+
	3 rows in set (0.00 sec)

	mysql>

删除row_id最大值对应的记录，不用更新其他记录。


现在我们可以根据tm表的最大row_id产生一个随机的row_id,然后获取对应的t_id,再从t表中根据t_id查找对应的记录，即达到了随机获取一条t表记录的效果。sql如下：

	select * from t join(select tm.row_id,tm.t_id from tm join (select  ceil(rand()*(select max(row_id) from tm)) as id) as temp1 where tm.row_id>temp1.id order by tm.row_id asc limit 1)as temp2 on t.id=temp2.t_id

为了测试效果，我们把数据多添加一些，直接调用init方法即可：
	
	mysql> call init(100);
	Query OK, 1 row affected (0.72 sec)

	然后我们删掉一部分记录，我把t表id从50到60的删除，从90到100的删除。请自行查看tm表的变化。

现在看一下随机获取记录的结果：

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+----+----------------------------------+------+
	| id | random                           | id   |
	+----+----------------------------------+------+
	| 43 | 8c4047a001d9b0a6e5064054e4e6b9df |   43 |
	+----+----------------------------------+------+
	1 row in set (0.00 sec)

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+----+----------------------------------+------+
	| id | random                           | id   |
	+----+----------------------------------+------+
	| 60 | b690e93552d8a7f96bcca58d3fabcded |   59 |
	+----+----------------------------------+------+
	1 row in set (0.00 sec)

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+----+----------------------------------+------+
	| id | random                           | id   |
	+----+----------------------------------+------+
	| 41 | f26b691d18b002cd663ec39b78d858cb |   41 |
	+----+----------------------------------+------+
	1 row in set (0.00 sec)

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+----+----------------------------------+------+
	| id | random                           | id   |
	+----+----------------------------------+------+
	| 40 | 1f2339e7e041815475e10da7bfec15ed |   24 |
	+----+----------------------------------+------+
	1 row in set (0.00 sec)

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+-----+----------------------------------+------+
	| id  | random                           | id   |
	+-----+----------------------------------+------+
	| 124 | a48ab3ea94f86a6b78e7772c54a17ca5 |  124 |
	+-----+----------------------------------+------+
	1 row in set (0.00 sec)

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+----+----------------------------------+------+
	| id | random                           | id   |
	+----+----------------------------------+------+
	| 42 | c7e5cf4cfab0ec0c550f9a289bd44301 |   42 |
	+----+----------------------------------+------+
	1 row in set (0.00 sec)
	
	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+-----+----------------------------------+------+
	| id  | random                           | id   |
	+-----+----------------------------------+------+
	| 100 | cb4af42f96156f4f477dce4cd5d66ecc |   92 |
	+-----+----------------------------------+------+
	1 row in set (0.00 sec)

	mysql> select * from t join (select ceil(rand()*(select max(id) from t)) as id) as t2 where t.id>= t2.id order by t.id asc limit 1;
	+----+----------------------------------+------+
	| id | random                           | id   |
	+----+----------------------------------+------+
	| 81 | a5c02b5f375bd49a01dd1f55e5acdda8 |   81 |
	+----+----------------------------------+------+


可以看到并没有增加t表有间断id的获取概率。

整篇文章使用的表中的数据前后可能不太一直，请根据自己表的数据查看。

参考文章链接：[http://jan.kneschke.de/projects/mysql/order-by-rand/](http://jan.kneschke.de/projects/mysql/order-by-rand/)

