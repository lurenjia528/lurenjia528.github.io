---
title: 清理docker镜像/容器
slug: chinese-test
date: 2019-06-26
categories:
- ubuntu
- docker
- shell
tags:
- docker
- shell
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
清理docker镜像/容器
<!--more-->

```bash

#!/bin/bash
docker images | grep none | awk '{print $3}' | xargs docker rmi -f
docker ps -a | grep "Exited (143)" | awk '{print $1}') |xargs docker rm -f
docker ps -a | grep "Exited (137)" | awk '{print $1}') |xargs docker rm -f
docker ps -a | grep "Exited (2)" | awk '{print $1}') |xargs docker rm -f
docker ps -a | grep "Exited (255)" | awk '{print $1}') |xargs docker rm -f
```