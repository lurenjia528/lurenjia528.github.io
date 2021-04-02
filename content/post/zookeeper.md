---
title: zookeeper日志查看
#slug: chinese-test
date: 2021-04-02
categories:
- linux
tags:
- zookeeper
thumbnailImagePosition: left
thumbnailImage: /img/zookeeper.jpeg
#coverImage: /img/vim.jpg
---

<!--more-->


# vim-problem

查看zookeeper日志：
```bash
root@master1:/var/log/zookeeper/version-2# java -cp /usr/local/kylincloud2-arm64/zookeeper-3.4.10/zookeeper-3.4.10.jar:/usr/local/kylincloud2-arm64/zookeeper-3.4.10/lib/slf4j-api-1.6.1.jar org.apache.zookeeper.server.LogFormatter log.8000b8bb3
```