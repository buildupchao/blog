---
title: 【技术挑战】Nacos自动刷新配置如何实现的？
tags: ['tech-talking', 'technology-challenge', 'Nacos', 'Java']
category: technology-challenge
keywords: technology-challenge,技术挑战
---

**技术挑战发展进度列表：**

- [三周技术挑战约定之缘起](http://www.buildupchao.cn/technology-challenge/2019/12/23/technology-challenge-for-3-weeks.html)
- [【技术挑战】Nacos自动刷新配置如何实现的？](http://www.buildupchao.cn/technology-challenge/2019/12/26/how-to-refresh-conf-automatically-for-nacos.html)


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

## HelloWorld之开始邂逅Nacos

为了方便测试，我们采用自构建jar包的方式使用Nacos

- 从Github下载Nacos代码：https://github.com/alibaba/nacos.git

{% highlight shell %}
git clone https://github.com/alibaba/nacos.git
{% endhighlight %}

- 打包下载好的nacos项目目录，执行编译打包命令

{% highlight shell %}
cd your_clone_nacos_project_dir
mvn -Prelease-nacos clean install -U -Dmaven.test.skip=true
{% endhighlight %}

- 打包完成，进入your_clone_nacos_project_dir/distribution/target/目录，会看到如下文件列表

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-jar.png?raw=true)

- 拷贝nacos-server-1.2.0-SNAPSHOT.tar.gz（xxx.zip包亦可）到你的服务器或者本地应用部署目录下然后解压缩

{% highlight shell %}
mv nacos-server-1.2.0-SNAPSHOT.tar.gz your_app_deploy_dir
cd your_app_deploy_dir
tar -zxvf nacos-server-1.2.0-SNAPSHOT.tar.gz
{% endhighlight %}

解压后，目录如下：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-server-dir.png?raw=true)

- 启动nacos-server，启动/关闭脚本在bin/目录下

关于nacos server的启动方式有两种：#1，采用单机模式，#2，集群模式。鉴于我们只是学习研究使用，无需采用集群模式（因为集群模式还需进行一系列配置），直接采用单机模式即可。

{% highlight shell %}
sh startup.sh -m standalone
{% endhighlight %}

启动后会看到如下信息：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/start-nacos-server.png?raw=true)

然后查看日志是否有报错，有错误信息，一般很简单已解决，实在不懂可自行google或者咨询nacos管理员。<br/>

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-log-info.png?raw=true)

此外，通过上图日志，我们可以知道nacos-server启动在8848端口（该端口我们后续会使用）。<br/>
<br/>
通过参考nacos下的example项目中NamingExample代码，在nacos下的test项目中编写如下代码进行发布、订阅服务数据：<br/>

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-example-code.png?raw=true)

发布一个HelloWorld服务（使用NamingService为:127.0.0.1:8848）:

{% highlight Java %}
package com.alibaba.nacos.test.hello;

import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingFactory;
import com.alibaba.nacos.api.naming.NamingService;

import java.util.concurrent.TimeUnit;

/**
 * @author buildupchao
 * @date 2019/12/24 01:52
 * @since JDK 1.8
 */
public class PublishService {

    public static void main(String[] args) throws NacosException, InterruptedException {
        // the name of service
        String serviceName = "HelloWorld";
        // construct a nacos instance with 127.0.0.1:8848
        NamingService namingService = NamingFactory.createNamingService("127.0.0.1:8848");
        // register a service with 1.1.1.1:8888
        namingService.registerInstance(serviceName, "1.1.1.1", 8888);
        // let main thread sleep so as to keeping heartbeat.
        TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
    }
}
{% endhighlight %}

订阅HelloWorld服务（使用NamingService为:127.0.0.1:8848）：

{% highlight Java %}
package com.alibaba.nacos.test.hello;

import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingFactory;
import com.alibaba.nacos.api.naming.NamingService;
import com.alibaba.nacos.api.naming.listener.NamingEvent;

import java.util.concurrent.TimeUnit;

/**
 * @author buildupchao
 * @date 2019/12/24 01:52
 * @since JDK 1.8
 */
public class SubscribeService {

    public static void main(String[] args) throws NacosException, InterruptedException {
        // the name of service
        String serviceName = "HelloWorld";
        // construct a nacos instance with 127.0.0.1:8848
        NamingService namingService = NamingFactory.createNamingService("127.0.0.1:8848");
        // subscribe service and listen to event data.
        namingService.subscribe(serviceName, event -> {
            if (event instanceof NamingEvent) {
                System.out.printf("subscribe data: [%s]", ((NamingEvent) event).getInstances());
            }
        });
        // let main thread sleep so as to seeing what Sub subscribes.
        TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
    }
}
{% endhighlight %}

我们先来启动PublishService服务，然后通过terminal命令行键入如下命令，可以获取到HelloWorld服务信息：

{% highlight shell %}
curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=HelloWorld'
{% endhighlight %}

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-service-info.png?raw=true)

然后我们启动SubscribeService服务，可以在控制台查看到订阅到的数据信息:

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/subscribe-nacos-service-console-info.png?raw=true)

其实，和上面``` curl ```命令获取到的服务信息一样。

## 熟悉游走于Nacos脉络结构之中

接下来，我们将开始潜如Nacos内部，去查看注册发布一份服务的流程活动。

## 回味Nacos自动刷新配置的实现

## 意味深长的望着Nacos

## 结尾

## 资料链接

- Nacos官网：[https://nacos.io/en-us/docs/what-is-nacos.html](https://nacos.io/en-us/docs/what-is-nacos.html)
- Nacos Github: [https://github.com/alibaba/nacos](https://github.com/alibaba/nacos)
