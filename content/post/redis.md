---
title: redis启动错误+springboot
#slug: chinese-test
date: 2020-07-24
categories:
- ubuntu
- redis
tags:
- redis
thumbnailImagePosition: left
thumbnailImage: /img/redis.jpg
---
redis启动错误
<!--more-->


redis启动报错
```bash
<jemalloc>: Unsupported system page size
<jemalloc>: Unsupported system page size
```
可能是系统pagesize问题

查看`getconf PAGESIZE`

# springboot k8s

redis集群以pod方式部署在k8s集群时，
springboot默认以lettuce连接池连接redis集群，
当redis集群重启或者故障转移后，lettuce仍然使用原来缓存的pod ip 连接redis集群，导致连接超时失败．
解决办法：
开启动态刷新集群拓扑
```yaml
spring:
  redis:
#    password:
    timeout: 3000ms
#    host: 192.168.17.154
    cluster:
      nodes:
        - redis-cluster.redis.svc.cluster.local:6379
      max-redirects: 3
    lettuce:
      pool:
        max-active: -1
        max-wait: -1
        max-idle: 10
        min-idle: 5
      cluster:
        refresh:
          dynamic-refresh-sources: true
          adaptive: true
          period: 5s
```