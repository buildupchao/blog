---
title: MySQL如何实现强制查询走索引和强制查询不缓存
date: 2017-08-16 09:19:07
tags: ['MySQL', '数据库']
category: db
---

- A:强制走索引

```
SELECT uid, uname
FROM table_name
force index(ind_id);
```

- B:强制查询不缓存

```
SELECT SQL_NO_CACHE uid, uanme
FROM table_name
```

不走逻辑IO，走物理IO

在此仅用以做记录备用。