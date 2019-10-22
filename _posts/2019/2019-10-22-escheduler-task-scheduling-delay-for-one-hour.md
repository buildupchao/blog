---
title: EasyScheduler线上任务调度延迟1小时问题排查
tags: ['EasyScheduler', '线上问题排查']
category: onlinetroubleshooting
---

## 一、背景

> 早上，暴躁君W来了条信息:"小时计算任务延迟一小时执行,导致应该6点启动的计算3点数据的任务到7点才被提交执行，而计算4点数据的任务跑了两次,帮忙排查下这个问题。"

## 二、那么问题来了

![easyscheduler-archtecture](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/easyscheduler/easyscheduler-arch.png?raw=true)

从上述架构图我们知道，MasterServer进行任务的生成，放至Task Queue中，WorkerServer从Task Queue中消费任务进行执行。
其次，EasyScheduler有一配置特性，如果当前结点CPU或者内存达到了80%以上，则不会进行新的任务的调度和执行。

综合上述两者，大概猜测到了是master结点CPU负载过高导致的.定位步骤如下:

- 通过公司的机器监控平台`of-dashboard`查看机器的CPU、内存最近一天使用情况

发现早上5点半开始到7点之间CPU负载呈现以80%为中心的正态分布，推断当时的确是触发了master结点的保护机制

- 查看master结点escheduler-master-xxx.log，观察早上06:00~07:00这段时间的日志

发现6点到7点这段时间一直处于80%高负载状态，持续打印 `[WARN] 2019-10-22 06:01:41.344 cn.escheduler.common.utils.OSUtils:[290] - load or availablePhysicalMemorySize(G) is too high, it's availablePhysicalMemorySize(G):200.01,loadAvg:23.3` 信息，完全确认是因为高负载导致。

但是高负载原因是什么呢？其实是因为我们混部了一些移动端采集任务在master结点上导致的（本意是处于充分利用机器）

- 从escheduler数据库中查询今日任务完成时间为6:00到7:18之间的任务实例信息

得出有三项数据计算任务均为高负载，且耗时跨度均为2~3小时，提取出该三项任务所属项目以及工作流定义信息，反馈给暴躁君，并让其进行任务优化或者时间段分摊负载以保证小时计算任务正常执行。

## 三、总结

- 问题定位都是有套路和步骤的，制定好troubleshooting的步骤，按部就班可以事半功倍

- EasyScheduler的自我保护机制是可配置的，只需在`install.sh`配置文件中配置如下两个参数即可

```bash
# master最大cpu平均负载,用来判断master是否还有执行能力
masterMaxCpuLoadAvg="10"

# master预留内存,用来判断master是否还有执行能力
masterReservedMemory="1"
```

- 你了解了EasyScheduler的套路了吗？
