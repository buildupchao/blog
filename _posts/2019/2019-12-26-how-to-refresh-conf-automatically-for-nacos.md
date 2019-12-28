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

> 文章很长，请做好心理准备。

## 1.Nacos是什么？

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

## 2.开始邂逅Nacos

### 2.1 初涉Nacos

为了方便测试，我们采用自构建jar包的方式使用Nacos

- 从Github下载Nacos代码：http://github.com/alibaba/nacos.git

{% highlight shell %}
git clone http://github.com/alibaba/nacos.git
{% endhighlight %}

- 打包下载好的nacos项目目录，执行编译打包命令（默认你已安装配置好Java环境和Maven环境）

{% highlight shell %}
cd your_clone_nacos_project_dir
mvn -Prelease-nacos clean install -U -Dmaven.test.skip=true
{% endhighlight %}

- 打包完成，进入your_clone_nacos_project_dir/distribution/target/目录，会看到如下文件列表
<br/>
![](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-jar.png?raw=true)
<br/>
- 拷贝nacos-server-1.2.0-SNAPSHOT.tar.gz（xxx.zip包亦可）到你的服务器或者本地应用部署目录下然后解压缩

{% highlight shell %}
mv nacos-server-1.2.0-SNAPSHOT.tar.gz your_app_deploy_dir
cd your_app_deploy_dir
tar -zxvf nacos-server-1.2.0-SNAPSHOT.tar.gz
{% endhighlight %}

解压后，目录如下：
<br/>
![](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-server-dir.png?raw=true)
<br/>
- 启动nacos-server，启动/关闭脚本在bin/目录下

关于nacos server的启动方式有两种：#1，采用单机模式，#2，集群模式。鉴于我们只是学习研究使用，无需采用集群模式（因为集群模式还需进行一系列配置），直接采用单机模式即可。

{% highlight shell %}
sh startup.sh -m standalone
{% endhighlight %}

启动后会看到如下信息：
<br/>
![](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/start-nacos-server.png?raw=true)
<br/>
然后查看日志是否有报错，有错误信息，一般很简单易解决，实在不懂可自行google或者咨询nacos管理员。<br/><br/>
![](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-log-console-page.png?raw=true)
<br/>
![](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/nacos-log-info.png?raw=true)
<br/>
此外，通过上图日志，我们可以知道nacos-server启动在8848端口（该端口我们后续会使用）。<br/>
其次，我们可以获取到另外两项信息：进程号以及Console控制台访问链接。
<br/><br/>
接下来，我们先来体验下nacos动态配置功能：<br/>
首先，我们对nacos example模块代码中```com.alibaba.nacos.example```包下的``` ConfigExample ```进行修改并启动。<br/>
修改后代码如下所示，其中``` TimeUnit.SECONDS.sleep(Integer.MAX_VALUE) ```代码是为了防止主线程退出而无法获取配置修改信息：<br/>

{% highlight Java %}
package com.alibaba.nacos.example;

import java.util.Properties;
import java.util.concurrent.Executor;
import java.util.concurrent.TimeUnit;

import com.alibaba.nacos.api.NacosFactory;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.config.listener.Listener;
import com.alibaba.nacos.api.exception.NacosException;
/**
 * Config service example
 *
 * @author Nacos
 */
public class ConfigExample {

    public static void main(String[] args) throws NacosException, InterruptedException {
        String serverAddr = "localhost";
        String dataId = "test";
        String group = "DEFAULT_GROUP";
        Properties properties = new Properties();
        properties.put("serverAddr", serverAddr);
        ConfigService configService = NacosFactory.createConfigService(properties);
        String content = configService.getConfig(dataId, group, 5000);
        System.out.printf("got content: %s\n", content);
        configService.addListener(dataId, group, new Listener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                System.out.printf("time: %d, receive: %s\n", System.currentTimeMillis(), configInfo);
            }

            @Override
            public Executor getExecutor() {
                return null;
            }
        });

        TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
    }
}
{% endhighlight %}
<br/>
控制台显示信息如下：
![deep-in-nacos-new-config-1](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-new-config-1.png?raw=true)
<br/><br/>
然后，登录nacos控制台，初始化默认账号/密码为：nacos/nacos，并新增一个配置信息。<br/>
![deep-in-nacos-new-config-2](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-new-config-2.png?raw=true)
<br/>
![deep-in-nacos-new-config-3](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-new-config-3.png?raw=true)
<br/><br/>
我们将在控制台查看到获取到如下配置信息：
<br/>
![deep-in-nacos-new-config-4](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-new-config-4.png?raw=true)
<br/><br/>
接下来，我们修改一次配置信息内容，并查看配置客户端listener是否可以获取到对应修改：
<br/>
![deep-in-nacos-update-config-1](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-update-config-1.png?raw=true)
<br/>
![deep-in-nacos-update-config-2](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-update-config-2.png?raw=true)
<br/><br/>
显然，客户端会获取到最新数据。到此为止，已经完成一个简单的动态配置管理功能（删除类似，不再敖述）。<br/>

### 2.1 小结

- Nacos服务端保存配置信息
- 客户端连接到服务端之后，通过dataId和group获取具体的配置信息
- 当服务端配置信息发生变更时，客户端会收到通知
- 客户端拿到变更后的配置信息后，然后做相应处理

那客户端是如何感知到nacos服务端配置信息变更呢？也就是说，客户端和服务端的数据交互是如何实现的。而根据经验来说，通常是两种交互方式：#1,服务端主动推送数据；#2，客户端从服务端拉取数据。具体如何实现，我们下面开始逐渐深入了解。

## <span id="client-rt-load-config">3.从客户端潜入Nacos实时更新配置原理</span>

### 3.1 ConfigFactory

- 首先，在``` ConfigExample ```代码中，我们首先看到通过``` NacosFactory.createConfigService(properties) ```创建了一个``` ConfigService ```，而``` NacosFactory ```是个工厂类，底层调用了对应xxxFactory的createxxxService方法

![deep-in-nacos-for-client-1](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-1.png?raw=true)
<br/>
可以看到实际上是调用了``` ConfigFactory#createConfigService ```，通过反射调用了带有一个Properties参数的``` NacosConfigService ```的构造方法来创建``` ConfigService ```。而且，此处并没有对实例进行缓存也没有采用单例模式，每次调用均会创建一个``` ConfigService ```实例。

### 3.2 NacosConfigService

- 接下来，我们来查看nacos client模块下``` com.alibaba.nacos.client.config ```包下的``` NacosConfigService ```构造方法都做了些什么工作

![deep-in-nacos-for-client-2](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-2.png?raw=true)
<br/>
由上图可知，``` NacosConfigService ```构造方法除去初始化命名空间，主要是做了两件事：<br/>
（1）采用装饰器模式，将ServerHttpAgent实例包装成MetricsHttpAgent对象<br/>
（2）实例化ClientWorker
<br/><br/>
#### 3.2.1 MetricsHttpAgent和ServerHttpAgent

MetricsHttpAgent和ServerHttpAgent均实现了```HttpAgent```接口。针对上述的agent，MetricsHttpAgent只是对ServerHttpAgent进行了包装，增加了一些耗时统计操作，实际上工作的类是ServerHttpAgent。而ServerHttpAgent构造方法中主要是初始化了```ServerListManager```用于获取nacos-server地址信息和初始化其他属性（encode、aksk、maxRetry）。
<br/><br/>
此外，agent作为参数传入ClientWorker构造方法，之后在ClientWorker中发挥作用。

#### 3.2.2 ClientWorker

![deep-in-nacos-for-client-3](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-3.png?raw=true)
<br/>
从上述代码可以看到，``` ClientWorker ```构造方法中，除了将```HttpAgent```和```ConfigFilterChainManager```维持在自己内部，还初始化了两个线程池：<br/>
（1）第1个线程池是只拥有一个线程且用于定时执行任务。每间隔10ms执行一次```checkConfigInfo()```方法，用于检查配置信息<br/>
（2）第2个线程池是用于长轮询的普通线程池（并未采用定时功能）。
<br/><br/>
接下来，我们来查看下```checkConfigInfo()```方法内部实现：<br/><br/>
![deep-in-nacos-for-client-4](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-4.png?raw=true)
<br/>
可以看出，``` checkConfigInfo() ```会取出来一部分任务，通过```executorService```线程池去执行```LongPollingRunnable```任务，且每个任务有一个taskId，``` LongPollingRunnable ```中会根据taskId获取``` CacheData ```。
<br/><br/>
接下来，我们看一下``` LongPollingRunnable ```内部实现：<br/>

{% highlight Java %}
class LongPollingRunnable implements Runnable {
    private int taskId;

    public LongPollingRunnable(int taskId) {
        this.taskId = taskId;
    }

    @Override
    public void run() {

        List<CacheData> cacheDatas = new ArrayList<CacheData>();
        List<String> inInitializingCacheList = new ArrayList<String>();
        try {
            // check failover config
            for (CacheData cacheData : cacheMap.get().values()) {
                if (cacheData.getTaskId() == taskId) {
                    cacheDatas.add(cacheData);
                    try {
                        checkLocalConfig(cacheData);
                        if (cacheData.isUseLocalConfigInfo()) {
                            cacheData.checkListenerMd5();
                        }
                    } catch (Exception e) {
                        LOGGER.error("get local config info error", e);
                    }
                }
            }

            // check server config
            List<String> changedGroupKeys = checkUpdateDataIds(cacheDatas, inInitializingCacheList);

            for (String groupKey : changedGroupKeys) {
                String[] key = GroupKey.parseKey(groupKey);
                String dataId = key[0];
                String group = key[1];
                String tenant = null;
                if (key.length == 3) {
                    tenant = key[2];
                }
                try {
                    String content = getServerConfig(dataId, group, tenant, 3000L);
                    CacheData cache = cacheMap.get().get(GroupKey.getKeyTenant(dataId, group, tenant));
                    cache.setContent(content);
                    LOGGER.info("[{}] [data-received] dataId={}, group={}, tenant={}, md5={}, content={}",
                        agent.getName(), dataId, group, tenant, cache.getMd5(),
                        ContentUtils.truncateContent(content));
                } catch (NacosException ioe) {
                    String message = String.format(
                        "[%s] [get-update] get changed config exception. dataId=%s, group=%s, tenant=%s",
                        agent.getName(), dataId, group, tenant);
                    LOGGER.error(message, ioe);
                }
            }
            for (CacheData cacheData : cacheDatas) {
                if (!cacheData.isInitializing() || inInitializingCacheList
                    .contains(GroupKey.getKeyTenant(cacheData.dataId, cacheData.group, cacheData.tenant))) {
                    cacheData.checkListenerMd5();
                    cacheData.setInitializing(false);
                }
            }
            inInitializingCacheList.clear();

            executorService.execute(this);

        } catch (Throwable e) {

            // If the rotation training task is abnormal, the next execution time of the task will be punished
            LOGGER.error("longPolling error : ", e);
            executorService.schedule(this, taskPenaltyTime, TimeUnit.MILLISECONDS);
        }
    }
}
{% endhighlight %}
<br/>
由代码可以看出，主要分为两部分内容：#1，检查本地配置信息（check failover config）；#2，获取到服务端配置信息并更新到本地（check server config）
<br/><br/>
（1）检查本地配置信息（check failover config）<br/>

首先，根据taskId获取到``` CacheData ```，然后对``` CacheData ```进行检查，主要是进行本地配置检查和监听器的md5检查。其中，本地检查主要是做一个故障容错，当服务端挂掉时，Nacos客户端可以从本地文件系统获取相关配置信息。而Nacos配置信息存储在如下目录中：<br/><br/>
![deep-in-nacos-for-client-5](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-5.png?raw=true)
<br/><br/>

（2）检查服务端配置信息（check server config）<br/>

![deep-in-nacos-for-client-6](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-6.png?raw=true)
<br/><br/>
首先，通过调用``` checkUpdateDataIds(cacheDatas, inInitializingCacheList) ```方法，从Server获取值变化了的DataID列表。<br/>
然后，通过调用``` getServerConfig(dataId, group, tenant, 3000L) ```方法，从Server获取最新的配置信息且把最新的配置信息保存到``` CacheData ```中。<strong style="color:red;">而```CacheData#setContent```中不仅会保存最新配置信息，还会更新该```CacheData```的md5值。</strong><br/>
![deep-in-nacos-for-client-10](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-10.png?raw=true)
<br/>
最后，调用``` cacheData.checkListenerMd5() ```方法（既然上面更新了md5值，那么这里进行check也就不足为奇了）。<br/>
此外，任务最后又重新提交了本任务 ``` executorService.execute(this); ```。
<br/><br/>

到此，我们就已经完成了``` ConfigService ```的创建，接下来就可以为该``` ConfigService ```添加一个``` Listener ```。``` ConfigService#addListener ```底层是调用了 ``` ClientWorker#addTenantListeners ```方法。
<br/>
![deep-in-nacos-for-client-7](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-7.png?raw=true)
<br/>
![deep-in-nacos-for-client-8](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-8.png?raw=true)
<br/>
![deep-in-nacos-for-client-9](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-9.png?raw=true)
<br/><br/>
由此观察，``` ClientWorker#addTenantListeners ```方法主要做了两件事：#1，根据dataId、group和tenant去获取```CacheData```对象；#2，将当前要添加的```Listener```对象添加到``` CacheData ```中（即```CacheData```持有```Listener```，所以可以回调```Listener#receiveConfigInfo```方法）。另外，``` CacheData#addListener ```方法中会将listener与CacheData的md5属性值一起作为参数构建```ManagerListenerWrap```对象并存储到``` CacheData```的listeners列表。
<br/><br/>
![deep-in-nacos-for-client-11](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-11.png?raw=true)
<br/><br/>
现在，我们已经大概了解了``` ConfigService ```，但是，还要一个问题待解决：```Listener```的回调方法```receiveConfigInfo```是在哪里被调用的？动脑子想想啊，既然要回调，肯定是检测到配置信息有变动了啊，那检测在哪里发生的？答案显而易见，没错，就是```CacheData#checkListenerMd5```。So cool，让我们开始从```LongPollingRunnable#run``` -> ```CacheData#checkListenerMd5```，checkListenerMd5代码如下：
<br/>
{% highlight Java %}
void checkListenerMd5() {
    for (ManagerListenerWrap wrap : listeners) {
        if (!md5.equals(wrap.lastCallMd5)) {
            safeNotifyListener(dataId, group, content, md5, wrap);
        }
    }
}
{% endhighlight %}
<br/>
通过代码可知，checkListenerMd5方法会检查```CacheData```当前的md5值与```CacheData```所持有的listener中保存的md5值是否一致，如果不一致，那么就会调用safeNotifyListener。看名字，应该是通知Listener的使用者，该Listener所监听的配置信息发生了变更。接下来，还是让我们看看safeNatofyListener代码，再得出最终结论吧（主要关注3行代码，完整详细代码请自行阅读源码）:
<br/>
![deep-in-nacos-for-client-12](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-client-12.png?raw=true)
<br/><br/>
如上，safeNatifyListener主要的三个步骤：#1，获取最新的配置信息；#2，调用Listener#receiveConfigInfo回调方法；#3，最后更新listenerWrap的md5值。Yahho~果然如此，长轮询内进行md5值比对后会决定是否触发回调。
<br/>

### 3.3 小结

到此为止，从客户端对配置中心的完整流程已经分析关闭，我们做一个小结：<br/>
- Nacos服务端创建一个配置后，客户端可对此配置信息进行监听;
- 客户端通过定时任务每间隔10ms来检查配置信息是否变更;
- 服务端配置信息发生变更，客户端将会获取到变更的数据，并将新的配置数据更新到CacheData中，并计算CacheData的新的md5属性值;
- 比较CacheData的新的md5值是否和所持有的listeners的md5值一致，不一致，则回调listener的receiveConfigInfo方法并更新listenerWrap的md5值;
- 其中，出于对服务端故障的考虑，客户端会将最新数据获取后会保存在本地的snapshot文件中，在此之后会优先从本地文件中获取配置信息。

<br/><br/>

## 4.从服务端潜入Nacos实时更新配置原理

### 4.1 ConfigServletInner && ClientLongPolling

我们从哪里作为切入点进行分析呢？还记得```ClientWorker#checkUpdateDataIds```方法吧，里面会去请求调用Nacos服务端API来获取值变化了的DataID列表。所以，我们可以轻易从```ClientWorker#checkUpdateDataIds``` -> ```ClientWorker#checkUpdateConfigStr``` -> ```HttpResult result = agent.httpPost(Constants.CONFIG_CONTROLLER_PATH + "/listener", headers, params, agent.getEncode(), readTimeoutMs);```，到此，我们可以得出客户端通过http请求的服务端API为``` v1/cs/configs/listener ```。我们直接从通过IDEA的双击Shift键进行快速查询定位该API所在位置，要记得当前是POST请求而不是GET请求哦。
<br/>
![deep-in-nacos-for-server-1](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-1.png?raw=true)
<br/><br/>
然后，我们可以定位到nacos config模块下```com.alibaba.nacos.config.server.controller.ConfigController#listener```方法，通过注释知道该API是用于比较MD5的。其中，通过对httpervletRequest参数进行解析转换后，交给inner对象去做长轮询。
<br/>
![deep-in-nacos-for-server-2](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-2.png?raw=true)
<br/><br/>
接下来，我们看一下```inner.doPollingConfig(request, response, clientMd5Map, probeModify.length())```内部主要做了什么呢？其中，该 inner 对象是``` com.alibaba.nacos.config.server.controller.ConfigServletInner ```。
<br/>
![deep-in-nacos-for-server-3](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-3.png?raw=true)
<br/><br/>
通过查看```ConfigServletInner#doPollingConfig```不仅支持长轮询，也支持短轮询的逻辑。在此，我们只看长轮询部分。
<br/>
![deep-in-nacos-for-server-4](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-4.png?raw=true)
<br/><br/>
我们会发现``` LongPollingService#addLongPollingClient ```中最后一行代码是把客户端的长轮询请求封装成一个```ClientLongPolling```对象提交给```scheduler```去异步执行。
<br/>
但是，有个问题：<strong style="color:red;">服务端拿到客户端指定的超时时间后，为何要减去500ms作为timeout时间呢？</strong>（注意：这里如果```isFixedPolling()```方法为true，timeout会是一个固定的时间间隔，默认是10000ms，也就是10s）<br/><br/>
我们先记录这个问题，然后继续往下走，去查看```ClientLongPolling```主要做了什么：
<br/>
![deep-in-nacos-for-server-5](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-5.png?raw=true)
<br/><br/>
我们发现```ClientLongPolling```主要做了这几件事：<br/>
（1）创建一个延迟调度任务，延迟时间为上述计算的timeout<br/>
（2）将该```ClientLongPolling```添加到```allSubs```中<br/>
（3）延迟时间到后，会将该```ClientLongPolling```实例从```allSubs```中删除（删除订阅关系）<br/>
（4）获取服务端中保存的对应客户端请求且未发生变更的changedGroupKeys，将其写入到reponse返回给客户端（之后客户端拿到changeGroupKeys所做的操作在[从客户端潜入Nacos实时更新配置原理](http://www.buildupchao.cn/technology-challenge/2019/12/26/how-to-refresh-conf-automatically-for-nacos.html#client-rt-load-config)部分已经解析）
<br/><br/>
这里有个疑问，为什么已经有延迟执行了，还要做一下```allSubs```添加、删除```ClientLongPolling```实例的操作呢？看代码注释提示是<strong style="color:red;">删除订阅关系</strong>，我们可以知道```ClientLongPolling```是被订阅的，但是这个订阅关系指的又是什么呢？我们猜测一下，之前对客户端实时更新配置分析时，我们知道一旦配置更新，客户端能立即得到变更信息，而服务端这里却是有timeout时延的，所以，这种订阅关系是不是和配置变更有关系呢？这里仅仅是猜测，我们依然先记录这个问题，继续从代码中寻求答案。

### 4.2 从更改配置操作作为切入点

<br/>
![deep-in-nacos-for-server-6](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-6.png?raw=true)
<br/><br/>
由浏览前F12请求情况可知，更新配置，会调用```POST:/nacos/v1/cs/configs```API，具体的方法时```ConfigController#publishConfig```方法：
<br/>
![deep-in-nacos-for-server-7](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-7.png?raw=true)
<br/><br/>
![deep-in-nacos-for-server-8](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-8.png?raw=true)
<br/><br/>
通过上述代码可知，修改配置后，服务端先通过```persistService.insertOrUpdate```将配置信息进行了更新，然后调用```EventDispatcher#fireEvent```触发一个```ConfigDataChangeEvent```事件。接下来，我们来查看一下fireEvent方法主要做了什么:
<br/>
![deep-in-nacos-for-server-9](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-9.png?raw=true)
<br><br/>
由此可见，fireEvent主要是根据事件class获取listener列表（```CopyOnWriteArrayList<EventDispatcher.AbstractEventListener>```），然后循环调用每个listener的onEvent方法。而listener是通过```EventDispatcher#addEventListener```添加到listeners中。因此，我们只需找到调用```EventDispatcher#addEventListener```方法的地方，即可得知需要触发哪些```AbstractEventListener```的onEvent回调方法。
<br/><br/>
![deep-in-nacos-for-server-10](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-10.png?raw=true)
<br/><br/>
我们发现```AbstractEventListener```构造方法调用了```EventDispatcher.addEventListener(this)```方法，但是显然```AbstractEventListener```是抽象类，应找具体的实现类，很巧，一个熟悉的身影出现在面前：```LongPollingService```。
<br/><br/>
所以，我们可以显而易见两条线路：<br/>
（1）在nacos控制台更改配置信息时 -> 调用```POST:/nacos/v1/cs/configs```API  -> 触发```ConfigDataChangeEvent```事件 -> 调用listener的onEvent方法 -> LongPollingService#onEvent<br/>
（2）在nacos控制台更改配置信息时 -> 调用```POST:/nacos/v1/cs/configs```API  -> 触发```ConfigDataChangeEvent```事件 -> 调用listener的onEvent方法 -> AsyncNotifyService#onEvent
<br/><br/>
那到底哪个是我们想要的呢？还记得我们触发的事件类型吗？没错，是``` ConfigDataChangeEvent ```事件，所以我们只需对比两条线路哪个是处理的该类型事件即可。
<br/><br/>
![deep-in-nacos-for-server-11](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-11.png?raw=true)
<br/>
![deep-in-nacos-for-server-12](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-12.png?raw=true)
<br/><br/>
经过上述```LongPollingService#onEvent```和```AsyncNotifyService#onEvent```初步查看，基本可以确定```AsyncNotifyService```是我们想要的，但是事实真的是如此吗？我们通过查看```AsyncNotifyService.AsyncTask#executeAsyncInvoke```中并没有任何相关性。那么你可能会说，```LongPollingService#onEvent```处理的是```LocalDataChangeEvent```岂不是更不相关？
<br/><br/>
然而答案却是```LongPollingService```，我们不要被表象所迷惑了。为什么呢？在Nacos中有一个DumpService，它会定时把变更后的数据dump到磁盘上。DumpService在Spring启动之后，会调用init方法启动几个dump任务，然后在任务执行结束之后，会触发一个LocalDataChangeEvent 的事件。
<br/><br/>
我们来看一下代码流转过程吧：
<br/>
![deep-in-nacos-for-server-13](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-13.png?raw=true)
<br/>
![deep-in-nacos-for-server-14](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-14.png?raw=true)
<br/>
![deep-in-nacos-for-server-15](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-15.png?raw=true)
<br/>
![deep-in-nacos-for-server-16](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-16.png?raw=true)
<br/><br/>
代码流程过程大致为：```DumpService#init``` -> ```DumpAllProcessor#process``` -> ```ConfigService#dump``` -> ```ConfigService#updateMd5``` -> ```EventDispatcher.fireEvent(new LocalDataChangeEvent(groupKey))```
<br/><br/>
所以，我们还是要会回到```LongPollingService#onEvent```中，其中会启动一个```DataChangeTask```线程，其中会有一个循环迭代器从```allSubs```里面获取```ClientLongPolling```对象，删除订阅关系后，调用```ClientLongPolling#sendResponse```将数据返回给客户端，这就是为什么配置信息可以实时触发更新的缘由了。
<br/><br/>
那如果```DataChangeTask```任务完成了数据返回客户端后，```ClientLongPolling```中延迟任务开始执行怎么办？
<br/>
haha~No Problem.因为在```DataChangeTask```调用```ClientLongPolling#sendResponse```返回数据给客户端时，会先取消超时任务，然后再反馈数据给客户端。代码如下:
<br/>
![deep-in-nacos-for-server-17](http://github.com/buildupchao/ImgStore/blob/master/blog/technologychallenge/nacos/deep-in-nacos-for-server-17.png?raw=true)

### 4.3 小结

到此为止，从服务端对配置中心的完整流程已经分析关闭，我们做一个小结:
<br/>
- 服务端接收到客户端发起的长轮询后，先比较缓存中的数据是否相同。
  - 如果不同，直接返回；
  - 如果相同，则通过schedule延迟29.5s后再执行比较
- 为保证服务端在29.5s内发生配置信息变更时能及时通知客户端，服务端采用了事件订阅的方式监听```LocalDataChangeEvent```事件
  - 收到```LocalDataChangeEvent```事件，触发```DataChangeTask```任务，遍历allSubs列队中的```ClientLongPolling```并将数据写回给客户端。
- 关于timeout事件去除500ms，这个要好好品。你品，你细品，你细细品。

## 5.结尾

留几个问题吧：<br/>
- 为什么Nacos采用客户端拉取服务端配置的方式，而不是采用服务端主动推送数据给客户端呢？
- 为何```ClientWorker```中采用上一个```LongPollingRunnable```线程执行完毕再通过```ScheduledExecutorService```进行提交下一次任务的方式，既然不采用```ScheduledExecutorService```的定时调度特性却如何创建了该类型线程池呢？
- 分析客户端实时获取配置信息时，```CacheData```高频出现，可见其重要性，自己主动去阅读下其源码？
- 通过上述分析，我们知道只要是使用配置的场景基本都可以用Nacos来进行管理，那么Nacos有哪些适用场景呢？举几个例子？
- nacos客户端和服务端设计上，都考虑到了哪些设计模式/理念呢？

就先留这几个问题吧。<br/>

## 6.资料链接

- Nacos官网：[http://nacos.io/en-us/docs/what-is-nacos.html](http://nacos.io/en-us/docs/what-is-nacos.html)
- Nacos Github: [http://github.com/alibaba/nacos](http://github.com/alibaba/nacos)

<br/><br/>
</strong>如有疑问欢迎留言。</strong>
