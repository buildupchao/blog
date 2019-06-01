---
title: Hive数据去重、多变一与一变多等实现
date: 2017-09-29 23:49:54
tags: ['大数据', 'Hive']
category: Hive
---

## 0. 数据准备

### 0.1 数据文件

本机的/usr/local/share/applications/hive/data/目录下创建 employees.txt 数据文件：

```
John Doe^A100000.0^AMary Smith^BTodd Jones^AFederal Taxes^C.2^BState Taxes^C.05^BInsurance^C.1^A1 Michigan Ave.^BChicago^BIL^B60600
Mary Smith^A80000.0^ABill King^AFederal Taxes^C.2^BState Taxes^C.05^BInsurance^C.1^A100 Ontario St.^BChicago^BIL^B60601
Todd Jones^A70000.0^A^AFederal Taxes^C.15^BState Taxes^C.03^BInsurance^C.1^A200 Chicago Ave.^BOak Park^BIL^B60700
Bill King^A60000.0^A^AFederal Taxes^C.15^BState Taxes^C.03^BInsurance^C.1^A300 Obscure Dr.^BObscuria^BIL^B60100
Boss Man^A200000.0^AJohn Doe^BFred Finance^AFederal Taxes^C.3^BState Taxes^C.07^BInsurance^C.05^A1 Pretentious Drive.^BChicago^BIL^B60500
Fred Finance^A150000.0^AStacy Accountant^AFederal Taxes^C.3^BState Taxes^C.07^BInsurance^C.05^A2 Pretentious Drive.^BChicago^BIL^B60500
Stacy Accountant^A60000.0^A^AFederal Taxes^C.15^BState Taxes^C.03^BInsurance^C.1^A300 Main St.^BNaperville^BIL^B60563
```

创建students.txt数据文件：

```
1,xiaoming,english,92
2,xiaoming,math,98
3,xiaoming,chinese,100
4,shifei,english,99
5,shifei,math,100
6,shigei,chinese,100
```

### 0.2 创建数据表

```
CREATE TABLE IF NOT EXISTS employees (
name STRING,
salary FLOAT,
subordinates ARRAY<STRING>,
deductions MAP<STRING, FLOAT>,
address STRUCT<street:STRING, city:STRING, state:STRING, zip:INT>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\001' COLLECTION ITEMS TERMINATED BY '\002' MAP KEYS TERMINATED BY '\003'
LINES TERMINATED BY '\ '
STORED AS TEXTFILE;
```
```
CREATE TABLE IF NOT EXISTS students (
id INT,
name STRING,
subject STRING,
score INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```

### 0.3 导入数据到数据表中

```
load data local inpath '/usr/local/share/applications/hive/data/employees.txt' overwrite into table employees;
```

```
load data local inpath '/usr/local/share/applications/hive/data/students.txt' overwrite into table students;
```

## 1. Hive实现一行数据变成多行

接下来我们以employees表为主进行相应操作。

### 1.1 首先，我们查看employees中每个员工的上下属关系

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-09-29-1.png?raw=true)

### 1.2 把subordinates字段进行分割，我们要得到name-salary-subordinate形式的数据，而不再是一个数组

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-09-29-2.png?raw=true)

现在，在查询结果集中，已经针对subordinates进行分割成一个个的subordinate了。但是，不对啊？！数据是不是少了一些？少了subordinates字段为空的记录，对吧？我需要把其为空的记录也显示出来，我有该怎么处理呢？很简单，使用lateral view outer 即可实现，SQL代码如下：

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-09-29-3.png?raw=true)

同理，如果想要针对deductions也是一样：

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-09-29-4.png?raw=true)

<strong>而针对结构体STRUCT要想实现该功能，则需要使用Hive最新版才支持。</strong>


## 2. Hive实现多行数据变成一行

接下来的所有操作均以students表为例。

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-09-29-5.png?raw=true)

<strong>通过collect_set和concat_ws以及concat函数实现。</strong>

## 3. Hive数据去重，并取指定的一条数据

场景描述：从students表中获取学生姓名。

![这里写图片描述](https://github.com/buildupchao/ImgStore/blob/master/blog/2017-09-29-6.png?raw=true)

<strong>通过ROW_NUMBER() OVER函数实现。</strong>

ROW_NUMBER() OVER函数的基本用法：

```
ROW_NUMBER() OVER(PARTITION BY COLUMN ORDER BY COLUMN)
```

ROW_NUMBER() 从1开始，为每一个分组记录返回一个数字。

## Summary

- 1. Hive通过lateral view (outer) 和explode、split函数实现数据一变多

- 2. Hive通过concat_ws、collect_set和concat函数实现数据多变一

- 3. Hive通过ROW_NUMBER() OVER函数实现数据去重

好了，Hive小记到此结束。博主设计，仅供参考！
