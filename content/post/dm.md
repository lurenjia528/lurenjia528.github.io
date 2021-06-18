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

# dm_svc.conf 
## 如何使用dm集群主备模式

dm服务配置文件(/etc/dm_svc.conf 路径文件名代码里写死的)

简单配置如下：
```bash
ysf@ysf:~$ cat /etc/dm_svc.conf 
TIME_ZONE=(480)
LANGUAGE=(cn)
DM=(192.168.17.187:5236,192.168.40.205:5236)
login=(1)
ysf@ysf:~$ 

```

springboot配置如下(原来的ip:port修改为服务名即可)
```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
  datasource:
    url: jdbc:dm://DM/CLOUD
    username: SYSDBA
    password: SYSDBA
    driver-class-name: dm.jdbc.driver.DmDriver
  jpa:
    database-platform: org.hibernate.dialect.DmDialect
    show-sql: false
    hibernate:
      ddl-auto: update
```

详细配置参考 《系统管理员手册》2.1.1.4