---
title: 初识Airflow任务调度平台
tags: ['Airflow', '大数据']
category: bigdataplatform
---

## 1.分布式和集群

- 集群和分布式对比

集群 | 分布式
-----|-------
一个物理形态 | 一种工作方式
只要是一堆机器，就可以称为集群，至于它们是否协作干活，这个谁也不知道 | 一个程序或者系统，只要运行在不同的机器上，就可以叫分布式（当然，C/S架构也可以叫分布式）
一般是物理集中、统一管理的 | 不强调物理集中、统一管理

- 小结

集群可能运行着一个或多个分布式系统，也可能根本没有运行分布式系统；

分布式系统可能运行在一个集群上，也可能运行在不属于一个集群的多台（2台也算是多台）机器上。

## 2.Airflow介绍

<strong style="color:green;">Airflow是Airbnb开源的一个用Python编写的调度工具。</strong>

### 2.1 Airflow中的作业和任务

- DAG
  - 概要：DAG（Directed Acyclic Graph）是有向无环图，也称为有向无循环图。在Airflow中，一个DAG定义了一个完整的作业。同一个DAG中的所有Task拥有相同的调度时间。
  - 参数：
    - dag_id: 唯一识别DAG，方便日后管理
    - default_args: 默认参数，如果当前DAG实例的作业没有配置相应参数，则采用DAG实例的default_args中的相应参数
    - schedule_interval: 配置DAG的执行周期，可采用crontab语法

- Task
  - 概要：Task为DAG中具体的作业任务，依赖于DAG，也就是必须存在于某个DAG中。Task在DAG中可以配置依赖关系（<strong style="color:red;">当然也可以配置跨DAG依赖，但是并不推荐。跨DAG依赖会导致DAG图的直观性降低，并给依赖管理带来麻烦</strong>）。
  - 参数：
    - dag: 传递一个DAG实例，以使当前作业属于相应DAG
    - task_id: 给任务一个标识符（名字），方便日后管理
    - owner: 任务的拥有者，方便日后管理
    - start_date: 任务的开始时间，即任务将在这个时间点之后开始调度

### 2.2 Airflow的调度时间

- start_date

在配置中，它是作业开始调度时间。而在谈论执行状况时，它是调度开始时间。

- schedule_interval

调度执行周期。

- execution_date

执行时间。在Airflow中称为执行时间，但其实它并不是真实的执行时间。

------------------------
<strong style="color:green;">[敲黑板，划重点]</strong>

所以，第一次调度时间：在作业中配置的start_date，且满足schedule_interval的时间点。记录的execution_date为作业中配置的start_date的第一个满足schedule_interval的时间。

<strong style="color:green;">[举个例子]</strong>

假设我们配置了一个作业的start_date为<strong>2019年6月2日</strong>，配置的schedule_interval为<strong>``` * 00 12 * * * ```</strong>，那么第一次执行的时间将是<strong>2019年6月3日 12点</strong>。因此execution_date并不是如期字面说的表示执行时间，真正的执行时间是execution_date所显示的时间的下一个满足schedule_interval的时间点。

------------------------

### 2.3 Airflow的调度方式

- 调度方式
  - SequentialExecutor: 顺序调度
  - LocalExecutor: 多进程调度
  - CeleryExecutor: 分布式调度

<strong style="color:red;">但是有时候仅仅靠配置作业依赖和调度执行周期并不能满足一些复杂的需求</strong>

- other任务调度方式
  - 1) 跳过非最新DAG Run（作业中出现故障，一段时间后恢复）
  - 2) 当存在正在执行的DAG Run时，跳过当前DAG Run（作业执行时间过长，长到下一次作业开始）
  - 3）Sensor的替代方案（Airflow中有一类Operator被称为Sensor，Sensor可以感应预先设定的条件是否满足，当满足条件后Sensor作业变为Success使得下游的作业可以执行。）

### 2.4 Airflow的服务构成

## 3.基于Docker的Airflow分布式搭建

## 4.彩蛋
