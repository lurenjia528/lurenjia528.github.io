---
title: 系统模块
slug: chinese-test
date: 2019-06-26
categories:
- linux
tags:
- modules
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---

<!--more-->

```bash
nf_conntrack_max文件不存在
检查系统是否存在conntrack模块
find /lib/modules/`uname  -r` -name *conntrack*
系统加载此模块
modprobe nf_conntrack
查看
lsmod
```
