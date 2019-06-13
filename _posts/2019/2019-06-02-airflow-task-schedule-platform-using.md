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

安装工具 | 版本 | 用途
-------|------|-----
Python | 3.6.5 | 安装airflow及其依赖包、开发airflow的dag使用
MySQL | 5.7 | 作为airflow的元数据库
Airflow | 1.10.0 | 任务调度平台

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

- 1）通过pip安装airflow脚手架

安装之前需要设置一下临时环境变量``` SLUGIFY_USES_TEXT_UNIDECODE ```，不然，会导致安装失败，命令如下：
```bash
export SLUGIFY_USES_TEXT_UNIDECODE=yes
```

安装airflow脚手架:
```bash
sudo pip install apache-airflow===1.10.0
```

airflow会被安装到python3下的site-packages目录下，完整目录为:``` ${PYTHON_HOME}/lib/python3.6/site-packages/airflow ```，我的airflow目录如下所示：
![python3-airflow-dir](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/python3-airflow-dir.png?raw=true)

- 2) 正式安装airflow

安装airflow前，我们需要先配置一下airflow的安装目录``` AIRFLOW_HOME ```，同时为了方便使用airflow的相关命令，我们也把airflow配置到环境变量中，一劳永逸。

<strong>编辑``` /etc/profile ```系统环境变量文件：</strong>
```bash
sudo vim /etc/profile
```

<strong>做如下修改（当然，具体目录需要修改成你自己对应的目录，不要照搬不动哦）：</strong>
![airflow-env-variable](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-env-variable.png?raw=true)

<strong>使修改后的环境变量立即生效:</strong>
```bash
sudo source /etc/profile
```

- 3）执行``` airflow ```命令做初始化操作

因为配置过airflow的环境变量``` SITE_AIRFLOW_HOME ```，我们在哪里执行如下命令都可：
```bash
airflow
```

到此，airflow会在刚刚的``` AIRFLOW_HOME ```目录下生成一些文件。当然，执行该命令时可能会报一些错误，可以不用理会！生成的文件列表如下所示：
![airflow-init-file](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-init-file.png?raw=true)

- 4) 为airflow安装mysql模块

```bash
sudo pip install apache-airflow[mysql]
```
