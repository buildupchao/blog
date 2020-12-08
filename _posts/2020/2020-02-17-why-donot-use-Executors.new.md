---
title: 为何不推荐直接采用 Executors.new 线程池的方式？
tags: ['java', 'threadpool', 'Executors']
category: java
keywords: java,executors,threadpool,线程池
layout: post
---


众所周知，Java并发编程是一大难点所在。<br/>
其实并不是我们不懂并发原理，而是我们往往忽略了细节，然而，<strong style='color:red'>“魔鬼总在细节中”！</strong>;<br/>
<br/>
作为以Java作为主语言的研发，应该都知道 Executors.new 各种线程池的时候（ScheduledThreadPool除外），底层都是通过 ThreadPoolExecutor来实现的。

<strong style="color:green;">[延拓]</strong>
<hr/>
```ThreadPoolExecutor```的5项基本参数为：
- int corePoolSize：核心线程数
- int maximumPoolSize：线程池最大可拥有的线程数（最高并发量）
- long keepAliveTime：线程idle态的最大存留时间
- TimeUnit unit：时间单位，配合 keepAliveTime 使用
- BlockingQueue<Runnable> workQueue：任务队列（当任务量打满核心线程数 or 最大线程数 时，用来存放排队等待执行的任务）
<hr/>

<br/>
以下主要以 ``` Executors.newFixedThreadPool(nThreads) ```进行分析：

![](https://github.com/buildupchao/ImgStore/blob/master/Java/concurrent/threadpool/newFixedThreadPool.png?raw=true)

由上图，我们可以明显得出两项信息：
- 线程池中线程不会进行回收，会一直存活，即使都一直处于idle状态（因为corePoolSize和maximumPoolSize被设置为等大）
- 任务队列直接采用```new LinkedBlockingQueue<Runnable>()```方式，是无界的，也就是说任务量飙升时会存在内存溢出风险

那还有没有其他信息呢？？？当然！！！
我们再来看通过```Executors.newFixedThreadPool```方式调用```ThreadPoolExecutor```时，存在哪些隐性操作：

![](https://github.com/buildupchao/ImgStore/blob/master/Java/concurrent/threadpool/Executors.newFixedThreadPool.png?raw=true)

![](https://github.com/buildupchao/ImgStore/blob/master/Java/concurrent/threadpool/Executors.defaultThreadFactory.png?raw=true)

啊！！！你看，内部直接采用了```Executors.defaultThreadFactory()```，它会做些什么？是以什么样的方式来给我们提供工作线程呢？

![](https://github.com/buildupchao/ImgStore/blob/master/Java/concurrent/threadpool/DefaultThreadFactory.newThread.png?raw=true)

发现问题严重性了吗？默认提供的工作线程均为<strong style="color:red;">非守护线程</strong>！<br/><br/>
也就是说，如果一旦发生意外，主线程都已经退出了，如果此时我们因为异常而没有关闭线程池的话，进程会作为一个僵尸进程一直存在的！！！

这也是近期排查线上问题时，发现有些小伙伴写代码太过于随意，根本不考虑边界情况以及异常case。
<br/><br/>
所以，我们平时设计并发编程，采用线程或线程池的时候，多一些思考，可能就会有不一样的结果。
<br/><br/>
在这里我也只是抛砖引玉，有兴趣的话，可以自己尝试去分析下```Executors.new```的另外几种方式又会存在哪些潜在风险呢？
