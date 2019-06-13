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

### 4.1基础篇

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
sudo pip install 'apache-airflow[mysql]'
```

airflow的包依赖安装均可采用该方式进行安装，具体可参考[airflow官方文档](https://airflow.apache.org/installation.html)

- 5) 采用mysql作为airflow的元数据库

<strong>修改airflow.cfg文件，配置mysql作为airflow元数据库:</strong>

_______________________
<strong style="color:red;">这里就巨坑无比了，很多人的教程里面都是这么直接写的，然后就完蛋了！！！不信你试试看，等下初始化数据库必死无疑！而且还没有任何有效的解决方案可以供你搜索！！！其次也不要相信什么改成<span style="color:green;">``` pymysql ```</span>配合pymysql包实现，这样更惨，会有数据类型解析问题，让你毫无头绪！！！切记切记！！！</strong>
```bash
sql_alchemy_conn = mysql://airflow:airflow@localhost:3306/airflow
```
或
```bash
sql_alchemy_conn = mysql+pymysql://airflow:airflow@localhost:3306/airflow
```
_______________________
那既然这种方式不可行，该怎么办呢？办法总比困难多的！因为Python3不再支持MySQLdb了，只有Python2才支持。但是呢，也只有MySQLdb才是最佳结果。

<strong>首先，我们通过pip安装一下mysqlclient:</strong>
```bash
sudo pip install mysqlclient
```

<strong>然后，再通过pip安装一下MySQLdb:</strong>
```bash
sudo pip install MySQLdb
```

<strong>最后，我们修改airflow.cfg文件中的sql_alchemy_conn配置:</strong>
```bash
sql_alchemy_conn = mysql+mysqldb://airflow:airflow@localhost:3306/airflow
```

到此，我们已经为airflow配置好元数据库信息且准备好依赖包。

- 6) 初始化元数据库信息（其实也就是新建airflow依赖的表）

```bash
airflow initdb
```

此时，我们的mysql元数据库（库名为airflow）中已经新建好了airflow的依赖表：

![airflow-metadata](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-metadata.png?raw=true)

- 7) 应用的基础命令
  - airflow组件：webserver, scheduler, worker, flower
  - 后台启动各组件命令：``` airflow xxx -D ```
  - 查看dag列表：``` airflow list_dags ```
  - 查看某个dag的任务列表：``` airflow list_tasks dag_id ```
  - 挂起/恢复某个dag：``` airflow pause/unpause dag_id ```
  - 测试某个dag任务：``` airflow test dag_id task_id execution_date ```

### 4.2进阶篇

- 1) 初识executor

![airflow-executor](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-executor.png?raw=true)

这里为什么要修改呢？因为SequentialExecutor是单进程顺序执行任务，默认执行器，通常只用于测试，LocalExecutor是多进程本地执行任务使用的，CeleryExecutor是分布式调度使用（当然也可以单机），生产环境常用，DaskExecutor则用于动态任务调度，常用于数据分析。

- 2）如何修改时区为东八区

为什么要修改时区呢？因为Airflow默认的时间是GMT时间，虽然可以在Airflow集群分布在不同时区时仍可保证时间相同，不会出现时间不同步的问题，但是这个时间比北京早8小时，不太符合我们的阅读习惯，也不够简洁直观。鉴于我们通常情况下，我们要么为单节点服务，要么即使扩展也是在同一个时区的，所以将时区修改为东八区，即北京时间，这样更便于我们使用。

Come on!

<strong>(1) 修改airflow.cfg文件:</strong>
```bash
default_timezone = Asia/Shanghai
```

这里修改的是scheduler的调度时间，也就是说在编写调度时间是可以直接写北京时间。

<strong>(2) 修改webserver页面上右上角展示的时间:</strong>

需要修改``` ${PATHON_HOME}/lib/python3.6/site-packages/airflow/www/templates/admin/master.html ```文件。

![airflow-webserver-time](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-webserver-time.png?raw=true)

修改后效果如图所示：

![airflow-webserver-time-2](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-webserver-time-2.png?raw=true)

<strong>(3) 修改webserver lastRun时间:</strong>

第1处修改``` ${PATHON_HOME}/lib/python3.6/site-packages/airflow/models.py ```文件。

```bash
def utc2local(self,utc):
       import time
       epoch = time.mktime(utc.timetuple())
       offset = datetime.fromtimestamp(epoch) - datetime.utcfromtimestamp(epoch)
       return utc + offset
```

效果如下：

![airflow-code-modify](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-code-modify.png?raw=true)

第2处修改``` ${PATHON_HOME}/lib/python3.6/site-packages/airflow/www/templates/airflow/dags.html ```文件。

```bash
dag.utc2local(last_run.execution_date).strftime("%Y-%m-%d %H:%M")
dag.utc2local(last_run.start_date).strftime("%Y-%m-%d %H:%M")
```

效果如下：

![airflow-code-modify-2](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-code-modify-2.png?raw=true)

<strong>修改完毕，此时可以通过重启webserver查看效果！</strong>

- 3) 添加用户认证

在这里我们采用简单的password认证方式足矣！

<strong>（1）安装password组件:</strong>
```bash
sudo pip install apache-airflow[password]
```

<strong>（2）修改airflow.cfg配置文件:</strong>
```bash
[webserver]
authenticate = True
auth_backend = airflow.contrib.auth.backends.password_auth
```

<strong>（3）编写python脚本用于添加用户账号:</strong>

编写``` add_account.py ```文件：

```python
import airflow
from airflow import models, settings
from airflow.contrib.auth.backends.password_auth import PasswordUser

user = PasswordUser(models.User())
user.username = 'airflow'
user.email = 'test_airflow@wps.cn'
user.password = 'airflow'

session = settings.Session()
session.add(user)
session.commit()
session.close()
exit()
```

执行``` add_account.py ```文件：
```python
python add_account.py
```

你会发现mysql元数据库表user中会多出来一条记录的。

- 4) 修改webserver地址

![airflow-webserver-url](https://github.com/buildupchao/ImgStore/blob/master/blog/bigdataplatform/airflow/airflow-webserver-url.png?raw=true)

### 4.3高级篇

- 1) 配置Airflow分布式集群

(未完待续...)
