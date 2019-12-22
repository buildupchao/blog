---
title: 论掌握一项脚本技术的必要性
tags: ['tech-talking']
category: tech-talking
---

工作过程中，我们常常需要对一些我们可能会临时需要的数据进行清洗或者格式化等处理。这个时候就需要借助于一些奇淫技巧或者一些工具，诸如Windows平台下的notepad++，Mac/Linux平台下的vim等。

最近大数据部在进行成本优化，需要对各业务使用带宽、数据量、访问量、以及pv、uv等各种可进行成本优化的信息进行分类统计，然后进行逐步缩减优化。期间就频繁多次的借助于shell脚本、Java程序以及HQL来解决了大量问题。

场景一：如何快速过滤出来包含某些内容的行？
{% highlight bash %}
cat your_file_name | grep "you_need_filter_content" > result_file
{% endhighlight %}
是不是发现很轻松得到了想要的所有行，而且再也不用通过n和N进行向下/上切换，或者通过ctrl+F或者ctrl+B翻页了。


场景二：
