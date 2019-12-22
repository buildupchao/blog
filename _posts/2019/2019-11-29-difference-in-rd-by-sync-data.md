---
title: 通过RDS事件数据同步来看不同软件工程师的区别
tags: ['Java']
category: arch
keywords: 架构设计,Java
---

> 由彼观彼，而不是由己观彼。

开始前，扯些许的题外话。我们经常会看到一些类似于初、中、高级软件工程师的区别的文章，觉得高级会如何，初中级又会如何，我们此次就不区分title，而是通过经验丰富与否、考虑问题是否全面、做事是否稳重来展开。

![](https://github.com/buildupchao/ImgStore/blob/master/blog/2019-11-29-1.jpg?raw=true){:width="300"}

## **1.问题描述**

有这样一个需求，我们需要在国内WEB端新增的事件数据，不仅要写入国内的RDS，同时也要同步到国外的三个不同地点的RDS中（不可采用@scheduled方式进行定时同步）。

## **2.如何做数据同步呢？步骤？代码逻辑？**

- 先来看一个步骤图：

![](https://github.com/buildupchao/ImgStore/blob/master/blog/2019-11-29-2.png?raw=true)

- 再来看一段<strong>伪代码</strong>（以下代码仅供参考，只是伪代码，不是标准Java代码）：
  - (1) 写入国内RDS、同步到国外其他RDS以及对应事务控制
{% highlight java %}
@Transactional(rollbackFor = Exception.class)
void newEvent(event) throws Exception {
    insertEventIntoInlandRds(event);
    asyncEventToOtherRds(event); // 异步同步数据
}
{% endhighlight %}

  - (2) 同步失败重试以及重试结果记录
```Java
class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {
  public void handleUncaughtException(Throwable throwable, Method method, Object... objects) {
      retry 5 times
      label = succeed after retring 5 times
      if label:
        insert async table successful label
      else:
        insert async table failed label and data
  }
}
```

## **3.潜在问题探讨**

通过上述方式，想必有一批直接使用Spring Boot做项目开发的工程师，都自以为标注上`@Transactional`就可以了，就会失败回滚，也不需要什么补救措施，那你就大错特错了。`@Transactional`支持的是单体应用，远程IO已经脱离了它的控制。

其次，同步失败，重试5次没问题，那同步概况记录表应该放在哪里呢？那些分别放到不同地域分别建立同步概况表的就不想说什么了，retry都失败了，你写入同步概况到国外RDS就一定能成功吗？答案是不能！所以记录同步概况的表当然需要放在主RDS里面（这里主从是相对概念，从A同步到B，则A是主；如果从C同步到A，则C是主），也就是国内RDS。

再次，如果`asyncEventToOtherRds`调用失败，导致`newEvent`事务回滚，所以`insertEventIntoInlandRds`也回滚，也就是国内RDS没有event这条记录了，但是海外还在不断retry，海外可能的结果是失败 or 成功。此时，会存在两种不同情况：1)海外同步失败，则国内外数据是一致的；2）海外同步成功，则国内外数据就是不一致的。所以这里存在的问题就是事务回滚却没有数据探测以及补偿措施。

然后，如果retry了5次仍然失败了，那该如何处理呢？这里同样需要有一定的补偿措施，而不是简单的记录一下失败同步记录和数据就完事大吉，长此以往，累积同步失败记录多了一定会出大问题的。

最后，就是这样的设计明显是没有完全理解业务的主次，没有做好责任划分和事务隔离。需要在保证国内RDS一定写入成功，提交事务后，再进行海外RDS数据同步，这样我们就可以针对性对症下药处理即可，而不用一边考虑国内失败如何，海外又失败如何，尽量责任单一。

## **4.结论呢？**

其实，这里写结论的时候，是很不想写的。为什么呢，因为其实这个无关乎title，真的只是关乎经验、考虑问题是否全面、是否有思辨思维。牵扯到数据方面的问题的时候，一定要打起12分的精神，不然数据都不准，谁还敢用你们的产品呢？

但是，到此，还是想说上两句：1）技术一定要扎实，不要对一门技术一知半解就觉得完全OK。比如上面的`@Transactional`注解的使用就是个很好的例子。2)一定要有思辨思维去看到每一件事情，凡事都是双面性的。

所以，这里的结论就是不做结论。希望自己思考，自己是否属于上述顾头不顾尾的往前跑，却不管产生的影响。

## **5.人性的弱点？**

之前，耗子叔曾说过，“学习本来就是件逆人性的事情，如果要成长，必须逼自己一把”。话没错，正式因为停留在技术的肤浅层面，最终才会出现上述问题。

也有人说，“失败是成功之母”。不，失败不会使你成功，总结失败才会助你成功。如果只是一味地不断重复错误，那你注定只能原地踏步。希望我们能够剥离这种潜意识！擅于总结问题，发现问题，积累经验，不断成长。

### **资料快链**

写完文章后，发现还是有必要贴上一些有助于理解上述问题的文章链接。就在网上找了下最佳解读的：

- [1] [两个与spring事务相关的问题](https://www.cnblogs.com/ASPNET2008/p/5570237.html)
- [2] [Spring Boot Reference Guide](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [3] [Spring Boot系列文章 – 纯洁的微笑](http://www.ityouknow.com/spring-boot.html)
- [4] [《Spring Boot Cookbook》阅读笔记](https://yq.aliyun.com/articles/54071)
