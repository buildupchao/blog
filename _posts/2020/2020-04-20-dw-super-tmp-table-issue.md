---
title: MySQL超大临时表问题回顾
tags: ['mysql', 'java', 'db']
category: onlinetroubleshooting
keywords: db,java,mysql,数据库,性能优化
layout: post
---

## 1.问题背景

  近期接到运维同学反馈，数仓线上MySQL数据库产生了一个500G的临时表。500G！500G！500G！

<!-- more -->

## 2.排查定位

通过pma访问数据库实例，执行``` show full processlist ```找到可疑的那个SQL（可疑查看Time字段，单位：秒）

![](https://github.com/buildupchao/ImgStore/blob/master/blog/db/tmp_table_1.png?raw=true)

可以通过设置“选项”显示完整SQL内容：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/db/tmp_table_2.png?raw=true)

回到项目代码中找到产生改SQL的地方（已经针对关键部分进行伪代码改写）：

```SQL
select distinct device as value from (select device from database_name.table_name where device <> 'your_value' and device <> '' order by date, device desc) a
```

<strong style='color:red;'>到这里需要说一句，该SQL寻找过程比较曲折，居然是通过String一点一点拼接的，强烈抗议！！！有ORM框架为何不用？？？</strong>

我们通过```explain```查看执行计划来继续分析SQL：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/db/tmp_table_3.png?raw=true)

嗯~的确是采用了临时表……

那我们把SQL改吧改吧，然后重新查看下执行计划：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/db/tmp_table_4.png?raw=true)

嗯，好，没有超级临时表问题了~

## 3.问题复盘

其实这个SQL并不复杂，但是为啥之前没有暴露出来问题呢？我想原因有两点：

- <strong style='color:red;'>数据量</strong>：在数据量没有达到一定量级的时候，一些问题往往会被掩盖了，很难凸显出来，或者对系统造成毁灭性的影响；
- <strong style='color:red;'>主动型</strong>：在问题没有发生的时候，当你看到这个SQL，你是否会有改掉它的意识并且执行起来呢？？？而不是表示对前人的尊敬呢？

“往往魔鬼总在细节中”。千里之堤毁于蚁穴，希望我们都能够引以为戒！
