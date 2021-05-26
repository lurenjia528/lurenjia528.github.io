---
title: dm相关
#slug: chinese-test
date: 2021-05-26
categories:
- dm
tags:
- dm
thumbnailImagePosition: left
thumbnailImage: /img/dmlogo.jpeg
coverImage: /img/dm.jpeg
---
dm相关
<!--more-->


# dm锁表 如何解决

查询事物视图,查到等待的事务id。
```sql
select * from v$trxwait;
```

利用sessions视图，查到sess_id。
```sql
select sess_id from v$sessions where tex_id in (select id from v$trxwait);
```
利用sp_close_session(sess_id);可以导致锁超时的语句。

```sql
sp_close_session(SESS_ID)
```

[参考](https://www.pianshen.com/article/84441029541/)
