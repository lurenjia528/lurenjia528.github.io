---
title: kube-proxy启用ipvs
slug: chinese-test
date: 2019-06-26
categories:
- k8s
- kube-proxy
- ipvs
tags:
- kube-proxy
- ipvs
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
kube-proxy启用ipvs
<!--more-->


# kube-proxy

kube-proxy 启用ipvs，ipset 需要 /etc/services 和 /etc/protocols

kube-proxy.yaml 需要挂载 /etc/services 和 /etc/protocols
