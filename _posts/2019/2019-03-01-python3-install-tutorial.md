---
title: Python3.x安装教程
tags: ['Python', '安装教程']
category: installtutorial
layout: post
---

## 1.前期准备

- 依赖包安装

Python安装会依赖一些环境，为了避免在安装过程中因为缺少依赖而产生不必要的麻烦，需要先执行以下命令，确保安装好以下依赖包：

```bash
yum -y install zlib zlib-devel
yum -y install bzip2 bzip2-devel
yum -y install ncurses ncurses-devel
yum -y install readline readline-devel
yum -y install openssl openssl-devel
yum -y install openssl-static
yum -y install xz lzma xz-devel
yum -y install sqlite sqlite-devel
yum -y install gdbm gdbm-devel
yum -y install tk tk-devel
yum install gcc
```

- Python源码下载

可以前往 [https://www.python.org/ftp/python/](https://www.python.org/ftp/python/)查看Python各个版本，这里，我们选择安装``` Python-3.6.5.tgz ```版本。

通过如下命令下载Python源码压缩包：
```bash
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
```

- 知识点补充：如果不知道``` configure, make, make install ```三个命令作用，[点这里查看](https://www.cnblogs.com/tinywan/p/7230039.html)

## 2.安装Python

- 1) 解压Python源码压缩包

```bash
tar -zxvf Python-3.6.5.tgz
```

____________________

<strong style="color:red;">步骤2~4均在Python-3.6.5目录下进行命令操作。</strong>

- 2）通过 configure 命令检测及校验平台

```bash
./configure --with-ssl --prefix=/service/python3
```

- 3）通过 make 命令编译Python源代码

```bash
make
```

- 4）通过 make install 命令安装Python

```bash
make install
```

____________________

- 5）Python的快捷方式之软连接

通常情况下，Linux默认自带Python2.x版本，所以，当我们在终端敲下``` python ```命令时，都是采用的Python2.x版本，为了方便我们使用，我们可以通过修改软连接的方式让``` python ```命令指向``` python3 ```。

<strong>做事之前先做好原始数据的备份工作（常识）:</strong>
```bash
sudo mv /usr/bin/python /usr/bin/python2.backup
```

<strong>制作新的指向Python3的软连接:</strong>
```bash
sudo ln -s /service/python/bin/python3 /usr/bin/python
```

<strong>验证是否生效:</strong>
```bash
python -V
```

到这里，Python3.x的安装已经完成，但是好事做到底，送佛送到西，当然要保证不影响系统其它命令和应用。Go on!

- 6）保证系统原有依赖的可用性

因为我们修改了Linux原有依赖的``` python ```软连接（从python2改成了python3），所以会导致有些命令使用时会报错，例如yum等。这时我们就需要保证命令的可用性。以yum为例，具体操作如下（其它命令使用时有影响亦如此修改即可）：

<strong>先找到yum命令所在位置（通常系统命令都在/usr/bin/目录下）:</strong>
![whereis-yum](https://github.com/buildupchao/ImgStore/blob/master/blog/installtutorial/whereis-yum.png?raw=true)

<strong>编辑yum文件，把python修改为python2.7(这个版本号和你的平台相关，比如centos7是2.7，centos6是2.6):</strong>
![modify-yum](https://github.com/buildupchao/ImgStore/blob/master/blog/installtutorial/modify-yum.png?raw=true)

至此，才算是真正完成了Python3的安装！
