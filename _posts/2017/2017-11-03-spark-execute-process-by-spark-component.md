---
title: 从Spark组件来剖析Spark的执行流程
tags: ['大数据', 'Spark']
category: spark
layout: post
---

## 0 从对比的视角来看Spark
### 0.1 Hadoop VS Spark

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-11-03-1.bmp?raw=true)

### 0.2 Spark VS Flink

- Flink 作为后起之秀，发展迅猛，有Ali 的 Blink 在支撑，流行起来是早晚的事
- 流计算：Flink要强一些，真正的流计算。不过，Spark Summit 2017 大会也有说到 Spark 2.2 后重点会是流计算与深度学习
- Spark SQL 或者 Hive on Spark 相对于 Flink 也是有优势的
- 如果深入了解了 Spark，Flink应用起来会简单很多。

## 1 从Spark组件来剖析Spark的执行流程
### 1.1 Spark组件结构图

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-11-03-2.png?raw=true)

### 1.2 Spark架构图

- 图1.2.1

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-11-03-3.jpeg?raw=true)

- 图1.2.2

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-11-03-4.jpeg?raw=true)


- Application: Spark中的Application和 Hadoop MR中的概念是相似的，指的是用户编写的Spark应用程序，内含了一个 Driver 功能的代码和分布在集群中多个节点上运行的Executor代码。

- Driver Program:Spark 中的 Driver 即运行上述 Application 的 main() 函数并且创建 SparkContext，其中创建 SparkContext 的目的是为了准备 Spark 应用程序的运行环境。在 Spark 中由 SparkContext 负责和 ClusterManager 通信，进行资源的申请、任务的分配和监控等；当 Executor 部分运行完毕后，Driver 负责将 SparkContext 关闭。通常 SparkContext 代表 Driver。

- Cluster Manager:指的是在集群上获取资源的外部服务，目前有:

- Standalone: Spark 原生的资源管理，由 Master 负责资源的分配；
Hadoop Yarn: 由 YARN 中的 ResourceManager 负责资源的分配；

- Worker:集群中任何可以运行 Application 代码的节点，类似于 YARN 的 NodeManager 节点。在 Standalone 模式中指的就是通过 Slave 文件配置的 Worker 节点，在Spark on Yarn 模式中指的就是 NodeManager 节点。

- Executor:Application 运行在 Worker 节点上的一个进程，该进程负责运行 Task，并且负责将数据存在内存或者磁盘上，每个 Application 都有各自独立的一批 Executor。在 Spark on Yarn模式下，其进程名称为 CoarseGrainedExecutorBackend，类似于Hadoop MapReduce中的 YarnChild。一个 CoarseGrainedExecutorBackend 进程有且仅有一个 executor 对象，它负责将 Task 包装成 taskRunner，并从线程池中抽取出一个空闲线程运行Task。每个 CoarseGrainedExecutorBackend能并行运行 Task的数量就取决于分配给它的 CPU的个数了。

- Job: 包含多个 Task组成的并行计算，往往由 Spark Action催生，一个Job包含多个RDD应用及作用于RDD上的各种Operation。

- Stage: 每个Job会被拆分很多Task，每组Task被称为Stage，也可称为 TaskSet，一个Job分为多个Stage

- Task: 被送到某个 Executor 上的工作任务

### 1.3 Spark的基本运行流程图

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-11-03-5.jpeg?raw=true)

步骤如下:

- 1) 构建Spark Application的运行环境，启动 Spark Context

- 2) SparkContext向资源管理器（Standalone，Mesos，Yarn）申请运行Executor资源，并启动StandaloneExecutorbackend，Executor向SparkContext申请Task

- 3) SparkContext将应用程序分发给Executor

- 4) SparkContext构建成DAG图，将DAG图分解成Stage，将TaskSet发送给Task Scheduler，最后由Task Scheduler将Task发送给Executor运行

- 5) Task在Executor上运行，运行完毕释放所有资源


### 1.4 Spark的完整运行流程图

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-11-03-6.jpeg?raw=true)

### 1.5 Spark运行架构的特点

- 1) 每个Application获取专属的Executor进程，该进程在Application期间一直驻留，并以多线程方式运行Task。这种Application隔离机制是有优势的，无论是从调度角度看（每个Driver调度他自己的任务），还是从运行角度看（来自不同Application的Task运行在不同JVM中），当然这样意味着Spark Application不能跨应用程序共享数据，除非将数据写入外部存储系统

- 2) Spark与资源管理器无关，只要能够获取Executor进程，并能保持相互通信就可以了

- 3) 提交SparkContext的Client应该靠近Worker节点（运行Executor的节点），最好是在同一个Rack里，因为Spark Application运行过程中SparkContext和Executor之间有大量的信息交换，如果在远程集群中运行，最好使用RPC 将 SparkContext提交给集群，不要远离Worker运行SparkContext

- 4) Task采用了数据本地性和推测执行的优化机制

## Reference Link

- [1] [Spark 执行流程](https://blog.xiaoxiaomo.com/2017/07/05/Spark-%E4%BB%8ESpark%E7%BB%84%E4%BB%B6%E6%9D%A5%E7%9C%8BSpark%E7%9A%84%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B/)

- [2] [Spark设计理念与基础架构](http://blog.csdn.net/beliefer/article/details/50561247)
