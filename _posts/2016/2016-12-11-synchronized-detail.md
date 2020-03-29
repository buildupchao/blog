---
title: synchronized原语图解
tags: ['Java']
category: java
---

``` synchronized ```关键字同步代码段时，编译后源码会自动生成``` monitorenter ```和``` monitorexit ```命令。而 ``` synchronized ```用来修饰 static方法时，则是依赖于方法修饰符上的``` ACC_SYNCHRONIZED ```来完成的。
<br/><br/>
``` synchronized ```原语图解如下：<br/>

![synchronized原语图解](https://github.com/buildupchao/ImgStore/blob/master/blog/concurrent/synchronized_detail.png?raw=true)
