---
layout: post
title: 通过IP地址访问postgresql数据库
category: postgressql
tags: [postgresql]
---

本地有一应用程序连接postgresql数据库，使用localhost连接没有问题，把localhost换成本机ip就不行，提示：


	2016-11-03 14:58:10.166 ERROR 12068 --- [           main] o.a.tomcat.jdbc.pool.ConnectionPool      : Unable to create initial connections of pool.

	org.postgresql.util.PSQLException: ��������: ������������ "172.18.19.56", ���� "postgres", ������ "test", SSL ���� �� pg_hba.conf ����
		at org.postgresql.core.v3.ConnectionFactoryImpl.doAuthentication(ConnectionFactoryImpl.java:293) ~[postgresql-9.1-901-1.jdbc4.jar:na]
		at org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:108) ~[postgresql-9.1-901-1.jdbc4.jar:na]
		at org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:66) ~[postgresql-9.1-901-1.jdbc4.jar:na]
		at org.postgresql.jdbc2.AbstractJdbc2Connection.<init>(AbstractJdbc2Connection.java:125) ~[postgresql-9.1-901-1.jdbc4.jar:na]
		at org.postgresql.jdbc3.AbstractJdbc3Connection.<init>(AbstractJdbc3Connection.java:30) ~[postgresql-9.1-901-1.jdbc4.jar:na]
		at org.postgresql.jdbc3g.AbstractJdbc3gConnection.<init>(AbstractJdbc3gConnection.java:22) ~[postgresql-9.1-901-1.jdbc4.jar:na]
		at org.postgresql.jdbc4.AbstractJdbc4Connection.<init>(AbstractJdbc4Connection.java:32) ~[postgresql-9.1-901-1.jdbc4.jar:na]


出现这个问题是因为postgresql默认不允许外部连接，需要配置才行。本机系统为win10，postgresql版本是：psql (PostgreSQL) 9.6beta4


修改配置如下：


	打开C:\Program Files\PostgreSQL\9.6\data\pg_hba.conf，找到

	# IPv4 local connections:
	host    all             all             127.0.0.1/32            md5

	在这个下面加一行

	host    all             all             0.0.0.0/0          md5

	重启postgresql服务器（在服务里面找到postgresql服务，点击重启）