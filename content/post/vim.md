---
title: vim回车换行
slug: chinese-test
date: 2021-01-06
categories:
- windows
- linux
- vim
tags:
- vim
thumbnailImagePosition: left
thumbnailImage: /img/vim.jpg
#coverImage: /img/vim.jpg
---

<!--more-->


# vim-problem

1. 在windows下的文本文件的每一行结尾，都有一个回车('\n')和换行('\r')
2. 在linux下的文本文件的每一行结尾，只有一个回车('\n');
3. 而在linux下打开windows编辑过的文件，就会在行末尾显示^M; 
4. 注：^M在vim中为ctrl+M

解决：

vim中输入^M为：

i进入输入模式

ctrl+v ctrl+m

方法1：

vim 1.txt

:%s/\r//g

方法2：

^M输入方法为： ctrl+v ctrl+m

vim 1.txt

:%s /^M//g




# 过滤空行和注释行
```bash
root@Kylin:/var/lib/influxdb# grep -Ev "^$|[#;]"  /etc/influxdb/influxdb.conf
reporting-disabled = true
[meta]
  dir = "/var/lib/influxdb/meta"
[data]
  dir = "/var/lib/influxdb/data"
  wal-dir = "/var/lib/influxdb/wal"
  wal-fsync-delay = "50s"
  index-version = "tsi1"
  query-log-enabled = false
  cache-max-memory-size = "1g"
  max-concurrent-compactions = 1
  max-series-per-database = 0
  max-values-per-tag = 0
  series-id-set-cache-size = 100
[coordinator]
  write-timeout = "60s"
  query-timeout = "60s"
[retention]
[shard-precreation]
[monitor]
  store-enabled = false
[http]
  enabled = true
  bind-address = ":8086"
  auth-enabled = true
[logging]
[subscriber]
  enabled = false
[[graphite]]
[[collectd]]
[[opentsdb]]
[[udp]]
[continuous_queries]
  run-interval = "10m"
[tls]

```
