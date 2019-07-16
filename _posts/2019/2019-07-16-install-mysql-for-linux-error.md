---
title: Linux安装MySQL报libc.so.6(GLIBC_2.17)(64bit)错误
tags: ['Linux', 'MySQL']
category: installtutorial
---

今天临时需要在某台Linux云主机上安装一下MySQL Client用于远程连接RDS，结果通过yum快速安装MySQL时出现如下问题：

![](https://github.com/buildupchao/ImgStore/tree/master/blog/installtutorial/yum-install-mysql-error.png?raw=true)

解决方案如下：

- 首先到 /etc/yum.repos.d/目录下
- 然后编辑该目录下的mysql-community.repo文件，命令为：``` sudo vi mysql-community.repo```
- 在该文件中找到 mysql-56-community
- 将enable设置为0
- 重新执行 ``` sudo yum install mysql-server ```安装
