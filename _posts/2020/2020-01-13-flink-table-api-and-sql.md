---
title: Flink Table API & SQL
tags: ['Flink', '大数据']
category: flink
keywords: flink,Flink,大数据,Flink SQL,Flink Table API
---

## Table API & SQL介绍

- Apache Flink具有两个关系API：表API和SQL，用于统一流和批处理。Table API是Scala和Java的语言集成查询API，查询允许组合关系运算符，例如过滤和连接。Flink SQL支持标准的SQL语法。

- Table API和SQL接口彼此集成，FLink的DataStream和DataSet API亦是如此。你可以轻松地基于API构建的所有API和库之间切换。

- 注意，到目前最新版本位置，Table API和SQL还有很多功能正在开发中。并非[Table API, SQL]和[Stream, Batch]输入的每种组合都支持所有操作。

<!-- more -->

### 1.为什么需要Table API & SQL

- Table API是一种关系型API，类SQL的API，用户可以像操作表一样地操作数据，非常的直观和方便。

- SQL作为一个"人所皆知"的语言，如果一个引擎提供SQL，它将很容易被人们接受。这已经是业界很常见的现象。

- Table & SQL API还有另外一个职责，就是流处理和批处理统一的API层。

### 2.Table API & SQL流处理概述

Flink的Table API和SQL支持是用于批处理和流处理的统一API。这意味着Table API和SQL查询具有相同的语义，无论它们的输入是有界批量输入还是无界流输入。因为关系代数（relational algebra）和SQL最初是为批处理而设计的，所以对于无界流输入的关系查询不像有界批输入上的关系查询那样容易理解。

#### 2.1.流数据上的关系查询

SQL和Relational algebra并没有考虑到流数据。因此，在关系代数（和SQL）和流处理之间有一些概念上的差距。
<br/>
<table>
    <thead>
        <tr>
            <th>关系代数/SQL</th>
            <th>流处理</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>关系（或表）是有界的（多）元组的集合</td>
            <td>流式无界的元组序列</td>
        </tr>
        <tr>
            <td>对批处理数据执行的查询（例如，关系数据库中的表）可以访问完整的输入数据</td>
            <td>流式查询在启动时无法访问所有数据，必须等待流式传输数据</td>
        </tr>
        <tr>
            <td>批处理查询在生成固定大小的结果后终止</td>
            <td>流式查询会根据收到的记录不断更新其结果，并且永远不会完成</td>
        </tr>
    </tbody>
</table>

#### 2.2动态表和连续查询

动态表（Dynamic table）是Flink Table API和SQL支持流数据的核心概念。与表示批处理数据的静态表（static table）相比，动态表会随时间而变化，并且可以像静态批处理表一样查询。查询动态表会生成连续查询（Continuous Query）。连续查询永远不会终止并生成动态表作为结果。查询不断更新其结果以反映其输入表的更改。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/flink/flink-foundation/dynamic-table-and-continuous-query-1.png?raw=true)

#### 2.3在流上定义表

为了使用关系查询处理流，必须将其转换为表。从概念上将，流的每个记录都被解释为对结果表的INSERT修改。下图显示了点击事件流（左侧）如何转换为表（右侧）。随着更多的点击事件的插入，结果表不断增长。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/flink/flink-foundation/dynamic-table-and-continuous-query-2.png?raw=true)

#### 2.4连续查询

在动态表上进行连续查询，并生成新的动态表。与批查询相反，连续查询不会停止更新其结果表。在任何时间点，连续查询的结果在语义上等同于在输入表的快照上以批处理模式执行的相同查询的结果。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/flink/flink-foundation/dynamic-table-and-continuous-query-3.png?raw=true)
<br/>
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/flink/flink-foundation/dynamic-table-and-continuous-query-4.png?raw=true)
<br/>

#### 2.5表转换到流

流查询的结果表将被动态更新，即，随着新纪录到达查询的输入流，它也发生变化。因此，将这样的动态查询转换成的DataStream需要对表的更新进行编码。
<br/>
将表转换为数据流有两种方式：
<br/>
- Append-only Mode：只有在动态Table仅通过INSERT更新修改时才能使用此模式，即它仅附加，并且以前发出的结果永远不会更新。如果更新或删除操作使用追加模式会失败报错。
- Retract Mode：始终可以使用此模式。返回值是boolean类型。它用true或false来标记数据的插入或撤回，返回true代表数据插入，false代表数据的撤回。

![](https://github.com/buildupchao/ImgStore/blob/master/blog/flink/flink-foundation/dynamic-table-and-continuous-query-5.png?raw=true)

### 3.Flink SQL Connector

Flink的表API和SQL程序可以连接到其他外部系统来读写批处理表和流表。<br/>
Table source提供对存储在外部系统（如数据库、键值存储、消息队列或文件系统）中的数据的访问。<br/>
Table Sink将表发送到外部存储系统。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/flink/flink-foundation/dynamic-table-and-continuous-query-6.png?raw=true)

### 4.Flink Table API

Table API使用一个Scala和Java的语言集成查询API，是基于Table类。Table类代表了一个流或者批表，并提供方法来使用关系型操作。这些方法返回一个新的Table对象，这个新的Table对象代表着输入的Table应用关系型操作后的结果。
<br/>
[代码：批表案例](https://github.com/buildupchao/flink-examples/blob/master/src/main/java/com/buildupchao/flinkexamples/batch/api/BatchTableExample.java)
<br/>
[代码：流表案例](https://github.com/buildupchao/flink-examples/blob/master/src/main/java/com/buildupchao/flinkexamples/stream/StreamTableApiAndSqlExample.java)
<br/>

### 5.Flink SQL

Flink SQL集成是基于Apache Calcite，Apache Calcite实现了标准的SQL。
<br/>
[代码：订单案例](https://github.com/buildupchao/flink-examples/blob/master/src/main/java/com/buildupchao/flinkexamples/batch/sql/BatchOrderCaseSQLExample.java)

## 资料快链

- [官网：Table API & SQL](https://ci.apache.org/projects/flink/flink-docs-stable/dev/table/)
- [官网：Flink保留字列表](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/sql.html)
