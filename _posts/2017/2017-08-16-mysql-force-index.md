---
title: MySQL如何实现强制查询走索引和强制查询不缓存
tags: ['MySQL', '数据库']
category: db
layout: post
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
