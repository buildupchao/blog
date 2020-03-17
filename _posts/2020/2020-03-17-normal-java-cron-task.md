---
title: 没有Spring cron时该怎么定点执行定时任务？
tags: ['java', 'cron']
category: java
keywords: java,cron,定时任务
---

最近有个小需求，在普通Java项目里面，不能借助于Spring，也不能使用复杂的jar，来实现cron定点定时任务。<br/>
通过查询资料，发现一个好用的工具：hutool<br/>

{% highlight maven %}
<!-- https://mvnrepository.com/artifact/cn.hutool/hutool-cron -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-cron</artifactId>
    <version>5.2.2</version>
</dependency>
{% endhighlight %}

参考资料如下：<a href="https://www.bookstack.cn/read/hutool/0f082d6e35363da6.md" target="_blank">https://www.bookstack.cn/read/hutool/0f082d6e35363da6.md</a><br/>

普通应用里面，只需引入这个简单的jar包，仅仅36KB，就可以方便快捷的实现cron方式执行任务。
