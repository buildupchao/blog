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
然后查看日志是否有报错，有错误信息，一般很简单已解决，实在不懂可自行google或者咨询nacos管理员。<br/>
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-log-info.png?raw=true)
<br/>
此外，通过上图日志，我们可以知道nacos-server启动在8848端口（该端口我们后续会使用）。<br/>
<br/>
通过参考nacos下的example项目中NamingExample代码，在nacos下的test项目中编写如下代码进行发布、订阅服务数据：<br/>
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-example-code.png?raw=true)
<br/>
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
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-service-info.png?raw=true)
<br/><br/>
然后我们启动SubscribeService服务，可以在控制台查看到订阅到的数据信息:
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/subscribe-nacos-service-console-info.png?raw=true)
<br/>
其实，和上面``` curl ```命令获取到的服务信息一样。

## 熟悉游走于Nacos脉络结构之中

接下来，我们将开始潜入Nacos内部，去查看注册发布一个服务的流程活动。

- 首先，从PublishService的``` NamingFactory.createNamingService ```方法潜入Nacos
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-1.png?raw=true)
<br/>
- 显然，``` NamingFactory.createNamingService(String serverList) ```方法通过反射调用了NacosNamingService的构造方法
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-2.png?raw=true)
<br/>
- 我们前往nacos-client模块下查看``` com.alibaba.nacos.client.NacosNamingService ```类，发现，除了构建参数外，还调用了init方法
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-3.png?raw=true)
<br/>
- 开始查看``` NacosNamingService#init ```方法到底做了什么
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-4.png)
<br/>
接下来我们主要说明几处关键代码：

(1) InitUtils.initWebRootContext()方法主要是构建了web context上下文，也就是上面你通过``` curl ```获取服务信息时候的 ``` nacos/v1/ns/instance ```
{% highlight Java %}
InitUtils.initWebRootContext();

InitUtils:
public static void initWebRootContext() {
    // support the web context with ali-yun if the app deploy by EDAS
    final String webContext = System.getProperty(SystemPropertyKeyConst.NAMING_WEB_CONTEXT);
    TemplateUtils.stringNotEmptyAndThenExecute(webContext, new Runnable() {
        @Override
        public void run() {
            UtilAndComs.WEB_CONTEXT = webContext.indexOf("/") > -1 ? webContext
                : "/" + webContext;

            UtilAndComs.NACOS_URL_BASE = UtilAndComs.WEB_CONTEXT + "/v1/ns";
            UtilAndComs.NACOS_URL_INSTANCE = UtilAndComs.NACOS_URL_BASE + "/instance";
        }
    });
}
{% endhighlight %}

(2) EventDispatcher用途

首先，启动了一个Notify线程：
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-5.png?raw=true)
<br/>
然后，Notify线程会一直不断轮询changedServices这个BlockingQueue。如果有Service相关信息改动，服务端就会推送下来数据到该BlokingQueue，而Notify就会从BlockingQueue获取到相关Service信息，然后拿到根据Service信息拿到EventListener列表，调用EventListener的回调函数onEvent。原理还是比较简单的，类似于生产者消费者 + 观察者模式。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-6.png?raw=true)
<br/>
(3) NamingProxy用途

NamingProxy构造方法除了初始化一些endpoint和namespace以及server地址外，就是调用了``` initRefreshSrvIfNeed ```方法。在``` initRefreshSrvIfNeed ```方法中是启动了一个定时线程，每30秒执行一次``` refreshSrvIfNeed ```方法。而``` refreshSrvIfNeed ```方法则是构建一个http请求，去nacos server获取一串nacos server集群的地址列表。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-7.png?raw=true)
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-8.png?raw=true)
<br/>

(4) BeatReactor用途

根据名字我们应该知道其肯定和心跳相关。没错，```BeatReactor```构造方法初始化了一个定时线程池，当我们调用```NamingService.registerInstance```方法时，就会自动调用```BeatReactor#addBeatInfo(serviceName,beatInfo)```方法启动心跳线程，时间间隔为5s。也就是说，每隔5s，去执行一次BeatTask里面的代码逻辑，而BeatTask是循环的去获取当前客户端注册好的实例，然后向服务端发送一个http的心跳通知请求，告诉客户端这个服务的健康状态。
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-9.png?raw=true)
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-11.png?raw=true)
<br/>
这里就是nacos客户端主动上报服务健康状况的逻辑了，是服务发现功能中比较重要的一个概念，即服务健康检查机制。常用的服务健康检查还有服务端主动去探测客户端的接口返回（比如K8S管理docker内服务时）。
<br/>
(5) HostReactor用途

![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-10.png?raw=true)

其中，除了根据配置来决定是否项目启动就从本地缓存目录加载服务信息列表，就是FailoverReactor和PushReceiver组件了。
<br/>
FailoverReactor组件是nacos客户端缓存容灾相关的，代码如下：
<br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-12.png?raw=true)
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-13.png?raw=true)
<br/><br/>
由上图可知，```FailoverReactor#init```方法中初始化了3个定时任务。
<br/>
第1个定时任务，首先判断是否有容灾开关，读取到容灾开关后，将值更新到内存中，后续解析地址列表时，首先会判断一下容灾开关是否打开，如果打开就读取缓存数据，否则从服务器获取最新数据。(容灾开关是一个磁盘文件的形式存在，通过容灾开关文件名字，判定容灾开关是否打开，1表示打开，0为关闭。)<br/>
{% highlight Java %}
class SwitchRefresher implements Runnable {
    long lastModifiedMillis = 0L;

    @Override
    public void run() {
        try {
            File switchFile = new File(failoverDir + UtilAndComs.FAILOVER_SWITCH);
            if (!switchFile.exists()) {
                switchParams.put("failover-mode", "false");
                NAMING_LOGGER.debug("failover switch is not found, " + switchFile.getName());
                return;
            }

            long modified = switchFile.lastModified();

            if (lastModifiedMillis < modified) {
                lastModifiedMillis = modified;
                String failover = ConcurrentDiskUtil.getFileContent(failoverDir + UtilAndComs.FAILOVER_SWITCH,
                    Charset.defaultCharset().toString());
                if (!StringUtils.isEmpty(failover)) {
                    List<String> lines = Arrays.asList(failover.split(DiskCache.getLineSeparator()));

                    for (String line : lines) {
                        String line1 = line.trim();
                        if ("1".equals(line1)) {
                            switchParams.put("failover-mode", "true");
                            NAMING_LOGGER.info("failover-mode is on");
                            new FailoverFileReader().run();
                        } else if ("0".equals(line1)) {
                            switchParams.put("failover-mode", "false");
                            NAMING_LOGGER.info("failover-mode is off");
                        }
                    }
                } else {
                    switchParams.put("failover-mode", "false");
                }
            }

        } catch (Throwable e) {
            NAMING_LOGGER.error("[NA] failed to read failover switch.", e);
        }
    }
}
{% endhighlight %}
<br/>
第2个定时任务，每隔24小时把内存中所有的服务数据，写一遍到磁盘中。其中，需要过滤掉一些非域名数据的特殊数据。<br/>
{% highlight Java %}
class DiskFileWriter extends TimerTask {
    @Override
    public void run() {
        Map<String, ServiceInfo> map = hostReactor.getServiceInfoMap();
        for (Map.Entry<String, ServiceInfo> entry : map.entrySet()) {
            ServiceInfo serviceInfo = entry.getValue();
            if (StringUtils.equals(serviceInfo.getKey(), UtilAndComs.ALL_IPS) || StringUtils.equals(
                serviceInfo.getName(), UtilAndComs.ENV_LIST_KEY)
                || StringUtils.equals(serviceInfo.getName(), "00-00---000-ENV_CONFIGS-000---00-00")
                || StringUtils.equals(serviceInfo.getName(), "vipclient.properties")
                || StringUtils.equals(serviceInfo.getName(), "00-00---000-ALL_HOSTS-000---00-00")) {
                continue;
            }

            DiskCache.write(serviceInfo, failoverDir);
        }
    }
}
{% endhighlight %}
<br/>
第3个定时任务，每隔10秒钟检查一次缓存目录是否存在，同时如果缓存目录里面值没有的话，主动触发一次缓存写磁盘的操作。<br/>
{% highlight Java %}
// backup file on startup if failover directory is empty.
executorService.schedule(new Runnable() {
    @Override
    public void run() {
        try {
            File cacheDir = new File(failoverDir);

            if (!cacheDir.exists() && !cacheDir.mkdirs()) {
                throw new IllegalStateException("failed to create cache dir: " + failoverDir);
            }

            File[] files = cacheDir.listFiles();
            if (files == null || files.length <= 0) {
                new DiskFileWriter().run();
            }
        } catch (Throwable e) {
            NAMING_LOGGER.error("[NA] failed to backup file on startup.", e);
        }

    }
}, 10000L, TimeUnit.MILLISECONDS);
{% endhighlight %}
<br/>
最后，就是关于```PushReceiver```组件了。这个组件主要是做变化通知的。而巧妙的地方就在于变化的通知push采用的是udp。这种比tcp长连接好很多，比http longpoling、grpc的http/2也要好很多。的确，通知不多的情况下，想到用udp也是极巧妙的。
<br/>
综上，我们已经大体游走了一番Nacos的脉络结构。我们会发现其中大部分都是在进行初始化多个线程池或者定时任务，各司其职。这其实也是我们后端开发的一惯性套路，用于提高系统的并发能力，同时也对任务进行分发和执行。

## 回味Nacos自动刷新配置的实现

## 意味深长的望着Nacos

## 结尾

## 资料链接

- Nacos官网：[https://nacos.io/en-us/docs/what-is-nacos.html](https://nacos.io/en-us/docs/what-is-nacos.html)
- Nacos Github: [https://github.com/alibaba/nacos](https://github.com/alibaba/nacos)
