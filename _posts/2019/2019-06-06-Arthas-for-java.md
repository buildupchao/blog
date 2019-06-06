---
title: 爱上Java诊断利器之Arthas
date: 2019-06-06 20:59:00
tags: ['Java', 'Arthas', '诊断利器']
category: Java
---

## 1.Arthas是什么？

摘自Arthas的Github介绍：
<blockquote>
  <p>Arthas is a Java Diagnostic tool open sourced by Alibaba.</p>
  <p>Arthas allows developers to troubleshoot production issues for Java applications without modifying code or restarting servers.</p>
</blockquote>

大意为：Arthas是阿里开源的一个Java诊断工具，可以帮助开发人员在不修改代码或重启服务器的情况下快速定位线上问题。

听起来确实是我们的程序员的一大福利。比如，我们就遇到一种情况，Spring Boot应用中有个cron定时任务为每天凌晨1点启动执行，但是测试起来很不方便，总不能每次修改cron时间来让QC测试吧？这样虽然是方便了测试妹子，但是却徒增了我们开发时间和迭代次数啊！！！那Arthas到底是否能够满足我们需求呢？Go on...

## 2.开启Arthas之旅

### 2.1安装Arthas

- 方式1：下载arthas-boot.jar包的方式

```shell
wget https://alibaba.github.io/arthas/arthas-boot.jar
```
此时在你当前所在目录下会有个``` arthas-boot.jar ```包。

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-1.png?raw=true)

尝试下arthas：

```shell
# 启动arthas，会进入命令行交互状态
java -jar arthas-boot.jar

# 查看arthas命令手册
java -jar arthas-boot.jar -h
```
![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-2.png?raw=true)

- 方式2：通过as.sh安装Arthas（<strong>强烈推荐</strong>）

```shell
# 该命令会下载 as.sh 到当前目录下
curl -L https://alibaba.github.io/arthas/install.sh | sh
```

尝试下arthas:

```shell
# 启动arthas，会进入命令行交互状态
./as.sh

# 查看arthas命令手册
./as.sh -h
```

### 2.2开始使用

下面演示我们以``` as.sh ```为主。

首先我们启动arthas，会查看到我们当前server上部署的应用已经被探测到，当前我的server上只有一个应用程序，只需输入数字1，即可和该应用进行交互：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-3.png?raw=true)
