---
title: 为何通过java.sql.DatabaseMetaData获取MySQL的table remarks总是为空？
tags: ['Java']
category: java
---

大数据平台存在这样一种场景，我们需要根据用户录入的配置信息，进行同步数据源/表相关的信息，此时就需要把数据库、数据表、数据表字段相关信息进行拉取。而我们获取数据源的元信息时，是在Connection连接通道之上，借助于java.sql.DatabaseMetaData进行获取详细信息（比如TABLE_NAME,  REMARKS, COLUMN_NAME等等），然而却出现了除REMARKS之外的其他信息都可以正常同步，唯有REMARKS一直为空。

通过查看相关资料得知，要想获取源数据库表的remarks信息，需要在构建Connection指定两个相关属性：
```Java
        Properties connectionProps = new Properties();
        // ...
        /**
         * If load remarks of table using java.sql.DatabaseMetaData, need to set these parameters.
         */
        connectionProps.put("remarks", "true");
        connectionProps.put("useInformationSchema", "true");
        Connection conn = DriverManager.getConnection(url, connectionProps);
```
如此，问题得以解决。

【参考文献】
[1] [mysql DatabaseMetaData 获取table remarks为空的解决办法](http://huqiji.iteye.com/blog/2205161)

[2] [DatabaseMetaData的用法](https://www.cnblogs.com/shide/p/3340906.html)
