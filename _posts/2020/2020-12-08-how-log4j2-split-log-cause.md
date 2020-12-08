---
title: Log4j2日志滚动策略的魔鬼槽点
tags: ['java', 'log4j2']
category: java
keywords: java,log4j2,日志滚动切分
layout: post
---

```TimeBasedTriggeringPolicy```参数说明：

<!-- more -->

|-|-|-|
|参数名称|类型|描述|
|-|-|-|
| interval | integer | 根据日期格式中最具体的时间单位来决定应该多久发生一次rollover。例如，在日期模式中小时为具体的时间单位，那么每4小时会发生4次rollover，默认值为1 |
| modulate | boolean | 表示是否调整时间间隔以使在时间间隔边界发生下一个rollover。例如：假设小时为具体的时间单元，当前时间为上午3点，时间间隔为4，第一次发送rollover是在上午4点，接下来是上午8点，接着是中午，接着是下午4点等发生。|
|-|-|-|

由上图我们展开两个测试用例：

- case 1：如何实现天粒度切分日志？

![](https://github.com/buildupchao/ImgStore/blob/master/Java/log4j2/day_split.jpg?raw=true)

- case 2: 如何实现小时粒度（基于秒）切分日志？

![](https://github.com/buildupchao/ImgStore/blob/master/Java/log4j2/seconds_split.jpg?raw=true)


所以，log4j2中```TimeBasedTriggeringPolicy```切分文件策略，是基于filePattern中的```%d{yyyy-MM-dd-HH-mm-ss}```来决定到底采用哪种时间单位（天、小时、分钟、秒等）。

那上面两种方式有没有错误呢？？？答案是 case 2 是有点问题的。

我们已经指定了 ```modulus``` （modulus就是modulate）为true，应该以0点自动校准进行文件切分时间规划的，然而，当我们设置了86400秒（也就是24小时）一切分的时候，却没有达到0点切分的目的，而是项目启动的当前时间推算24小时。

我们再来看下Log4j2基于时间切分逻辑底层```org.apache.logging.log4j.core.appender.rolling.PatternProcessor```判断逻辑(部分截图)：

![](https://github.com/buildupchao/ImgStore/blob/master/Java/log4j2/PatternProcessor-1.png?raw=true)

![](https://github.com/buildupchao/ImgStore/blob/master/Java/log4j2/PatternProcessor-2.png?raw=true)

所以，这种奇葩的实现逻辑，略有些逆人性，真的很容易踩坑。

但是说到底还是对log4j2切分策略不够熟悉，导致使用上有所偏差。

等后面有空，再对log4j2的disruptor使用进行详细总结~
