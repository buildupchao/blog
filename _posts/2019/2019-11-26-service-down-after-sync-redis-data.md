---
title: 我只是同步了下Redis数据，怎么就服务瘫痪了？
tags: ['线上问题排查','Java']
category: onlinetroubleshooting
keywords: 线上问题排查,onlinetroubleshooting,线上事故,问题排查,Java
layout: post
---

![](https://github.com/buildupchao/ImgStore/blob/master/blog/2019-11-26-0.jpeg?raw=true)

## **背景**

> bug千千万，今天到我家。

简要描述：数仓WEB端进行新增事件后，会注入Redis中进行缓存，供给动参服务进行响应各端SDK的请求。

<!-- more -->

下午，发现海内外redis中存储的事件数据中仅有停用事件，而没有启用事件数据，以为是个bug，然后找数据负责人、动参负责人以及产品等进行确认，对比数据及综合意见后统一归咎为错误数据，经排查业务代码逻辑，发现数仓WEB端业务逻辑也是按照颠倒方式进行缓存事件数据（即仅缓存停用事件，却不缓存启用事件），然后对代码进行更新后，于晚上进行所有事件的缓存同步。

sync完所有事件数据到海内外redis后，同时也伴随着3个应用的发版，动参一度瘫痪，不能响应请求。

## **问题排查与定位**

从动参的表现来看，似乎是服务出问题了，测试发送请求，确认的确不可行后，通过`jstack processId > processId.log`进行查看线程状态，发现所有线程全部`BLOCKED`在调用`org.apache.commons.beanutils.BeanUtils.closeBean(BeanUtils.java:108)`时，锁等待在`PropertyDescritor#getReadMethod`（方法声明：`public synchronized Method getReadMethod()`）。

![](https://github.com/buildupchao/ImgStore/blob/master/blog/2019-11-26-1.png?raw=true)
<br/><br/>
![](https://github.com/buildupchao/ImgStore/blob/master/blog/2019-11-26-2.png?raw=true)

其次，结合今天另外两个条件：#1，三个应用发版；#2，修复颠倒业务逻辑后，会导致缓存中数据量增多（启用事件>停用事件等）。

所以多个小概率事件的叠加总和，导致触发了动参服务中使用的`BeanUtils#cloneBean`坑点，这也是进行多次大对象copy时的服务性能严重下降甚至瘫痪的根源。

知道问题所在就好解决了，暂时关闭需要调用BeanUtils#cloneBean的API，同时快速进行Redis数据回退，一切恢复正常。

## **事后反思**

其实很多工具类在多线程模式下都是多多少少存在性能或者安全问题的，比如`BeanUtils#cloneBean`，`SimpleDateFormat`等等。所以，我们需要时刻注意自己所写代码，也要注意成长积累，避开那些槽点，同时规避潜在的不安全点，也算是防御性编程思想的运用吧！（此处推荐阅读下阿里的Java开发手册）

既然要做一名程序员，那么请从每一行代码做起，保证代码质量，魔鬼总在细节中，不要最终千里之堤溃于蚁穴。各位，come on, hold on!

#### **资料快链**

[1] [各类对象属性拷贝工具性能测试对比：BeanCopier、BeanUtils、DozerBeanMapper、PropertyUtils](https://blog.csdn.net/u012534326/article/details/102611483)

[2] [BeanUtils拷贝属性容易忽视的坑](https://blog.csdn.net/qq_21033663/article/details/71794648)
