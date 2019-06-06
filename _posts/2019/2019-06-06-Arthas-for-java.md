---
title: 爱上Java诊断利器之Arthas
date: 2019-06-06 20:59:00
tags: ['Java', 'Arthas', '诊断利器']
category: Java
---

## Arthas是什么？

摘自Arthas的Github介绍：
<blockquote>
  <p>Arthas is a Java Diagnostic tool open sourced by Alibaba.</p>
  <p>Arthas allows developers to troubleshoot production issues for Java applications without modifying code or restarting servers.</p>
</blockquote>

大意为：Arthas是阿里开源的一个Java诊断工具。Arthas可以在不修改代码或重启服务器的情况下帮助开发人员快速定位线上问题。

听起来确实是我们的程序员的一大福利。比如，我们就遇到一种情况，Spring Boot应用中有个cron定时任务为每天凌晨1点启动执行，但是测试起来很不方便，总不能每次修改cron时间来让QC测试吧？这样虽然是方便了测试妹子，但是却徒增了我们开发时间和迭代次数啊！！！那Arthas到底是否能够满足我们需求呢？Go on...

## Arthas
