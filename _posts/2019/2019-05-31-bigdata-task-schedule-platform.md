---
title: 大数据任务调度平台
tags: ['大数据', '大数据平台']
category: bigdataplatform
---

## 任务调度平台对比

比对项 |Airflow | Azkaban | EasyScheduler
------|--------|---------|--------------
API | [Airflow API文档](http://airflow.apache.org/api.html) | [Azkaban API文档](https://azkaban.readthedocs.io/en/latest/ajaxApi.html) | [EasyScheduler API文档](http://52.82.13.76:8888/easyscheduler/doc.html?language=zh_CN&lang=cn)
调度对比 | 更偏向于工作流方式，有强大的任务依赖管理和调度支持 | 更偏向于底层，跟Hadoop结合的比较紧 | 更贴切于建立DAG以及任务依赖，进行任务调度
环境搭建 | 分为WebServer和Executor端，属于中轻量级环境 | 环境搭建根据网站手册搭建，也没有太大问题 | 符合高可用需求
账户管理 | 数据库中存储，可集成第三方插件UI操作 | XML配置方式 | 可配置
社区活跃度 | [Airflow Github](https://github.com/apache/airflow) | [Azkaban Github](https://github.com/azkaban/azkaban) | [EasyScheduler Github](https://github.com/analysys/EasyScheduler)

## 总结

- API方面

总的来说，Azkaban和EasyScheduler的API更充分、更有利于实现应用端业务需求。

- 调度方面

Azkaban和Airflow更偏向于底层执行端的调度，但是Airflow有强大的任务依赖管理。

我们需要做的是针对上层任务之间的前后关系调度，需要编码包装任务这一层，也就是建立一个DAG的调度。需要我们来通过代码自定义好DAG依赖。

说白了，调度平台好像做的只是关乎任务的资源分配，排队，执行、暂停和监控以及日志输出之类。（该块是调度的核心，下面执行端的话交给基础架构那块即可）

- 环境搭建

三者环境搭建难度适宜，没有特别繁琐的过程。

- 账户管理

因为并不是让用户直接解决任务调度平台，所以用户量并不多，保证开发账户通行即可。

- 社区活跃度
  - Github star数：Airflow > Azkaban > EasyScheduler
