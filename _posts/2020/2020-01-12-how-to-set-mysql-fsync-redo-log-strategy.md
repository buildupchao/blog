---
title: 如何控制MySQL事务提交后，刷redo-log的策略？
tags: ['redo-log', 'MySQL']
category: db
keywords: database,db,MySQL,数据库,redo log
layout: post
---

既然涉及到事务提交，那么我们就是以`InnoDB`来说明的。
<br/><br/>
MySQL有一个参数，能够控制事务提交时，刷redo log的策略。该参数为：`innodb_flush_log_at_trx_commit`。

![](https://github.com/buildupchao/ImgStore/blob/master/blog/db/mysql-refresh-redo-log-1.png?raw=true)

- 策略1,`set global innodb_flush_log_at_trx_commit = 0`

该方式可以获得最佳性能。<br/>
每隔1秒种，才将Log Buffer中的数据批量写入OS Cache，同时MySQL主动fsync。<br/>
<strong style="color:red;">这种策略，如果数据库崩溃，会有1秒的数据丢失。</strong>

- 策略2,`set global innodb_flush_log_at_trx_commit = 1`

该方式可以获得强一致性。<br/>
每次事务提交，都将Log Buffer中数据写入到OS Cache，同时MySQL主动fsync。<br/>
<strong style="color:red;">这种策略是InnoDB默认配置，是为了保证事务ACID特性。</strong>

- 策略3,`set global innodb_flush_log_at_trx_commit = 2`

该方式则是一种trade-off的结果。<br/>
每次事务提交，都将Log Buffer中的数据写入OS Cache；<br/>
每隔1秒，MySQL主动将OS Cache中数据批量fsync。<br/>
<strong style="color:red;">这种策略，如果操作系统崩溃，最多会有1秒的数据丢失。（磁盘IO次数是不确定的，因为OS的fsync的频率并不是MySQL能控制的；OS也会进行fsync，而MySQL主动fsync的周期是1秒，所以最多丢1秒数据）</strong>

<br/>
那么，透漏个小秘密，高并发业务场景下，最佳实践会如何配置呢？<br/>
<strong style="color:green;">答案是`set global innodb_flush_log_at_trx_commit = 2`。因为不仅可以保证性能，也相对可以保障安全性（只要操作系统不出问题，数据就不会丢。而比起MySQL应用出问题，操作系统出问题概率小很多）</strong>
