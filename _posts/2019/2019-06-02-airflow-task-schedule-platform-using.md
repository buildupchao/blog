---
title: Airflow[v1.10]任务调度平台的安装教程
tags: ['Airflow', '大数据']
category: bigdataplatform
---

![airflow安装教程](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-install-header.JPG?raw=true)

## 0.背景

真的是想不通，Airflow不论社区活跃度还是Github的star数都是远胜于Azkaban还有EasyScheduler的，但是为何却连一个完备的安装教程都没有呢？是我的需求太高？真的是心累不已，整整把搜索引擎还有youtube翻来覆去也没让我感到满足……不过好在，一步一坑一脚印的最终搭建连通好了环境以及Operator。好了，废话不多说，开始Airflow今日份安装教程。

## 1.安装前准备工作

- 安装版本说明

安装工具 | 版本
-------|------
Python | 3.6.5
MySQL | 5.7
Airflow | 1.10.0

<strong style="color:red;">请选择一台干净的物理机或者云主机。不然，产生任何多余影响或者后果，本人概不负责！</strong>

- 请确保你熟悉Linux环境及基本操作命令，顺便会一些Python基础命令，如果不熟悉，请出门左转充完电再来

## 2.安装Python3

[Python3的安装可以参考我之前的文章，在此不再敖述](http://www.buildupchao.cn/installtutorial/2019/03/01/python3-install-tutorial.html)

## 3.安装MySQL

3年前也写过一个关于[Centos安装MySQL的教程](http://www.buildupchao.cn/installtutorial/2016/12/09/install_mysql5.6_on_centos7.html)，但是虽然实用，但是内容太久，在此我们用最简方式快速安装MySQL并配置用户（当然，如果你用现成的``` RDS ```也可以，省去了安装过程，可直接跳转至为Airflow建库建用户步骤了）。

- 老规矩，卸载mariadb

```bash
rpm -qa | grep mariadb

rpm -e --nodeps mariadb-libs-5.5.52-1.el7.x86_64

sudo rpm -e --nodeps mariadb-libs-5.5.52-1.el7.x86_64

rpm -qa | grep mariadb
```

- 下载mysql的repo源

```bash
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

- 通过rpm安装

```bash
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

- 安装mysql并授权

```bash
sudo yum install mysql-server
sudo chown -R mysql:mysql /var/lib/mysql
```

- 启动mysql

```bash
service mysqld start
```

______________________

<strong style="color:red;">以下操作均在mysql客户端上进行操作，首先需要连接并登录mysql。</strong>

用root用户连接登录mysql:
```bash
mysql -uroot
```

- 重置mysql密码

```bash
use mysql;

update user set password=password('root') where user='root';

flush privileges;
```
- 为Airflow建库、建用户

<strong>建库:</strong>
```bash
create database airflow;
```

<strong>建用户:</strong>
```bash
create user 'airflow'@'%' identified by 'airflow';

create user 'airflow'@'localhost' identified by 'airflow';
```

<strong>为用户授权:</strong>
```bash
grant all on airflow.* to 'airflow'@'%';

flush privileges;

exit;
```
______________________

## 4.安装Airflow

<strong>万事既已具备，让我们开始进入今天的主题！</strong>
