---
title: redis启动错误
slug: chinese-test
date: 2020-07-24
categories:
- redis
- ubuntu
tags:
- redis
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
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
