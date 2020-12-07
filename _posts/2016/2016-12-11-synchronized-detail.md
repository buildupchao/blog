---
title: synchronized原语图解
tags: ['Java']
category: java
layout: post
---


### 1.synchronized同步原语

``` synchronized ```关键字同步代码段时，编译后源码会自动生成``` monitorenter ```和``` monitorexit ```命令。而 ``` synchronized ```用来修饰 static方法时，则是依赖于方法修饰符上的``` ACC_SYNCHRONIZED ```来完成的。
<br/><br/>
``` synchronized ```原语图解如下：<br/>

![synchronized原语图解](https://github.com/buildupchao/ImgStore/blob/master/blog/concurrent/synchronized_detail.png?raw=true)

<br/>
由上图可以看出，<br/>
任意线程对Object（Object由synchronized保护）的访问，首先要获得Object的监视器。<br/>
如果获取失败，线程进入同步队列，线程状态变为BLOCKED。<br/>
当访问Object的前驱（获得了锁的线程）释放了锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新尝试对监视器的获取。<br/>

### 2.synchronized编程范式

<br/>
等待/通知的编程范式：
<br/>
- 等待方：

{% highlight java %}
synchronized (对象) {
  while (条件不满足) {
    对象.wait();
  }
  对应的处理逻辑
}
{% endhighlight %}

- 通知方：

{% highlight java %}
synchronized (对象) {
  改变条件
  对象.notifyAll();
}
{% endhighlight %}
