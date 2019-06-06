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

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-3-new.png?raw=true)

- 通过``` dashboard ```命令可以实时查看应用监控数据

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-4.png?raw=true)

- 通过``` thread ```命令查看应用程序中所有线程情况

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-5.png?raw=true)
其中第一列为线程的ID。

- 通过``` thread threadId ```命令查看指定线程状态信息

比如我们要查看线程ID为506的线程状态信息：
![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-6.png?raw=true)

当然，因为是命令行交互，也是支持管道流式操作：
![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-7.png?raw=true)

- 通过``` sc -d *yourClassName* ```去查看JVM加载的类信息

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-8.png?raw=true)

- 通过``` jad yourFullClassName ```去查看反编译后的完整代码信息

![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-9.png?raw=true)

- 通过``` watch ```命令去查看方法的参数、返回值和异常信息

- 通过``` sm ```命令查看类的方法信息
  - case 1: ``` sm java.math.RoundingMode ```

  ![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-10.png?raw=true)

  - case 2: ``` sm -d java.math.RoundingMode ```

  ![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-11.png?raw=true)

  - case 3: ``` sm java.math.RoundingMode <init> ```

  ![](https://github.com/buildupchao/ImgStore/blob/master/blog/arthas/arthas-12.png?raw=true)
