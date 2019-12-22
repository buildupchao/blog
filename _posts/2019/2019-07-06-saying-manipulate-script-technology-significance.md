---
title: 论掌握一项脚本技术的必要性
tags: ['tech-talking']
category: tech-talking
---

工作过程中，我们常常需要对一些我们可能会临时需要的数据进行清洗或者格式化等处理。这个时候就需要借助于一些奇淫技巧或者一些工具，诸如Windows平台下的notepad++，Mac/Linux平台下的vim等。

最近大数据部在进行成本优化，需要对各业务使用带宽、数据量、访问量、以及pv、uv等各种可进行成本优化的信息进行分类统计，然后进行逐步缩减优化。期间就频繁多次的借助于shell脚本、Java程序以及HQL来解决了大量问题。

## 场景一：如何快速过滤出来包含某些内容的行？（shell）
{% highlight shell %}
#!/bin/bash

cat your_file_name | grep "you_need_filter_content" > result_file
{% endhighlight %}
是不是发现很轻松得到了想要的所有行，而且再也不用通过n和N进行向下/上切换，或者通过ctrl+F或者ctrl+B翻页了。

## 场景二：实时调用API获取数据并录入MySQL。（shell）

{% highlight shell%}
#!/bin/bash

# 以Get请求为例
reqUrl="yourapi"
echo "request url is : $reqUrl"

resData=$(curl $reqUrl)
echo "get data : <$resData>"

parseJson(){
  echo $1 | sed 's/.*'$2':\([^,}]*\).*/\1/'
}

# 假设返回的json数据有两个字段，分别为total和fail
total=$(parseJson $resData '"total"')
fail=$(parseJson $resData '"fail"')

# 对数据进行入mysql库
insertSql="INSERT INTO your_mysql_table(total,fail) values($total, $fail)"
echo "general sql: <$insertSql>"

cnt=$(mysql -hxxxxxx -P3306 -uxxxxxx -pxxxxx your_db_name -e "$insertSql")
if($cnt>0);then
  echo 'insert record successfully!'
else
  echo 'no insert!'
fi
{% endhighlight %}

## 场景三：如何批量删除redis中指定key数据（以set数据类型为例）？（shell）

假设已经准备好需要删除的redis keys of set member列表文件: wait_deleted_redis_keys
{% highlight shell %}
#!/bin/bash

cat wait_deleted_redis_keys | while read member_key
do
  `redis-cli -h your_redis_host -p your_redis_password srem your_redis_set_key $member_key`
  echo "success:["$member_key"]" >> result_deleted_redis_keys.log
done
echo ">>>>>>>>>>>>>>DONE<<<<<<<<<<<<<" >> result_deleted_redis_keys.log
{% endhighlight %}

## 场景四：如果没有权限执行flushall清空redis中数据，那么如何处理？（shell）
{% highlight shell %}
#!/bin/bash

redis-cli -h your_redis_host -p your_redis_password --scan --pattern "*" | xargs redis-cli -h your_redis_host -p your_redis_password del
echo ">>>>>>>>>>>>>>clear redis data DONE<<<<<<<<<<<<<"
{% endhighlight %}

## 场景五：如何实时监控某个进程的存活状态，失败并重新拉起？(python)
{% highlight python %}
# -*- coding:UTF-8 -*-

import os

# 检测进程是否存在
def isRunning(process_name):
    try:
        process = len(os.popen('ps aux | grep "' + process_name + '" | grep -v grep | grep -v tail | grep -v keepH5ssAlive').readlines())
        if process >= 1:

            return True
        else:
            return False
    except:
        print("Check process ERROR!!!")
        return False

# 重新拉起进程
def startProcess(process_script):
    try:
        result_code = os.system(process_script)
        if result_code == 0:
            return True
        else:
            return False
    except:
        print("Process start Error!!!")
        return False

# 服务监控入口
def monitorService(process_name):
    if isRunning(process_name) == False:
       startProcess('your_start_process_script')

{% endhighlight %}

---

以上只是通过几个例子来说明脚本（shell/python等等）可以帮助我们快速实现各种小需求和数据处理。当然也不反对你些Java之类程序，然后打包部署启动再执行...
