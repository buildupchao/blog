---
title: 三周技术挑战约定之缘起
tags: ['tech-talking', 'technology-challenge']
category: technology-challenge
keywords: technology-challenge,技术挑战
layout: post
---

2019年12月23日，突发奇想，相约三周时间进行技术挑战。

参赛人数：两人

人员名单：buildupchao & 另一个不能暴露身份的神秘人（后面就统一“神秘人”称之）

挑战说明：在接下来三周时间内，总共6项选题，每人选两项且不交叉的极速学习方式。期间，需要形成文档，可以采用每日Github or 印象笔记 or 手记方式记录，于每周末（周日）进行一次对战，验收情况，以问倒对方为主进行查漏补缺，同时每次对战后，需要将每周文档同步录入到SegmentFault进行汇总。第三周周日晚上完结，进行终极PK，查看下是否有技术上的不一样（比如短时间快速Get到技术且消化的能力 or 技术突破等）以及习惯的养成。

选题如下：

| 选题 | 选择人 |
| --- | --- |
| MySQL InnoDB存储引擎 | 神秘人 |
| Hive进阶 | - |
| Netty in Action && Spring Cloud Alibaba | buildupchao |
| Flink官方文档 && Action in research and development | - |
| Kafka进阶 | - |
| Presto进阶 | - |

选题研究内容：

要求文档内容追求原创，内容包含：“背景 -> 制造问题（or 寻求问题） -> 如何逐步渐进解决问题 -> 回顾总结 -> 问题延拓 -> 提供技术相关资料链接”

* MySQL InnoDB存储引擎
    * 请带我们分析下InnoDB是如何实现行锁的？
* Hive进阶
    * MR执行流程（Hadoop1 && Hadoop2），如果小文件很多，对此弊端是是什么？该如何优化？
    * Hive分区过多会怎么样？如何及时感知数据上报进行repair修分区，聊聊修分区的设计？
    * 数据倾斜产生情况，如何进行优化？
    * 常见HQL问题汇总以及说明
* Netty in Action && Spring Cloud Alibaba
    * nacos自动刷新配置如何实现的
    * sentinel限流原理
* Flink官方文档 && Action in research and development
* Kafka进阶
    * 如何保证消息不被重复消费 or 如何保证消息消费时的幂等性？
    * 如何保证消息被可靠传输不丢失（需要考虑producer发送、kafka传输、consumer消费）？
    * 如何保证从MQ里拿到数据按顺序执行？
* Presto进阶
    * Presto的预处理原理是什么？如何让查询过一次的SQL再次重复查询而不采用预处理？


后续，每次完成一个选项，才会开启下一个选项的选择，不会一开始就展开一系列问题。希望后续能收到不错的结果。

产出文章列表：<br/>
- [三周技术挑战约定之缘起](http://www.buildupchao.cn/technology-challenge/2019/12/23/technology-challenge-for-3-weeks.html)
- [【技术挑战】Nacos自动刷新配置如何实现的？](http://www.buildupchao.cn/technology-challenge/2019/12/26/how-to-refresh-conf-automatically-for-nacos.html)
- [【技术挑战】Sentinel限流原理？](http://www.buildupchao.cn/technology-challenge/2019/12/28/traffic-limit-control-for-sentinel.html)
