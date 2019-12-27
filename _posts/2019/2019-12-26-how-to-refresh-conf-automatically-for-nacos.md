---
title: 【技术挑战】Nacos自动刷新配置如何实现的？
tags: ['tech-talking', 'technology-challenge', 'Nacos', 'Java']
category: technology-challenge
keywords: technology-challenge,技术挑战
---

**技术挑战发展进度列表：**

- [三周技术挑战约定之缘起](http://www.buildupchao.cn/technology-challenge/2019/12/23/technology-challenge-for-3-weeks.html)
- [【技术挑战】Nacos自动刷新配置如何实现的？](http://www.buildupchao.cn/technology-challenge/2019/12/26/how-to-refresh-conf-automatically-for-nacos.html)

<!-- more -->

## Nacos是什么？

摘自Nacos官网：

> Nacos is committed to help you discover, configure, and manage your microservices. It provides a set of simple and useful features enabling you to realize dynamic service discovery, service configuration, service metadata and traffic management.

大意为：Nacos致力于帮助你发现、配置、管理微服务。Nacos提供了一系列简单易用的特性，它们可以帮助你实现动态服务发现、服务配置、服务元数据以及流量管理。

> Key features of Nacos:
> Service Discovery And Service Health Check
> Dynamic configuration management
> Dynamic DNS service
> Service governance and metadata management

大意为：Nacos的主要特性：服务发现和服务的健康检查，动态配置管理，动态DNS服务，服务治理和元数据管理。

综上，我们大概可以知道Nacos是致力于动态服务发现、配置管理、服务元数据以及流量管理的平台。相比大多数人已经对这几个关键字耳熟能详了，就不做过多解释，详情可根据文章末尾资料链接前往官网查看。

## 开始邂逅Nacos

为了方便测试，我们采用自构建jar包的方式使用Nacos

- 从Github下载Nacos代码：https://github.com/alibaba/nacos.git

{% highlight shell %}
git clone https://github.com/alibaba/nacos.git
{% endhighlight %}

- 打包下载好的nacos项目目录，执行编译打包命令（默认你已安装配置好Java环境和Maven环境）

{% highlight shell %}
cd your_clone_nacos_project_dir
mvn -Prelease-nacos clean install -U -Dmaven.test.skip=true
{% endhighlight %}

- 打包完成，进入your_clone_nacos_project_dir/distribution/target/目录，会看到如下文件列表
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-jar.png?raw=true)
<br/>
- 拷贝nacos-server-1.2.0-SNAPSHOT.tar.gz（xxx.zip包亦可）到你的服务器或者本地应用部署目录下然后解压缩

{% highlight shell %}
mv nacos-server-1.2.0-SNAPSHOT.tar.gz your_app_deploy_dir
cd your_app_deploy_dir
tar -zxvf nacos-server-1.2.0-SNAPSHOT.tar.gz
{% endhighlight %}

解压后，目录如下：
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-server-dir.png?raw=true)
<br/>
- 启动nacos-server，启动/关闭脚本在bin/目录下

关于nacos server的启动方式有两种：#1，采用单机模式，#2，集群模式。鉴于我们只是学习研究使用，无需采用集群模式（因为集群模式还需进行一系列配置），直接采用单机模式即可。

{% highlight shell %}
sh startup.sh -m standalone
{% endhighlight %}

启动后会看到如下信息：
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/start-nacos-server.png?raw=true)
<br/>
然后查看日志是否有报错，有错误信息，一般很简单易解决，实在不懂可自行google或者咨询nacos管理员。<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-log-console-page.png?raw=true)
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-log-info.png?raw=true)
<br/>
此外，通过上图日志，我们可以知道nacos-server启动在8848端口（该端口我们后续会使用）。<br/>
其次，我们可以获取到另外两项信息：进程号以及Console控制台访问链接。
<br/>


## 从客户端潜入Nacos实时更新配置原理

## 从服务端潜入Nacos实时更新配置原理

## 结尾

## 资料链接

- Nacos官网：[https://nacos.io/en-us/docs/what-is-nacos.html](https://nacos.io/en-us/docs/what-is-nacos.html)
- Nacos Github: [https://github.com/alibaba/nacos](https://github.com/alibaba/nacos)
